---
title: "Hugo + GitHub Pages + github-styleでウェブサイトを構築する"
date: 2023-03-21T18:04:49+09:00
draft: true
categories: "tech"
tags: ["hugo"]
---

## はじめに

自分の活動や経歴を紹介するウェブサイトが欲しかったので、静的サイトジェネレーターであるHugoとPaperModというHugoのテーマを使って、Netlify上に作成しました。

## Hugoの始め方

以下の手順に沿って進めることで、ウェブサイトを簡単に構築できます。

https://gohugo.io/getting-started/quick-start/

1. Hugoのインストール

Ubuntuを使っているので、以下のコマンドでHugoをインストールします。
```js
sudo apt install hugo
```

2. ウェブサイトの構築

[Hugo Themes](https://themes.gohugo.io/)で公開されている様々なテーマを使うことができます。今回は[Paper](https://themes.gohugo.io/themes/hugo-papermod/)のテーマを選びました。
```js
hugo new site kkato.dev
cd kkato.dev
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/github-style
rm config.yaml

hugo server
```

`http://localhost:1313/`から作成したウェブサイトを確認することができます。

3. 記事の作成

以下のコマンドで記事を作成し、markdownで記載します。　

```
hugo new posts/my-first-post.md
```

## テーマの設定
github-styleの設定は[こちら](https://github.com/MeiK2333/github-style/blob/master/config.template.toml)を参考に行います。

```js:config.toml
baseURL = "このウェブサイトのURL"
languageCode = "en-us"
title = "タイトル"
theme = "github-style"
googleAnalytics = "Google Analytics"
pygmentsCodeFences = true #コードブロックを言語ごとにカラーリング
pygmentsUseClasses = true #カラーリングをCSSを使ってカスタマイズ

[params]
  author = "名前"
  description = "自己紹介"
  github = "GitHubのアカウント"
  facebook = "Facebookのアカウント"
  twitter = "Twitterのアカウント"
  linkedin = "LinkedInのアカウント"
  instagram = "Instagramのアカウント"
  tumblr = "Tumblrのアカウント"
  email = "メールアドレス"
  url = "自己紹介の欄に表示させるURL"
  keywords = "blog, google analytics"
  rss = true #RSSを公開する
  lastmod = true #最後にいつ編集されたか表示する
  avator = "アイコン画像へのパス"
  favicon = "ブラウザのタブに表示されるアイコン画像へのパス"
  location = "ロケーション"
  userStatusEmoji = "😀"
  enableGitalk = true #Gitalkを使ってコメントを表示する
  
  [[params.links]]
    title = "アカウント"
    href = "そのURL"
    icon = "アイコン画像へのパス"
```

ちなみに私の場合はこんな感じです。
```
baseURL = 'http://kkato.github.io/'
languageCode = 'en-us'
title = 'kkato.github.io'
theme = 'github-style'

[params]
  author = "Ken Kato"
  github = "kkato"
  linkedin = "ken-kato"
  keywords = "hugo, blog, github-style"
  rss = false
  lastmod = false
  avatar = "/images/sagarifuji.svg"
  favicon = "/images/sagarifuji.svg"
  location = "Tokyo, Japan"
  enableGitalk = false
```

## GitHub Pagesへのデプロイ

予め"ユーザ名.github.io"のようなレポジトリを作成しておいてください。レポジトリが準備できたら、以下のコマンドを実行してコードをレポジトリにpushします。
```
cd kkato.github.io
git add -A
git commit -m "Initial commit"
git remote add origin https://github.com/kkato/kkato.github.io
$git push -u origin master                                
```

そして、GitHub ActionsによるGitHub Pagesへの自動デプロイを設定します。Settings > Pages> Build and deployment > Sourceで"GitHub Actions"を選択します。
![](/images/hugo/githubpages1.png)

そうすると以下のようにHugoのworkflowが勝手に提案されます。続けてConfigureを押して、hugo.ymlを作成してください。
![](/images/hugo/githubpages2.png)

そうすると、Actionsに以下のような項目が表示され、緑になっていればhttps://ユーザ名.github.ioにデプロイされています。
![](/images/hugo/githubpages3.png)

## おわりに

HugoとGitHub Pagesを使ってウェブサイトを構築しました。想像以上に簡単に構築できるので、ウェブサイトを作ってみたいと考えている方は試してみてください。

https://kkato.github.io/