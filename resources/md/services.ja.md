LuminusでRESTサービスを書くおすすめのやり方は、[Compojure-API](https://github.com/metosin/compojure-api)ライブラリを使用しするやり方です。
この`Compojure-API`ライブラリは、エンドポイントのリクエストとレスポンスのパラメータを生成しチェックするために[Prismatic Schema](https://github.com/Prismatic/schema)を使用します。

Swaggerのサポートを追加する最も簡単なやり方は、`+swagger`profileを使うやり方です。

```
lein new luminus swag +swagger
```

上記で生成された結果のプロジェクトには、ルーティング定義のいくつか実装例とともに`swagger.routes.services`名前空間が含まれます。

## Working with Swagger

通常使う`compojure.core/GET`の代わりに`compojure.api.sweet/GET*`などのCompojure-APIのヘルパーを使ってルーティングが宣言されるということが分かります。

これらのエンドポイントに対する構文は、以下に見るような個々のサービスの処理を注釈することを必要とする以外は、通常のCompojureの構文と似ています。

```clojure
(GET* "/plus" []
        :return       Long
        :query-params [x :- Long {y :- Long 1}]
        :summary      "x+y with query-parameters. y defaults to 1."
        (ok (+ x y)))
```

上記のサービスの処理は、次のようにClojureScriptからも呼び出し可能です。

```clojure
(ns swag.core
  (:require [reagent.core :as reagent :refer [atom]]
            [ajax.core :refer [GET]]))

(defn add [params result]
  (GET "/api/plus"
       {:headers {"Accept" "application/transit+json"}
        :params @params
        :handler #(reset! result %)}))

(defn int-value [v]
  (-> v .-target .-value int))

(defn home-page []
  (let [params (atom {})
        result (atom nil)]
    (fn []
      [:div
       [:form
        [:div.form-group
         [:label "x"]
         [:input 
          {:type :text
           :on-change #(swap! params assoc :x (int-value %))}]]
        [:div.form-group
         [:label "y"]
         [:input
          {:type :text
           :on-change #(swap! params assoc :y (int-value %))}]]]
       [:button.btn.btn-primary {:on-click #(add params result)} "Add"]
       (when @result
         [:p "result: " @result])])))

(reagent/render-component [home-page] (.getElementById js/document "app"))
```  

返り値の型とクエリ・パラメータの型を明示し、それぞれのサービスの処理の説明文を与えなければいけません。複雑な型を扱う場合は、それぞれに対してschemaの定義を行わなければいけません。

```clojure
(s/defschema Thingie {:id Long
                      :hot Boolean
                      :tag (s/enum :kikka :kukka)
                      :chief [{:name String
                               :type #{{:id String}}}]})

(POST* "/echo" []
        :return   Thingie
        :body     [thingie Thingie]
        :summary  "echoes a Thingie from json-body"
        (ok thingie))
```
このプロジェクトは、このサービスのドキュメンテーションのページを生成するようにセットアップされます。このAPIドキュメントのページは[ring-swagger-ui](https://github.com/metosin/ring-swagger-ui)ライブラリを使用したものであり、`/swagger-ui`というURLから利用可能です。

```clojure
(ring.swagger.ui/swagger-ui
   "/swagger-ui"
   :api-url "/swagger-docs")
```

## CSRF

CSRF保護は、[ring-anti-forgery](https://github.com/ring-clojure/ring-anti-forgery)ミドルウェアによって提供され、これはデフォルトで有効です。`+swagger`profileにおいては、CSRF保護から除外された`service-routes`というルートが生成されます。

```clojure
(def app
  (-> (routes
        service-routes ;; no CSRF protection
        (wrap-routes home-routes middleware/wrap-csrf)
        base-routes)
      middleware/wrap-base))
```
