## Sessions

Luminusはデフォルトでイン・メモリのセッションを使用します。

[Immutant](http://immutant.org/)サーバを使うと、[wrap-session](http://immutant.org/documentation/2.0.2/apidoc/immutant.web.middleware.html#var-wrap-session)ミドルウェアによって提供されるサーブレットのセッションと統合されます。

セッションのミドルウェアは、`wrap-base` 関数によって`<app>.middleware`名前空間の中で初期化されます。セッションのタイムアウト時間は、秒で指定することができ、デフォルトでは30分何も動作がないとタイムアウトします。

```clojure
(defn wrap-base [handler]
  (-> handler
      wrap-dev
      wrap-formats
      wrap-webjars
      (wrap-defaults
        (-> site-defaults
            (assoc-in [:security :anti-forgery] false)
            (dissoc :session)))
      wrap-flash
      wrap-session
      wrap-context
      wrap-internal-error))
```

あるいは、[ring-ttl-session](https://github.com/boechat107/ring-ttl-session)ライブラリでセッションを機能強化します。この`ring-ttl-session`ライブラリを使えば、生存時間（TTL）付きでイン・メモリにセッションを保存することができます。

メモリに保存するデフォルトの実装は簡単に別の実装に挿し替えられます。例えばクッキーに保存する実装など。以下のコードでは、独自のセッション保存場所として`example-app-session`というnameで`ring.middleware.session.cookie/cookie-store`を指定しています。

```clojure
(wrap-defaults
  (-> site-defaults
      (assoc-in [:security :anti-forgery] false)
      (assoc-in [:session :store] (cookie-store))
      (assoc-in [:session :cookie-name] "example-app-sessions")))
```

`:max-age`キーを使って、セッション・クッキーの maximum age を指定することも可能です。

```clojure
(wrap-defaults
  (-> site-defaults
      (assoc-in [:security :anti-forgery] false)
      (assoc-in [:session :store] (cookie-store))
      (assoc-in [:session :cookie-attrs] {:max-age 10})))
```

セッションをクッキーに保存する場合、クッキーの暗号化に用いる秘密の鍵（16文字）を指定することも重要です。鍵を明示的に指定しない場合、アプリケーションが起動するタイミング都度ランダムな鍵が生成され、以前に作成されたセッションは破棄されるでしょう。

```clojure
(wrap-defaults
  (-> site-defaults
      (assoc-in [:security :anti-forgery] false)
      (assoc-in [:session :store] (cookie-store {:key "BuD3KgdAXhDHrJXu"}))
      (assoc-in [:session :cookie-name] "example-app-sessions")))
```

セッション・ストアとして[Redis](http://redis.io/)を試したいと思うかもしれません。[Carmine](https://github.com/ptaoussanis/carmine)があるおかげでRedisにセッションを作成するのは簡単です。単にRedisのコネクションを定義し、そのコネクションを`taoensso.carmine.ring/carmine-store`とともに使用するだけで済みます。

```clojure
(def redis-conn {:pool {<opts>} :spec {<opts>}})


(wrap-defaults
  (-> site-defaults
      (assoc-in [:security :anti-forgery] false)
      (assoc-in [:session :store] (carmine-store redis-conn))))
```

より詳細な情報は[APIの公式ドキュメント](http://ptaoussanis.github.io/carmine/taoensso.carmine.ring.html)を見てください。

### Accessing the session

Ringはリクエストmapを使ってセッションを追跡します。現在のセッションは`:session`キー以下にあります。以下はセッションをやり取りする簡単な例です。

```clojure
(ns myapp.home
  (:require [compojure.core :refer [defroutes GET]]
            [ring.util.response :refer [response]]))

(defn set-user! [id {session :session}]
  (-> (str "User set to: " id)
      response
      (assoc :session (assoc session :user id))))

(defn remove-user! [{session :session}]
  (-> (response "")
      (assoc :session (dissoc session :user))))

(defn clear-session! []
  (dissoc (response "") :session))

(defroutes app-routes
  (GET "/login/:id" [id :as req] (set-user! id req))
  (GET "/remove" req (remove-user req))
  (GET "/logout" req (clear-session!)))
```

デフォルトの`<app>.layout/render`関数はセッションをセットすることができないことに注意してください。この`render`関数は、ページ・レンダリングが本来の責務であり、コントローラのいかなる処理とも一緒になるべきではありません。セッションをセットしてページもレンダリングしたいというシナリオにおいては、リダイレクトするのがおすすめのアプローチです。

### Flash sessions

Flash session とは、1つのリクエストの間だけ有効なセッションを言います。この Flash session を使うには、通常のセッションに利用される`:session`キーの代わりに`:flash`キーを利用します。

## Cookies

クッキーは、リクエストの`:cookies`キー以下ににあります。

```clojure
{:cookies {"username" {:value "Bob"}}}
```

ということは逆に言えば、レスポンスにクッキーをセットしたければ、単にレスポンスmapを目的のクッキーの値に更新すればOKです。

```clojure
(-> "cookie set" response (update-in [:cookies "username" :value] "Alice"))
```

クッキーは、`:value`キー以外に以下のような追加の属性を持っています。


* :domain - クッキーを特定のドメインに制限する
* :path - クッキーを特定のパスに制限する
* :secure - trueの場合はクッキーをHTTPSのURLに制限する
* :http-only - trueの場合はクッキーをHTTPのURLに制限する（これは、例えばJavaScript経由でのアクセスができなくなる）
* :max-age - クッキーを破棄するまでの秒数
* :expires - クッキーを破棄する明示的な日時
