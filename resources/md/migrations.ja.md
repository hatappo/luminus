## Migrations
デフォルトでは、Luminusはデータベースのマイグレーションとスキーマの管理に[Migratus](https://github.com/yogthos/migratus)ライブラリを使います。`+mysql`や`+postgres`といったprofileを選ぶと、`profiles.clj`ファイルにマイグレーションの設定が追加されます。

### Migrations with Migratus 

開発環境用のデータベースの設定は`profiles.clj`ファイルに書きます。この`profiles.clj`ファイルは、あなたのローカル環境の設定を規定するので、ソースコード・リポジトリにはチェック・インすべきではありません。

```clojure
{:profiles/dev  {:env {:database-url "jdbc:postgresql://localhost/myapp_dev?user=appuser&password=secret"}}
 :profiles/test {:env {:database-url "jdbc:postgresql://localhost/myapp_test?user=test&password=test"}}}
```

本番環境では、それらの設定は環境変数に存在することが期待されます。例えば、データベースのURLを指す`DATABASE_URL`というshell変数を用意する、といったかたちです。設定のオプションの全リストについては[Environの公式ドキュメント](https://github.com/weavejester/environ)を見てください。

Migratusは、アプリケーションの`<app>.core`名前空間内の`-main`関数から起動されます。

デフォルトでは、プロジェクトのルートの`resources/migrations`ディレクトリにマイグレーションのSQLファイルが置いてあることが期待されます。ディレクトリの変更は、`<app>.db.migrations`名前空間の中で設定することができます。設定した独自ディレクトリを作成すれば、そこにマイグレーションのSQLスクリプトを追加していくことができるようになります。

それでは、2つのスクリプトを作ってみましょう。1つはマイグレーション用で、もう1つは、ロールバック用です。次のようMigratusのプラグインを使ってこの2つのスクリプト・ファイルを生成することができます。

```
lein migratus create add-users-table
```

このようにすると、Migratusの規定のディレクトリに適切なファイルが生成されるでしょう。`up`マイグレーション・ファイルの中身を、テーブルを作成するためのSQLスクリプトに書き換えます。

`resources/migrations/201506120401-add-users-table.up.sql`

```SQL
CREATE TABLE users (id INT, name VARCHAR(25));
```

それから、上記のファイルに対応する`down`マイグレーション・ファイルを、作成したテーブルを廃棄するSQLスクリプトに書き換えます。

`resources/migrations/201506120401-add-users-table.down.sql`

```SQL
DROP TABLE users;
```

これで、次のようにマイグレーションを起動することができるようになりました。

```
lein run migrate
```

ロールバックはこんな感じでいけます。

```
lein run rollback
```

特定のマイグレーションを適用するには、適用したいマイグレーションのidとともに上記コマンドを実行すればOKです。

```
lein run migrate 201506104553 201506120401
lein run rollback 201506104553 201506120401
```

マイグレーション一式は、アプリケーションがコンパイルされる時に一緒にパッケージされます。これにより、アプリケーションがサーバにデプロイされたときにマイグレーションを適用するといったことが可能になります。

```
lein uberjar
java -jar target/my-app.jar migrate
```

## Popular Migrations Alternatives

現在利用可能で有名なマイグレーション・ライブラリを以下に全て列挙します。

* [Drift](https://github.com/macourtney/drift) - DriftはClojureで書かれたマイグレーション・ライブラリです。Driftは、全てのマイグレーション・ファイルを収めたディレクトリを使った、Railsとよく似たマイグレーション処理をします。Driftはどのマイグレーション・ファイルを適用しなければいけないかを検出し、適切に実行します。
* [Lobos](https://github.com/budu/lobos) - Lobosは、データベース・スキーマのSQLの操作とマイグレーションのライブラリで、Clojureで書かれています。現在は、H2、MySQL、PostgreSQL、SQLite、そして SQL Server をサポートしています。
* [Ragtime](https://github.com/weavejester/ragtime) - データベース・マイグレーションを実装した、汎用的なマイグレーション・フレームワークです。
* [Joplin](https://github.com/juxt/joplin) - Joplinは、データストアのマイグレーションとシード・データの作成に柔軟に対応したライブラリです。
