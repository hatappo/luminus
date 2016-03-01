[12 factor](http://12factor.net/)スタイルのアプリケーション開発を促進することを目指しています。12 factor のアプローチは、設定とコードとは分離されるべきだと述べています。デプロイされる環境ごとに分けてパッケージされるべきではありません。

Luminusプロジェクトでは、デフォルトで次の環境変数を使用しています。

* `PORT` - アプリケーションをバインドするHTTPのポート番号。デフォルトは3000。
* `NREPL_PORT` - 特定のポートでnREPLサーバを起動するようにアプリケーションを設定してるなら、開発用にデフォルトで7000番が使用されます。
* `DATABASE_URL` - データベース・コネクションのURLです。
* `APP_CONTEXT` - アプリケーションのルートに対して任意のコンテキストを指定するのに使用します。
* `LOG-CONFIG` - 外部のログ設定ファイルを指定するのに使用します。resourcesフォルダ内の`log4j.properties`がデフォルトでは使用されます。

## Managing Environment Variables

アプリケーションで使用する環境変数は、[luminus/config](https://github.com/luminus-framework/config)ライブラリを使って管理されます。この`luminus/config`ライブラリは、EDN形式での設定、shell変数、そしてjavaのシステム・プロパティをサポートしています。

この`luminus/config`ライブラリは、クラスパス上の`config.edn`ファイルを探します。この`config.edn`ファイルの中身は、`System/getenv`と`System/getProperties`で見つかる環境変数の値とマージされます。

`config.edn`に書かれた設定は例えば次のようなmapからなります。

```clojure
{:port 4000}
```

あるいは、`config`環境変数を使って別の設定ファイルを指定することもできます。

```clojure
export CONFIG="prod-config.edn"
```

設定は、次の順序で解決されます。


1. クラスパス上の`config.edn`
2. `config`環境変数を使って指定されたEDN形式の設定ファイル
3. プロジェクトのディレクトリの`.lein-env`ファイル
4. プロジェクトのディレクトリの`.boot-env`ファイル
5. 環境変数
6. javaのシステム・プロパティ

既存の変数は後から見つかった設定値で上書きされます。例えば、`config.edn`ファイル内であるキーが宣言されたとしても、`.lein-env`でそのキーが設定されていれば上書きされます。他もそうです。

`config.edn`ファイルは、特定の環境においてのみ取り込むために、`env/dev/resources`や`env/prod/resources`に置かれます。それぞれの環境においても同様に異なるバージョンを指定することが可能です。

### Using Shell Variables

shell変数は次のように宣言することができます。

```clojure
export DATABASE_URL=jdbc:postgresql://localhost/app?user=app_user&password=secret"
```

システム・プロパティはコマンドライン引数として`java`コマンドに`-D`フラグを付けて渡すことができます。

```clojure
java -Ddatabase_url="jdbc:postgresql://localhost/app?user=app_user&password=secret" -jar app.jar
```

Luminusの設定は、変数名をCloureスタイルのキーワードに変換するように配慮してくれます。変数名は小文字で、`_`と`.`の文字は全て`-`の文字に変換されます。上述の環境変数に対応したキーワード名は次のとおりです。

* `:port`
* `:nrepl-port`
* `:database-url`
* `:app-context`
* `:log-config`

これらの変数は、`config.core/env`map内に展開され、以下の例で見るようにアクセスできます。

```clojure
(ns <app>.db.core
  (:require [config.core :refer [env]]))

(def database-url
  (env :database-url))
```

## Environment Specific Code

いくつかのコード ― 例えばブラウザにスタックトレースを表示する開発環境用のミドルウェアなど ― は、アプリケーションが実行されるモードに依存します。例えば、上記のミドルウェアは開発時にだけ走らせて、本番環境ではクライアントに表示しないということができます。

Luminusは、`env/env/clj`と`env/pord/clj`というパスをこの目的のために使用します。デフォルトではこれらのパスは、その環境固有の設定を保持した`<app>.config`名前空間を持ちます。`dev`の設定は次のようになっています。

```clojure
(ns <project-ns>.config
  (:require [selmer.parser :as parser]
            [clojure.tools.logging :as log]
            [<project-ns>.dev-middleware :refer [wrap-dev]]))

(def defaults
  {:init
   (fn []
     (parser/cache-off!)
     (log/info "\n-=[app started successfully using the development profile]=-"))
   :middleware wrap-dev}
```

同様のパスに存在する`<app>.dev-middleware`名前空間はこの`<app>.config`の設定が参照しています。開発環境に固有のミドルウェアは全てこの名前空間に置かれるべきです。

他方、`prod`の設定がこんな感じになるわけではありません。

```clojure
# 訳注：Luminusは以下のようなコードを prod 用として生成したりはしない。
(ns <project-ns>.config
  (:require [clojure.tools.logging :as log]))

(def defaults
  {:init
   (fn []
     (log/info "\n-=[app started successfully]=-"))
   :middleware identity})
```

単に、`<app>.middleware`名前空間で定義されたミドルウェアのみが本番環境で実行されるというだけです。
