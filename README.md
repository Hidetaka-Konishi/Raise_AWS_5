# UnicornとNginxを利用するときによく使うコマンド
・unicornを起動(development)　```bundle exec unicorn -c /home/ec2-user/raisetech-live8-sample-app/config/unicorn.rb -E development -D```

・unicornを停止(マスタープロセスのPIDは以下の場合は15406)　```kill -9 [マスタープロセスのPID]```

![](./image/ps_aux_grep_unicorn.png)

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

・後半の20行だけログを表示　```sudo tail -n 20 unicorn.log```

・gemファイルの変更を反映　```bundle install```

・特定のファイルやディレクトリを検索する　`find / -name "ファイル名またはディレクトリ名"`

・ブラウザからEC2上のアプリにアクセスするにはEC2のパブリックIPv4 アドレスをブラウザに入力

・ブラウザからALBを経由してEC2上のアプリにアクセスするにはALBのDNS名をブラウザに入力

# EC2上のNginxとUnicornにアプリをデプロイしてALB経由でS3に画像をアップロード
※インストールするパッケージのバージョンはREADMEに書かれている

【手動構築の手順(IAMロール)】
1. VPCの作成手順は通常通り。
2. EC2のセキュリティグループはインバウンドルールにポート番号8080のTCPの0.0.0.0/0とポート番号22のTCPのマイIPからの通信のみを許可する。S3FullAccessのポリシーをアタッチしたIAMロールをEC2に割り当てる。
3. RDSのセキュリティグループはインバウンドルールにタイプ`MYSQL/Aurora`のTCPのポート番号3306のソースはEC2からの通信のみを許可する。
5. マネジメントコンソールのEC2の画面から左のサイドバーにある「ロードバランサー」→「ロードバランサーの作成」をクリックして ALBを選択する。
6. 「基本的な設定」の項目では、「ロードバランサー名」に名前をつける。
7. 「ネットワークマッピング」の項目では、「VPC」にEC2があるVPCを選択、AZを二つ選択し、サブネットは⚠️のマークが表示されないサブネットを選択する。
8. 「セキュリティグループ」の項目では、タイプHTTPのTCPのポート番号80の0.0.0.0/0を許可する。
9. 「リスナーとルーティング」の項目では、「デフォルトアクション」の「ターゲットグループの作成」をクリックする。
10. 「基本的な設定」の項目では、「ターゲットグループ名」に名前をつけて、EC2が設置されているVPCを選択。
11. 一番下までスクロールして「次へ」をクリックする。
12. 「使用可能なインスタンス」の項目で対象のEC2にチェックをつけ、「選択したインスタンスのポート」で8080を入力し、「保留中として以下を含める」をクリックする。
13.  一番下までスクロールして「ターゲットグループの作成」をクリックする。
14.  ALBを作成しているページに戻って「デフォルトアクション」にさっき作成したターゲットグループを選択する。
15.  一番下までスクロールして「ロードバランサーの作成」をクリックする。
17. マネジメントコンソールのS3の画面から「バケットを作成」をクリックする。
18. 「一般的な設定」の項目では、「バケット名」で名前をつけて、「AWS リージョン」でEC2が配置されているリージョンを選択する。EC2が配置されているリージョンはEC2のAZがus-east-1aとなっていた場合、us-east-1がリージョン名になる。
19. 一番下までスクロールして「バケットを作成」をクリックする。
20. 作成したS3バケットの詳細ページに行き、「アクセス許可」をクリックする。下にスクロールするとCORSを編集できる項目があるので以下のコードを追加する。
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

21. `sudo yum install git -y`
22. 2. `git clone [リポジトリのURL]`
23. `sudo yum update -y`
24. `sudo yum install -y curl gpg gcc gcc-c++ make`
25. `curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -`
26. `curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -`
27. `gpg2 --keyserver hkp://keyserver.ubuntu.com --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB`
28. `curl -sSL https://get.rvm.io | bash -s stable`
29. `echo 'source ~/.rvm/scripts/rvm' >> ~/.bashrc`
30. `source ~/.rvm/scripts/rvm && rvm list | grep 'ruby-[rubyのバージョン]'`
31. `source ~/.rvm/scripts/rvm && rvm install ruby-[rubyのバージョン]`
32. `source ~/.rvm/scripts/rvm && gem install bundler -v [bundlerのバージョン]`
33. `sudo yum install mysql-devel -y`
34. `cd [プロジェクトディレクトリ名]`
35. `bundle install`
36. `sudo yum install python3-pip -y`
37. `sudo pip3 install PyMySQL`
38. `sudo yum install mysql -y`
39. `mysql -h [RDSのエンドポイント] -u [RDSのマスターユーザー名(デフォルトはadmin)] -p[RDSのマスターパスワード] -e "CREATE DATABASE IF NOT EXISTS [任意のデータベーステーブル名];"`
40. `cd config`
41. `cp database.yml.sample database.yml`
42. `vi database.yml`
43. `username`の値をRDSのマスターユーザー名(デフォルトはadmin)に変更。
44. `default`の`password`にRDSのマスターパスワードを追加。
45. `development`と`test`の`database`の値を19で作成したデータベーステーブル名に変更。
46. `host`キーと値であるRDSのエンドポイントを`development`と`test`に追加。
47. `port`キーと値であるRDSのポート番号(デフォルトは3306)を`development`と`test`に追加。
48. `development`と`test`の`socket`をコメントアウトにする。データの保存先がRDSなので`socket`をコメントアウトにしている。
49. ファイルを保存する
50. `cd ..`
51. `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v[nvmのバージョン]/install.sh | bash`
52. `cd`
53. `echo 'export NVM_DIR="$HOME/.nvm"' >> /home/ec2-user/.bashrc`
54. `cd [プロジェクトディレクトリ名]`
55. `[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"`
56. `. ~/.nvm/nvm.sh`
57. `nvm install [Node.jsのバージョン]`
58. `cd`
59. `export PATH="/home/ec2-user/.nvm/versions/node/v[Node.jsのバージョン]/bin:$PATH"`
60. `npm install -g yarn`
61. `sudo yum install -y ImageMagick`
62. `cd [プロジェクトディレクトリ名]`
63. `sudo amazon-linux-extras install nginx1 -y`
64. `rvmsudo -u ec2-user bundle install`
65. `cd config`
66. `vi unicorn.rb`
67. worker_processesの次の行に追加　`working_directory "/home/ec2-user/[プロジェクトディレクトリ名]"`。この設定をすることでUnicornがアプリケーションを実行するディレクトリを認識できるようになる。
68. listenの最後に追加　`, :backlog => 64`。`:backlog => 64`とすることで64個の接続リクエストを待機させることができる。
69. listenの次の行に追加　`listen 8080, :tcp_nopush => true`。`listen 8080`はUnicornがポート番号8080でHTTP通信を許可することを設定していて、`:tcp_nopush => true`はTCPの性能を最適化するための設定。
70. pidの次の行に追加　`stdout_path "/home/ec2-user/[プロジェクトディレクトリ名]/unicorn.log"`。これによってUnicornのログが出力されるファイルを固定化することができる。
71. stdout_pathの次の行に追加　`stderr_path "/home/ec2-user/[プロジェクトディレクトリ名]/unicorn.log"`。これによってUnicornのエラーのログが出力されるファイルを固定化することができる。
72. ファイルを保存する。
73. `sudo vi /etc/nginx/nginx.conf`
74. `user nginx;`を`user ec2-user;`に変更。こうすることで`unicorn.sock`の実行権限が`ec2-user`になっていてもNginxはUniconとやり取りできるようになる。
75. 以下をhttpブロックに追加。このブロックを追加することでUnicornにリクエストを送信できるようになる。

```
upstream app {
        server unix:/home/ec2-user/[プロジェクトディレクトリ名]/unicorn.sock;
    }
```

![](./image/upstream_app.png)

76. serverブロックに以下を追加。

```
        location / {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            proxy_pass http://app;
        }

        location ^~ /assets/ {
            root /home/ec2-user/[プロジェクトディレクトリ名]/public;
            gzip_static on;
            expires max;
            add_header Cache-Control public;
        }
```

![](./image/server_listen_80.png)

77. ファイルを保存する。
78. `vi environments/development.rb`
79. `config.active_storage.service = :local`を`config.active_storage.service = :amazon`に書き換える。
80. `config.assets.debug = true`を以下に書き換える。

```
  config.assets.debug = false
  config.assets.compile = true
```

81. ファイルを保存する。
82. `cd ..`
83. `RAILS_ENV=development bundle exec rake assets:precompile`
84. `sudo su - ec2-user -c 'bin/rails db:migrate RAILS_ENV=development'`
85. `cd`
86. `sudo su - ec2-user -c 'cd /home/ec2-user/[プロジェクトディレクトリ名] && bin/rails db:migrate RAILS_ENV=development'`
87. `cd /home/ec2-user/[プロジェクトディレクトリ名]/config/storage.yml`
88. `region`を対象のS3バケットがあるリージョン、`bucket`を対象のS3バケット名に変更する。
89. NginxとUnicornを起動するとBlocked hostが表示される。
90. `vi config/environments/development.rb`
91. ファイルの末尾にBlocked hostで表示された`config.hosts << "ALBのDNS名"`を記載する。
92. NginxとUnicornを停止して起動する。

【手動構築の手順(アクセスキー)】
`storage.yml`の`access_key_id`と`secret_access_key`をS3へのアクセスを許可したIAMユーザーのアクセスキーとシークレットアクセスキーに変更する。今回はアクセスキーを使うので手順2で割り当てたIAMロールを取り外す。

# S3にオブジェクトが保存されているかEC2上から確認
1. AWS CLIの設定　```aws configure```
2. 以下の情報を入力します
```
AWS Access Key ID: IAMユーザーのアクセスキーID
AWS Secret Access Key: IAMユーザーのシークレットアクセスキー
Default region name: S3バケットのリージョン名（例：ap-northeast-1）
Default output format: json
```
3. S3バケット内のオブジェクトの確認　aws s3 ls s3://S3バケット名/

# S3のCORSについて
## CORSとは
S3バケットに対してWebサイトやWebアプリケーションからアクセスを許可するかのルールを設定する項目
## CORSのコード解説
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
### AllowedHeaders
リクエストが持っているどの情報に対して許可するかを設定する。
### *(アスタリスク)
すべてを許可することを意味する。
### AllowedMethods
リクエストのどの行動を許可するかを設定する。
### PUT
更新
### POST
作成
### HEAD
ヘッダーの情報
### AllowedOrigins
どのWebサイトやWebアプリケーションからのアクセスを許可するかを設定する。
### ExposeHeaders
ブラウザから見ることができるヘッダーの情報を指定する。
### []
[]の中に何もない場合は何も設定や指定をしていないことを意味する。
### MaxAgeSeconds
ブラウザがS3から提供してもらった情報を覚えていられる時間を設定する。設定した時間が過ぎたらブラウザは再度S3に対してデータを共有しても良いのかをS3に尋ねる必要がある。
### 3000
3000秒のこと。
# RDSのセキュリティグループを変更
AWSのマネジメントコンソールからセキュリティグループを変更したいRDSの詳細情報が書かれているページに行き、右上の「変更」をクリックしてそこに書かれているセキュリティグループを変更する。
