# ★RailsにJSファイルを読込させる方法
JavaScriptをRailsで読み込む方法を解説していく。
やり方としてはRails7のesbuild形式の場合となる。


## 目標
JavaScriptをビューファイルに適用する


## 概要
JavaScriptのロジック記入用ファイルを新規作成して、記述したロジックをモジュール化して
ビューファイルに適用するまでの過程を解説する。


## 🐻‍❄️実装工程
🌱 ステップ1：新規の .js ファイルを作る
🌱 ステップ2：application.js に import 追加
🌱 ステップ3：application.html.erb に JS を読み込ませる
🌱 ステップ4：HTMLに id をつける


## 実装
### 🌱 ステップ1：共通の JS ファイルを作る
JavaScriptのコードを記入する為の新規ファイルを作成
```
touch app/javascript/sample.js
```

作成したファイルを編集する
```
export function sample_js() {


任意のJavaScriptのコードを記述



}
```
sample_js の部分は任意のモジュール名でOKです。


### 🌱 ステップ2：application.js に import 追加
最初に作成した sample.js をapp/javascript/application.jsにimportします。
```
#app/javascript/application.js

import { sample_js } from "./sample";
document.addEventListener("DOMContentLoaded", sample_js);
console.log("😂JS同期モジュール読み込まれました！");  // 確認用
```

確認用のログは適用させたいページが読み込まれたときに開発者ツールのコンソールに表示されます。

次に application.js を以下のコードを実行してビルドします。
```
docker compose exec web ./bin/dev
```
ただし、コンテナのWebサーバが起動済みの場合は自動でビルドされるので実行不要です。


