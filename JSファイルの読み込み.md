# ★RailsにJSファイルを読込させる方法
JavaScriptをRailsで読み込む方法を解説していく。<br>
やり方としてはRails7のesbuild形式の場合となる。<br>
<br>
### 🐻‍❄️JavaScriptの適用形式の判断
JavaScriptをどの形式で適用しているかの簡単な判断基準としてWebサーバー起動時の<br>
ログの<br>
```
docker compose exec web bin/dev
20:03:59 css.1  | started with pid 15
20:03:59 js.1   | started with pid 16
20:04:00 js.1   | yarn run v1.22.22
20:04:00 js.1   | $ esbuild app/javascript/*.* --bundle --sourcemap -- <=ここ！！ outdir=app/assets/builds --public-path=assets --loader:.js=jsx --watch
```
esbuildと表示されている部分にどの形式でJavaScriptを読み込んでいるかが表示されます。<br>
<br>
<br>
## 目標
JavaScriptをビューファイルに適用する<br>
<br>
<br>
## 概要
JavaScriptのロジック記入用ファイルを新規作成して、記述したロジックをモジュール化して<br>
ビューファイルに適用するまでの過程を解説する。<br>
<br>
<br>
## 🐻‍❄️実装工程
🌱 ステップ1：新規の .js ファイルを作る<br>
🌱 ステップ2：application.js に import 追加<br>
🌱 ステップ3：application.html.erb に JS を読み込ませる<br>
<br>
<br>
## 実装
### 🌱 ステップ1：共通の JS ファイルを作る
JavaScriptのコードを記入する為の新規ファイルを作成<br>
```
touch app/javascript/sample.js
```
　
作成したファイルを編集する<br>
```
export function sample_js() {


任意のJavaScriptのコードを記述



}
```
sample_js の部分は任意のモジュール名でOKです。<br>
<br>
<br>
### 🌱 ステップ2：application.js に import 追加
最初に作成した sample.js をapp/javascript/application.jsにimportします。<br>
```
#app/javascript/application.js

import { sample_js } from "./sample";
document.addEventListener("DOMContentLoaded", sample_js);
console.log("😂JS同期モジュール読み込まれました！");  // 確認用
```
<br>
確認用のログは適用させたいページが読み込まれたときに開発者ツールのコンソールに表示されます。<br>
<br>
次に application.js を以下のコードを実行してビルドします。<br>

```
docker compose exec web ./bin/dev
```
ただし、コンテナのWebサーバが起動済みの場合は自動でビルドされるので実行不要です。<br>
<br>
<br>
### 🌱 ステップ3：application.html.erb に JS を読み込ませる
JavaScriptのコードを適用させたいビューのレイアウトファイル(application.html.erb等)の<br>
header にJavaScriptを読み込むためのヘルパーメソッドを記述します。<br>
```
<script type="module" src="/assets/application.js"></script>
```
<br>
これでJavaScriptの適用手順は終了です。<br>
