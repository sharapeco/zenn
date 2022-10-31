---
title: "新しいヘッドレスCMS「Newt」"
emoji: "🥚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Newt", "CMS", "HeadlessCMS"]
published: true
---

[Newt](https://www.newt.so/) は2022年3月にリリースされたばかりの新しいCMS。SaaS製品を提供している株式会社プレイドのメンバーが2021年4月に創業したようだ。

少し試してみたので機能を紹介しつつ所感を述べる。

# 構成

1つのアカウントにつき、スペース（プロジェクト）を複数作成でき、その中にAppを複数作成できる。そしてAppごとにデータのスキーマ（モデル）を定義できるようになっている。

- スペース
	- App
		- モデル

現在はまだリリースされていないが、スペースごとやAppごとに権限設定ができるようになるようだ。

# 料金プラン

Free, Professional, Enterprise が予定されているが、Free 以外は準備中となっている。

Freeプランの制限は次の通り。

- データ転送量 100 GB/月
- 1メディアあたりの容量 50 MB
- APIリクエスト数 2,000,000
- プロジェクトロール（管理者権限のみ）
- Appロール（管理者権限のみ）
- APIキー数 1
- Webhook 1

また、有料プラン含め今後予定されている機能は次の通り。

- 予約投稿
- コンテンツ変更履歴
- コンテンツレビュー
- コンテンツの国際化
- CSVインポート
- IP制限
- ステージング環境
- Appの公開ステータス設定

有料プランは2022年6月のリリースを目指しているとのこと。

この中で注目しているのはコンテンツの国際化機能で、画像コンテンツの扱いや、ロケールに依存しないコンテンツをどう分かりやすく管理画面に落とし込むのか、非常に興味がある。

# モデルデータの種類

- 単一行テキスト
	- ユニーク制約をつけられる
	- 最小・最大文字数を設定できる
	- コンテンツの一覧を表示するときにタイトルとして表示するかを選べる
- リッチテキスト
	- エディタは [Quill](https://quilljs.com/) を使用している
	- エディタのカスタマイズには非対応
	- 表組みができない
- Markdown
	- エディタで undo/redo ができない
	- 表組みをしたければこっち
- 数値
	- 最小値と最大値を設定できる
	- `step` 属性は設定できない
- 日時
	- 日付のみ、時刻のみの選択はできない
- 画像
	- ひとつのフィールドで単数・複数を選べる
- リストボックス
	- 値を数値と文字列から選べる
- チェックボックス
- カラー
	- あらかじめ色とラベルを設定しておき、その中からひとつ色を選ぶことができる
- 絵文字
- 他のモデルデータを参照
	- 単数・複数を選べる
- マルチタイプ
	- 上記の型から任意の値を設定できるフィールド
	- 単数・複数を選べる
- カスタムフィールドタイプ
	- 上記の型を複数組み合わせた子項目を作成できる

少しトリッキーだが、「カスタムフィールドタイプ」を「マルチタイプ」と組み合わせることで、ページ内のみで使用する繰り返し項目を表現することができる。

![](/images/newt-repeat-sections.png)

# ビュー

コンテンツは「ビュー」を使い、モデルごとに一覧形式を見やすいものから選ぶことができる。

- テーブル
	- 表示する項目を選ぶことができる
- ギャラリー
	- 画像を大きく見せることができる
	- 表示する項目を選ぶことができる
- リスト
	- タイトルのみのシンプルなリスト

![](/images/newt-view.png)

# 簡単に使えるテンプレート

Newtではよく使用されるサイトのテンプレートが用意されている。Appをテンプレートから作成し、フロント側のコードをGitHubからcloneするだけですぐに使用できる。

とてもわかりやすくできていて、テンプレート通りのサイトをNewtに登録してから [Vercel](https://vercel.com/) にデプロイするまで10分ほどでできた。

テンプレートは2022年4月現在、次の8種類が用意されている。

- News — ニュース・プレスリリース
- Help center — ヘルプコンテンツ
- Docs — ドキュメンテーション
- Updates — Newsと似ている
- Blog — シンプルなブログ
- Blog2 — WordPressの次を目指す意思を感じるブログ
- Member — 社員紹介
- Landing page — ヒーローエリア、ロゴ、特徴、Call to action、FAQを含むランディングページ

フロントのコードはNuxt.jsとNext.jsの両方が用意されている。

Next.jsのBlog2テンプレートはTypeScriptで書かれていた。

# 画像コンテンツの扱い

コンテンツとして登録した画像は、Newtでの管理もできるほか、外部サービスをアップロード先として選ぶことができる。

Newt内部ではGoogle Cloud Storageを使用しているようだ。

APIでは参照可能なURLとして `src` プロパティが提供されている。`img` 要素に `width` や `height` 属性を指定したいこともあるので、APIから幅や高さが取得できる [GraphCMS](https://graphcms.com/) と比べると心許ない。

# おわりに

すでに商用として十分使える機能を持ち、全体として非常に使いやすいと思った。

早く有料プランで使いたい。