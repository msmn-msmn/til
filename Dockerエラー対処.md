#　★Dockerのトラブルシューティング#
## コンテナが停止できなくなる
　●対処法<br>
　　パソコンを再起動すると停止するので、問題のコンテナを削除して<br>
  ```
　　docker compose build
```
　　で新しいコンテナを生成する<br>

  🔹 特定のコンテナを削除する<br>
　　　まず、削除したいコンテナのIDまたは名前を確認します<br>
　　　bash<br>
   ```
　　　　docker ps -a
```

　　　その後、以下のコマンドでコンテナを削除します<br>
　　　bash<br>
   ```
　　　　docker rm [コンテナIDまたは名前]
```

　　　実行中のコンテナを削除する場合は、-f オプションを使用して強制的に削除できます<br>
　　　bash<br>
   ```
　　　　docker rm -f [コンテナIDまたは名前]
```

　🔹 すべての停止中のコンテナを一括削除する<br>
　　　停止中のすべてのコンテナを一度に削除するには、以下のコマンドを使用します<br>
　　　bash<br>
   ```
　　　　docker rm $(docker ps -a -q)
```

　　　このコマンドは、すべてのコンテナIDを取得し、それらを削除します。​<br>
<br>
　🔹 使用されていないすべてのコンテナ、イメージ、ボリューム、ネットワークを一括削除する<br>
　　　未使用のリソースを一括で削除するには、以下のコマンドを使用します<br>
　　　bash<br>
   ```
　　　　docker system prune -a
```

　　　このコマンドは、未使用のコンテナ、イメージ、ボリューム、ネットワークをすべて削除します。​<br>
<br>
　⚠️ 注意点<br>
　　　docker rm は停止中のコンテナを削除します。​<br>
　　　実行中のコンテナを削除するには、-f オプションを使用して強制的に削除します。​<br>


# ★`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`が出てdocker関連のコマンドが実行できない時
　●対処法<br>
　　Dockerのデスクトップアプリを再インストールする<br>
　　※RailsのDBがアンインストールで消えてしまうので<br>
　　　`docker compose exec web bin/rails db:create`　#DB構築<br>
　  　`docker compose exec web rails db:migrate`　#構築したDBにマイグレーションを実行<br>
　　　コマンドを実行してDBを再構築する<br>
<br>
<br>
   # ★Webコンテナが起動後即停止してしまう<br>
   ### ‼️症状
　　Webコンテナを起動後数行で勝手にWebコンテナが停止してしまう<br>
   
   ```
　　　　$ docker compose exec web bin/dev

------------------中略--------------------------------

　　　　10:07:47 web.1  | => Booting Puma
　　　　10:07:47 web.1  | => Rails 6.1.7.3 application starting in development
　　　　10:07:47 web.1  | => Run `bin/rails server --help` for more startup options
　　　　10:07:48 web.1  | A server is already running. Check /app/tmp/pids/server.pid.
　　　　10:07:48 web.1  | Exiting
　　　　10:07:48 web.1  | exited with code 1
　　　　10:07:48 system | sending SIGTERM to all processes
　　　　10:07:48 js.1   | exited with code 1　　　　　　　<=ここまでで停止する
```
　●原因<br>
 　ログに<br>
  ```
10:07:48 web.1  | A server is already running. Check /app/tmp/pids/server.pid.
```
 　とあり、これは起動中のコンテナがすでに起動している事を示しています。<br>
 　よって、起動済みと判断されて立ち上げ処理が終了してしまう。<br>
 　これが、起動後に即停止してしまう原因です。<br>
 　何故起動済みと判断されるかというとこの部分<br>
<br>
```
　　　　Check /app/tmp/pids/server.pid.
```

 　本来、正常に終了されているなら削除されるはずのpidファイルが残っているからです。<br>
 　原因として、docker compose up で既にWebサーバーが起動しているのでなければ<br>
 　これが強制終了等の理由で削除されずに残ってしまっているのが根本の原因となります。<br>
 　docker compose downで終了した場合はこのファイルは削除されます。<br>
 　pidファイルを消してからWebサーバーを起動させた時にまたpidファイルが原因で<br>
 　起動プロセスが終了する場合はdocker compose up で既にWebサーバーが起動している<br>
 　可能性が高いです。<br>
 　そうでないとした場合は次の流れで起動プロセスが終了しています。<br>
```
　　　　pidファイルの中身
　　　　tmp/pids/server.pid
　　　　1
```
 　ひとつだけ書かれた数字はRails ServerのPIDです。<br>
 　Linuxでは各プロセスにIDをつけて管理しています。またDockerでは, <br>
 　プロセスIDが1であるプロセスが終了した場合, コンテナも終了するようになっています。<br>
<br>
 　この状態で起動を試みると, <br>
```
　　　　１．Railsは server.pid を見て「もうサーバー起動してる」と勘違いして起動をやめる

　　　　２．起動しない＝プロセスがすぐ終了

　　　　３．PIDが1のプロセスが終わる＝Dockerコンテナも終了
```
 　という処理の流れになります。<br>
<br>
### ✅対処法
 　pidファイルが残っているのが原因なのでそれを削除すればOKです。<br>
```
　　　　rm tmp/pids/server.pid
```
