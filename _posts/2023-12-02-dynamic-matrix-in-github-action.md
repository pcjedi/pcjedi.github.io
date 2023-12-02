---
layout: post
---
# How to write one GitHub Action that tests exactly those resources that have been changed

{% raw %}

Imagine you have a repository with a folder structure list this:

```plaintext
Project Root
├── .github
│   └── workflows
│       └── test-gluejobs.yaml
├── aws-glue-resource
│   ├── resource1
│   │   ├── script.py
│   │   ├── requirements.txt
│   │   ├── test-script.py
│   │   └── test-requirements.txt
│   └── resource2
│       ├── script.py
│       ├── requirements.txt
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
      - "aws-glue-resource/**"
jobs:
  tests:
    runs-on: ubuntu-latest
    matrix:
      dir:
        - jresource1
        - resource2
    steps:
      - uses: actions/checkout@main
      - uses: actions/setup-python@main
        with:
          python-version: "3.10"
      - run: pip install -r aws-glue-resources/${{ matrix.dir }}/test-requirements.txt
      - run: pytest aws-glue-resources/${{ matrix.dir }}/test-script.py
```

The Problem with this setup is, that it will test all resources stated in the matrix, regardless if they have been changed or not. To overcome this, we have to write a init job, that dynamically finds all the resources that are changing with the PR and feed the matrix with the result.

{% endraw %}
