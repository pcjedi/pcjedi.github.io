---
layout: post
---

# Using Favicon in your Jekyll based Webpage

Having a Favicon in you webpage is not only a nice touch, but eases the navigation of multiple tabs and booksmarks tremendously. By setting this up, it will make your and your readers life much simpler. As for simplicity, this article is meant to be as simple as possible to do this.

## Goal

By the end of this article you will have a favicon installed. In doing so, you might get a little better understanding on how jekyll works, regarding plugins, dynamic and static content.

## Process

I am using the [Minima Theme](https://rubygems.org/gems/minima). The procedure might be different if you are not using this theme. The first step will be to determine your current html head and set it explicitly. If you are already setting it explicitly, you can jump ahead. But then again, the rest might be trivial for you. Having said that:

1. If not already there, create [`_includes`](https://jekyllrb.com/docs/includes/) directory

    ```bash
    mkdir -p _includes
    ```

    You can read more about this folder in the [official documentation](https://jekyllrb.com/docs/includes/)
1. Copy the default head.html to make it explicite and edible.

    ```bash
    cp $(bundle info --path minima)/_includes/head.html _includes/head.html
    ```

    This will change nothing so far, but make your head be explicitly stated.
1. Add you [Favicon](https://www.w3schools.com/html/html_favicon.asp) into this head.html

    ```html
    <link rel="icon" type="image/x-icon" href="https://avatars.githubusercontent.com/u/22711712">
    ```

    I use my GitHub avatar directly. So if I change it in the future, it will be changed here aswell.
1. (optional) add assets folder with static image

    ```bash
    mkdir -p assets
    ```

    You can use this folder to provide static files
1. (optional) reference static file as icon.

    ```html
    <link rel="icon" type="image/x-icon" href="/assets/favicon.ico">
    ```

    If you have a `favicon.ico` in you assets folder, you can reference it like that.

## Result

You now have a Favicon on your Jekyll based webpage. Either dynamically fetched from another site, or (optionally) references as static content.

## Discussion

Adding a Favicon to your Jekyll based webpage is a [frequently discussed issue](https://github.com/jekyll/minima/issues?q=favicon). In my opinion, it should be part of the `_config.yml` file. Other themes might make it possible to implement it there. A [commit from Feb 12, 2020](https://github.com/jekyll/minima/commit/8b8177e5c8fe52d756f533ad23c1bf4db98e905c) introduces a `custom-head.html` into the minima theme, which would make this process a little simpler.
