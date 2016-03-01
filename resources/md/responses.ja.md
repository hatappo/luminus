## Responses

Ringのレスポンスは[ring-http-response](https://github.com/metosin/ring-http-response)ライブラリを使って生成されます。この`ring-http-response`ライブラリはHTTPの各ステータスコードを持ったレスポンスを作成するための数多くのヘルパーを提供しています。

例えば、`ring.util.http-response/ok`ヘルパーは、ステータスコード`200`のレスポンスを生成するのに使います。次のコードは、有効なレスポンスmapを生み出し、その`:body`キーにはコンテンツがセットされます。

```clojure
(ok {:foo "bar"})

;;result of calling response
{:status  200
 :headers {}
 :body    {:foo "bar"}}
```

レスポンスのボディは単なるstring、sequence、file、あるいは input stream のうち1つを利用できます。レスポンス・ボディは、そのステータスコードと適切に対応していなければいけません。

stringの場合、クライアントにそのまま送信されます。sequenceの場合、各要素を表す文字列がクライアントに送信されます。そして、レスポンスがfileや input stream の場合、サーバはその中身をクライアントに送信します。

### Response encoding

ルートが`:body`キーを持つmapを返すとき、デフォルトでは、[ring-middleware-format](https://github.com/ngrunwald/ring-middleware-format)ミドルウェアをレスポンス・タイプの推測に使います。

```clojure
(GET "/json" [] {:body {:foo "bar"}})
```

`ring-middleware-format`はアプリケーションの`<app-name>.middleware`名前空間にあります。このミドルウェア関数は`wrap-formats`から呼び出されるので、`wrap-formats`が呼び出されると、`ring-middleware-format`の持つ JSON/Transit encoding のサポートも有効になります。

```clojure
  (defn wrap-base [handler]
  (-> handler
      wrap-dev
      wrap-formats ;; JSON/Transit のシリアライゼーションとデシリアライゼーションを有効に
      (wrap-defaults
        (-> site-defaults
            (assoc-in [:security :anti-forgery] false)
            (assoc-in  [:session :store] (ttl-memory-store (* 60 30)))))
      wrap-servlet-context
      wrap-internal-error))
```

フォーマットは、`:formats`キーによって次のように選択され、制御されます。

```clojure
(defn wrap-formats [handler]
  (wrap-restful-format handler {:formats [:json-kw :transit-json :transit-msgpack]}))
```

利用可能なフォーマットは

* :json - :params と :body-params 内に文字列のキーを持つJSON
* :json-kw - :params と :body-params 内にキーワード化されたキーを持つJSON
* :edn - Clojureのネイティブなフォーマット
* :yaml - YAMLフォーマット
* :yaml-kw - :params と :body-params 内にキーワード化されたYAMLフォーマット 
* :yaml-in-html - HTMLページ内のyaml

`Accept`ヘッダにフォーマットの指定がなかったり不明なフォーマットが指定されていたりすると、ハンドラ内の`formats`vectorの先頭のフォーマットが使用されます（デフォルトはJSONです）。


## Setting headers

`ring.util.http-response/header`を呼び出してヘッダのmapを渡すことで、レスポンス・ヘッダを追加することができます。ヘッダmapのキーは**文字列でなければならない**ことに注意してください。

```clojure
(-> "hello world" response (header "x-csrf" "csrf"))
```

## Setting content type

`ring.util.http-response/content-type`関数を使うことで、独自のレスポンス・タイプを設定できます。例えば、

```clojure
(GET "/project" []
  (-> (clojure.java.io/input-stream "report.pdf")
      response
      (content-type "application/pdf")))
```

## Setting custom status

`ring.util.http-response/status`関数にステータス・コードを渡すことで、独自のHTTPステータスを設定できます。

```clojure
(GET "/missing-page" []
  (-> "your page could not be found"
      response
      (status 404)))
```

## Redirects

リダイレクトの制御には`ring.util.http-response/found`関数を使います。この`found`関数はレスポンスに `302` redirect のステータスを設定します。

```clojure
(GET "/old-location" []
  (found "/new-location"))
```

利用可能なその他のヘルパーについては[ring-http-response](https://github.com/metosin/ring-http-response/blob/master/src/ring/util/http_response.clj)を参照してください。
