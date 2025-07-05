---
layout: post
title:  "gitとjelyllでブログを始める"
date:   2025-06-08T14:25:52-05:00
author: rukaszz
categories: [git]
tags: [environment, blog]
permalink: /posts/git
---

# Gitとjekyllでブログを始める

## Jekyllとは

Jekyll(ジキル)とは静的なサイトを生成するRubyのツール．サイト構成のための要素(カテゴリの列挙など)を，HTMLやMarkdownで記述して実行することで，Webサイトを構成するHTMLファイルを一式で出力する．

外枠は有志作成のテンプレートが複数あり，ブログの作者としてはメモ的に記述したMarkdownファイルを多少手直しするだけでブログとして出力できる非常にありがたいツールである．

### 静的サイト

静的と動的という言葉があり，静的とはユーザの操作に対してサイトのコンテンツを生成したりすることが無いということで，例えばコメントフォームにコメントを書いて投稿すると，サイト下部にコメントが出現するというのはわかりやすい動的な要素です．

静的サイトはユーザがサーバへリクエストした内容(ここではHTMLファイル)をそのまま返すだけのサイトと言えます．基本的にJekyllで作成するサイトは，ユーザのリクエストに対してデータベースからデータを返すような処理はしません．

### Jekyllによる自動生成

HTMLファイルにHTMLを1行ずつ書くことでWebページは作れます．情報系の授業でWebページを作成された経験を持つ人も多いと思います．しかし手作業で大量のHTMLソースを記述するのは大変ですし，繰り返し要素などはいわゆるfor文などで自動で作りたいと思うでしょう．

JekyllはJekyllを用意できる環境があれば，サーバに一式のWebページの設定や投稿内容を記述したファイルを上げれば，それだけでブログのようなページが作れます．すごくざっくりと言えば，次のような手順で外部にWebページを公開できます：

1. AWSのサービスへログインする(AWS マネジメントコンソール)
2. AWSのEC2サービスからインスタンスを起動
3. ruby，Jekyllをコマンド叩いて必要なソフトウェアをインストール
4. Nginxなどでhttp(ポート80)でアクセスできるように設定(できれば)
5. Jekyllのテンプレートや投稿用ファイルの配置
6. 外部からアクセス可能(Nginxで設定すれば)

Jekyllの構成の利点は，1つのディレクトリにまとめられるほど手軽な構成であること，基本的にテキストベースなのでgitなどのバージョン管理が容易であることがあげられます．なお，画像データは必要です．

欠点としては，動的な構成(コメントフォーム)などがJekyll単体では実装できないこと．Jekyllの環境が必要なことくらいでしょう．

### JekyllとGit

Gitで管理が容易という話はしましたが，Github上でブログを公開できるサービスがGithub上に用意されています．詳しくは次節で話しますが，gitのリポジトリで管理しながらブログも外部へ公開できるのは，非常に手軽で強力な利点でしょう．

## Gitでレポジトリを切る

1. gitアカウントを作る
2. gitで新しくレポジトリを切る
3. ダウンロードしてきたjekyllテンプレートをファーストコミット
4. レポジトリの設定の箇所で，「pages」的なタブがあるので，そこで静的ページとしてアクセスできる設定をつける
5. Visit siteでアクセスできる※反映に時間が係るっぽいので，間を開けてからurlへアクセスしたほうが良いかも

## Jekyllを用いてローカルでブログをチェックする

毎度コミットしてブログの内容を微調整するのは面倒かつコミット履歴も膨らむので管理的にも好ましくない．そこで，ローカルでJekyllのブログを構成していきます．

環境は次の通り：

```Bash
# Ubuntuのバージョン
~$ cat /etc/os-release
→NAME="Ubuntu"
→VERSION="20.04.6 LTS (Focal Fossa)"
→ID=ubuntu
→ID_LIKE=debian
→PRETTY_NAME="Ubuntu 20.04.6 LTS"
→VERSION_ID="20.04"

# Rubyのバージョン
~$ ruby -v
→ruby 3.4.1 (2024-12-25 revision 48d4efcb85) +PRISM [x86_64-linux]

# bundleのバージョン
~$ bundle -v
→Bundler version 2.6.9

# gemのバージョン
~$ gem -v
→3.6.9
```

rubyのインストール：

```Bash
sudo apt update
sudo apt install ruby ruby-dev
# 必要なら
gem update --system
```

Jekyllのインストール：

```Bash
gem install bundler jekyll
```

テンプレートはWeb上に無数に公開されていますが，ダウンロードして作業用ディレクトリへ配置すれば，それを使ってブログを立ち上げられます：

URL：[https://jekylltips-ja.github.io/templates/](https://jekylltips-ja.github.io/templates/)

作業用ディレクトリを「./myBlog/」として，以降は勧めていきます．このmyBlogディレクトリに，テンプレートなどが配置されていると仮定します．

ブログ用ディレクトリ構成概要：

```Bash
$ tree
.
├── Gemfile
├── Gemfile.lock
├── LICENSE
├── LICENSE.md
├── README.md
├── _config.yml
├── _includes
│   ├── footer.html
│   ├── head.html
│   ├── header.html
│   ├── nav_links.html
│   ├── page_divider.html
│   └── tooltips
│       └── example.html
├── _layouts
│   ├── archive.html
│   ├── default.html
│   ├── page.html
│   └── post.html
├── _posts
│   ├── aws
│   │   └── 2025-01-26-aws_glossary.md
│   ├── cpp
│   │   └── 2025-05-01-cpp_environment_with_vscode.md
│   ├── docker
│   │   └── 2025-05-10-about_Docker.md
│   └── git
│       ├── 2025-06-01-about_git.md
│       └── 2025-06-01-about_gitblog_with_jekyll.md
├── _sass
│   ├── _layout.scss
│   ├── base
│   │   ├── _base.scss
│   │   ├── _buttons.scss
│   │   ├── _forms.scss
│   │   ├── _grid-settings.scss
│   │   ├── _lists.scss
│   │   ├── _tables.scss
│   │   ├── _typography.scss
│   │   └── _variables.scss
│   ├── bourbon
│   │   ├── _bourbon-deprecated-upcoming.scss
│   │   ├── _bourbon.scss
│   │   ├── addons
│   │   │   ├── _border-color.scss
│   │   │   ├── _border-radius.scss
│   │   │   ├── _border-style.scss
│   │   │   ├── _border-width.scss
│   │   │   ├── _buttons.scss
│   │   │   ├── _clearfix.scss
│   │   │   ├── _ellipsis.scss
│   │   │   ├── _font-stacks.scss
│   │   │   ├── _hide-text.scss
│   │   │   ├── _margin.scss
│   │   │   ├── _padding.scss
│   │   │   ├── _position.scss
│   │   │   ├── _prefixer.scss
│   │   │   ├── _retina-image.scss
│   │   │   ├── _size.scss
│   │   │   ├── _text-inputs.scss
│   │   │   ├── _timing-functions.scss
│   │   │   ├── _triangle.scss
│   │   │   └── _word-wrap.scss
│   │   ├── css3
│   │   │   ├── _animation.scss
│   │   │   ├── _appearance.scss
│   │   │   ├── _backface-visibility.scss
│   │   │   ├── _background-image.scss
│   │   │   ├── _background.scss
│   │   │   ├── _border-image.scss
│   │   │   ├── _calc.scss
│   │   │   ├── _columns.scss
│   │   │   ├── _filter.scss
│   │   │   ├── _flex-box.scss
│   │   │   ├── _font-face.scss
│   │   │   ├── _font-feature-settings.scss
│   │   │   ├── _hidpi-media-query.scss
│   │   │   ├── _hyphens.scss
│   │   │   ├── _image-rendering.scss
│   │   │   ├── _keyframes.scss
│   │   │   ├── _linear-gradient.scss
│   │   │   ├── _perspective.scss
│   │   │   ├── _placeholder.scss
│   │   │   ├── _radial-gradient.scss
│   │   │   ├── _selection.scss
│   │   │   ├── _text-decoration.scss
│   │   │   ├── _transform.scss
│   │   │   ├── _transition.scss
│   │   │   └── _user-select.scss
│   │   ├── functions
│   │   │   ├── _assign-inputs.scss
│   │   │   ├── _contains-falsy.scss
│   │   │   ├── _contains.scss
│   │   │   ├── _is-length.scss
│   │   │   ├── _is-light.scss
│   │   │   ├── _is-number.scss
│   │   │   ├── _is-size.scss
│   │   │   ├── _modular-scale.scss
│   │   │   ├── _px-to-em.scss
│   │   │   ├── _px-to-rem.scss
│   │   │   ├── _shade.scss
│   │   │   ├── _strip-units.scss
│   │   │   ├── _tint.scss
│   │   │   ├── _transition-property-name.scss
│   │   │   └── _unpack.scss
│   │   ├── helpers
│   │   │   ├── _convert-units.scss
│   │   │   ├── _directional-values.scss
│   │   │   ├── _font-source-declaration.scss
│   │   │   ├── _gradient-positions-parser.scss
│   │   │   ├── _linear-angle-parser.scss
│   │   │   ├── _linear-gradient-parser.scss
│   │   │   ├── _linear-positions-parser.scss
│   │   │   ├── _linear-side-corner-parser.scss
│   │   │   ├── _radial-arg-parser.scss
│   │   │   ├── _radial-gradient-parser.scss
│   │   │   ├── _radial-positions-parser.scss
│   │   │   ├── _render-gradients.scss
│   │   │   ├── _shape-size-stripper.scss
│   │   │   └── _str-to-num.scss
│   │   └── settings
│   │       ├── _asset-pipeline.scss
│   │       ├── _prefixer.scss
│   │       └── _px-to-em.scss
│   └── neat
│       ├── _neat-helpers.scss
│       ├── _neat.scss
│       ├── functions
│       │   ├── _new-breakpoint.scss
│       │   └── _private.scss
│       ├── grid
│       │   ├── _box-sizing.scss
│       │   ├── _direction-context.scss
│       │   ├── _display-context.scss
│       │   ├── _fill-parent.scss
│       │   ├── _media.scss
│       │   ├── _omega.scss
│       │   ├── _outer-container.scss
│       │   ├── _pad.scss
│       │   ├── _private.scss
│       │   ├── _row.scss
│       │   ├── _shift.scss
│       │   ├── _span-columns.scss
│       │   ├── _to-deprecate.scss
│       │   └── _visual-grid.scss
│       └── settings
│           ├── _disable-warnings.scss
│           ├── _grid.scss
│           └── _visual-grid.scss
├── _site
│   ├── 2025
│   │   ├── 01
│   │   │   ├── 26
│   │   │   │   └── index.html
│   │   │   └── index.html
│   │   ├── 05
│   │   │   ├── 02
│   │   │   │   └── index.html
│   │   │   ├── 11
│   │   │   │   └── index.html
│   │   │   └── index.html
│   │   ├── 06
│   │   │   ├── 02
│   │   │   │   └── index.html
│   │   │   └── index.html
│   │   └── index.html
│   ├── Gemfile
│   ├── Gemfile.lock
│   ├── LICENSE
│   ├── LICENSE.md
│   ├── README.md
│   ├── about
│   │   └── index.html
│   ├── assets
│   │   ├── header_image.jpg
│   │   ├── icons
│   │   │   ├── android-icon-144x144.png
│   │   │   ├── android-icon-192x192.png
│   │   │   ├── android-icon-36x36.png
│   │   │   ├── android-icon-48x48.png
│   │   │   ├── android-icon-72x72.png
│   │   │   ├── android-icon-96x96.png
│   │   │   ├── apple-icon-114x114.png
│   │   │   ├── apple-icon-120x120.png
│   │   │   ├── apple-icon-144x144.png
│   │   │   ├── apple-icon-152x152.png
│   │   │   ├── apple-icon-180x180.png
│   │   │   ├── apple-icon-57x57.png
│   │   │   ├── apple-icon-60x60.png
│   │   │   ├── apple-icon-72x72.png
│   │   │   ├── apple-icon-76x76.png
│   │   │   ├── apple-icon-precomposed.png
│   │   │   ├── apple-icon.png
│   │   │   ├── browserconfig.xml
│   │   │   ├── favicon-16x16.png
│   │   │   ├── favicon-32x32.png
│   │   │   ├── favicon-96x96.png
│   │   │   ├── favicon.ico
│   │   │   ├── manifest.json
│   │   │   ├── ms-icon-144x144.png
│   │   │   ├── ms-icon-150x150.png
│   │   │   ├── ms-icon-310x310.png
│   │   │   └── ms-icon-70x70.png
│   │   ├── instacode.png
│   │   ├── logo.png
│   │   └── profile-placeholder.gif
│   ├── category
│   │   ├── aws
│   │   │   └── index.html
│   │   ├── cpp
│   │   │   └── index.html
│   │   ├── docker
│   │   │   └── index.html
│   │   └── git
│   │       └── index.html
│   ├── circle.yml
│   ├── css
│   │   └── main.css
│   ├── feed.xml
│   ├── index.html
│   ├── posts
│   │   ├── Docker.html
│   │   ├── aws.html
│   │   ├── cpp.html
│   │   ├── git.html
│   │   └── index.html
│   ├── robots.txt
│   ├── sitemap.xml
│   ├── stackbit.yaml
│   ├── tag
│   │   └── environment
│   │       └── index.html
│   └── typography
│       └── index.html
├── about.md
├── assets
│   ├── header_image.jpg
│   ├── icons
│   │   ├── android-icon-144x144.png
│   │   ├── android-icon-192x192.png
│   │   ├── android-icon-36x36.png
│   │   ├── android-icon-48x48.png
│   │   ├── android-icon-72x72.png
│   │   ├── android-icon-96x96.png
│   │   ├── apple-icon-114x114.png
│   │   ├── apple-icon-120x120.png
│   │   ├── apple-icon-144x144.png
│   │   ├── apple-icon-152x152.png
│   │   ├── apple-icon-180x180.png
│   │   ├── apple-icon-57x57.png
│   │   ├── apple-icon-60x60.png
│   │   ├── apple-icon-72x72.png
│   │   ├── apple-icon-76x76.png
│   │   ├── apple-icon-precomposed.png
│   │   ├── apple-icon.png
│   │   ├── browserconfig.xml
│   │   ├── favicon-16x16.png
│   │   ├── favicon-32x32.png
│   │   ├── favicon-96x96.png
│   │   ├── favicon.ico
│   │   ├── manifest.json
│   │   ├── ms-icon-144x144.png
│   │   ├── ms-icon-150x150.png
│   │   ├── ms-icon-310x310.png
│   │   └── ms-icon-70x70.png
│   ├── instacode.png
│   ├── logo.png
│   └── profile-placeholder.gif
├── circle.yml
├── css
│   └── main.scss
├── feed.xml
├── index.html
├── js
├── posts.md
├── stackbit.yaml
└── typography.md

```

各ディレクトリやファイルの意味は次の通りです(私が理解している範囲で，全部ではないです)：

- GemFile：Jekyllでサイトを構成する際に，実行に必要なものを記述する．テンプレートを使うなら，サーバ立ち上げ時にエラーなどが表示された場合，要求されるパッケージなどはここに記述することで解消できます
- _config.yml：ディレクトリの構成などを記述します．例えば投稿したファイルをディレクトリ単位で管理したい場合などは，ここに記述を追加する必要がありました
- _posts：自分が記述したブログ投稿用ファイルなどはここへ起きます．原則_postsディレクトリの配下にサブディレクトリを切るような管理は出来ないそうですが，_config.ymlなどを触れば行ける
- _layouts：親ページの構成管理など．posts全体の構成などが記述されたhtmlファイルなどが配置されています．テンプレートの構成を変更するつもりがなければ，触らなくても良いかも？
- _includes：サイトの外枠というか構成の部分が記述されているhtmlファイルがある．テンプレートを使うなら基本的にはさわんないはず
- _site：jekyll用のサーバをローカルで立ち上げた時，jekyllが生成したhtmlファイルなどが格納されるディレクトリ．ブログ用のディレクトリ(myBlog)のコピーが，毎度生成され中身が上書きされる，あるいは生成されるイメージ．ここを触って変更を加えても意味がない
