---
title: "高速で記述が簡単な find コマンドの代替「fd」"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CLI", "find", "fd", "コマンド"]
published: true
---

# まえがき

GNU find を使ってファイルを探すのにパラメータの指定に嫌気がさし、もっと便利なものがあるはずだと思って検索したら見つけた。

`fd` はたぶん同じように `find` のパターン指定の冗長さに嫌気がさした人が作ったんだと思う。

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

```shell-session
wget https://github.com/sharkdp/fd/releases/download/v8.2.1/fd_8.2.1_amd64.deb
sudo dpkg -i fd_8.2.1_amd64.deb
```

## Windows 10 へのインストール

Scoop もしくは Chocolatey を使ってインストールできる。

```shell-session
scoop install fd
```

# 使い方の例

## ファイル名で検索

`fd` に続けてパターンを指定するとファイル名・ディレクトリ名で検索できる。

```shell-session
$ fd pon
components
img/pon.svg
```

## ディレクトリ内を検索

GNU find と違い `fd pattern... directory` のようにパターンとディレクトリが逆順になる。

```shell-session
$ fd pon img
img/pon.svg
```

## デフォルトで正規表現が使える

```shell-session
$ fd '^a.*e$'
App.vue
components/pages/ApplyDone.vue
components/pages/ApplyForm.vue
components/pages/ApplyIndex.vue
```

## ファイル名の OR 条件で検索

正規表現が使えるので `|` を使用すればよい。

```shell-session
$ fd 'copy|preview'
src\css\public\sections\preview.styl
tasks\copy.js
```

## 拡張子で検索

`-e ext` または `--extension ext` を指定すると拡張子で検索できる。`.` は不要。複数指定すると OR 検索になる。

```shell-session
$ fd -e js -e vue
App.vue
ajaxzip.js
api.js
...
```

## 拡張子と名前の組み合わせ

単純にそれぞれを順不同で並べる。下記の例では `fd map -e xml` としてもよい。

```shell-session
$ fd -e xml map
common/views/sitemap.xml
```

## 特定のパターンを除外する

`-E pattern` または `--exclude pattern` で特定のファイルやディレクトリを除外できる。

```shell-session
$ fd map -E vendor
common/supports/SitemapBase.php
common/views/partials/use_google_maps.html
common/views/sitemap.xml
```

ここは正規表現ではなく glob 形式となっている。

```shell-session
$ fd map -E '*.bak'
```

## ファイルのみ、ディレクトリのみを検索する

`-t` または `--type` に続けてファイルの種類を指定する。種類は次の通り。

- `f` または `file` …… 通常のファイル
- `d` または `directory` …… ディレクトリ
- `l` または `symlink` …… シンボリックリンク (Windows ではジャンクションも含む)
- `e` または `empty` …… 空のファイルまたはディレクトリ

このうち `file`, `directory`, `symlink` は複数指定すると OR 検索となる。`empty` は他の条件との AND 検索となる。

## 最大のディレクトリ階層を指定する

`-d` または `--max-depth` に続けて最大の階層数を指定する。

`--max-depth 1` とするとカレントディレクトリのみから検索する。

## 検索結果に対してプログラムを実行する

`-x` または `--exec` に続けてコマンドを記述する。

後述のプレースホルダを記述しなかった場合は、コマンドの最後に引数として検索結果が追加される。

```shell-session
$ fd -e html map -x wc -l
49 public/views/sitemap.html
6 public/views/partials/require_google_maps.html
1 common/views/partials/use_google_maps.html
```

便利なプレースホルダを使って ImageMagick で形式を変換する例。

```shell-session
$ fd -e png -x convert {} {.}.jpg
==> convert image.png image.jpg のように展開される
```

プレースホルダは次のものが使用できる。

- `{}`: 検索結果のパスそのまま (documents/images/party.jpg)
- `{.}`: 拡張子を除いたもの (documents/images/party)
- `{/}`: ファイル名 (basename) のみ (party.jpg)
- `{//}`: 親ディレクトリ (documents/images)
- `{/.}`: 拡張子を除いたファイル名 (basename) のみ (party)

## ファイルサイズで検索する

`-S` または `--size` に続けてサイズを記述することでファイルサイズで検索できる。

- `fd -S +10M` …… 10 MB 以上のファイルを検索
- `fd -S -1k` …… 1 kB 以下のファイルを検索
- `fd -S 923b` …… 923 バイトちょうどのファイルを検索
- `fd -S +10M -S -12M` …… 10 MB 以上 12 MB 以下のファイルを検索

単位は 10 進の SI 接頭辞 (1 kB = 10³ bytes) の `k`, `M`, `G`, `T` と、2 進ベース (1 KiB = 2¹⁰ bytes) の `Ki`, `Mi`, `Gi`, `Ti` が使用できる。接頭辞のつかないバイト単位の場合は `b` をつける。大文字小文字どちらでもよい。

## 更新日時でファイルを検索する

`--changed-within` や `--changed-before` に続けて日時や期間を指定する。

期間の指定に使える単位には、年 `y` `year`、月 `month`、週 `w` `week`、日 `d` `day`、時間 `h` `hour`、分 `m` `min` `minute`、秒 `s` `sec` `second` がある。

日時は `YYYY-MM-DD HH:MM:SS` 形式で記述し、ローカル時間ではなく UTC で指定する必要があることに注意。

例： 3時間以内に変更されたファイルを検索する

```shell-session
$ fd --changed-within 3h
```

例： 3年より前に変更されたファイルを検索する

```shell-session
$ fd --changed-before 3y
```

例: 2020年5月9日 00:00:00 (UTC) 以降に変更されたファイルを検索する

```shell-session
$ fd --changed-within '2020-05-09 00:00:00'
```

例: 2020年5月9日 00:00:00 (UTC) から 2020年5月12日 20:00:00 (UTC) の間に変更されたファイルを検索する

```shell-session
$ fd --changed-within '2020-05-09 00:00:00' --changed-before '2020-05-12 20:00:00'
```
