## Guestbook Application

このチュートリアルは、Luminusを使った簡単なゲストブックのアプリケーションの構築の手引きです。
このゲストブックでは、ユーザがメッセージを残すこと、そして他の人が残したメッセージを一覧することができます。
このゲストブック・アプリケーションでは、HTMLテンプレート、データベース接続、そしてプロジェクトのファイル構成を実際に確認していきます。

もし、Clojure用の好みのエディタがまだ決まっていないのなら、このチュートリアルを進めていくのには[Light Table](http://www.lighttable.com/)をおすすめします。

### Installing JDK

ClojureはJVM上で実行されるため、JDKがインストールされている必要があります。
もしあなたの環境にJDKがまだ入っていないのなら、OpenJDKが[ここ](http://www.azul.com/downloads/zulu/)からダウンロードできるので、これをおすすめします。
Luminusがデフォルトの設定で動作するのには JDK 8 が必要です。

### Installing Leiningen
Luminusで作業するには[Leiningen](http://leiningen.org/)が必要です。
Leiningenのインストールは、次のステップで片付きます。

1. スクリプトをダウンロードする。
3. スクリプトに実行権限を付与します（例えば`chmod +x lein`）。
2. スクリプトを$PATHに配置します（例えば`~/bin`）。
4. `lein`を実行して自己インストールが完了するまで待ちます。

```
wget https://raw.github.com/technomancy/leiningen/stable/bin/lein
chmod +x lein
mv lein ~/bin
lein
```

### Creating a new application

Leiningenをインストールしたら、ターミナルで次のコマンドを実行することができます。新しいアプリケーションが初期化されます。

```
lein new luminus guestbook +h2
cd guestbook
```

上記により、[組み込みデータベースのH2](http://www.h2database.com/html/main.html)のエンジンをサポートした新しいテンプレート・プロジェクトが作成されます。

### Anatomy of a Luminus application

新しく作成されたアプリケーションは次の構造をしています。

```
guestbook
│
├── Procfile
├── README.md
├── env
│   ├── dev
│   │   ├── clj
│   │   │   ├── guestbook
│   │   │   │   ├── dev_middleware.clj
│   │   │   │   └── env.clj
│   │   │   └── user.clj
│   │   └── resources
│   │       ├── config.edn
│   │       └── log4j.properties
│   ├── prod
│   │   ├── clj
│   │   │   └── guestbook
│   │   │       └── env.clj
│   │   └── resources
│   │       ├── config.edn
│   │       └── log4j.properties
│   └── test
│       └── resources
│           └── config.edn
├── profiles.clj
├── project.clj
├── resources
│   ├── docs
│   │   └── docs.md
│   ├── migrations
│   │   ├── 20160221190951-add-users-table.down.sql
│   │   └── 20160221190951-add-users-table.up.sql
│   ├── public
│   │   ├── css
│   │   │   └── screen.css
│   │   └── favicon.ico
│   ├── sql
│   │   └── queries.sql
│   └── templates
│       ├── about.html
│       ├── base.html
│       ├── error.html
│       └── home.html
├── src
│   └── clj
│       └── guestbook
│           ├── config.clj
│           ├── core.clj
│           ├── db
│           │   └── core.clj
│           ├── handler.clj
│           ├── layout.clj
│           ├── middleware.clj
│           └── routes
│               └── home.clj
└── test
    └── clj
        └── guestbook
            └── test
                ├── db
                │   └── core.clj
                └── handler.clj
```

アプリケーションのルートのフォルダにあるファイル群が何なのか見ていきましょう。

* `Procfile` - Herokuへのデプロイに使います
* `README.md` - 慣例的に置くことになってる、アプリケーションについてドキュメントする場所。
* `project.clj` - Leiningenによって、プロジェクトの設定と依存関係を管理するのに使われます。
* `profiles.clj` - リポジトリにチェックインすべきではないローカル設定に使われます。
* `.gitignore` - Gitから除外するための、ビルドで生成されるファイルなどのアセットの一覧です。

### The Source Directory

全てのソースコードは`src/clj`フォルダ配下にいます。
このアプリケーションはguestbookという名前なので、guestbookという名前がプロジェクトの名前空間のルートにもなっています。
生成された全ての名前空間についても見ていきましょう。

#### guestbook

* `core.clj` - これは、サーバを起動と停止するためのロジックを持っている、アプリケーション自体のエントリ・ポイントです。
* `handler.clj` - アプリケーションの基本となるルーティングを定義しており、ここがアプリケーションを利用する側のエントリ・ポイントです。
* `layout.clj` - ページの中身をレンダリングするのに使われるレイアウト・ヘルパーのための名前空間です。
* `middleware.clj` - このアプリケーション独自のミドルウェアを保持する名前空間です。

#### guestbook.db

`db`名前空間は、アプリケーションのmodelを定義することと永続化層を扱うことに使われます。

* `core.clj` - データベースとやり取りする関数を格納するのに使われます。

#### guestbook.routes

`routes`名前空間は、トップページとaboutページのルーティングとコントローラの場所です。
認証やあるいは特定のなんらかの処理のルーティングをさらに追加する場合、それらのための名前空間をここに作ります。

* `home.clj` - アプリケーションのトップページとaboutページを定義する名前空間。

### The Env Directory

環境に固有のコードやリソースは、`env/dev`、`env/test`、そして`env/prod`というパスのは以下に配置されます。
`dev`設定は、開発時に、`test`はテスト時に使用され一方で、`prod`設定は、アプリケーションが本番環境用にパッケージされたときに使用されます。

#### `dev/clj`

* `user.clj` - REPLでの開発中に実行したいユーティリティのコードを置くための名前空間です。
* `guestbook/env.clj` - 開発環境での設定のデフォルトを保持します。
* `guestbook/dev_middleware.clj` - 開発時にのみ使用するミドルウェアで、本番環境においてはコンパイルすべきではないものを保持します。

#### `dev/resources`

* `config.edn` - 開発環境用のデフォルトの環境変数です。
* `log4j.properties` 開発環境用のロギング設定に使用するファイルです。

#### `test/resources` folder contains the `log4j.properties` file used to configure the development logging profile.

* `config.edn` - テスト環境用のデフォルトの環境変数です。

#### `prod/clj`

* `guestbook/env.clj` 本番環境用の設定の名前空間です。

#### `prod/resources`

* `config.edn` - 本番のアプリケーションに同梱されるデフォルトの環境変数です。
* `log4j.properties` - 本番環境用のデフォルトのロギング設定です。

### The Test Directory

ここはアプリケーションのテストを置く場所です。
テストのサンプルが2つ既に定義されています。

### The Resources Directory

ここはアプリケーションの全ての静的リソースを置く場所です。
`resources`の下の`public`ディレクトリ内のコンテンツは、サーバからクライアントに配布されます。
いくつかのCSSリソースが既に作成されていることに気付くでしょう。

#### HTML templates

`templates`ディレクトリは、アプリケーションのHTMLページを表現する[Selmer](https://github.com/yogthos/Selmer)テンプレートに予約されています。

* `about.html` - aboutページ
* `base.html` - サイトの基本レイアウト
* `home.html` - homeページ（訳注：トップページ）


#### SQL Queries

SQLは、`resources/sql`フォルダ内にあります。

* `queries.sql` - SQLとSQL関連の関数名を定義します。

#### The Migrations Directory

Luminusは、マイグレーションに[Migratus](https://github.com/yogthos/migratus)を使用します。
マイグレーションは、upとdownのSQLファイルを使って管理されます。（訳注：Migratusで使うSQLファイルには`.up.sql`か`.down.sql`という接尾辞が付きます。）
規約上、このマイグレーション用のSQLファイルは、日付を利用したバージョン情報を持ちファイル生成日時の順に適用されます。

* `20150718103127-add-users-table.up.sql` - usersテーブルを作成するためのマイグレーションファイル
* `20150718103127-add-users-table.down.sql` - usersテーブルを破棄するためのマイグレーションファイル


### The Project File

既に説明したように、全ての依存関係は、`project.clj`ファイルの更新によって管理されます。
私達さくせしたこのアプリケーションの`project.clj`ファイルは、ルート・フォルダにあり、次のような感じでしょう。

```clojure
(defproject guestbook "0.1.0-SNAPSHOT"

  :description "FIXME: write description"
  :url "http://example.com/FIXME"

  :dependencies [[org.clojure/clojure "1.8.0"]
                 [selmer "1.0.0"]
                 [markdown-clj "0.9.86"]
                 [ring-middleware-format "0.7.0"]
                 [metosin/ring-http-response "0.6.5"]
                 [bouncer "1.0.0"]
                 [org.webjars/bootstrap "4.0.0-alpha.2"]
                 [org.webjars/font-awesome "4.5.0"]
                 [org.webjars.bower/tether "1.1.1"]
                 [org.webjars/jquery "2.2.0"]
                 [org.clojure/tools.logging "0.3.1"]
                 [com.taoensso/tower "3.0.2"]
                 [compojure "1.4.0"]
                 [ring-webjars "0.1.1"]
                 [ring/ring-defaults "0.1.5"]
                 [ring "1.4.0" :exclusions [ring/ring-jetty-adapter]]
                 [mount "0.1.10-SNAPSHOT"]
                 [cprop "0.1.5"]
                 [org.clojure/tools.cli "0.3.3"]
                 [luminus-nrepl "0.1.3"]
                 [luminus-migrations "0.1.0"]
                 [conman "0.4.5"]
                 [com.h2database/h2 "1.4.191"]
                 [org.webjars/webjars-locator-jboss-vfs "0.1.0"]
                 [luminus-immutant "0.1.3"]
                 [luminus-log4j "0.1.2"]]

  :min-lein-version "2.0.0"

  :jvm-opts ["-server" "-Dconf=.lein-env"]
  :source-paths ["src/clj"]
  :resource-paths ["resources"]

  :main guestbook.core
  :migratus {:store :database}

  :plugins [[lein-cprop "1.0.1"]
            [migratus-lein "0.2.6"]]
  :profiles
  {:uberjar {:omit-source true

             :aot :all
             :uberjar-name "guestbook.jar"
             :source-paths ["env/prod/clj"]
             :resource-paths ["env/prod/resources"]}
   :dev           [:project/dev :profiles/dev]
   :test          [:project/test :profiles/test]
   :project/dev  {:dependencies [[prone "1.0.2"]
                                 [ring/ring-mock "0.3.0"]
                                 [ring/ring-devel "1.4.0"]
                                 [pjstadig/humane-test-output "0.7.1"]
                                 [mvxcvi/puget "1.0.0"]]


                  :source-paths ["env/dev/clj" "test/clj"]
                  :resource-paths ["env/dev/resources"]
                  :repl-options {:init-ns user}
                  :injections [(require 'pjstadig.humane-test-output)
                               (pjstadig.humane-test-output/activate!)]}
   :project/test {:resource-paths ["env/dev/resources" "env/test/resources"]}
   :profiles/dev {}
   :profiles/test {}})
```

見れば分かりますが、`project.clj`ファイルは単なるClojureのlistです。アプリケーションの様々な側面を表現したKey/Valueのペア持っています。

最もよく行う作業は、新しいライブラリのプロジェクトへの追加です。
追加されるライブラリは、`:dependencies`vectorを用いて指定されます。
この私達のプロジェクトで新しいライブラリを使うには、単純にその依存をここに追記しなければいけません。

`:plugins`vector内の項目は、環境変数を読み取る`lein-cprop`プラグインなどのような、追加の機能を提供するのに使用されます。

`:profiles`は、プロジェクトの様々な設定のmapを持ちます。これは、本番環境のビルドか開発環境のどちらかを初期化するために使われます。
The `:profiles` contain a map of different project configurations that are used to initialize it for either development or production builds.

このプロジェクトでは、`:dev`と`:test`を合成したprofileがセットアップされることに注意してください。
これらのprofileは、`profiles.clj`内にある`:profiles/dev`と`:profiles/test`と同様に、`:project/dev`と`:project/test`のprofileの変数を保持します。
`profiles.clj`は、共有されるリポジトリにチェックインしない、ローカルだけの環境変数を設定しましょう。

`project.clj`というビルド・ファイルの構造についての詳細は、[Leiningenの公式ドキュメント](http://leiningen.org/#docs)を参照してください。

### Creating the Database

まずは、このアプリケーションのmodel（訳注：MVCでいうところのModel）を作りましょう。`migrations`フォルダ配下に置かれた`<date>-add-users-table.up.sql`ファイルを開きます。
このファイルの中身は次のとおりです。

```sql
CREATE TABLE users
(id VARCHAR(20) PRIMARY KEY,
 first_name VARCHAR(30),
 last_name VARCHAR(30),
 email VARCHAR(30),
 admin BOOLEAN,
 last_login TIME,
 is_active BOOLEAN,
 pass VARCHAR(100));
```

`users`テーブルを、私達のアプリケーションによりふさわしいテーブルに置き換えましょう。

```sql
CREATE TABLE guestbook
(id INTEGER PRIMARY KEY AUTO_INCREMENT,
 name VARCHAR(30),
 message VARCHAR(200),
 timestamp TIMESTAMP);
```

guestbookテーブルは、コメントした人の名前や、メッセージの内容、そしてタイムスタンプなどの、メッセージを表現するための全てのフィールドを格納します。です。
このファイルの変更を保存して、プロジェクトのルートで次のコマンドを実行しましょう。

```
lein run migrate
```

ここまで全てうまくいっていれば、私達のデータベースが初期化されます。

### Accessing The Database

次に、`src/guestbook/db/core.clj`を見ていきましょう。
ここでは、私達のデータベースへの接続の定義が既にあるのが分かる思います。

```clojure
(ns guestbook.db.core
  (:require
    [conman.core :as conman]
    [mount.core :refer [defstate]]
    [guestbook.config :refer [env]]))

(defstate ^:dynamic *db*
          :start (conman/connect!
                   {:datasource
                    (doto (org.h2.jdbcx.JdbcDataSource.)
                          (.setURL (-> env :database :url))
                          (.setUser "")
                          (.setPassword ""))})
          :stop (conman/disconnect! *db*))

(conman/bind-connection *db* "sql/queries.sql")
```

データベースの接続設定は、実行時に環境変数のmapから読み取られます。
デフォルトでは、`:database`キーは、データベース固有の設定の変数を指しています。
このmapには、データベースへの接続URLを保持した`:url`キーがあります。
この`:url`キーの値は、開発時には`profiles.clj`ファイルから伝播して取得され、本番環境時には環境変数としてセットされなければなりません。本番環境では例えば、

```
export DATABASE_URL="jdbc:h2:./guestbook.db"
```

このプロジェクトでは組み込みのH2データベースを使用しているので、データはURLで指定されたファイルに格納されます。このURLのパスは、プロジェクトが実行された所からの相対パスになります。

データベースへのクエリに対応する関数は、`bind-connection`が呼び出されたときに生成されます。
既に見てきたように、クエリ関数は`sql/queries.sql`ファイルを参照します。
この`sql/queries.sql`ファイルは、`resources`フォルダ配下にあります。
ファイルを開いて中を見てみましょう。

```sql
-- :name create-user! :! :n
-- :doc 新しいユーザのレコードを作成します
INSERT INTO users
(id, first_name, last_name, email, pass)
VALUES (:id, :first_name, :last_name, :email, :pass)

-- :name update-user! :! :n
-- :doc 既に存在するユーザのレコードを更新します
UPDATE users
SET first_name = :first_name, last_name = :last_name, email = :email
WHERE id = :id

-- :name get-user :? :1
-- :doc 与えられたIDのユーザえお取得します
SELECT * FROM users
WHERE id = :id
```

 上記で見たように、それぞれのクエリ関数は、`-- :name`から始まり関数名が続く特殊なコメントの記法を使って定義されます。
As we can see each function is defined using the comment that starts with `-- :name` followed by the name of the function.
その次の行のコメントは、その関数の doc string になります。
そして、その次の行からが素のSQLで書かれたボディです。

パラメータは、`:`記号を使って表現されます。
それでは、今あるクエリを私達のguestbookに必要なものに書き換えましょう。


```sql
-- :name save-message! :! :n
-- :doc creates a new message
INSERT INTO guestbook
(name, message, timestamp)
VALUES (:name, :message, :timestamp)

-- :name get-messages :? :*
-- :doc selects all available messages
SELECT * from guestbook
```

これで、modelがセットアップされました。アプリケーションを動かしていきましょう。

### Running the Application

次のように、アプリケーションをdevelopmentモードで起動することができます。

```
>lein run
[2016-02-28 15:05:34,970][DEBUG][org.jboss.logging] Logging Provider: org.jboss.logging.Log4jLoggerProvider
[2016-02-28 15:05:36,067][INFO][com.zaxxer.hikari.HikariDataSource] HikariPool-0 - is starting.
[2016-02-28 15:05:36,252][INFO][luminus.http-server] starting HTTP server on port 3000
[2016-02-28 15:05:36,294][INFO][org.xnio] XNIO version 3.4.0.Beta1
[2016-02-28 15:05:36,344][INFO][org.xnio.nio] XNIO NIO Implementation Version 3.4.0.Beta1
[2016-02-28 15:05:36,406][INFO][org.projectodd.wunderboss.web.Web] Registered web context /
[2016-02-28 15:05:36,407][INFO][luminus.repl-server] starting nREPL server on port 7000
[2016-02-28 15:05:36,422][INFO][guestbook.core] #'guestbook.config/env started
[2016-02-28 15:05:36,422][INFO][guestbook.core] #'guestbook.db.core/*db* started
[2016-02-28 15:05:36,422][INFO][guestbook.core] #'guestbook.core/http-server started
[2016-02-28 15:05:36,423][INFO][guestbook.core] #'guestbook.core/repl-server started
[2016-02-28 15:05:36,423][INFO][guestbook.env]
-=[guestbook started successfully using the development profile]=-
```

サーバが起動すると、[http://localhost:3000](http://localhost:3000)を開くことができるので、アプリが実行されているのを確認しましょう。
`PORT`環境変数を設定するか以下のように引数としてポート番号を渡すことで、違うポートでサーバを起動することができます。

```
lein run -p 8000
```

開いたページは、データベースを初期化するためにマイグレーションを実行するよう訴えていることに注意してください。
しかしながら、それはさっき済ましてしまったので改めて実効する必要はありません。

### Creating Pages and Handling Form Input

ルーティングは、`guestbook.routes.home`名前空間の中に定義されています。
ファイルを開いて、データベースのデータをレンダリングするロジックを追記しましょう。
まずは、[Bouncer](https://github.com/leonardoborges/bouncer)のバリデータと[ring.util.response](http://ring-clojure.github.io/ring/ring.util.response.html)とあわせて`db`名前空間を追加します。

```clojure
(ns guestbook.routes.home
  (:require
    ...
    [guestbook.db.core :as db]
    [bouncer.core :as b]
    [bouncer.validators :as v]
    [ring.util.response :refer [redirect]]))
```

次に、フォーム・パラメータをバリデートするための関数を作成します。

```clojure
(defn validate-message [params]
  (first
    (b/validate
      params
      :name v/required
      :message [v/required [v/min-count 10]])))
```

この関数は、`:name`と`:message`のキーが指定したルールに従うか確かめるために、Bouncerの`validate`関数を使用します。
特に、nameは必須項目であり、massageは最低でも10文字以上でなければいけません。
`:message`のバリデーションでやっているように、Bouncerは複数のルールをバリデータに渡すためにverctorの構文を使用します。。
バリデータがさらに引数を取る場合には、`min-count`でやっているようにここでもまたvectorの構文が使用できます。
バリデーションされる値は、暗黙的に第一引数としてバリデータに渡されます。

それでは、メッセージをバリデートし、保存するための関数を追加しましょう。

```clojure
(defn save-message! [{:keys [params]}]
  (if-let [errors (validate-message params)]
    (-> (response/found "/")
        (assoc :flash (assoc params :errors errors)))
    (do
      (db/save-message!
       (assoc params :timestamp (java.util.Date.)))
      (response/found "/"))))
```

この関数は、リクエストからフォーム・パラメータを保持している`:params`キーを取り出します。
`validate-message`関数がエラーを返すとき、`/`にリダイレクトさせ、渡ってきたパラメータにそのエラーを付け足してレスポンスの`:flash`キーにセットします。
エラーが返らない場合、データベースにメッセージを保存しリダイレクトします。

最後に、`home-page`ハンドラ関数を次に示す内容に変えましょう。

```clojure
(defn home-page [{:keys [flash]}]
  (layout/render
   "home.html"
   (merge {:messages (db/get-messages)}
          (select-keys flash [:name :message :errors]))))
```

この関数は、homeページのテンプレートをレンダリングします。そのとき、バリデーションのエラーなどの`:flash`セッションの全てのパラメータとともに、現在保存されているメッセージをテンプレートに渡します。

私達のルーティングでは、`home-page`と`save-message!`の両方のハンドラにリクエストが渡されなければいけません。

```clojure
(defroutes home-routes
  (GET "/" request (home-page request))
  (POST "/" request (save-message! request))
  (GET "/about" [] (about-page)))
```

`compojure.core`から`POST`を参照するのを忘れないでくださいね。

```
(ns guestbook.routes.home
  (:require ...
            [compojure.core :refer [defroutes GET POST]]
            ...))
```

こうして、私達のコントローラが完成したので、`resources/templates`配下に置かれた`home.html`テンプレートを開きましょう。
現状では、`home.html`テンプレートは単順にcontentブロック内の`content`変数の内容をレンダリングします。

```xml
{% extends "base.html" %}
{% block content %}
  <div class="jumbotron">
    <h1>Welcome to guestbook</h1>
    <p>Time to start building your site!</p>
    <p><a class="btn btn-primary btn-lg" href="http://luminusweb.net">Learn more &raquo;</a></p>
  </div>

  <div class="row">
    <div class="span12">
    {{docs|markdown}}
    </div>
  </div>
{% endblock %}
```

登録された全てのメッセージをイテレートして（順に処理して）リスト形式で表示するように、`content`ブロックを更新しましょう。

```xml
{% extends "base.html" %}
{% block content %}
  <div class="jumbotron">
    <h1>Welcome to guestbook</h1>
    <p>Time to start building your site!</p>
    <p><a class="btn btn-primary btn-lg" href="http://luminusweb.net">Learn more &raquo;</a></p>
  </div>

  <div class="row">
    <div class="span12">
        <ul class="messages">
            {% for item in messages %}
            <li>
                <time>{{item.timestamp|date:"yyyy-MM-dd HH:mm"}}</time>
                <p>{{item.message}}</p>
                <p> - {{item.name}}</p>
            </li>
            {% endfor %}
        </ul>
    </div>
</div>
{% endblock %}
```

上記を見て分かるとおり、全メッセージを順になめるために`for`イテレータを使用しています。
各メッセージは、message、name、そしてtimestampのキーを持ったmapなので、その名前でアクセスできます。

timestampを人間に読みやすい形式にフォーマットする`date`フィルタを使っていることにも注目してください。

最後に、ユーザがメッセージを送信することができるようにフォームを作成しましょう。

nameやmessageなどの値が与えられたら、それらの値を伝播させ、そしてそれらに関連するエラーは全てレンダリングします。
このフォームでは、CSRF（cross-site request forgery）防御に必要なとなる`csrf-field`タグが使われていることに注意してください。

```xml
<div class="row">
    <div class="span12">
        <form method="POST" action="/">
                {% csrf-field %}
                <p>
                    Name:
                    <input class="form-control"
                           type="text"
                           name="name"
                           value="{{name}}" />
                </p>
                {% if errors.name %}
                <div class="alert alert-danger">{{errors.name|join}}</div>
                {% endif %}
                <p>
                    Message:
                <textarea class="form-control"
                          rows="4"
                          cols="50"
                          name="message">{{message}}</textarea>
                </p>
                {% if errors.message %}
                <div class="alert alert-danger">{{errors.message|join}}</div>
                {% endif %}
                <input type="submit" class="btn btn-primary" value="comment" />
        </form>
    </div>
</div>
```

最終的な`home.html`テンプレートの次のような感じになります。

```xml
{% extends "base.html" %}
{% block content %}
<div class="row">
    <div class="span12">
        <ul class="messages">
            {% for item in messages %}
            <li>
                <time>{{item.timestamp|date:"yyyy-MM-dd HH:mm"}}</time>
                <p>{{item.message}}</p>
                <p> - {{item.name}}</p>
            </li>
            {% endfor %}
        </ul>
    </div>
</div>
<div class="row">
    <div class="span12">
        <form method="POST" action="/">
                {% csrf-field %}
                <p>
                    Name:
                    <input class="form-control"
                           type="text"
                           name="name"
                           value="{{name}}" />
                </p>
                {% if errors.name %}
                <div class="alert alert-danger">{{errors.name|join}}</div>
                {% endif %}
                <p>
                    Message:
                <textarea class="form-control"
                          rows="4"
                          cols="50"
                          name="message">{{message}}</textarea>
                </p>
                {% if errors.message %}
                <div class="alert alert-danger">{{errors.message|join}}</div>
                {% endif %}
                <input type="submit" class="btn btn-primary" value="comment" />
        </form>
    </div>
</div>
{% endblock %}
```

最後に、私達のフォームをより格好良くフォーマットするために`resources/public/css`フォルダ内に置かれた`screen.css`ファイルを更新しましょう。

```css
html,
body {
	font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  height: 100%;
  line-height: 1.4em;
	background: #eaeaea;
	width: 520px;
	margin: 0 auto;
}
.navbar {
  margin-bottom: 10px;
}
.navbar-brand {
  float: none;
}
.navbar-nav .nav-item {
  float: none;
}
.navbar-divider,
.navbar-nav .nav-item+.nav-item,
.navbar-nav .nav-link + .nav-link {
  margin-left: 0;
}
@media (min-width: 34em) {
  .navbar-brand {
    float: left;
  }
  .navbar-nav .nav-item {
    float: left;
  }
  .navbar-divider,
  .navbar-nav .nav-item+.nav-item,
  .navbar-nav .nav-link + .nav-link {
    margin-left: 1rem;
  }
}

.messages {
  background: white;
  width: 520px;
}
ul {
	list-style: none;
}

ul.messages li {
	position: relative;
	font-size: 16px;
	padding: 5px;
	border-bottom: 1px dotted #ccc;
}

li:last-child {
	border-bottom: none;
}

li time {
	font-size: 12px;
	padding-bottom: 20px;
}

form, .error {
	padding: 30px;
	margin-bottom: 50px;
	position: relative;
  background: white;
}
```

ブラウザでページをリロードすると、guestbookページに出迎えられるでしょう。
実際にフォームにコメントを書き込んで、全てが期待通りに動作するかテストすることができます。

## Adding some tests

今や動くアプリケーションが出来上がったので、テストを追加することができます。
`test/clj/guestbook/test/db/core.clj`名前空間を開いて、それを次のように書き換えましょう。

```clojure
(ns guestbook.test.db.core
  (:require [guestbook.db.core :refer [*db*] :as db]
            [luminus-migrations.core :as migrations]
            [clojure.test :refer :all]
            [clojure.java.jdbc :as jdbc]
            [guestbook.config :refer [env]]
            [conman.core :refer [with-transaction]]
            [mount.core :as mount]))

(use-fixtures
  :once
  (fn [f]
    (mount/start
      #'guestbook.config/env
      #'guestbook.db.core/*db*)
    (migrations/migrate ["migrate"] (-> env :database :url))
    (f)))

(deftest test-users
  (jdbc/with-db-transaction [t-conn *db*]
      (jdbc/db-set-rollback-only! t-conn)
      (let [message {:name "test"
                     :message "test"
                     :timestamp (java.util.Date.)}]
        (is (= 1 (db/save-message! t-conn message)))
        (is (= [(assoc message :id 1)] (db/get-messages t-conn {})))))
  (is (empty? (db/get-messages))))
```

これで、データベースとやり取りしている動作が期待通りであるか確認するためにターミナルで`lein test`を実行することができるようになりました。

## Packaging the application

次のコマンドを実行して、このアプリケーションをスタンドアローンでのデプロイ用にパッケージすることができます。

```
lein uberjar
```

このコマンドは、以下のように実行することが可能な「実行可能jarファイル」を作成します。

```
export DATABASE_URL="jdbc:h2:./guestbook_dev.db"
java -jar target/guestbook.jar
```

パッケージングしていないアプリケーションとは違って、jarで実行するときは`DATABASE_URL`環境変数が与えなければならないことに注意してください。

***

このチュートリアルの完全なソースコードは[ここ](https://github.com/yogthos/guestbook)から入手できます。
より完成された例としては、[Github](https://github.com/yogthos/luminus)でこのサイトのソースコードを見ることができます。
