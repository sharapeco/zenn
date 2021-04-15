---
title: "高速で記述が簡単な find コマンドの代替「fd」"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CLI", "find", "fd", "コマンド"]
published: true
---

# まえがき

GNU find を使ってファイルを探すのにパラメータの指定に嫌気がさし、もっと便利なものがあるはずだと思って検索したら見つけた。

`fd` はたぶん同じように `find` のパターン視程の冗長さに嫌気がさした人が作ったんだと思う。

# fd の特徴

- はやい
- パターンが短く書ける
- デフォルトで正規表現を使える
- スマートな case 判定：パターンに大文字を含む場合のみ大文字小文字を区別する
- 結果が色付きで見やすい
- `.` で始まるファイルや `.gitignore` で指定されたファイルをデフォルトで無視する
- コマンドが `find` に比べて 50% 短い

# インストール

ここに方法が書いてある。

- [sharkdp/fd: A simple, fast and user-friendly alternative to 'find'](https://github.com/sharkdp/fd)

## Ubuntu 18 on WSL 2 へのインストール

`dpkg` を用いてインストールする。

```sh
wget https://github.com/sharkdp/fd/releases/download/v8.2.1/fd_8.2.1_amd64.deb
sudo dpkg -i fd_8.2.1_amd64.deb
```

## Windows 10 へのインストール

Scoop もしくは Chocolatey を使ってインストールできる。

```sh
scoop install fd
```

# 使い方の例

## ファイル名で検索

`fd` に続けてパターンを指定するとファイル名・ディレクトリ名で検索できる。

```sh
% fd pon
components
img/pon.svg
```

## ディレクトリ内を検索

GNU find と違い `fd pattern... directory` のようにパターンとディレクトリが逆順になる。

```sh
% fd pon img
img/pon.svg
```

## デフォルトで正規表現が使える

```sh
% fd '^a.*e$'
App.vue
components/pages/ApplyDone.vue
components/pages/ApplyForm.vue
components/pages/ApplyIndex.vue
```

## 拡張子で検索

`-e ext` を指定すると拡張子で検索できる。`.` は不要。複数指定すると OR 検索になる。

```sh
% fd -e js -e vue
App.vue
ajaxzip.js
api.js
...
```

## 拡張子と名前の組み合わせ

単純にそれぞれを順不同で並べる。下記の例では `fd map -e xml` としてもよい。

```sh
% fd -e xml map
common/views/sitemap.xml
```

## 特定のパターンを除外する

`-E pattern` で特定のファイルやディレクトリを除外できる。

```sh
% fd map -E vendor
common/supports/SitemapBase.php
common/views/partials/use_google_maps.html
common/views/sitemap.xml
```

ここは正規表現ではなく glob 形式となっている。

```sh
% fd map -E '*.bak'
```

## 検索結果に対してプログラムを実行する

`-x` に続けてコマンドを記述する。それぞれのファイルに対して処理される。

```sh
% fd -e html map -x wc -l
49 public/views/sitemap.html
6 public/views/partials/require_google_maps.html
1 common/views/partials/use_google_maps.html
```
