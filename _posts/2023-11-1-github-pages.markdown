---
layout: post
---
## How to create your own github.io page

Creating a personal website has never been more accessible, thanks to the combination of GitHub Pages, Jekyll, and the powerful capabilities of GitHub Codespaces. In this guide, I will walk you through the steps to build your GitHub.io page with Jekyll using GitHub Codespaces, a cloud-based development environment.

### What is GitHub Pages, Jekyll, and GitHub Codespaces?

#### GitHub Pages:
GitHub Pages is a GitHub feature that allows users to host static websites directly from their GitHub repositories. It's an excellent platform for showcasing your portfolio, projects, or even a personal blog.

#### Jekyll:
Jekyll is a static site generator that simplifies the process of building a website. It transforms your content written in Markdown, HTML, Liquid, or other formats into a static website ready for hosting.

#### GitHub Codespaces:
GitHub Codespaces provides a cloud-based development environment that allows you to develop, build, and test your code directly in your browser. It eliminates the need for local setup and configuration, making collaboration and development more streamlined.

1. **setup a new repository**
    - head to [GitHub to create a new repository](https://github.com/new)
    - the repo name must start with the owner name followed by `.github.io`. resulting in the format `<github-user-name>.github.io`
    - for easier usage, initialize with a README.md
1. **setup as page**
    - in repo `Settings` go to `Pages`
    - select `Deploy from a branch` as Source and select the `main` branch unter `Branch` and click `Save`
    - wait until Deploy action finishes
    - reload site
    - follow link to the deployed site. your README.md should be visible
1. **start a codespace**
    - in the repo, click on the green `Code`-Button and then on `Create codespace on main`
    - wait until codespace has started up
1. **initialize the repository with default jekyll structure**
    - type `jekyll new .` after startup into the terminal
1. **customize the content**
    - delete or edit entries in the `_config.yml` file.
    - delete or edit the `about.markdown` file.
    - delete the markdown file in `_posts` and create a new one.

    The name filepaths in _posts must follow the format `<YEAR>-<MONTH>-<DAY>-<TITLE>.<FILEEXTENSION>`. In order to inherit the post layout, the file must start with: 
    ```
    ---
    layout: post
    ---
    ```
    You can also add `title` and `date` but this seems redundant to me. `categories` is another field you can fill, but it will make the url look strange and I haven't seen any other advantages (like listing all posts from one category). When using `#`-style headings, avoid the single resulting in `<h1>`, it has the same size as `<h3>`.
1. view and push results
    - type `bundle exec jekyll serve` to view the result
    - push when satisfied.


### Conclusion:

By leveraging the power of GitHub Codespaces, you've built and deployed your GitHub.io page with Jekyll effortlessly. This approach enhances collaboration and flexibility, allowing you to work on your website from any device with an internet connection. Happy coding!