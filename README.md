## Install

```
brew update && brew install hugo
```

## Create a New Site

```
cd ~/dev && hugo new site engblog
```

## Make it a Git Repo

```
git init
```

## Edit Site Config 

```
cd engblog && vim config.toml
```

To be whatever you'd like:

```
title = "Eng Blog"
baseurl = "https://psengblog.herokuapp.com"
```

## Start Dev Server

```
hugo server && open localhost:1313
```

## Create a New Post

Generate a file:

```
hugo new post/removing-null-from-csharp.md
```

Edit the file, and follow your heart.

## Edit the Frontmatter

The frontmatter lives at the top of your post.md file.  It is post metadata:

```
title = "Title Case Me"
draft = false
```

## Create a Layout

Put it in the layouts dir:

```
vim layouts/index.html
```

Loop through the pages you've created:

```
{{ range .Data.Pages }}
  <a href="{{ .URL }}">{{ .Title }}</a>
{{ end }}
```

## Create a Stylesheet

Put it in the static dir:

```
vim static/css/style.css
```

Make something basic:

```
body {
  background: #ddd;
  font-family: sans-serif;
}
```

Link in the head:

```
<link href="{{ .Site.BaseURL}}/css/style.css" rel="stylesheet" />
```

## Create a Post Page

The _default section will be used to render posts:

```
vim layouts/_default/single.html
```

The template has the context of a page object:

```
<h1>{{ .Title }}</h1>
<p>
  {{ .Content }}
</p>
```

## Add Code Highlighting

In head:

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.9.0/highlight.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.9.0/styles/atom-one-dark.min.css" type="text/css" media="all" />
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
