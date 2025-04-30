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
