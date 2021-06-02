---
title: "Apache 2.4 で静的アセットのキャッシュ制御を行う設定"
emoji: "🕰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Apache"]
published: true
---

# はじめに

ウェブページの表示速度を改善する方法のひとつとして、適切なキャッシュ設定を行うことが挙げられる。

この文書では、静的アセット（画像、CSS、JavaScript、フォント）が共通キャッシュに 1 週間保存されるよう設定するのが目標である。つまり応答ヘッダに次の内容を追加されるように設定する。

```
Cache-Control: public, max-age=604800
Expires: Wed, 09 Jun 2021 04:55:22 GMT
```

ウェブサーバは Apache 2.4 とし、必要なモジュールや .htaccess で設定を行う場合に必要な設定の情報をまとめてある。

# 基本設定

この設定を使用するには次のモジュールが必要なので、あらかじめ読み込まれるよう設定しておく。

- Expires モジュール (mod_expires)
- Headers モジュール (mod_headers)

## Expires ヘッダを出力する

画像、CSS、JavaScript、フォントに対して Expires ヘッダを出力するには、次の設定をサーバ設定ファイル、バーチャルホスト、ディレクトリ、.htaccess のいずれかに記述する。

```
ExpiresActive On

ExpiresByType image/gif "access plus 1 weeks"
ExpiresByType image/jpeg "access plus 1 weeks"
ExpiresByType image/png "access plus 1 weeks"
ExpiresByType image/svg+xml "access plus 1 weeks"
ExpiresByType image/webp "access plus 1 weeks"
ExpiresByType text/css "access plus 1 weeks"
ExpiresByType text/javascript "access plus 1 weeks"
ExpiresByType application/javascript "access plus 1 weeks"
ExpiresByType font/woff "access plus 1 weeks"
ExpiresByType font/woff2 "access plus 1 weeks"
```

期間の単位には years, months, weeks, days, hours, minutes, seconds が使用できる。環境によっては単数形も使用できることがあるが、エラーになる場合もあるので必ず複数形を使用する。

もし静的アセットのみが配置されたディレクトリに対して適用する場合であれば、ファイルの種類の判別が不要なので次のように書ける。

```
ExpiresActive On
ExpiresDefault "access plus 1 weeks"
```

もしかすると WebP や Web フォントに対して MIME タイプが設定されていないかもしれないので、その場合は MIME モジュール（mod_mime）を使用してタイプを追加する。

```
AddType image/webp webp
AddType font/woff woff
AddType font/woff2 woff2
# EOT, TrueType はすでに必要ないだろう
```

- [mod_expires - Apache HTTP サーバ バージョン 2.4](https://httpd.apache.org/docs/2.4/ja/mod/mod_expires.html)
- [AddType ディレクティブ - mod_mime - Apache HTTP サーバ バージョン 2.4](https://httpd.apache.org/docs/2.4/ja/mod/mod_mime.html#addtype)

## Cache-Control ヘッダを出力する

画像、CSS、JavaScript、フォントに対して Cache-Control ヘッダを出力するには、次の設定をサーバ設定ファイル、バーチャルホスト、ディレクトリ、.htaccess のいずれかに記述する。

```
<FilesMatch "\.(gif|jpe?g|png|svgz?|webp)$">
    Header set Cache-Control "public, max-age=604800"
</FilesMatch>

<FilesMatch "\.(css)$">
    Header set Cache-Control "public, max-age=604800"
</FilesMatch>

<FilesMatch "\.(js)$">
    Header set Cache-Control "public, max-age=604800"
</FilesMatch>

<FilesMatch "\.(woff2?)$">
    Header set Cache-Control "public, max-age=604800"
</FilesMatch>
```

もし静的アセットのみが配置されたディレクトリに対して適用する場合であれば、ファイルの種類の判別が不要なので次のように書ける。

```
Header set Cache-Control "public, max-age=604800"
```

ここでは静的アセットを想定しているのですべて `public, max-age=604800` に設定しているが、必要に応じて `private`, `no-store`, `no-cache`, `must-revalidate` を使い分ける。

- [Header ディレクティブ - mod_headers - Apache HTTP サーバ バージョン 2.4](https://httpd.apache.org/docs/current/ja/mod/mod_headers.html#header)
- [Cache-Control - HTTP | MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Cache-Control)

## ETag を出力しないようにする（必要な場合のみ）

Apache にはファイルをそのまま HTTP レスポンスとして返す場合、リソースの識別子である ETag を自動的に出力する機能があり、デフォルトでオンになっている。

ところが Apache の ETag はディスク上の管理番号である inode 番号に基づいているため、配信サーバが複数あると同じリソースを指していても異なる ETag になってしまう。

そのような場合は次の設定を、サーバ設定ファイル、バーチャルホスト、ディレクトリ、.htaccess のいずれかで行う。

```
FileETag None
```

- [ETag - HTTP | MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/ETag)
- [FileETag ディレクティブ - core - Apache HTTP サーバ バージョン 2.4](https://httpd.apache.org/docs/2.4/ja/mod/core.html#fileetag)

# .htaccess で設定する場合

上記設定を .htaccess で行う場合には FileInfo と Indexes の上書きが許可されている必要がある。

設定の上書きを許可するにはディレクトリに対して次の設定を追加する。

```
<Directory /path/to/www/assets>
	# 既存の設定に加え FileInfo と Indexes を追加
	AllowOverride FileInfo Options AuthConfig Indexes
</Directory>
```

AllowOverride ディレクティブは、次のようにワイルドカードを使用したディレクトリに対しても適用できる。

```
<Directory "/path/to/www/assets-*">
	# 既存の設定に加え Indexes を追加
	AllowOverride FileInfo Options AuthConfig Indexes
</Directory>
```

正規表現を使った `<Directory ~ "...">` のようなセクションに対しては適用できないことに注意する。

- [AllowOverride ディレクティブ - core - Apache HTTP サーバ バージョン 2.4](https://httpd.apache.org/docs/2.4/ja/mod/core.html#allowoverride)
