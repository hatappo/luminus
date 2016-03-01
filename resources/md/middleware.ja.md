## Adding custom middleware
Luminusはアプリケーション・ハンドラのルーティングにRingを使用しています。使用しているのは標準的なRingハンドラなので、他のRingベースのアプリケーションと全く同じようにミドルウェアでラップすることができます。

ミドルウェアによって、リクエストがどう処理されるかを修正することができる関数の中でハンドラをラップすることができます。ミドルウェア関数は、Ringハンドラの基本的な機能を拡張するのにしばしば利用されます。この拡張によって、Ringハンドラをアプリケーション固有のニーズに合わせることができます。

ミドルウェアは単なる関数です。任意のパラメータとともに既存のハンドラを受け取って、追加の処理を持った新しいハンドラを返します。例えばミドルウェア関数はこんな感じになります。

```clojure
(defn wrap-nocache [handler]
  (fn [request]
     (let [response (handler request)]
        (assoc-in response [:headers  "Pragma"] "no-cache"))))
```

スタックトレースを表示するミドルウェアといったような開発環境用のミドルウェアは全て、`<app>.dev-middleware`名前空間の中にある`wrap-dev`関数に追加されるべきです。この`<app>.dev-middleware`名前空間は、`env/dev/clj`というパスに配置されており、developmentモードの時にのみ取り込まれます。

```clojure
(defn wrap-dev [handler]
  (-> handler
      wrap-reload
      wrap-error-page
      wrap-exceptions))  
```

各ミドルウェア関数によってリクエストが加工されることについて、「ミドルウェアの順序」が重要であることに注意してください。例えば、セッションに依存するミドルウェア関数は全て、セッションを生成するミドルウェアである`wrap-defaults`より前に置かれなければいけません。その理由は、リクエストが、より内側のミドルウェア関数に到達する前により外側のミドルウェア関数を通してくることによります。

例えば、以下のように`wrap-formats`と`wrap-defaults`を使ってラップされたハンドラがあるとします。

```clojure
(-> handler wrap-formats wrap-defaults)
```

このリクエストは、次のような順序でミドルウェア関数を通過してきます。

```
handler <- wrap-formats <- wrap-defaults <- request
```

## Useful ring middleware
* [ring-ratelimit](https://github.com/myfreeweb/ring-ratelimit) - レート制限ミドルウェア
* [ring-etag-middleware](https://github.com/mikejs/ring-etag-middleware) - ringレスポンスのetagを処理し問題なければ304をレスポンスする
* [ring-gzip-middleware](https://github.com/mikejs/ring-gzip-middleware) - Gzipを扱えるユーザ・エージェントのためのGzipされたringレスポンス
* [ring-upload-progress](https://github.com/joodie/ring-upload-progress) - ringセッションにアップロードの進捗を提供する
