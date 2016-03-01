Clojureにおけるアプリケーション開発の典型的なワークフローでは、アプリケーションのインスタンスを実行しているREPLにエディタを接続します。Luminusでは、2つの接続方法を提供しています。

## Starting the Application from the REPL

プロジェクトのディレクトリでREPLを起動すると、最初に`yourapp.core`名前空間に移動するでしょう。この名前空間は、`http`という名前で`luminus.http-server`名前空間を参照します。この名前空間には、`start`と`stop`が入っており、それぞれHTTPサーバの起動と停止に使用します。

`start`関数は、[Ring](https://github.com/ring-clojure/ring)ハンドラ、初期化関数、そしてポートを引数として受け取ります。

```clojure
(http/start {:port 3000 :init init :handler app})
```

`stop`関数は、停止用の関数を取ります。

```clojure
(http/stop destroy)
```

## Connecting to the nREPL

Luminusでは、埋め込み型の[nREPL](https://github.com/clojure/tools.nrepl)も提供されています。これを使うと、実行中のサーバ・インスタンスにエディタを接続することができます。デフォルトのnREPLのポートは`project.clj`ファイルの中に、開発環境用である`:project/dev`profile配下に設定します。

```clojure
:profiles
{...
 :project/dev {...
               :env {:dev        true
                     :port       3000
                     :nrepl-port 7000
                     :log-level  :trace}}}
```

`lein repl`を使ってアプリケーションを実行すると、`7000`番ポートにネットワークREPLが作成され、`localhost:7000`を使ってエディタを接続することができます。`NREPL_PORT`環境変数がセットされていると、nREPLが本番環境でも有効になってしまい、開発環境でやっているのと同じようにして本番のアプリケーションの中を覗くことができるようになってしまうので注意してください。

本番環境でnREPLを利用可能にする手順については[deployment section](http://www.luminusweb.net/docs/deployment.md#enabling_nrepl)を見てください。
