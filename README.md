# How to start your Hugo Blog 

[More about hugo](https://gohugo.io/)

## Install

```
brew update && brew install hugo
```

## Create a New Site

```
hugo new site engblog
```

## Make it a Git Repo

```
git init
```

## Edit Site Title

```
cd engblog && vim config.toml
```

## Create a New Post

```
hugo new post/removing-null-from-csharp.md
```

## Edit the Frontmatter

```
title = "Title Case Me"
draft = false
```

## Create a Layout

```
vim layouts/index.html
```

## Create a Stylesheet

```
<link href="{{ .Site.BaseURL}}/css/style.css" rel="stylesheet" />
```

```
vim static/css/style.css
```

```
body {
 background: yellow;
}
```

```
{{ range .Data.Pages }}
  <a href="{{ .URL }}">{{ .Title }}</a>
{{ end }}
```

## Create a Post Page

```
vim layouts/_default/single.html
```

```
<h1>{{ .Title }}</h1>
<p>
  {{ .Content }}
</p>
```

## Add Code Highlighting

In head:

```
<script src="http://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.9.0/highlight.min.js"></script>
<link rel="stylesheet" href="http://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.9.0/styles/atom-one-dark.min.css" type="text/css" media="all" />
```

Before body close:

```
<script>hljs.initHighlightingOnLoad();</script>
```

## RSS Built In

```
open localhost:1313/index.xml
```

## Deploy to Heroku

```
heroku create engblog
heroku buildpacks:set https://github.com/jaketrent/heroku-buildpack-hugo.git
git add .
git commit -m "Initial commit"
git push heroku master
```
