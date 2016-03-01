## Configuring the Database
Luminusはデフォルトでは、データベースのマイグレーションには[Migratus](https://github.com/yogthos/migratus)を、データベースの操作には[HugSQL](http://www.hugsql.org/)を、それぞれ使います。例えば、`+postgres`のようなデータベースprofileを使えば、マイグレーションとデフォルトのDBコネクションは自動でセットアップされるでしょう。

### Configuring Migrations
まず初めに、自分のデータベースの接続設定を`profiles.clj`に書かなければいけません。デフォルトでは、**development**と**testing**という2つのprofileがそれぞれに生成されます。

```clojure
{:profiles/dev  {:env {:database-url "jdbc:postgresql://localhost/my_app_dev?user=db_user&password=db_password"}}
 :profiles/test {:env {:database-url "jdbc:postgresql://localhost/myapp_test?user=db_user&password=db_password"}}}
```

そうしたら、データベース・スキーマをマイグレーションやロールバックするためのSQLスクリプトを書きます。このSQLスクリプトは、idを数値として捉えてその順に適用されます。そして、`resources/migrations`フォルダ配下に置かれることが期待されます。Luminusテンプレートは、userというテーブルに対するサンプルのマイグレーションファイルを生成してくれます。

```
resources/migrations/20150720004935-add-users-table.down.sql
resources/migrations/20150720004935-add-users-table.up.sql
```

以上の準備を経ることで、以下のようにマイグレーションが実行可能になります。

```
lein run migrate
```

適用されたマイグレーションは次のようにしてロールバックできます。

```
lein run migrate
```

`migratus`プラグインを使って、次のようにさらに追加のマイグレーションファイルを生成することができます。

```
lein migratus create add-guestbook-table
```

詳細は[データベース・マイグレーション](http://www.luminusweb.net/docs/migrations.md)を参照してください。

### Setting up the database connection

DBのコネクションの設定は、アプリケーションの`<app>.db.core`ネームスペースの中にあります。デフォルトでは、DBのコネクションは`DATABASE_URL`環境変数として渡されることが期待されます。

コネクション・プールを行うために、[conman](https://github.com/luminus-framework/conman)ライブラリが使われてます。

DBのコネクションは、`conman/connect!`関数がDB設定のmapとコネクションを格納するatomを渡して呼び出されることによって初期化されます。`connect!`関数は、[HikariCP](https://github.com/brettwooldridge/HikariCP)ライブラリを使ってプールされたJDBCコネクションを作成します。`disconnect!`関数が呼び出されると、作成されたコネクションは閉じられます。

```clojure
(ns myapp.db.core
  (:require
    ...
    [config.core :refer [env]]
    [conman.core :as conman]
    [mount.core :refer [defstate]]))
            
(def pool-spec
  {:adapter    :postgresql
   :init-size  1
   :min-idle   1
   :max-idle   4
   :max-active 32}) 

(defn connect! []
  (conman/connect!
    (assoc
      pool-spec
      :jdbc-url (env :database-url))))

(defn disconnect! [conn]
  (conman/disconnect! conn))

(defstate ^:dynamic *db*
          :start (connect!)
          :stop (disconnect! *db*))
```

コネクションは`defstate`マクロを使って規定されます。`*db*`componentが`:start`の状態に変わるときに`connect!`関数が呼び出され、`*db*`componentが`:stop`の状態に変わるときに`disconnect!`関数が呼び出されます。

このコネクションは、`conman.core/with-transaction`のスコープの中でクエリ関数が呼び出されるときにトランザクションが使用可能なコネクションに自動的に切り替えたいので動的(dynamic)である必要があります。
`*db*`componentのライフサイクルは、[mount](https://github.com/tolitius/mount)ライブラリによって管理されます。これについては[componentのライフサイクル管理](http://www.luminusweb.net/docs/components.md)の節で議論されています。

`<app>.handler/init`と`<app>.handler/destroy`関数は、`(mount/start)`や`(mount/stop)`を呼び出すことによって`defstate`を用いて定義されたあらゆるcomponentの初期化と停止をそれぞれ行います。この仕組みによって、DBコネクションはサーバが起動するときに利用可能になりサーバが停止するときに綺麗に始末されます。

複数のデータベースを相手にする場合、データベースのコネクションを各々追跡する必要があるので、それぞれに別々のatomが必要となります。

### Working with HugSQL

HugSQLはクエリに定義するのに素のSQLを使用します。各クエリに対して関数名と doc string を割り当てるためにSQLのコメントを利用します。

規約として、クエリは全て`resources/sql/queries.sql`ファイルに書きます。しかしながら、アプリケーションが育って大きくなってくれば当然クエリを複数のファイルに分割したくなるでしょう。

クエリファイルのフォーマットは、以下を見るとよく分かるでしょう。

```sql
-- :name create-user! :! :n
-- :doc creates a new user record
INSERT INTO users
(id, first_name, last_name, email, pass)
VALUES (:id, :first_name, :last_name, :email, :pass)
```

生成された関数の名前は`:name`コメントによって規定されます。この関数名の後には、「コマンド」と「（問い合わせ）結果」という2つのフラグが続きます。

次のコマンドフラグが利用可能です。

* `:?` - result-setをともなうクエリ（デフォルト値）
* `:!` - いかなるステートメントにも対応
* `:<!` - `INSERT ... RETURNING`をサポート
* `:i!` - insertとjdbcの`.getGeneratedKeys`をサポート

（問い合わせ）結果フラグです

* `:1` - 単一行のデータをhash-mapとして返す
* `:*` - 複数行のデータをhash-mapのvectorとして返す
* `:n` - 影響を与えた（つまり 挿入or更新or削除 された）行数を返す
* `:raw` - 結果を加工せずにそのまま返します（デフォルト値）

クエリ自体は、コロンを接頭辞として付加したパラメータ名によって印された動的なパラメータと素のSQLを用いて書きます。

クエリ関数は、`conman/bind-connection`マクロを呼び出すことによって生成します。この`conman/bind-connection`マクロは、上述されたようなクエリのファイルを1つ以上と、コネクションのvarを受け取ります。

```clojure
(conman/bind-connection conn "sql/queries.sql")
```

`bind-connection`が実行されると、さっき定義したクエリは`myapp.db.core/create-user!`関数にマッピングされます。`bind-connection`によって生成された関数は、デフォルトでは`conn`atom内のコネクションを使用するようになります。ただし、関数の実行時に明示的に別のコネクションを渡せばこの限りではありません。

```clojure
(create-user!
  {:id "user1"
   :first_name "Bob"
   :last_name "Bobberton"
   :email "bob.bobberton@mail.com"
   :pass "verysecret"})
```

生成されたこの関数はパラメータなしでも実行できます。

```clojure
(get-users)
```

この関数に明示的にコネクションを渡すこともできます。トランザクションの中での実行のようなケースに対応するためです。

```clojure
; デフォルトとは異なるコネクションを定義
(def some-other-conn 
  "jdbc:postgresql://localhost/myapp_test?user=test&password=test")
  
(create-user!
  {:id "user1"
   :first_name "Bob"
   :last_name "Bobberton"
   :email "bob.bobberton@mail.com"
   :pass "verysecret"}
   some-other-conn) ; <- 明示的にデフォルトとはコネクションを渡している
```

`conman`ライブラリは、トランザクションを使ってステートメントを実行するための`with-transaction`マクロも提供しています。この`with-transaction`マクロはそのボディの内部においては、当該コネクションを「トランザクション付きのコネクション」に再束縛します。`bind-connection`の実行によって生成されたSQLクエリ関数は全て、デフォルトでは`with-transaction`マクロ内では「トランザクション付きのコネクション」を使用します。

```clojure
(with-transaction [t-conn conn]
  (jdbc/db-set-rollback-only! t-conn)
  (create-user!
    {:id         "foo"
     :first_name "Sam"
     :last_name  "Smith"
     :email      "sam.smith@example.com"})
  (get-user {:id "foo"}))
```

詳細は[公式ドキュメント](http://www.hugsql.org/)を参照してください。

### SQL Korma
> KormaはClojureのドメイン固有言語です。あなたの好きなRDBMSでの作業で発生する苦痛を取り除いてくれます。柔軟性に重点を置いて設計され、速度に重点を置いて構築されました。Kormaは、後味の悪いデータに対してシンプルで直感的なインターフェイスを提供します。

> Korma is a domain specific language for Clojure that takes the pain out of working with your favorite RDBMS. Built for speed and designed for flexibility, Korma provides a simple and intuitive interface to your data that won't leave a bad taste in your mouth.

http://sqlkorma.com/

既存のプロジェクトにKormaのサポートを追加するのはけっこう簡単です。まず必要なのは`project.clj`にKormaの依存を書き足すことです。

```clojure
[korma "0.4.0"]
```

次に、Kormaを使い出すために`korma.db`への参照を加えなければいけません。

```clojure
(ns myapp.db.core
  (:require
        [korma.core :refer :all]
        [korma.db :refer [create-db default-connection]]))
```

Kormaは、`create-db`を使ってコネクションを生成することを必要とします。そしてデフォルトのコネクションは`default-connection`関数によってセットすることができます。

```clojure
(connect! [] (create-db db-spec))

(defstate ^:dynamic *db*
          :start (connect!))

(default-connection *db*)
```

この関数は、[c3p0](http://sourceforge.net/projects/c3p0/)ライブラリを使って、あなたのDB設定に対するコネクション・プールを作成します。注意点として、全てのクエリのデフォルトとして最後に作成されたコネクション・プールがセットされます。

Kormaは、SQLのテーブルを表現するのにエンティティを用います。エンティティは、クエリの構築における中核要素にあたります。エンティティは、`defentity`マクロを使うことによって作成されます。

```clojure
(defentity users)
```

usersエンティティを使って、さっき登場したクエリをuserを作成するためのクエリに書き直すことができます。以下のようになります。

```clojure
(defn create-user [user]
  (insert users
          (values user)))
```

userを取得するクエリはこんなふうに書き直せるでしょう。

```clojure
(defn get-user [id]
  (first (select users
                 (where {:id id})
                 (limit 1))))
```

Kormaとその機能に関するより詳しいドキュメントなら、[公式ドキュメントのページ](http://sqlkorma.com/docs)を参照してください。

----
# 訳注

* profileという単語が2通り使われている。
    0. Luminusのテンプレート作成の際に指定する追加のコンポーネントの指定。  
    例えば`lein new luminus guestbook +h2`ってやるときの`+h2`がprofile。
    0. 「development」とか「testing」とか「production」とかの環境で分ける設定や情報のこと。  
    この意味では`profiles.clj`とかがそう。

* 文中で**component**って出てくる場合は、単に「構成要素」を意味してるんじゃなくて、[component](https://github.com/stuartsierra/component)ライブラリのことを指しているのがほとんどじゃないかと思う。componentライブラリについてそもそもよく知らないので、なるべく「component」ってそのまま残している。
