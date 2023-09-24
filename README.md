# UnicornとNginxを利用するときによく使うコマンド
・unicornを起動(development)　```bundle exec unicorn -c /home/ec2-user/raisetech-live8-sample-app/config/unicorn.rb -E development -D```

・unicornを停止(マスタープロセスのPIDは以下の場合は15406)　```kill -QUIT [マスタープロセスのPID]```

![スクリーンショット 2023-09-24 134519](https://github.com/Hidetaka-Konishi/Raise_AWS_5/assets/142459457/6f523b6d-c8a1-4667-873d-89dc9be7da0d)

・unicornが起動しているかの確認(上記のようなmasterやworkerが表示されない場合はunicornサーバーが停止している)　```ps aux | grep unicorn```

・単一のファイルを削除　```rm unicorn.log```

・unicornのログファイルの中身を表示　```cat unicorn.log```

・Nginxの起動　```sudo systemctl start nginx```

・Nginxが起動しているかの確認　```sudo systemctl status nginx```

・Nginxの停止　```sudo systemctl stop nginx```

・Nginxのエラーログやアクセスログのファイルがあるか確認　```sudo ls /var/log/nginx/```

・error.logまたはaccess.logの中身を見る　```sudo cat /var/log/nginx/error.log```

・error.logまたはaccess.logの削除　```sudo rm /var/log/nginx/error.log```

・error.logまたはaccess.logの作成　```sudo touch /var/log/nginx/error.log```

・ブラウザからEC2上のアプリにアクセスするにはEC2のパブリックIPv4 アドレスをブラウザに入力

・ブラウザからALBを経由してEC2上のアプリにアクセスするにはALBのDNS名をブラウザに入力

# EC2に組み込みサーバーとしてアプリをデプロイする手順(データベースはRDSを使用)
※インストールするパッケージのバージョンはREADME.mdに書かれている
1. gitパッケージをインストール　```sudo yum install git -y```
2. リポジトリをクローン　```git clone [リポジトリのURL]```
3. システム上のすべてのパッケージを最新のバージョンに更新　```sudo yum update -y```
4. ソフトウェアパッケージをLinuxシステムにインストール　```sudo yum install curl gpg gcc gcc-c++ make -y```
5. RVMの開発者の公開鍵を信頼できる鍵としてローカルのGPGキーリングに追加　```curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -```
6. RVMのGPGキーをインポート　```curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -```
7. RVMの安定版をインストール　```\curl -sSL https://get.rvm.io | bash -s stable```
8. RVMのスクリプトをシェルにロード　```source ~/.rvm/scripts/rvm```
9. システムが適切に準備されているかを確認　```rvm requirements```
10. Rubyをインストール　```rvm install ruby-3.1.2```
11. Bundlerをインストール　```gem install bundler:2.3.14```
12. mysql-develパッケージをインストール　```sudo yum install mysql-devel```
13. アプリのプロジェクトディレクトリに移動
14. gemをインストール　```bundle install```
15. サンプルファイルをコピーして、自分の環境に合わせて設定値を変更　```cp config/database.yml.sample config/database.yml```
16. MySQLに接続　```mysql -h [RDSのエンドポイント] -P 3306 -u admin -p```
17. 新しいデータベースインスタンスを作成　```CREATE DATABASE [新しく作成するデータベース名];```
18. アプリのプロジェクトディレクトリ→configに移動
19. database.ymlを開く　```nano database.yml```
20. developmentとtestのdatabaseの値をさっき作成したデータベース名に変更
21. usernameの値をRDSのユーザー名(デフォルトはadmin)に変更
22. defaultのpasswordにRDSのパスワードを追加
23. hostキーと値であるRDSのエンドポイントをdevelopmentとtestに追加
24. portキーと値であるRDSのポート番号(デフォルトは3306)をdevelopmentとtestに追加
25. developmentとtestのsocketをコメントアウトにすることで適用されないようにする
26. Ctrl + Oで保存、Enterキーでファイル名を決定、Ctrl + Xでnanoエディタを閉じる
27. アプリのプロジェクトディレクトリに移動
28. nvmをインストール　```- curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash```
    v0.39.5は最新を確認→https://github.com/nvm-sh/nvm
29. 
# RDSのセキュリティグループを変更
AWSのマネジメントコンソールからセキュリティグループを変更したいRDSの詳細情報が書かれているページに行き、右上の「変更」をクリックしてそこに書かれているセキュリティグループを変更する。
