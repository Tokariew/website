---
title: "Past Sketches – part 2"
date: 2021-08-06T10:58:25+02:00
url: /past-sketches-part-2/
tags:
  - sketches
  - Fountain Pens
  - hugo
draft: false
include_toc: false

---
I updated setup on hugo a little, now scaled down images will be presented,
which are automatically generated using
[Image Processing](https://gohugo.io/content-management/image-processing/) and
[Markdown Render Hooks](https://gohugo.io/getting-started/configuration-markup/#markdown-render-hooks).

<!--more-->

![seras](img/seras.webp "Seras Victoria from Hellsing")
![girl](img/girl1.webp "Who knows…")
![king](img/king.webp "King from One-Punch Man")
![sailor](img/sailormoon.webp "Usagi Tsukino from Sailor Moon and Grid")

### Magic Code

Following snippet will generate preview images with width of 800px, save it in
following path: `layouts/_default/_markup/render-image.html`.

```
{{ $src := resources.GetMatch (.Destination) }}
{{ $small := $src.Resize "800x" }}
<figure><a href={{ $src.RelPermalink }}><img
    src="{{ $src.RelPermalink }}" alt="{{ .Text }}"
    {{ if ge $src.Width "800"}}srcset='{{ $small.RelPermalink }} 800w'{{ end }}/></a>
    {{ if .Title }}<figcaption><h4>{{ .Title }}</h4></figcaption>{{ end }}
</figure>
```
