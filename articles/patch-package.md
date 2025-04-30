---
title: "patch-package で npm パッケージに簡単にパッチを当てる"
emoji: "🩹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["npm", "patch", "patchpackage", "jspdf"]
published: true
---

# まえがき

patch-package を使うと手間を最小限に npm パッケージにパッチを当てることができる。

他の方法としては、(1) 手動でパッチを作成し適用する方法 や (2) パッケージをクローンしてローカルからインストールする方法、(3) パッケージを fork して自分のリポジトリに置く方法 がある。

patch-package を使うと、共同開発者に手動でパッチを適用してもらう手間をなくし (1)、Dependabot などのツールをフルに活用 (2) (3) できる。

今回は parallax/jsPDF を使用しているプロジェクトで、CIFF (Canon Camera Image File Format) の画像がサポートされていないファイルタイプと表示されエラーになる問題を解決するために、patch-package を使ってパッチを当てた。

- [AddImage does not support files of type 'UNKNOWN' · Issue #3692 · parallax/jsPDF](https://github.com/parallax/jsPDF/issues/3692)

# 手順

## 1. スクリプトを追加

package.json に以下のスクリプトを追加する。

```json
  "scripts": {
+    "postinstall": "patch-package"
  }
```

## 2. patch-package をインストール

```bash
npm install patch-package
```

## 3. パッケージを修正

今回の jsPDF の例では `node_modules/jspdf/dist/jspdf.*.js` にあるビルド後のファイルにパッチを当てたいので、まずは jsPDF をクローンして修正したあと、ビルドを実行する。

```bash
cd ~/repos
git clone https://github.com/parallax/jsPDF.git
cd jsPDF
git checkout v3.0.1
npm install
${EDITOR} src/modules/addimage.js
npm run build
```

こうしてビルドしたファイルを `node_modules/jspdf/dist/jspdf.*.js` にコピーする。

```bash
cd ~/repos/your-project
cp ~/repos/jsPDF/dist/jspdf.*.js node_modules/jspdf/dist/
```

## 4. パッチを作成

次のコマンドを実行すると `patches/jspdf+3.0.1.patch` が作成される。

```bash
npx patch-package jspdf
```

# パッチが適用されているか確認

パッチが適用されるかどうかを確認するため、いったん `node_modules` を削除してから、再インストールする。

```bash
rm -rf node_modules
npm install
```

うまくいけば次のようにパッケージを適用したメッセージが表示される。

```shellsession
> postinstall
> patch-package

patch-package 8.0.0
Applying patches...
jspdf@3.0.1 ✔
```

パッケージのバージョンが変わった場合、マージできる場合は自動的にマージされ、できない場合はエラーになる。

# 参考

- [ds300/patch-package: Fix broken node modules instantly 🏃🏽‍♀️💨](https://github.com/ds300/patch-package)
- [patch-package を使ってnpmパッケージに安全にパッチを当てる方法 | N Inc. Developers Blog](https://dev.blog.n.inc/2021-06-21-patch-package/)
- [npmでローカルのパッケージをinstallする方法 #JavaScript - Qiita](https://qiita.com/suin/items/c9c342f557bd885dbe06)
