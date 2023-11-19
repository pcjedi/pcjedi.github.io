---
layout: post
---
## How to create your own github.io page

1. create a repository with the name `<github-user-name>.github.io`.
1. open a codespace in this repository.
1. type `jekyll new .` after startup.
1. delete or edit entries in the `_config.yml` file.
1. delete or edit the `about.markdown` file.
1. delete the markdown file in `_posts` and create a new one.

    The name MUST follow the format `<YEAR>-<MONTH>-<DAY>-<TITLE>.<FILEEXTENSION>`. I tried some things to change this to `/` instead of `-` so that my soon to be thousands of post are in a well defined folder structure (this is irony). But it seems too hardwired into jekyll to change this. In order to inherit the default layout, the file must start with: 
    ```
    ---
    layout: post
    ---
    ```
    You can also add `title` and `date` but this seems redundant to me. `categories` is another field you can fill, but it will make the url look strange and I haven't seen any other advantages (like listing all posts from one category). When using `#`-style headings, avoid the single resulting in `<h1>`, it has the same size as `<h3>`.
1. type `bundle exec jekyll serve` to view the result
1. push when satisfied.
1. in repo `Settings` go to `Pages` and select the branch from which to serve.
1. see the action running
1. go back to repo `Settings` in `Pages` and follow the link to see the result.
