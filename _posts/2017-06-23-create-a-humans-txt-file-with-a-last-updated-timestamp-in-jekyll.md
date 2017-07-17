---
title: Create a humans.txt file with a Last Updated timestamp in Jekyll
date: 2017-06-23 11:34:00 +02:00
permalink: '/jekyll/create-a-humans-txt-file-with-a-last-updated-timestamp-in-jekyll/'
categories:
- jekyll
layout: post
published: true
---

I wanted to create a [humans.txt][humans] file but at the same time I did not want to manually update the `Last update` variable in `/* SITE */` section every time I made a change. I wanted to automate the process so that I never have to worry about it again. 

## Table of Contents
<!-- MarkdownTOC -->

- [Create a new Jekyll template](#create-a-new-jekyll-template)
- [Create Humans Markdown file](#create-humans-markdown-file)
- [Linking your template and define a permalink](#linking-your-template-and-define-a-permalink)
- [Adding the meta-tag to your template](#adding-the-meta-tag-to-your-template)
- [Give it a whirl](#give-it-a-whirl)
- [Support](#support)

<!-- /MarkdownTOC -->

[Click here][my-humans] to see my `humans.txt` file. The below is what I did to get it to work and conform to standards.

## Create a new Jekyll template
For this to work you need to create a template so that when the file is accessed it won't display any of your site template but rather output raw text.

* Create a new file in `/_layouts/`
* Inside this file, paste the following code:
```html
{{ content | strip_html }}
```
* Save the file as `/_layouts/humans.html`

What you have done is create a completely new template that will only output the content and ignore any template files you may have. 

By default, Jekyll will wrap your text in a paragraph (`<p>`) tag so the `strip_html` part in the above tells Jekyll to simply dump the raw text without any HTML tags. 

## Create Humans Markdown file
Go ahead and create a file in the root of your Jekyll site and save it as `humans.md`. You have to create the Markdown file so that Jekyll can compile the page when it generates the site. If you create a `txt` file you lose the ability to access Jekyll's Liquid variables and we need this.

To comply to [the humans standard][standard] you will need to have at least these two sections in your `humans.md` file:
 1. `/* TEAM */`
 2. `/* SITE */`

I have more in [my file][my-humans] but add what you feel is necessary for yours. In this example let's assume your `humans.md` file now looks like this:
```
/* TEAM */
    Boss: Your name
    Email: be weary of spam
    Twitter: @username
    From: The country you live in

/* SITE */
    Last update: ???????????
    Language: English
    Doctype: HTML5
    IDE: Sublime Text 3, Github Pages, Jekyll
```
The `Last update` field is where we use Jekyll's Liquid variable `{{ site.time }}`. Copy and paste `{{ site.time }}` next to the `Last update:` field so that your Markdown includes the tag and ends up looking like this:
```
/* SITE */
    Last update: {{ site.time }}
    Language: English
    Doctype: HTML5
    IDE: Sublime Text 3, Github Pages, Jekyll
```

## Linking your template and define a permalink
You now need to ensure that `humans.md` is accessing your `humans.html` template file you created. Go ahead and open up your `/humans.md` file. Add the following code snippet to the top of your file:

```
---
layout: humans
permalink: '/humans.txt'
---
```
Your file should end up looking like this:

```
---
layout: humans
permalink: '/humans.txt'
---
/* TEAM */
    Boss: Your name
    Email: be weary of spam
    Twitter: @username
    From: The country you live in

/* SITE */
    Last update: {{ site.time }}
    Language: English
    Doctype: HTML5
    IDE: Sublime Text 3, Github Pages, Jekyll
```
Setting the `permalink` to `/humans.txt` is important because we want to conform to standards and we want to render a plain txt page for search engines and the like.

## Adding the meta-tag to your template
The final step is to add a valid meta-tag to your `<head>` section in your main template file. In my case my main template is `/_includes/default.html`

Add the following line to your template:
```html
<link rel='author' href='humans.txt' type='text/plain'>
```

If you aren't sure where to put this meta-tag look for the structure similar to the below example which is in your template.
```html
<head>
    <title></title>
    <meta name="description" content="" />
    <link rel="stylesheet" type="text/css" href="style.css">
    <meta content='text/html; charset=utf-8' http-equiv='Content-Type'>
</head>
```

## Give it a whirl 
Once you have published your changes you should be able to access your newly created humans file at `yourdomain.com/humans.txt`.

## Support
If you need any help, found a bug or to report any issues please [click here][support] and log a support request. 

[my-humans]: /humans.txt
[humans]: http://humanstxt.org
[standard]: http://humanstxt.org/Standard.html
[support]: https://github.com/justinhartman/howto/issues/new
