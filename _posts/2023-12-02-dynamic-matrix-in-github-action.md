---
layout: post
---
# How to write one GitHub Action that tests exactly those resources that have been changed

TL;DR: check out [this workflow file](https://github.com/pcjedi/dynamic-matrix/blob/main/.github/workflows/test-aws-glue-resources.yaml)

{% raw %}

Imagine you have a repository with a folder structure list this:

```plaintext
Project Root
├── .github
│   └── workflows
│       └── test-gluejobs.yaml
├── aws-glue-resource
│   ├── resource1
│   │   ├── requirements.txt
│   │   ├── script.py
│   │   ├── test-requirements.txt
│   │   └── test-script.py
│   └── resource2
│       ├── requirements.txt
│       ├── script.py
│       ├── test-script.py
│       └── test-requirements.txt
├── README.md
└── LICENSE

```

How can we write a github action, that tests all resources which changes with a PR? All resources have the same structure. So we can use a job with a matrix, that iterates and parralalizes the folders. Something like this:

```yaml
on:
  pull_request:
    paths:
      - "aws-glue-resources/**"
jobs:
  tests-aws-glue:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        dir:
          - resource1
          - resource2
    steps:
      - uses: actions/checkout@main
      - uses: actions/setup-python@main
        with:
          python-version: "3.10"
      - run: pip install -r aws-glue-resources/${{ matrix.dir }}/test-requirements.txt
      - run: pytest aws-glue-resources/${{ matrix.dir }}/test-script.py
```

The Problem with this setup is, that it will test all resources stated in the matrix, regardless if they have been changed or not. To overcome this, we have to write an init job, that dynamically finds all the resources that are changing with the PR and feed the matrix with the result. Something like this:

```yaml
on:
  pull_request:
    paths:
      - "aws-glue-resources/**"
jobs:
  init:
    runs-on: ubuntu-latest
    outputs:
      dirs: ${{ steps.output-dirs.outputs.dirs }}
    steps:
      - uses: actions/checkout@main
        with:
          fetch-depth: 0
      - id: output-dirs
        run: >
          echo "dirs=$(
            git diff --name-only origin/${{ github.base_ref }} -- aws-glue-resources/* | 
            cut -d/ -f2 | 
            uniq | 
            jq --compact-output --raw-input --slurp 'split("\n")[:-1]'
          )" >> "$GITHUB_OUTPUT"
  tests-aws-glue:
    needs: init
    runs-on: ubuntu-latest
    strategy:
      matrix:
        dir: ${{ fromJson(needs.init.outputs.dirs) }}
    steps:
      - uses: actions/checkout@main
      - uses: actions/setup-python@main
        with:
          python-version: "3.10"
      - run: pip install -r aws-glue-resources/${{ matrix.dir }}/test-requirements.txt
      - run: pytest aws-glue-resources/${{ matrix.dir }}/test-script.py
```

The interesting part is the last step of the init job. Let's break this appart a bit to understand it:

- `run: echo "key=value" >> "$GITHUB_OUTPUT"` [with this step, we can specify an output for another job](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idoutputs).
- `${{ github.base_ref }}` [gives us the branch in which this pull request want to merge into](https://docs.github.com/en/actions/learn-github-actions/contexts#github-context)
- `git diff -- <path>` [shows information about version controll changes and filters these to path](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt-ltpathgt82308203)
- `cut -d/ -f2` [splits the path at backslash and returns the second element](https://dyn.manpages.debian.org/bookworm/coreutils/cut.1.en.html)
- `jq` [is a command line tool to handle json](https://github.com/jqlang/jq)

This works very well already. But what if we implement a change into the workflow file? In that case we want to test everything! In this case our init job becomes a bit different.

```yaml
    steps:
      - uses: actions/checkout@main
        with:
          fetch-depth: 0
      - run: >
          echo "dirs=$(
            git diff --name-only origin/${{ github.base_ref }} -- aws-glue-resources/* | 
            cut -d/ -f2 | 
            uniq | 
            jq --compact-output --raw-input --slurp 'split("\n")[:-1]'
          )" >> "$GITHUB_ENV"
      - name: check if workflow file changed
        run: echo "workflowfilechanged=$(git diff origin/${{ github.base_ref }} --name-only -- ${{ github.workflow }} )" >> "$GITHUB_ENV"
      - name: all testable dirs if workflowfile changed
        if: env.workflowfilechanged
        run: echo "dirs=$(ls -d aws-glue-resources/* | cut -d/ -f2 | jq --compact-output --raw-input --slurp 'split("\n")[:-1]')" >> $GITHUB_ENV
      - id: output-dirs
        run: echo "dirs=$dirs" >> "$GITHUB_OUTPUT"
```

After the checkout, we first fetch the changed directories just as before, but we only write it into the environment, not the output. Then, we check if the workflow file changes and if so, we write the path of the workflow file into an environemnt variable, if not the variable remains empty. We can use this Environment Variable like a boolean operator, to conditionally overwrite the previously fetch directory list with all possible directories. This directory list then goes into the output.

The next thing I would like to improve, is to add a testrun with every push on main. The trigger is easily added:

```yaml
on:
  push:
    paths:
      - "aws-glue-resources/**"
      - .github/workflows/test-aws-glue-resources.yaml
```

But now we must introduce a conditional `git diff`. If we are in a pull_request, the diff shall be made against the base branch, if we are in a push, the diff shall be made against the last commit. The workflow thus becomes like this:

```yaml
jobs:
  init:
    runs-on: ubuntu-latest
    outputs:
      dirs: ${{ steps.output-dirs.outputs.dirs }}
    steps:
      - uses: actions/checkout@main
        with:
          fetch-depth: 0
      - if: github.event_name == 'pull_request'
        run: echo "gitdiffref=origin/${{ github.base_ref }}" >> "$GITHUB_ENV"
      - if: github.event_name == 'push'
        run: echo "gitdiffref=${{ github.event.before }}" >> "$GITHUB_ENV"
      - run: >
          echo "dirs=$(
            git diff --name-only ${{ env.gitdiffref }} -- aws-glue-resources/* | 
            cut -d/ -f2 | 
            uniq | 
            jq --compact-output --raw-input --slurp 'split("\n")[:-1]'
          )" >> "$GITHUB_ENV"
      - name: check if workflow file changed
        run: echo "workflowfilechanged=$(git diff ${{ env.gitdiffref }} --name-only -- ${{ github.workflow }} )" >> "$GITHUB_ENV"
      - name: all testable dirs if workflowfile changed
        if: env.workflowfilechanged  ==  github.workflow  ||  github.event_name == 'workflow_dispatch'
        run: echo "dirs=$(ls -d aws-glue-resources/* | cut -d/ -f2 | jq --compact-output --raw-input --slurp 'split("\n")[:-1]')" >> $GITHUB_ENV
      - id: output-dirs
        run: echo "dirs=$dirs" >> "$GITHUB_OUTPUT"
```

As you can see, I also added a workflow_dispatch trigger. In the case of workflow_dispatch, also all testable dirs shall be tested. You can find the complete working example on [my repo](https://www.github.com/pcjedi/dynamic-matrix)

{% endraw %}
