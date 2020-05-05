---
title: "Netlify + Hugo + ox-hugoでブログを初める第一歩"
author: ["conao3"]
date: 2020-05-05T16:34:00+09:00
lastmod: 2020-05-05T16:20:00+09:00
tags: ["meta", "hugo"]
draft: false
---

## tl;dr {#tl-dr}

-   Netlify + Hugo + ox-hugoでブログを書くことにしました
-   多少の方針決定とコーディングが必要になりましたが、快適なブログ執筆環境が整いました
-   Netlify + Hugo + ox-hugoはいいぞ


## Intro {#intro}

HTMLジェネレータの[seml-mode.el](https://github.com/conao3/seml-mode.el)とOrg->HTMLジェネレータ拡張の[orglyth.el](https://github.com/conao3/orglyth.el)を作っていて、自分のブログくらいはそれらを使って構築しようと思っていました。

しかし結局落ち着かないまま1年が経過し、[leaf.el](https://github.com/conao3/leaf.el)のドキュメントや、専門分野のまとめなどネットで公開したいものだけ溜まっていました。

[conao3.com](https://conao3.com)のドメインも2017年から持っていますが、3年もの間まともなコンテンツは提供されませんでした。

ここらへんで諦めてさっさとHugoとNetlifyで公開してしまえということで、きちんと整備されたツールを使わせてもらい、この度ようやく[conao3.com](https://conao3.com)でまともなコンテンツを提供できるようになりました。


## Netlify {#netlify}

よい情報で溢れているので省略します。

NetlifyはDNSも提供しているようなので、「さくらのドメイン」で取得したconao3.comをNetlify DNSで解決させるようにしました。

また、デフォルトでは古いHugoが動いているようなので、「Build & Deploy」->「Environment」->「Environment variables」で「HUGO\_VERSION」をローカルのバージョンで設定します。
私の場合「0.69.2」でした。


## Hugo {#hugo}


### Hugoのインストール {#hugoのインストール}

[Install Hugo](https://gohugo.io/getting-started/installing/)を参考にHugoをインストールします。

記載がありませんが、Archの場合はpacmanで簡単に入ります。([Archlinux - Package](https://www.archlinux.jp/packages/community/x86%5F64/hugo/))

```shell
# pacman -S hugo
$ hugo version
Hugo Static Site Generator v0.69.2/extended linux/amd64 BuildDate: unknown
```


### ブログテンプレートの展開 {#ブログテンプレートの展開}

[Quick Start](https://gohugo.io/getting-started/quick-start/)を参考にテンプレートを展開します。

私はローカルのレポジトリ置き場は `~/dev/repos` にしています。

```shell
$ cd ~/dev/repos
$ hugo new site netlify-src
```

以下、 `netlify-src` はhugoサイトのルートディレクトリということにします。


### ブログテーマの適用 {#ブログテーマの適用}

[Hugo themes](https://themes.gohugo.io/)からいい感じのテーマを探し、submoduleでレポジトリに持ちます。

テーマはローカルで簡単にカスタマイズできる仕組みがあるので、多少気に入らないところがあっても大丈夫です。

```shell
$ cd quickstart
$ git init
$ git submodule add https://github.com/lxndrblz/anatole.git themes/anatole
```

テーマごとにカスタマイズする方法が違うので、テーマのREADMEを読んで設定をして下さい。

なお、私は当時ローカルで簡単にカスタマイズできる機構を知らず、forkでカスタマイズするしかないと思っていたので、「[Make a Hugo blog from scratch](https://zwbetz.com/make-a-hugo-blog-from-scratch/)」を参考に0からテーマを作りました。

私が作ったテーマは[conao3/anatole-ext](https://github.com/conao3/anatole-ext)で公開しています。


### サンプル記事の追加 {#サンプル記事の追加}

サンプルの記事を追加します。

テーマが `exampleSite` というフォルダを持っている場合は、それらのファイルを `netlify-src/` にコピーするだけでいいです。

コマンドでテンプレートを作成するには次のコマンドを実行します。
実行後、 `content/blog/my-first-post.md` が作成されているので、よしなに編集します。

```shell
$ hugo new blog/my-first-post.md
```


### ローカルサーバーの起動 {#ローカルサーバーの起動}

Hugoは簡単にローカルサーバーを起動でき、さらに依存ファイルが変更されたときに自動でブラウザにリロードさせることができます。

```shell
$ hugo server -D
```

サーバーを起動すると `localhost:1333` で配信されます。


### 静的サイトのビルド {#静的サイトのビルド}

サイト全体のビルドは下記で行います。

Netlifyで公開するならローカルでビルドする必要はないのですが、最終的にどんなツリーになっているかを確認することができます。

```shell
$ hugo
```


## ox-hugo {#ox-hugo}


### ox-hugoのインストールと設定 {#ox-hugoのインストールと設定}

[ox-hugo](https://github.com/kaushalmodi/ox-hugo)をインストールし、適宜設定を行ないます。

「[hugo | IMADENALE](https://pxaka.tokyo/blog/categories/hugo/)」を参考に無難な構成を考えましたが、もっといい方法がある気がします。
とりあえず私は下記の設定と方針でしばらくやってみます。

```emacs-lisp
(leaf ox-hugo
    :doc "Hugo Markdown Back-End for Org Export Engine"
    :req "emacs-24.4" "org-9.0"
    :tag "docs" "markdown" "org" "emacs>=24.4"
    :added "2020-05-05"
    :url "https://ox-hugo.scripter.co"
    :emacs>= 24.4
    :ensure t
    :after ox
    :custom ((org-hugo-auto-set-lastmod . t)
             (org-hugo-allow-spaces-in-tags . nil)
             (org-hugo-front-matter-format . "yaml")))
```


### orgファイル管理の方針 {#orgファイル管理の方針}

ox-hugo用のorgファイルは `netlify-src/org/{{user}}.org` に持つことにしました。

```org
#+title: Hugo blog source
#+author: conao3
#+date: <2020-05-05 Tue>
#+options: ^:{}

#+hugo_base_dir: ../
#+hugo_section: blog

#+link: files file+sys:../static/files/

* blog
:PROPERTIES:
:EXPORT_HUGO_SECTION: blog
:END:

** DONE test                                                      :meta:hugo:
CLOSED: [2020-05-05 Tue 19:21]
:PROPERTIES:
:EXPORT_FILE_NAME: test
:EXPORT_DATE: 2020-05-05T00:00:00+09:00
:EXPORT_HUGO_LASTMOD: [2020-05-05 Tue 16:20]
:END:

testestest.
```

もし将来的にこのorgファイルが超巨大なファイル(1万行~)になれば、適宜 `netlify-src/org/archive-{{user}}-{{num}}.org` に移すことにします。

レベル1は[セクション](https://gohugo.io/content-management/sections/)の分類に使い、レベル2のheadingから記事のツリーと解釈されます。


### 静的ファイル管理の方針 {#静的ファイル管理の方針}

filesのlinkは「[画像の埋め込みテスト | imadenale](https://pxaka.tokyo/blog/2018/a-test-of-images/)」を参考にしました。

スクリーンショットやPDFは `netlify-src/static/files` 以下に持つことにします。

`netlify-src/static` はレポジトリ肥大化を避けて[conao3/netlify-src](https://github.com/conao3/netlify-src)から[conao3/netlify-src-blob](https://github.com/conao3/netlify-src-blob)に切り出し、submoduleで持つことにします。

参考記事ではfilesのリンクをURLで設定していましたが、「[Org Modeのリンク機能で情報集約 | Qiita](https://qiita.com/takaxp/items/96629bbcc4a9403f0213)」を参考に `file+sys:` 指定を使うとorgの画像インライン表示もできますし、きちんとox-hugoによってリンクが修正され、正しいマークダウンが出力されました。

-   Input

    ```org
    #+link: files file+sys:../static/files/

    #+attr_html: :width 128px
    [[files:logo.jpg]]
    ```

-   Output

    ```markdown
    \{\{< figure src="/files/logo.jpg" width="128px" >\}\}
    ```

-   Rendering

    {{< figure src="/files/logo.jpg" width="128px" >}}


## Deploy {#deploy}

NetlifyのデプロイはGitHubにpushするだけです!

Hugoのビルドはとても早く、pushしてNetlifyのログを見に行くともう終わっています。

ビルドできたら個人Slackに通知飛したりするのをまた今度やりたいと思っています。


## Conclusion {#conclusion}

Netlify + Hugo + ox-hugoで快適なブログ執筆環境を整えることができました!

ようやく情報発信する環境が整ったので、どんどん情報を発信していきたいと思います。

まずは見やすいleaf.elのドキュメントを書きます!

また、執筆現在は[conao3/netlify-src](https://github.com/conao3/netlify-src)と[conao3/netlify-src-blob](https://github.com/conao3/netlify-src-blob)をパブリックレポジトリとしていますが、後でプライベートレポジトリに変更し、[Patreon](https://www.patreon.com/conao3)の特典にします。

本来公開するものを非公開にすることでしか価値を付けられないことをお許しください。

ぜひ[Patreon](https://www.patreon.com/conao3)で私の活動のサポートを頂けるとうれしいです!
