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

# EC2上のNginxとUnicornにアプリをデプロイする手順(データベースはRDSを使用)
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
15. サンプルファイルをコピー　```cp config/database.yml.sample config/database.yml```
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
    
29. NVM_DIRを設定　```export NVM_DIR="$HOME/.nvm"```
30. nvmの初期化スクリプトが存在し、かつサイズが0でない場合に、そのスクリプトを実行　```[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"```
31. Node.jsをインストール　```nvm install 17.9.1```
32. yarnをインストール　```npm install -g yarn```
33. プロジェクトのセットアップと初期化　```bin/setup```
34. ImageMagickソフトウェアパッケージをインストール　```sudo yum install -y ImageMagick```
35. アプリケーションサーバーを起動　```bin/dev```
36. Nginxをインストール　```sudo amazon-linux-extras install nginx1 -y```
37. 依存関係を解決　```bundle install```
38. configディレクトリに移動
39. unicorn.rbファイルを開く　```nano unicorn.rb```
40. worker_processesの次の行に追加　```working_directory "/home/ec2-user/アプリのプロジェクト名"```
41. listenの最後に追加　```, :backlog => 64```
42. listenの次の行に追加　```listen 8080, :tcp_nopush => true```
43. pidの次の行に追加　```stdout_path "/home/ec2-user/アプリのプロジェクト名/unicorn.log"```
44. stdout_pathの次の行に追加　```stderr_path "/home/ec2-user/アプリのプロジェクト名/unicorn.log"```
45. Ctrl + Oで保存、Enterキーでファイル名を決定、Ctrl + Xでnanoエディタを閉じる
46. nginx.confファイルを開く　```sudo nano /etc/nginx/nginx.conf```
47. httpブロックに追加
```
upstream app {
        server unix:/home/ec2-user/アプリのプロジェクト名/unicorn.sock;
    }
```
![スクリーンショット 2023-09-24 190618](https://github.com/Hidetaka-Konishi/Raise_AWS_5/assets/142459457/7a28e7d5-5eeb-430a-95a9-1b2b2dbd2247)

48. serverブロックに追加
```
location / {
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Host $http_host;
proxy_redirect off;
proxy_pass http://app;
}
```

![スクリーンショット 2023-09-24 190921](https://github.com/Hidetaka-Konishi/Raise_AWS_5/assets/142459457/31ce7241-091d-413a-8f53-08952e7aaf1a)

49. Nginxの設定をテスト　```sudo nginx -t```
50. EC2のセキュリティグループでHTTPを許可するようにする
51. unicorn.sockのパーミッションの所有者をnginxユーザーに変更　```sudo chown nginx:nginx /home/ec2-user/アプリのプロジェクト名/unicorn.sock```
52. パーミッションを変更　```chmod o+x /home/ec2-user```
53. Unicornと Nginxを起動する。
# EC2上のNginxとUnicornにアプリをデプロイしたものにALBを追加する手順
1. 上記の「EC2上のNginxとUnicornにアプリをデプロイする手順(データベースはRDSを使用)」を行う。
2. マネジメントコンソールのEC2の画面から左のサイドバーにある「ロードバランサー」→「ロードバランサーの作成」→をクリックして ALBを選択する。
3. 「基本的な設定」の項目では、「ロードバランサー名」に名前をつける。
4. 「ネットワークマッピング」の項目では、「VPC」にEC2があるVPCを選択、AZを二つ選択し、サブネットは⚠️のマークが表示されないサブネットを選択する。
5. 「セキュリティグループ」の項目では、EC2と同じセキュリティグループを選択
6. 「リスナーとルーティング」の項目では、「デフォルトアクション」の「ターゲットグループの作成」をクリックする。
7. 「基本的な設定」の項目では、「ターゲットグループ名」に名前をつけて、EC2が設置されているVPCを選択。
8. 一番下までスクロールして「次へ」をクリックする。
9. 「使用可能なインスタンス」の項目でEC2を選択し、「保留中として以下を含める」をクリックする。
10.  一番下までスクロールして「ターゲットグループの作成」をクリックする。
11.  ALBを作成しているページに戻って「デフォルトアクション」にさっき作成したターゲットグループを選択する。
12.  一番下までスクロールして「ロードバランサーの作成」をクリックする。
13.  UnicornとNginxを停止して、起動させる。
14.  ALBのDNS名をブラウザに入力してアクセスする。
15.  Blocked hostというエラーが発生するので、ブラウザのエラー文に表示されている```config.hosts << "ec2-34-239-115-7.compute-1.amazonaws.com"```をコピーしてconfig→environments→development.rbファイルの末尾に書き込む
16.  ファイルやディレクトリの所有者と権限を確認するために、unicorn.sockがあるディレクトリで```ls -l```を実行する。
17.  srwxrwxr-x 1 nginx nginx 0 Sep 19 12:41 unicorn.sockが表示されてUnicornをec2-userとして実行している場合、所有者と権限の変更をする必要があるのでunicorn.sockがあるディレクトリで```sudo chown ec2-user:ec2-user unicorn.sock```を実行する。
18.  ファイルの権限も適切に設定するために、unicorn.sockがあるディレクトリで```sudo chmod 600 unicorn.sock```を実行する。
19. UnicornとNginxを停止して、起動させる。

# ALBを追加したEC2にS3を追加して、RDSではなくS3をデータ保存として変更する手順
1. マネジメントコンソールのS3の画面から「バケットを作成」をクリックする。
2. 「一般的な設定」の項目では、「バケット名」で名前をつけて、「AWS リージョン」でEC2が配置されているリージョンを選択する。EC2が配置されているリージョンはEC2のAZがus-east-1bとなっていた場合、us-east-1がリージョン名になる。
3. 一番下までスクロールして「バケットを作成」をクリックする。
4. マネジメントコンソールのIAMの画面から「ユーザーの作成」をクリックする。
5. 「ユーザー名」に名前をつけて「次へ」をクリックする。
6. 「許可のオプション」の項目では、「ポリシーを直接アタッチする」を選択して、「許可ポリシー」の検索欄に許可したいポリシーで検索してそれを選択する。
7. 一番下までスクロールして「次へ」をクリックして、また一番下までスクロールして「ユーザーの作成」をクリックする。
8. 作成したIAMユーザーの詳細ページに行き、「セキュリティ認証情報」→「アクセスキーを作成」をクリックする。
9. 「AWS コンピューティングサービスで実行されるアプリケーション」を選択して、一番下のチェックボックスにチェックを入れて、「次へ」をクリックする。
10. 「アクセスキーを作成」→「.csvファイルをダウンロード」をクリックする。ダウンロードされたファイルにはアクセスキーとシークレットアクセスキーの情報が書いてあるので大切に保管する必要がある。最後に「完了」をクリックする。
11. ターミナルでconfigディレクトリに移動する
12. storage.ymlを開く　```nano storage.yml```
13. S3の設定を変更する
```
amazon:
  service: S3
  access_key_id: <%= Rails.application.credentials.dig(:aws, :access_key_id) %>
  secret_access_key: <%= Rails.application.credentials.dig(:aws, :secret_access_key) %>
  region: us-east-1
  bucket: your_bucket_name_here
```
<%= Rails.application.credentials.dig(:aws, :access_key_id) %>をIAMユーザーのAccess key IDに変更

<%= Rails.application.credentials.dig(:aws, :secret_access_key) %>をIAMユーザーのSecret access keyに変更

us-east-1をS3バケットが配置されているリージョンに変更

your_bucket_name_hereをS3バケット名に変更

14. 依存関係のインストール　```bundle install --with development```
15. マイグレーション　```RAILS_ENV=development bundle exec rails db:migrate```
16. 作成したS3バケットの詳細ページに行き、「アクセス許可」をクリックする。下にスクロールするとCORSを編集できる項目があるので以下のコードを追加する。
```
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET",
            "PUT",
            "POST",
            "DELETE",
            "HEAD"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": [],
        "MaxAgeSeconds": 3000
    }
]
```
17. UnicornとNginxを停止して、起動させる。
18. ALBのDNS名をブラウザに入力してアクセスする。
19. development.logの末尾に```[ActionDispatch::HostAuthorization::DefaultResponseApp] Blocked host: etl.colorblockplaaygame.com```といったエラーがあり、このエラーはホストをブロックしているというエラーなのでconfig/environments/development.rbに```config.hosts << "etl.colorblockplaaygame.com"```といったものを追加してあげる。
20. UnicornとNginxを停止して、起動させる。

# S3にオブジェクトが保存されているかEC2上から確認
1. AWS CLIの設定　```aws configure```
2. 以下の情報を入力します

AWS Access Key ID: IAMユーザーのアクセスキーID

AWS Secret Access Key: IAMユーザーのシークレットアクセスキー

Default region name: S3バケットのリージョン名（例：ap-northeast-1）

Default output format: json

3. S3バケット内のオブジェクトの確認　aws s3 ls s3://S3バケット名/
# RDSのセキュリティグループを変更
AWSのマネジメントコンソールからセキュリティグループを変更したいRDSの詳細情報が書かれているページに行き、右上の「変更」をクリックしてそこに書かれているセキュリティグループを変更する。
