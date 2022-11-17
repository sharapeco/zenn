---
title: "SSHトンネルが使えないFTPサーバに対し、力技でファイルを同期する"
emoji: "👺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["FTP", "SSH"]
published: true
---

皆さんは Web アプリケーションを本番環境にデプロイするとき、一切の暗号化なしの FTP サーバの接続条件を渡されたので SFTP にできないかと相談したら、もう動いているサーバでそれはできないと断られ、かわりに Web サーバと同じネットワーク上に SSH 接続できるサーバを用意してもらい、トンネル経由で FTP 接続しようと思っていたら MLSD コマンドが何らかの原因で使えず、平文 FTP サーバを用意する業者だから原因究明もまあ無理だろうと慣れ親しんだツールが使えなくて困った経験はありませんか？

```
ローカル ーSSH→ 中継サーバ —FTP→ Webサーバ
```

この記事ではそんなときに役立つファイル同期の力技を紹介します。

# 用意するもの

- bash
- expect
	- 対話型のコマンドラインプログラムを自動操作するためのツール
	- とっても詳しくて役に立つ：[Tclの使い方 - Qiita](https://qiita.com/hana_shin/items/ebd7339da657d5b376dd)
- [fd](https://github.com/sharkdp/fd)
	- 以前に書いた紹介：[高速で記述が簡単な find コマンドの代替「fd」](https://zenn.dev/21f/articles/fd-find-alternative)
- PHP
	- シェルスクリプトでは書きにくい処理をするために使用。書きやすければ他のツールでも可
- ssh, scp
	- 標準で入っていると思う

# 処理の流れ

1. fd で更新ファイルを抽出し、PHP で FTP クライアントのコマンドリストに変換する
2. SSH で中継サーバに作業ディレクトリを作成する
3. scp を使い対象ファイルをすべて中継サーバにコピーする
4. expect を使い、1. で作成したリストを用いて Web サーバにファイルを転送する

# スクリプトの内容

プロジェクトディレクトリ直下にディレクトリ `transfer` を作成し、以下のスクリプトを作成した。

- run.sh
- make-commands.php
- transfer.exp

Git でバージョン管理しているので `.gitignore` に `/transfer` を追加する。
また、`.gitignore` は fd で更新ファイルを検索する際に除外対象として働くので、Git を使っていなくても有用。

## run.sh

`./transfer/run.sh 5h` のように使用する。引数は fd の `--changed-within` オプションに渡すもの。

中継サーバの作業ディレクトリを最初に全て空にしているのは、`scp -r` で既存のファイルが更新されなかったため。

```shell
#!/usr/bin/env bash

# プロジェクトルートに移動
cd `dirname $0`
cd ..

# コマンドリストを作成する
# 引数のチェックも（書きやすいので）任せる
/usr/bin/env php transfer/make-commands.php "$1"

if [ $? != 0 ]; then
  exit
fi

# 中継サーバに作業ディレクトリを作成する
ssh host "rm -r ~/ftp && mkdir -p ~/ftp/public_html"

# 中継サーバに全部コピー
scp -pr app host:ftp/app
scp -pr vendor host:ftp/vendor
scp -pr html/assets host:ftp/public_html/assets

# ファイルを転送
expect transfer/transfer.exp
```

## make-commands.php

```php
<?php

// プロジェクトルートに移動
chdir(dirname(__DIR__));

exit(main($argv));

function main(array $argv): int
{
	$period = filterFdPeriod($argv[1] ?? '');
	if (!$period) {
		echo "Usage: ./transfer/run.sh <interval>\n";
		return 1;
	}

	$files = array_merge(
		// 除外ファイルがあれば -E オプションを追加する
		getLines("fd . --type file -E '/tests/**' --changed-within {$period}"),
		// .gitignore で除外しているディレクトリがあればこのように書く
		getLines("fd . vendor/ --type file --changed-within {$period}")
	);

	$commands = [];
	$dirsCreated = [];
	foreach ($files as $file) {
		// サーバとパスが違うところは変換する
		$file = convertPath($file);

		// ディレクトリを浅い順に作成する
		$dirs = [];
		$dir = dirname($file);
		while ($dir !== '.' && $dir !== '') {
			$dirs[] = $dir;
			$dir = dirname($dir);
		}
		foreach (array_reverse($dirs) as $dir) {
			if (!array_key_exists($dir, $dirsCreated)) {
				$commands[] = "mkdir {$dir}";
			}
		}

		$commands[] = "put {$file}";
	}

	file_put_contents(__DIR__ . '/updates.txt', implode("\n", $commands), LOCK_EX);

	return 0;
}

function convertPath(string $file): string
{
	// 先頭に ./ がついている場合は取り除く
	$file = preg_replace('#^\./#', '', $file);

	// html → public_html
	$file = preg_replace('#^html/#', 'public_html/', $file);

	return $file;
}

function filterFdPeriod(string $val): ?string
{
	if (preg_match('/\\A[1-9][0-9]*(?:y|year|month|w|week|d|day|h|hour|m|min|minute|s|sec|second)\\z/', $val)) {
		return $val;
	}
	return null;
}

function getLines(string $command): array
{
	exec($command, $output, $result);
	if ($result !== 0) {
		throw new RuntimeException(implode("\n", $output));
	}
	return array_values(array_filter($output, fn ($line) => trim($line) !== ''));
}
```

## transfer.exp

```tcl
#!/usr/bin/expect

spawn ssh host

expect "\\\$"
send "cd ftp \r"

expect "\\\$"
send "ftp ftp.example.com \r"

expect "Name"
send "USERNAME\r"

expect "Password"
send "PASSWORD\r"

set fp [open "transfer/updates.txt"]

while {! [eof $fp]} {
	expect "ftp>"
	gets $fp command
	send "$command\r"
}

close $fp

expect "ftp>"
```

# まとめ

これで何とかコマンドひとつでファイル転送できるようになりました。やれやれ。
