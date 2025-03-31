#Dockerのトラブルシューティング#
◎コンテナが停止できなくなる
　●対処法
　　パソコンを再起動すると停止するので、問題のコンテナを削除して
　　`docker compose build`
　　で新しいコンテナを生成する

　　★`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`が出てdocker関連のコマンドが実行できない時
　　　●対処法
　　　　Dockerのデスクトップアプリを再インストールする
　　　　※RailsのDBがアンインストールで消えてしまうので
　　　　　`docker compose exec web bin/rails db:create`　#DB構築
　  　　　`docker compose exec web rails db:migrate`　#構築したDBにマイグレーションを実行
　　　　　コマンドを実行してDBを再構築する
