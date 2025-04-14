#Dockerのトラブルシューティング#
★コンテナが停止できなくなる
　●対処法
　　パソコンを再起動すると停止するので、問題のコンテナを削除して
　　`docker compose build`
　　で新しいコンテナを生成する

  🔹 特定のコンテナを削除する
　　　まず、削除したいコンテナのIDまたは名前を確認します
　　　bash
　　　　docker ps -a
　　　
　　　その後、以下のコマンドでコンテナを削除します
　　　bash
　　　　docker rm [コンテナIDまたは名前]
　　　
　　　実行中のコンテナを削除する場合は、-f オプションを使用して強制的に削除できます
　　　bash
　　　　docker rm -f [コンテナIDまたは名前]

　🔹 すべての停止中のコンテナを一括削除する
　　　停止中のすべてのコンテナを一度に削除するには、以下のコマンドを使用します：​
　　　bash
　　　　docker rm $(docker ps -a -q)
　　　このコマンドは、すべてのコンテナIDを取得し、それらを削除します。​

　🔹 使用されていないすべてのコンテナ、イメージ、ボリューム、ネットワークを一括削除する
　　　未使用のリソースを一括で削除するには、以下のコマンドを使用します：​
　　　bash
　　　　docker system prune -a
　　　このコマンドは、未使用のコンテナ、イメージ、ボリューム、ネットワークをすべて削除します。​

　⚠️ 注意点
　　　docker rm は停止中のコンテナを削除します。​
　　　実行中のコンテナを削除するには、-f オプションを使用して強制的に削除します。​


★`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`が出て
  docker関連のコマンドが実行できない時
　●対処法
　　Dockerのデスクトップアプリを再インストールする
　　※RailsのDBがアンインストールで消えてしまうので
　　　`docker compose exec web bin/rails db:create`　#DB構築
　  　`docker compose exec web rails db:migrate`　#構築したDBにマイグレーションを実行
　　　コマンドを実行してDBを再構築する
