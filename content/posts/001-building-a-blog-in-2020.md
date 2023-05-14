---
title: "Building a personal dev blog in 2020"
date: 2020-06-26T22:26:46-04:00
categories: ["Web Development"]
tags: ["hugo", "blog", "building", "scratch", "custom", "theme"]
summary: Where I explain how I leveraged Hugo to finally get into the modern era of blogging. 
---


In what now looks like another life, I had a mildly famous tech blog. It all started 10 years (yikes) ago, when I still worked with ABAP. I partnered with a friend to create an ABAP-themed blog called [ABAPZombie](https://www.abapzombie.com/), using [WordPress](https://wordpress.com/). If you think the name is weird, you should take a look at [one of our first page designs](https://web.archive.org/web/20120202221336/http://www.abapzombie.com/).

SAP is viewed as a tedious platform by most programmers [^1] , and back then we believed a lot of this had to deal with how people perceived the format of the information being shared about the ABAP language. Our blog was an attempt to share dev knowledge without corporate strings attached to the content. And, of course, a platform to vent when needed.

Long story short, the site started to get popular, I gave talks in Brazilian SAP events, and even wrote a book. But since I decided to leave the SAP world for good, due to many different reasons I never went back into writing. But this ends today!

## Static Site Generators

I had many issues trying to keep a WordPress website up for over 10 years. Plugins, themes, databases, "core" updates - you name it. So this time, [CMS](https://en.wikipedia.org/wiki/Content_management_system)s were out of question.

My goal was to create a personal site that could be as simple as possible, so using a static site generator (SSG) was an obvious choice. It's basically a tool that estipulate a certain workflow pattern, which in the end will result in a bunch of generated html/js/css files, ready to be served in the web. 

Three main things drew me towards a SSG over a CMS: markdown, speed and maintainability. Just thinking about all the caching solutions for WordPress gives me the chills. I really like markdown. And with static files, I don't have to care about my website being down just because a syntax-highlight plugin got updated [^2].

[Jekyll](https://jekyllrb.com/) has been around since forever, and was the tool I first intended to use for this website. But influenced by a co-worker of mine, I ended up deciding to go with [Hugo](https://goHugo.io/).

## Hugo

I'm still amazed by how easy it was to setup everything for this site with Hugo. The [quickstart](https://goHugo.io/getting-started/quick-start/) docs are really easy to follow if you're in Linux or OSX (`brew install` and off you go).

There are [plenty of themes](https://themes.goHugo.io/) available. A piece of advice if you're testing out lots of free themes: some of them require _very_ specific things in your site's `config.toml` file. Make sure to double check if you're following the theme dev's guidelines for configuration, before moving on to a theme "that works".

Most themes I tried still felt like too much for what I wanted, so I decided to go and create my own. [This guide](https://www.pakstech.com/blog/create-Hugo-theme/) by Janne Kemppainen was extremely helpful. Basically, I used his examples as base taking _all_ the "clutter" out of the way. 

Only two folders live inside `content`:

```
> about
  - index.md
> posts
  - 001-....md
  - 002-....md
```

The numbers in front of the posts are not a theme hard requirement, just something I decided to use to keep things organized. Plus, I can semantically tie images to posts, like `/images/001/whatever.png`.

My `index.html` is basically the list of posts + a simpler version of the `pagination` widget from Janne's post:

```html
{{ $pag := $.Paginator }}
{{ if gt $pag.TotalPages 1 }}
<nav class="pagination is-small is-rounded">
<ul class="pagination-list">
    {{ with $pag.First }}
    <li>
        <a href="{{ .URL }}" class="pagination-link" {{ if not $pag.HasPrev }} disabled{{ end }} aria-label="First"><span aria-hidden="true">&laquo;&laquo;</span></a>
    </li>
    {{ end }}
    <li>
        <a href="{{ if $pag.HasPrev }}{{ $pag.Prev.URL }}{{ end }}" class="pagination-link" {{ if not $pag.HasPrev }} disabled{{ end }} aria-label="Previous"><span aria-hidden="true">&laquo;</span></a>
    </li>
    
    {{ range $pag.Pagers }}
    {{ end }}
    <li>
    <a href="{{ if $pag.HasNext }}{{ $pag.Next.URL }}{{ end }}" class="pagination-link" {{ if not $pag.HasNext }}disabled{{ end }} aria-label="Next"><span aria-hidden="true">&raquo;</span></a>
    </li>
    {{ with $pag.Last }}
    <li>
        <a href="{{ .URL }}" class="pagination-link" {{ if not $pag.HasNext }}disabled{{ end }} aria-label="Last"><span aria-hidden="true">&raquo;&raquo;</span></a>
    </li>
    {{ end }}
</ul>
</nav>
{{ end }}
```

That code will render:

![pagination](/images/001/pagination.png)

In my version, there are no numbers for pages, just simple controls to skip to next/prev or first/last. The new CSS classes come straight from [Bulma](https://bulma.io/) (also recommended by Janne). Think of it as a lighter version of Bootstrap, kinda like what [PureCSS](https://purecss.io/) does. I'm not well versed on all the hidden mysteries of CSS, so any open source CSS framework that's modular and simple to use works for me.

Here's my override to `theme/folder/index.html`:

```html
{{ define "main" }}
<div class="container">
    <div class="section">
        <div class="columns is-centered">
            <div class="column max-content-width">
                <div class="content">
                    {{ .Content }}
                    {{ $paginator := .Paginate (where site.RegularPages "Type" "in" site.Params.mainSections) }}
                    {{ range .Paginator.Pages }}
                    <div class="container">
                        <div class="title is-5">
                            <a href="{{ .Permalink }}">{{ .Title }}</a>
                        </div>
                        <div class="subtitle is-7">
                            <time>{{ .PublishDate.Format "January 2, 2006"}}</time>
                            <div class="summary">
                            {{ .Summary }}
                            </div>
                        </div>
                        <div class="subtitle">
                        </div>
                    </div>
                    {{ end }}
                </div>
            </div>
        </div>
        <div class="columns is-centered">
            <div class="column max-content-width">
                {{ partial "widgets/pagination.html" . }}
            </div>
        </div>
    </div>    
</div>
{{ end }}
```

Just a simple list of _posts_ (notice the rules for `$paginator`), with title, date and summary. Bulma's [column](https://bulma.io/documentation/columns/basics/) ensure a modular and responsive layout, giving me that feeling that I'm back into using `<tr><td>`s for everything in Dreamweaver (I really wished I had a way to find the first ever Quake website I made).

I've added [fontawesome](https://fontawesome.com/icons?d=gallery&m=free) to the theme, which is where those social icons up there are coming from, and the free font [Merriweather](https://fonts.google.com/specimen/Merriweather) by Google. [JQuery](https://jquery.com/) is only used to deal with the "burguer" icon that appears when you access the website using smaller screens. Aside from some minor CSS tweaks and overrides, that's pretty much it. 

All that was left was to host the site somewhere.

## Github Pages

The idea of publishing my site as a public github repo always appealed to me, so I decided to give it a go. Again, the [Hugo docs](https://goHugo.io/hosting-and-deployment/) are of great use when it comes to hosting/deployment. The only small change I made to the instruction was using `ssh` over `https` for my `submodule`, with pub/priv keys for authentication.

One thing I'll say thought, is that when I first publish the `public` folder to the new github repo, all I got was a huge 404 error. Googling led me to think there might be some problem with the files generated by Hugo, since Github's SSG of choice is, obviously, [Jekyll](https://help.github.com/en/github/working-with-github-pages/about-github-pages#static-site-generators) (it was created by one of Github's [co-founders](https://en.wikipedia.org/wiki/Jekyll_(software))). I even thought about hosting Hugo in GitLab, which natively supports Hugo, Jekyll and others... but all it took was a few minutes for my site to pop up correctly. A little bit more patience was all I needed.

Another small thing I forgot to do was fixing the `baseURL` in my site's `config.toml`. That's what Hugo uses to prefix all relative links when the `public` files get generated.

I'll probably add a custom domain to my site's repo at some point, since I don't like `github.io` that much. You can do that [directly inside github](https://help.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site), which is pretty convenient.

## What now?

Well, I guess now I'm finally back into writing. It's been a while (last time I wrote for a blog [was in 2017](https://www.abapzombie.com/mundo-sap/2017/05/31/exite-vida-apos-o-abap/)), and I definitely missed it. Hopefully this will serve as a platform to improve not only my tech, but also english writing skills.

Thanks for reading, and I see you in the next one!


[^1]: I have no data to back up this claim, but I encourage you to try and ask any non-SAP dev you know that had to touch anything ABAP-related.

[^2]: This happened to me once. And again with other plugins.
