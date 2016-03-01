Luminusは、Webアプリケーションを作るためのしっかりとしたデフォルト構成を提供することを目的としています。そのため、デフォルトで数多くのライブラリとともにパッケージされます。これらのライブラリには、セキュリティの[Buddy](https://github.com/funcool/buddy)、バリデーションの[Bouncer](https://github.com/leonardoborges/bouncer)、HTMLテンプレートの[Selmer](https://github.com/yogthos/Selmer)、国際化対応とその他細々とした機能のための[Tower](https://github.com/ptaoussanis/tower)が含まれます。もちろん、ClojureのWeb開発のためのライブラリはそれ以外にもたくさんあります。ClojureとClojureScriptの便利なライブラリについて、Luminusに既に含まれているもの以外をさらにここに列挙しようと思います。

## Assets

* [Stefon](https://github.com/circleci/stefon) - アセット・パイプラインのringミドルウェア
* [lein-asset-minifier](https://github.com/yogthos/lein-asset-minifier) - CSSとJSのアセットをminifyするためのLeiningenのプラグイン

## Authentication

* [Friend](https://github.com/cemerick/friend) - 拡張可能な認証と認可のライブラリ
* [clj-ldap](https://github.com/pauldorman/clj-ldap) -LDAPサーバとしゃべるためのライブラリ
* [ring-basic-authentication](https://github.com/remvee/ring-basic-authentication) - BASIC認証を敷くためのringミドルウェア

## Caching

* [Spyglass](https://github.com/clojurewerkz/spyglass) - Memcached（とCouchbaseとKestrel）のクライアント
* [core.cache](https://github.com/clojure/core.cache) - 様々なキャッシュ・ストラテジを実装したキャッシングのライブラリ

## Configuration

* [lein-init-script](https://github.com/strongh/lein-init-script) - *nixのinitスクリプトを生成するためのプラグイン

## ClojureScript

* [Om](https://github.com/swannodette/om) - FacebookのReactに対するClojureScriptのインターフェイス
* [Kioo](https://github.com/ckirkendall/kioo) - Reagent/OmのためのDOM操作とテンプレートのライブラリ
* [Hickory](https://github.com/davidsantiago/hickory) - HTMLをClojureのデータ構造にパースする
* [Sente](https://github.com/ptaoussanis/sente) - WebSocketとAjax（自動縮退）の両方での双方向・同期/非同期コミュニケーション
* [Datascript](https://github.com/tonsky/datascript) - 全てのアプリケ―ションの状態を管理するための中央集権的で統一的なアプローチ
* [Garden](https://github.com/noprompt/garden) - ClojureとClojureScriptにおいてCSSをレンダリングするためのライブラリ
* [Dommy](https://github.com/Prismatic/dommy) - 余分なもののない実用的テンプレートと（素早い）DOM操作ライブラリ
* [json-html](https://github.com/yogthos/json-html) - JSON/EDNエンコードされたデータを人間に分かりやすい表現に生成する
* [lein-externs](https://github.com/ejlo/lein-externs) - JSライブラリのexternファイルを自動生成するためのプラグイン

## Database clients

* [CongoMongo](https://github.com/aboekhoff/congomongo) - mongo-dbのJAVAのAPIのラッパー
* [Monger](http://clojuremongodb.info/) - MongoDBのクライアント
* [Clutch](https://github.com/clojure-clutch/clutch) - Apache CouchDB のクライアント
* [Neocons](https://github.com/michaelklishin/neocons) - よく使う表現の機能が豊富なNeo4Jの REST API ためのクライアント
* [Welle](http://clojureriak.info/) - Riakのための表現豊かなクライアント
* [Cassaforte](https://github.com/clojurewerkz/cassaforte) - Apache Cassandra 1.2+ のための若いクライアント
* [Rotary](https://github.com/weavejester/rotary) - DynamoDB API
* [Rummage](https://github.com/cemerick/rummage) - Amazon's SimpleDB (SDB) のクライアント・ライブラリ
* [Carmine](https://github.com/ptaoussanis/carmine) - ClojureのRedisクライアントとメッセージ・キュー

## Database Migrations

* [Drift](https://github.com/macourtney/drift) - マイグレーションのライブラリ
* [Lobos](http://budu.github.com/lobos/) - Lobosはデータベース・スキーマを作成・変更するのを手助けするライブラリです
* [Ragtime](https://github.com/weavejester/ragtime) - データベースに依存しないマイグレーション・ライブラリ

## SQL Libraries

* [Honey SQL](https://github.com/jkk/honeysql) - SQLを組み立てるためDSLで、Kormaの代替
* [clojure.java.jdbc](https://github.com/clojure/java.jdbc) - Java JDBC の低レベル層のラッパー
* [blackwater](https://github.com/bitemyapp/blackwater) - Kormaとclojure.java.jdbcにおいて、SQLとその実行時間をロギングするためのライブラリ

## Dependency Injection

* [mount](https://github.com/tolitius/mount)
* [yoyo](https://github.com/jarohen/yoyo)

## Email

* [Mailer](https://github.com/clojurewerkz/mailer) - ActionMailerに影響を受けたmailerライブラリ
* [Postal](https://github.com/drewr/postal) - ClojureのEメールのサポート

## Graphics

* [Analemma](http://liebke.github.com/analemma/) - Clojureベースの SVG DSL とチャート描画ライブラリ
* [Monet](https://github.com/rm-hull/monet) - canvasを簡単に（効率良く）使えるようにするためのClojureScriptの小さなライブラリ

## Template Languages

* [Cuma](https://github.com/liquidz/cuma) - Clojureの拡張可能なマイクロ・テンプレート・エンジン
* [Basil](https://github.com/kumarshantanu/basil) - 汎用用途のテンプレート・ライブラリ
* [Stencil](https://github.com/davidsantiago/stencil) - 速い、Mustache互換の実装
* [Enlive](https://github.com/cgrand/enlive) - セレクタをベースにした（CSSの）テンプレートと変換system

## Miscellaneous

* [clj-pdf](https://github.com/yogthos/clj-pdf) - PDF report generation library
* [clj-rss](https://github.com/yogthos/clj-rss) - RSSフィードを生成するためのライブラリ
* [ring-logger-timbre](https://github.com/nberger/ring-logger-timbre) - ringのリクエストとレスポンスをTimbreを使ってロギングする
* [slf4j-timbre](https://github.com/fzakaria/slf4j-timbre) - ClojureのTimbreというロギング・ライブラリのSLF4Jバインディング
* [ring-rewrite](https://github.com/ebaxt/ring-rewrite) - リライトのルールを定義市適用するためのringミドルウェア
* [Pantomime](https://github.com/michaelklishin/pantomime) - MIMEタイプに基いて処理するためのライブラリ
* [Route One](https://github.com/clojurewerkz/route-one) - （Ruby on Rails や 同種のモダンなWebアプリケーション・フレームワークにおけるのと同じような）HTTPリソースのルーティングを生成するライブラリ
* [Schema](https://github.com/prismatic/schema) - 宣言的なデータ記述とバリデーションを行うClojure(Script)ライブラリ
* [Urly](https://github.com/michaelklishin/urly) - URI、URL、そして相対hrefのようなURLライクな値の解析を統一的に扱うライブラリ
* [Validateur](http://clojurevalidations.info/articles/getting_started.html) - RubyのActiveModelに影響を受けたバリデーション・ライブラリ
* [aging-session](https://github.com/diligenceengine/aging-session) - メモリを使ったringのセッション・ストアで、時間の概念を取り入れている
* [Timbre](https://github.com/ptaoussanis/timbre) - Clojure/Scriptのロギングとプロファイリングのライブラリ
* [Throttler](https://github.com/brunoV/throttler) - （例えば入ってくるリクエストなどの）関数呼び出しに対する、全体的な割合と一時的なバーストの割合をともに制御するための token bucket algorithm
* [Elastisch](https://github.com/clojurewerkz/elastisch) - モダンな分散検索エンジンであるElasticSearchの、ミニマルなClojureクライアント
* [cronj](http://docs.caudate.me/cronj/) - スケジューリング・タスクのライブラリ
* [ring-async](https://github.com/ninjudd/ring-async) - 非同期レスポンスをサポートするためのRingアダプタ

## Web Services 

* [sweet-liberty](https://github.com/RJMetrics/sweet-liberty) - データベースを背後に持ったRESTfulなサービスを構築するためのライブラリ
* [Liberator](http://clojure-liberator.github.com/) - RESTサービスを作成するためのライブラリ
* [necessary-evil](https://github.com/brehaut/necessary-evil) - Clojureの XML RPC ライブラリ

ここに挙げた内容はカテゴリがかなり少なく、テスト、データ・バリデーション、テキスト検索、ランダム・データの生成、JSON解析、例外制御、SQLの抽象化、そしてそれ以外にもWeb開発と関連した多くのライブラリを次のWebサイトで見つけることができます。[The Clojure Toolbox](http://www.clojure-toolbox.com/)、[ClojureSphere](http://www.clojuresphere.com/)、そして[ClojureWerkz](http://clojurewerkz.org/)です。
