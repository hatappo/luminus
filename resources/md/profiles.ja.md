## Profiles

`lein new luminus myapp`を実行すると、デフォルトのprofileのテンプレートを使ってアプリケーションが作成されます。しかしながら、あなたが使用するテンプレートにより多くの機能を盛り込みたかったら、テンプレートに加えたい機能のprofileヒントを追記することができます。

### web servers

Luminusは、デフォルトで[Immutant](http://immutant.org/)をWebサーバとして使用しますが、別のサーバもサポートしています。次のとおりです。

* +aleph    - [Aleph](https://github.com/ztellman/aleph)サーバのサポートをプロジェクトに追加
* +jetty    - [Jetty](https://github.com/mpenet/jet)のサポートをプロジェクトに追加
* +http-kit - [HTTP Kit](http://www.http-kit.org/)のWebサーバをプロジェクトに追加

### databases

* +h2       - `db.core`名前空間とH2データベースの依存関係を追加
* +sqlite   - `db.core`名前空間とSQLiteデータベースの依存関係を追加
* +postgres - `db.core`名前空間とPostreSQLの依存関係を追加
* +mysql    - `db.core`名前空間とMySQLの依存関係を追加
* +mongodb  - `db.core`名前空間とMongoDBの依存関係を追加

### miscellaneous

* +auth     - [Buddy](https://github.com/funcool/buddy)の依存関係と認証のミドルウェアを追加
* +cljs     - ClojureScriptのサポートをその使用例とともにプロジェクトに追加
* +cucumber - cucumberとclj-webdriverのprofile
* +swagger  - [compojure-api](https://github.com/metosin/compojure-api)ライブラリを用いた[Swagger-UI](https://github.com/swagger-api/swagger-ui)のサポートを追加
* +sassc    - [SassC](https://github.com/sass/sassc)コマンドライン・コンパイラを用いた[SASS/SCSS](http://sass-lang.com/)ファイルのサポートを追加
* +war      - Apache Tomcat などのサーバにデプロイするためのWARアーカイブのビルドのサポートを追加
* +site     - 指定されたデータベース（デフォルトではH2）とClojureScriptを使ったサイトのテンプレートを作成

profileを足すには、作成するアプリケーション名の後ろに単に引数としてprofileを渡すだけです。例えば、

```
lein new luminus myapp +cljs
```

アプリケーションを作成する際に複数のprofileを組み合わせることも可能です。例えば、

```
lein new luminus myapp +cljs +swagger +postgres
```
