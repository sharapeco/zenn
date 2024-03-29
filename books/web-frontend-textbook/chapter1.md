---
title: "HTMLに親しむ"
---

HTML は文書構造を表現するためのマークアップ言語です。この章では HTML を自力で調べて書けるようになることを目標とします。

# 基本構造

HTML の基本構造は次のようになっています。

```html
<!DOCTYPE html>
<html lang="ja">
	<head>
		<meta charset="UTF-8">
		<title>Webページ</title>
	</head>
	<body>
		<h1>Webページ</h1>
		<p>これはすごいWebページです</p>
	</body>
</html>
```

`<` から始まり `>` で終わる部分を**タグ**といい、このタグをつけることによって文書を構造化していきます。また、タグをつけていくことを**マークアップ**といいます。

HTML は木構造になっています。`<body>` から `</body>` までが木構造における要素で、 `<body>` を開始タグ、 `</body>` を終了タグといいます。開始タグと終了タグの間にあるのがその子要素となります。

つまり、この例にある「これはすごいWebページです」というテキストは `html > body > p > #text` という階層に位置することになります。

また、終了タグの存在しない要素もあります。**空要素**といって内容を含めることができない要素です。この例では `<meta charset="UTF-8">` が該当します。

開始タグにある `charset="UTF-8"` のような部分を**属性**といいます。`属性名="属性値"` のように記述します。

:::message
#### HTML の規格

現在の HTML の規格は Web Hypertext Application Technology Working Group (WHATWG) という団体が策定している HTML Living Standard というものです。

過去には World Wide Web Consortium (W3C) が策定した HTML 4.01 や XHTML 1.0 などがあり、たまに見かけることがあります。
:::

### 演習

1. 普段見ている Web ページも同じ構造になっているか確認してみなさい。Web ブラウザのメニューから「ソースの表示」を選ぶと HTML の記述を確認することができます。
2. 先ほどの例を実際にファイルを作って Web ブラウザで表示してみなさい。また、文章を変えてみなさい。

# 見出しと段落

本を読んでいると第 1 章や 1.1 節（あるいは数字のみ）といった表記を見かけることがあります。この章や節のタイトルにあたる部分が**見出し**で、 `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>` を使用します。

また、本文を記述するときには**段落**（形式段落）ごとに `<p>` でマークアップします。

```html
<h1>第1章 HTMLに親しむ</h1>
<p>HTML は文書構造を表現するためのマークアップ言語です。</p>

<h2>基本構造</h2>
<p>HTML の基本構造は次のようになっています。</p>
...
```

見出しは最も大きいものを `<h1>`、最も小さいものを `<h6>` で表します。通常、ページには必ず `<h1>` が現れるはずです。

見出しの大きさには厳格なルールがあるわけではありませんが、階層が上位のものほど数字が小さくなるようにします。また、それほど重要でない見出しには大きな数字を使います。

### 演習

1. 手近にあるチラシや案内文書などから文字情報を抜き出し、見出しと段落でマークアップしてみなさい。

# リスト

見出しと段落がマークアップしたら、それだけでは足りないと思うことがあるでしょう。

HTML には 3 種類のリストが用意されています。

## 箇条書き（順不同リスト）

項目を列挙するための箇条書きは `<ul>` と `<li>` を使って記述します。

```html
<p>明日の遠足の持ち物は次の通りです。</p>
<ul>
	<li>お弁当</li>
	<li>水筒（水またはお茶）</li>
	<li>おやつ（300円まで）</li>
	<li>ゴミ袋</li>
</ul>
```

デフォルトでは項目の先頭にビュレット • がつく表示になります。

## 順序付きリスト

順序に意味があるリストは `<ol>` と `<li>` を使って記述します。

```html
<p>明日の遠足は次のように行動してください。</p>
<ol>
	<li>午前8:30までに集合</li>
	<li>点呼</li>
	<li>バスに乗車</li>
</ol>
```

デフォルトでは項目の先頭に「1.」「2.」「3.」と番号がついた表示になります。

## 説明リスト（連想リスト）

用語とその定義、情報の名前とその値、質問と回答など対になる情報を記述する際には `<dl>` `<dt>` `<dd>` を使用します。

```html
<dl>
	<dt>バナナはおやつに入りますか？</dt>
	<dd>バナナは三食の食事でも食べ、食事以外のときにも食べるということは、
		おやつでもあり、おやつではないということです。</dd>
</dl>
```

### 演習

1. さきほど見出しと段落でマークアップした文書にリストを加えなさい。リストとしてマークアップできるものがないときはリストとしてマークアップできる内容を加えなさい。

# ハイパーリンク

HTML の最も重要な機能として、他の文書を参照できる**ハイパーリンク**があり、`<a>` 要素を使ってマークアップします。

```html
<p>
	HTML の仕様を厳密に確認したいときは
	<a href="https://html.spec.whatwg.org/multipage/">HTML Living Standard</a>
	を確認してね。
</p>
```

`target` 属性には参照先のリソースの URL を記述します。

- 絶対 URL と相対 URL

# テーブル

# メタデータ

- link rel prev など
- meta name="description" など

# ビューポートとスタイルシート

