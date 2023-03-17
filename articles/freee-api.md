---
title: "freee API をアプリ公開申請せずに使う"
emoji: "🕊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["freee", "TypeScript"]
published: true
---

クラウド会計ソフト [freee 会計](https://secure.freee.co.jp/)には API が用意されており、次の項目などを読み書きできる。

- 取引（収入・支出）
- 取引先
- 勘定科目
- 経費科目
- 部門メモタグ
- 見積書
- 請求書

Freee 会計では事業用口座や専用のクレジットカードを使えば自動で取引を仕分けしてくれるというコンセプトなのだが、あいにく使えないケースもある。そのような場合に自分で必要なプログラムを書いて自動、あるいは半自動で連携できるのは魅力だ。

# アプリケーション申請の壁

Freee API では API のアクセスを認可に [OAuth 2.0](https://www.rfc-editor.org/rfc/rfc6749) を採用している。この RFC では grant types として5通りの方法が示されているが、freee API で使用できるのは Authorization Code Grant のみだ（実際に他の方法も試した）。

- [アクセストークンを取得する - freee Developers Community](https://developer.freee.co.jp/startguide/getting-access-token)

OAuth 認証画面を開くため、このページで示されている方法に従い [freeeアプリストア](https://app.secure.freee.co.jp/developers/applications)でアプリを作ったが、アプリタイプを「プライベートアプリ」に設定しているにもかかわらず OAuth 認証画面を開くとアプリが存在しませんとエラーになる。

アプリを開発するために OAuth 認証画面が欲しいのに、アプリが完成していないと OAuth 認証画面にアクセスできないというよく分からない状態になっている。

# 解決方法

試行錯誤した末、OAuth 認証画面を使用せずにアクセストークンを取得できる方法が見つかった。

Freee API のセットアップを済ませると、開発用にアクセストークンを取得するためのリンクを取得できるので、それを用いる。このアクセストークン取得ページがいつまで有効かは分からないが、今のところ半月くらいは使えている。

具体的には次のようにする。

## 会計APIの準備

[freee API スタートガイド](https://developer.freee.co.jp/startguide) にしたがって「1. セットアップ」を済ませる。

セットアップが済むと次の件名のメールが送られてくる。

```
[freee API] 開発用テスト事業所の作成完了のお知らせ
```

ここでメールの「アクセストークン取得ページへ」のリンク先を覚えておく。

## プログラム例

プログラム側ではアクセストークンがない場合、あるいは期限が切れている場合、アクセストークン取得ページへ案内する。そしてユーザは取得したアクセストークンをプログラムに入力する。

コンソールアプリケーションを TypeScript で書く場合は次のようになる。

```typescript
import * as readline from "readline";
import { createLocalStorage } from "localstorage-ponyfill";

// ブラウザの localStorage と同様の機能を提供する
const localStorage = createLocalStorage();

async function main(token: string) {
	...
}

// プロンプトを表示し、ユーザから入力を受け付ける
function question(prompt: string): Promise<string> {
	const rl = readline.createInterface({
		input: process.stdin,
		output: process.stdout,
	});
	return new Promise((resolve) => {
		rl.question(prompt, (input) => {
			rl.close();
			resolve(input);
		});
	});
}

let getAccessToken = localStorage.getItem("getAccessToken");
if (getAccessToken == null) {
	getAccessToken = await question("アクセストークン取得ページのURL: ");
	localStorage.setItem("getAccessToken", getAccessToken);
}

let succeeded = false;
while (!succeeded) {
	try {
		const token = localStorage.getItem("token");
		if (token == null) {
			throw new Error("invalid_access_token");
		}
		await main(token);
		succeeded = true;
	} catch (e) {
		if (e instanceof Error && /\b(?:invalid|expired)_access_token\b/.test(e.message)) {
			console.log("アクセストークン取得ページからアクセストークンを取得してください");
			console.log(getAccessToken);
			const token = await question("アクセストークン: ");
			localStorage.setItem("token", token);
		} else {
			console.error(e);
			break;
		}
	}
}
```
