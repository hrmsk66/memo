---
title: "Github Pages + Hugo + remark.js セットアップメモ"
date: 2020-03-07T23:09:34+09:00
draft: false
tags: ["html","css","hugo"]
---

# HUGO

## Install

```
brew install hugo
hugo new site memo
cd memo
```

記事を作成。ローカルサーバを起動してどうレンダリングされるか確認しながら編集。

```
hugo new post/helloworld.md --editor="code"
hugo server -D --watch
```

## Markdown

半角スペース x 2 で改行できる。`</br>` と同じ。

## Theme

Theme は [Kiss](https://github.com/ribice/kiss) を使用。  
`hugo`コマンドでビルド、`public/`をGithub PagesのリポジトリにPush。  
親ディレクトリで`themes/kiss`と`public/`をsubmoduleとして設定してPush。

```
git init
git submodule add git@github.com:ribice/kiss.git themes/kiss
git submodule add git@github.com:hrmsk66/hrmsk66.github.io.git public
```

Themeをカスタマイズしたくなったら`layout/`や`static/`の下に同じ名前のファイルを置いてオーバライドする。  
`theme/`配下のファイルは編集しない。

### layouts/index.html
記事一覧のサマリを削除したかったのでこのパートを削除。
```
  <div class="content">
    {{ .Summary | plainify | safeHTML }}
    {{ if .Truncated }}
    <a class="button is-link" href="{{ .Permalink }}" style="height:28px">
      Read more
    </a>
    {{ end }}
  </div>
```

### layouts/partials/nav.html
右上にメニューを置いた。

```
<div class="nav-right">
  <nav id="nav-items" class="nav-item level is-mobile">
*    {{ range .Site.Menus.main.ByWeight }}
*      <a class="subtitle is-6 level-item" href="{{ .URL }}">{{ .Name }}</a>
*    {{ end }}
```

### static/css/custom.css
タイトルの色を変更。

```
.content h1,
.content h2,
.content h3,
.content h4,
.content h5,
.content h6 {
    color: #000; // Custom Color
}
```

### static/*.png
[Favicon Generator](https://realfavicongenerator.net/)で作ったイメージ(favicon)を`/static`に配置。

## config.toml
コンテンツは記事とスライドの2種類。これを`menu.main`に追加。先述の`layouts/partials/nav.html`への変更によりHome右上に各コンテンツへのリンクが配置される。
コンテンツのURLは日付+ファイル名とする。

```
baseURL = "https://hrmsk66.github.io/"
languageCode = "ja"
title = "memo"
theme = "Kiss"
Paginate = 5
enableRobotsTXT = true

[params.info]
adsense = "" # Adsense ID (ID only, without ca-pub-)
enableSocial = true # Adds OpenGraph and Twitter cards
homeTitle = "" # Title for home page
poweredby = false # Adds powered by hugo and kiss below Copyright
related = true # Includes related articles at the bottom of the article
codeCopy = true # Add copy button above code blocks
taxonomiesCount = true # Add taxonomies count

[params.social]
github = "hrmsk66"

[params.social.config]
platforms = ["github"]

[params.assets]
customCSS = ["css/custom.css"]

[taxonomies]
tag = "tags"

[[menu.main]]
    name = "home"
    pre = "<i class='fa fa-heart'></i>"
    weight = -110
    url = "/"
[[menu.main]]
    name = "post"
    pre = "<i class='fa fa-road'></i>"
    weight = -100
    url = "/post/"
[[menu.main]]
    name = "slide"
    pre = "<i class='fa fa-road'></i>"
    weight = -90
    url = "/slide/"

[permalinks]
post = "/post/:year/:month/:day/:filename/"
slide = "/slide/:year/:month/:day/:filename/"
```

## archetypes

### archetypes/post.md

```
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
categories: ["post"]
tags: ["", ""]
---
```

### archetypes/slide.md

```
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
categories: ["post"]
tags: ["", ""]
---
```

## 複数テンプレートを使い分ける

`/post` のコンテンツには記事用のレイアウト、`/slide` のコンテンツにはスライド用のレイアウトを適用したい。
とりあえず`layouts/slide/single.html`を配置。スライドモードになるともとのページに戻れないのが不便なのでおいおい何とかしたい。

```
<html>
  <head>
  </head>
<body>
<textarea id="source">
{{- .RawContent -}}
</textarea>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/remark/0.14.0/remark.js"></script>
    <script>
      var slideshow = remark.create({
        ratio: '16:9' 
      });
    </script>
</body>
</html>
```

- [Using with Hugo](https://github.com/gnab/remark/wiki/Using-with-Hugo)
- [My experiences with Hugo’s template lookup order](https://discourse.gohugo.io/t/my-experiences-with-hugos-template-lookup-order/9959)

---

# remark.js

https://github.com/gnab/remark/wiki

縦横比を16:9に変更。

```
<script>
  var slideshow = remark.create({
    ratio: '16:9' 
  });
</script>
```

Getting StartedのHTMLではこのURLが使われているがHTTPなのでMixed Contents問題がおきる。

```
<script src="https://remarkjs.com/downloads/remark-latest.min.js">
```

こっちを使用。↑のレイアウト用HTMLは修正済み。

```
<script src="https://gnab.github.io/remark/downloads/remark-latest.min.js"></script>
```


ページをクローンすることで自分はカンペ付きの発表者用ページを、オーディエンスには通常スライドを表示することができる。
`Shift + C`でスライド画面のクローンを作成、`Shift + P` で発表者用画面を表示。
