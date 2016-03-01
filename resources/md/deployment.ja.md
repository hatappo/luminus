## Running Standalone

アプリケーションをスタンドアローンで実行できるようにするには、単純にこう実行すればOKです。

```
lein uberjar
```

出来上がった`jar`は`target`フォルダにあります。次のように実行できます。

```
java -jar myapp.jar
```

デフォルトでは、スタンドアローンのアプリケーションは、起動に組み込みのImmutantサーバを使用します。しかしながら、例えば`+jetty`などのprofileを使ったとしたら、代わりに別のサーバが使用されます。独自のポート番号を指定するには`$PORT`環境変数をセットする必要があります。例をあげると:

```
export PORT=8080
java -jar myapp.jar
```

## Deploying on Immutant App Server

Immutantアプリケーションサーバのデプロイについては、[Immutantの公式のドキュメント](http://immutant.org/documentation/2.0.2/apidoc/guide-installation.html)で概略されているステップをなぞってみてください。[Linode](http://linode.com/)にも、[Ubuntu14.04上でImmutantとWildFlyを使ったLuminusアプリケーションのデプロイの仕方](https://linode.com/docs/applications/development/clojure-deployment-with-immutant-and-wildfly-on-ubuntu-14-04)についての手引きがあります。

## Deploying to Tomcat

Apache Tomcat のようなコンテナにアプリケーションをデプロイするためにはWAR形式のアーカイブを生成する必要があります。これは、`+war`profileを使用すると同梱される[lein-uberwar](https://github.com/luminus-framework/lein-uberwar)プラグインだけがサポートしています。

`lein-uberwar`プラグインを有効にするには、`project.clj`ファイルに次の設定を手動で追加してください。

```clojure
:plugins [...
          [lein-uberwar "0.1.0"]]

  :uberwar {:handler <app>.handler/app
            :init <app>.handler/init
            :destroy <app>.handler/destroy
            :name "<app>.war"}
```

WARを作成するためにアプリケーションをパッケージすることができます。こう実行します。

```
lein uberwar
```

次に、生成された`<app>.war`を単純にTomcatの`webapps`フォルダにコピーすればOKです。例えば:

```
cp target/<app>.war ~/tomcat/webapps/
```

すると、あなたのアプリケーションはTomcatを起動したときに`/<app>`コンテキストで利用可能になります。アプリケーションをルート・コンテキストにデプロイしたかったら、単に`webapp`配下にWARを`ROOT.war`という名前にしてコピーすればOKです。

## VPS Deployment

[DigitalOcean](https://www.digitalocean.com/)のような仮想プライベート・サーバ（VPS）は、Clojureアプリケーション向けの安価なホスティング・オプションを提供しています。あなたのDigitalOceanサーバをセットアップするには[この手引き](https://www.digitalocean.com/community/tutorials/how-to-create-your-first-digitalocean-droplet-virtual-server)にとおりやってください。いったんサーバが作成されると、[ここに書かれているように](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-12-04)Ubuntuをインストールすることができます。最終的に、[この手順](https://help.ubuntu.com/community/Java)に従ってあなたのUbuntuにjavaをインストールします。以降で示す手順は、Ubuntu15.04かそれより新しいバージョンに対応しています。

### Application deployment

このステップでは、サーバにあなたのアプリケーションをデプロイし、サーバを起動すると自動でアプリケーションが開始することを確認します。これを実現するのに`systemd`を使用します。あなたのアプリケーションを実行する`deploy`ユーザをを作成してください。

```
sudo adduser -m deploy
sudo passwd -l deploy
```

あなたのアプリケーション用に、サーバ上に`/var/myapp`のようなディレクトリを作成してください。そして、`scp`を使ってサーバにアプリケーションをアップロードしてください。

```
$ scp myapp.jar user@<domain>:/var/myapp/
```

ここまできたら、アプリケーションが起動可能か一度テストしてみるべきです。`ssh`を使ってサーバに接続し、アプリケーションを実行します。

```
java -jar /var/myapp/myapp.jar
```

もし全てうまくいっていると、アプリケーションはローカルで起動します。次のコマンドでアプリケーションが期待通り動いているか確認します。

```
curl http://127.0.0.1:3000/
```

あなたのアプリケーションは、今やサーバ上で`http://<domain>:3000`でアクセスすることも可能です。もし、アプリケーションがアクセスできなかったら、ファイヤーウォールがこのポートのアクセスを許可しているか確認してください。あなたが利用しているVPSのプロバイダによっては、3000番ポートに対するアクセス・ポイントを作成する必要があるかもしれません。

* [Azure でアクセス・ポイントを作成する](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-set-up-endpoints/)
* [Amazon EC2 でアクセス・ポイントを作成する](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html#adding-security-group-rule)

ここで、アプリケーションのインスタンスを一度停止して、アプリケーションのライフサイクルを管理する`systemd`の設定を作成しましょう。とりわけ、サーバ起動時にアプリケーションが開始することに注意しましょう。`/lib/systemd/system/myapp.service`というファイルを作成し、中身を次のとおりにします。

```
[Unit]
Description=My Application
After=network.target

[Service]
WorkingDirectory=/var/myapp
EnvironmentFile=-/var/myapp/env
Environment="DATABASE_URL=jdbc:postgresql://localhost/app?user=app_user&password=secret"
ExecStart=/usr/bin/java -jar /var/myapp/myapp.jar
User=deploy

[Install]
WantedBy=multi-user.target
```

デフォルトのJVMはかなりアグレッシブなメモリの使い方をすることに注意してください。もしJVMが使うメモリの量を減らしたかったら、	次の1行を`[Service]`設定に追加することができます。

```
[Service]
...
_JAVA_OPTIONS="-Xmx256m"
ExecStart=/usr/bin/java -jar /var/myapp/myapp.jar
```

この設定は、JVMに使用を許可するメモリの最大量を制限します。ここまでくると、次のコマンドで、システムに再起動がかかるたびに毎回アプリケーションを開始するように`systemd`に伝えることができます。

```
sudo systemctl daemon-reload
sudo systemctl enable myapp.service
```

システムが再起動するとき、あなたのアプリケーションは開始しリクエストを処理する準備が整います。テストしたくなるんじゃないでしょうか。単純にあなたのマシンを再起動し、プロセスを確認してください。

```
ps -ef | grep java
```

このコマンドを実行すると、何かこんな感じのを返します。`UID`に注目してください。これが`deploy`になっているべきです。`root`のままになっていたら重大なセキュリティ・リスクになってしまうでしょう。

```
deploy     730     1  1 06:45 ?        00:00:42 /usr/bin/java -jar /var/mysite/mysite.jar
```

### Fronting with Nginx

次のコマンドでNginxをインストールしてください。

```
$ sudo apt-get install nginx
```

次に、`/etc/nginx/sites-available/default`にデフォルトの設定のバックアップを行って、それをあなたのアプリケーション独自の設定ファイルに置き換えてください。例えばこんなふうに:

```
server{
  listen 80 default_server;
  listen [::]:80 default_server ipv6only=on;
  server_name localhost mydomain.com www.mydomain.com;

  access_log /var/log/myapp_access.log;
  error_log /var/log/myapp_error.log;

  location / {
    proxy_pass http://localhost:3000/;
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_redirect  off;
  }
}
```

こう実行してNginxを再起動してください。

```
sudo service nginx restart
```

それから、アプリケーションが`http://<domain>`で利用可能なことをテストしてください。

追加で、アプリケーションの静的リソースをNginxに配布させるように設定することもできます。これをやるには、全ての静的リソースが`static`などの共通の接頭辞を使って配信されることを保証しなければいけません。次に、プロジェクトのフォルダで次のようなコマンドを実行して、`resources/public/static`フォルダをアプリケーションからサーバ上の`/var/myapp/static`などの場所にアップロードしてください。

```
scp -r resources/public/static user@<domain>:/var/myapp/static
```

次の追加設定オプションを、Nginxの設定の`server`セクション以下に追記してください。

```
location /static/ {
    alias /var/myapp/static/;
  }
```

`http://<domain>/static`へのあらゆるリクエストについて、あなたのアプリケーションを迂回しその代わりに直接配信するようにNginxさせます。

圧縮を有効にするために、次の設定があなたの`/etc/nginx/nginx.conf`に存在することを確かめてください。

```
gzip on;
gzip_disable "msie6";

gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_http_version 1.1;
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;">
```

### Setting up SSL

もしあなたのサイトがなんらかのユーザ認証を行うなら、HTTPSを使いたいでしょう。まず必要なのは、SSL証明書とその鍵です。この2つをそれぞれ`cert.crt`と`cert.key`と呼びます。

#### Setting up SSL Certificate using Let's Encrypt

インストール・ツールをダウンロードし、次のコマンドで証明書を生成してください。

```
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
/letsencrypt-auto certonly --email <you@email.com> -d <yoursite.com> -d <www.yoursite.com> --webroot --webroot-path /var/www/html
```

追加で、証明書を自動で更新するためにCronのジョブをセットアップしてください。`root`でこう実行することでcrontabが更新できます。

```
su
crontab -e
```

次の1行を追加してください。

```
0 0 1,15 * * /path-to-letsencrypt/letsencrypt-auto certonly --keep-until-expiring --email <you@email.com> -d <yoursite.com> -d <www.yoursite.com> --webroot --webroot-path /var/www/html
```

OpenSSLのデフォルトを使用する代わりに、より強力なDHEパラメータを生成しましょう。これは、鍵交換に1024-bitの鍵を込めます。

```
cd /etc/ssl/certs
openssl dhparam -out dhparam.pem 4096
```

次に、`/etc/nginx/sites-available/default`内の設定を次のように更新したくなるでしょう。

```
server {
    listen 80;
    return 301 https://$host$request_uri;
}

server {

    listen 443;
    server_name localhost mydomain.com www.mydomain.com;

    ssl_certificate           /etc/letsencrypt/live/<yoursite.com>/fullchain.pem;
    ssl_certificate_key       /etc/letsencrypt/live/<yoursite.com>/privkey.pem;

    ssl on;
    ssl_prefer_server_ciphers  on;
    ssl_session_timeout        180m;
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'AES256+EECDH:AES256+EDH';
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
    add_header Strict-Transport-Security 'max-age=31536000';

    access_log /var/log/myapp_access.log;
    error_log /var/log/myapp_error.log;

    location / {

      proxy_set_header        Host $host;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;

      # Fix the “It appears that your reverse proxy set up is broken" error.
      proxy_pass          http://localhost:3000;
      proxy_read_timeout  90;

      proxy_redirect      http://localhost:3000 https://mydomain.com;
    }
  }
```

上記の設定はNginxに、HTTPのリクエストをHTTPSにリダイレクトし、配信でさっきの証明書を使用するにようにさせます。

最終的に、次のコマンドを実行することによって、特定のポートだけを許可するようにファイヤー・ウォールを設定してください。

```
$ sudo ufw allow ssh
$ sudo ufw allow http
$ sudo ufw allow https
$ sudo ufw enable
```

[SSL Server Test](https://www.ssllabs.com/ssltest/)を使用すれば、SSLの設定をテストすることができます。

## Heroku Deployment

まず、[Git](http://git-scm.com/downloads)と[Heroku toolbelt](https://toolbelt.heroku.com/)がインストールされているか確認し、以下のステップに従ってください。

やりたかったらで良いですが、アプリケーションがローカルで起動することをテストしてみてください。こうすればOKです。

```
heroku local
```

ここまできたら、あならのgitリポジトリを初期化し、アプリケーションのコードをコミットすることができます。

```
git init
git add .
git commit -m "init"
```

Herokuにあなたのappを作成

```
heroku create
```

任意で、アプリケーションで使うデータベースを作成

```
heroku addons:create heroku-postgresql
```

接続設定は、appのadd-ons配下の[Heroku dashboard](https://dashboard.heroku.com/apps/)にあります。

アプリケーションをデプロイ

```
git push heroku master
```

これで、あなたのアプリケーションはHerokuにデプロイできるはずです！データベースを更新または初期化するために

```
heroku run lein run migrate
```

さらなる手順は、[公式ドキュメント](https://devcenter.heroku.com/articles/clojure)を見てください。

## Enabling nREPL

LuminusはサーバのREPLに接続することを可能にする[nREPL](https://github.com/clojure/tools.nrepl)のセットアップを備えています。この機能は、稼働中のアプリケーションをhotfixするだけでなくデバッグ全般に便利です。nREPLのサポートを有効にするには、`NREPL_PORT`環境変数に利用したいポート番号を設定します。

```
export NREPL_PORT=7000
```

REPLの接続をテストするなら、単に次のコマンドを実行してください。

```
lein repl :connect 7001
```

ローカルのREPLに接続するのと全く同じようにして、好きなIDEからリモートのREPLに接続することができます。

リモートのサーバでnREPLを起動するとき、SSHを使ってローカル・マシンにREPLのポートをフォワードすることをおすすめします。

```
ssh -L 7001:localhost:7001 remotehost
```
