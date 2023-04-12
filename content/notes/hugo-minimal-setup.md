---
title: "Hugo Minimal Setup"
date: 2023-04-03T11:34:38-03:00
draft: false
---

## Needs
Time to time I needed publish set of documents or notes and updating it in while. Org-mode, markdown, wiki - works perfectly. What about publishing?

Recommended Wordpress configuration from AWS. Everything is good... but too much.

![WordPress on AWS](aws-wordpress.png "WordPress on AWS reference architecture")

After review [top generators](https://jamstack.org/generators/) - [Hugo](https://gohugo.io/) looks like my best choice.

## System setup
[How to install](https://gohugo.io/installation/)

To check `hugo version`:
```out
hugo v0.111.3-5d4eb...+extended windows/amd64
```

## Minimal file set
Add configuration without tags and taxonomy.
```yml
baseURL: http://localhost
menu:
  main:
  - name: Start
    pageRef: /
    weight: 10
  - name: Items
    pageRef: /items
    weight: 20
disableKinds:
- taxonomy
- term
```

Add default template for *item*: `file:archetypes/default.md`
```
---
title: {{ replace .Name "-" " " | title }}
date: {{ .Date }}
draft: false
---
```

Add content dir: `mkdir content`
Add test *item*: `hugo new items/todo.md`
```out
Content "content/items/todo.md" created
```

Add start page `hugo new _index.md` and edit `file: content/_index.md`
```
---
title: :)
date: <TIMESTAMP>
draft: false
---
Hello world!
```

Create default HTML template `file:layouts/_default/baseof.html`:
```html
<!DOCTYPE html>
<html lang="{{ .Lang }}">
<head>
    <meta charset="UTF-8">
    <title>{{ .Title | safeHTML }}</title>
</head>
<body>
  <nav>
  {{ range .Site.Menus.main }}
    <a href="{{ .URL }}">{{ .Name | safeHTML }}</a>
  {{ end }}
  </nav>
  <main>
    {{- block "main" . }}{{- end }}
  </main>
</body>
</html>
```

HTML template for start page `file:layouts/index.html`:
```
{{ define "main" }}
<section>
  {{ .Content }}
</section>
{{ end }}
```

HTML template for single *item* `file:layouts/_default/single.html`:
```
{{ define "main" }}
  <article class="single">
    <h1>{{ .Title | safeHTML }}</h2>
    <summary>{{ .Content }}</summary>
  </article>
{{ end }}
```

HTML template for index page `file:layouts/_default/list.html`:
```
{{ define "main" }}
  <section>
    {{ .Content }}
  </section>
  {{ range .Pages.ByPublishDate.Reverse }}
    <article class="list">
      <h2>
        <a href="{{ .RelPermalink }}">{{ .Title | safeHTML }}</a>
      </h2>
      <summary>{{ .Summary | truncate 50 | safeHTML }}</summary>
    </article>
  {{ end }}
{{ end }}
```

Start live-rebuild server: `hugo server --disableFastRender --navigateToChanged`
```out
Start building sites ...
hugo ...
Built in 7 ms
Watching ...
Web Server is available at http://localhost:nnnn/
```

Empty page rendered in 1ms

![GET Time](1ms-page.png "Render time of empty page")

## Ignore working files
Ignore for source control `file:.gitignore`
```
public/*
resources/*
.hugo_build.lock
```

## Notes
Put images to folder with name equals to *item*. Page `todo` can use image from `file:content/items/todo/pic.png`:
```
![Image alternative](pic.png "Mouse hover title")
```
