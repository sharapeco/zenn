---
title: "Zenn を使ってみた感想"
emoji: "🐤"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["zenn"]
published: false
---

2021年2月3日、春になったので Zenn を使ってみた。

Zenn には GitHub リポジトリを投稿できるという機能がある。これならサービスが終了しても記事が手元に残る、それを手間なくできるところが魅力で使ってみる気になった。

# セットアップ

2021年2月3日現在、次の方法で記事のディレクトリの初期化ができる。かんたんかんたん。

```
mkdir zenn && cd zenn

npm init --yes
npm install zenn-cli
npx zenn init

git init
git commit -m "🐤"
git branch -M main
git remote add origin git@github.com:sharapeco/zenn.git
git push -u origin main
```

## 余談：GitHub のブランチ名について

久しぶりに GitHub のリポジトリを作ったら `git branch -M main` という見慣れない表示があった。気になって調べてみると今後はデフォルトのブランチ名が `main` になるらしい。

- [GitHub、これから作成するリポジトリのデフォルトブランチ名が「main」に。「master」から「main」へ変更 － Publickey](https://www.publickey1.jp/blog/20/githubmainmastermain.html)
- [The default branch for newly-created repositories is now main - GitHub Changelog](https://github.blog/changelog/2020-10-01-the-default-branch-for-newly-created-repositories-is-now-main/)

# Zenn CLI の使い方

記事を作ったりブラウザでプレビューしたり。プレビューはファイルを更新すると即座に反応されるので非常に具合がよい。

```
zenn new:article
zenn new:article --slug 記事のスラッグ
zenn preview
```

オプション `--slug` を指定することでファイル名 (URL) を任意に指定できる。Slug に指定できるのは `/^[0-9a-z-]{12,50}$/` で結構難しい。

# 感想

- もとのデータがテキストデータでしかも手元に何もしなくても残るので、サービスがなくなっても心配いらない。好きなエディタが使えるのがいいし、VSCode なら貼り付けたときにインデントをほどよくしてくれるのでは？
- ただしファイル名が何らかのハッシュ値なのはもう少し人にやさしくならなかったかなと思う。ファイル名に ASCII 以外を使うのは可搬性の問題が絡んでくるだろうし（特に濁点）、記事を書き始めてみたらあとでファイル名を変えたくなることもあるかも。URL にはファイル名がそのまま使われるようだ。
- ファイル名を日本語にするとどうなる？ → おこられる
	- > slugの値（すずめ）が不正です。半角英数字（a-z0-9）とハイフン（-）の12〜50字の組み合わせにしてください
