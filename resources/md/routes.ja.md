Luminusでは、Compojureを使ってアプリケーションのルーティングを定義します。ルーティングは、アプリケーションの入り口となり、サーバとクライアント間のコミュニケーション・プロトコルを確立するのに使用されます。

### Routes

Compojureのルーティングの定義は、[リクエストmapを受け取りレスポンスmapを返す](https://github.com/mmcgrana/ring/blob/master/SPEC)単なる関数です。個々のルーティングは、URLマッチング・パターン、引数リスト、そしてボディにHTTPメソッドを紐付けたものです。

```clojure
(GET "/" [] "なにかを表示する")
(POST "/" [] "なにかを作成する")
(PUT "/" [] "なにかを置換する")
(PATCH "/" [] "なにかを修正する")
(DELETE "/" [] "なにかを無効にする")
(OPTIONS "/" [] "なにかをAppease譲歩する")
(HEAD "/" [] "なにかをプレビューする")
```

ボディは、パラメータとしてリクエストを受け取る関数になるでしょう。

```clojure
(GET "/" [] (fn [req] "リクエストでなにかする"))
```

あるいは、ルーティングの第二引数としてリクエストを宣言すれば、直接リクエストmapを使用することも可能です。

```clojure
(GET "/foo" request (interpose ", " (keys request)))
```

上記のルーティングは、リクエストmapから全てのキーを読み出して表示します。出力は、次のような見た目です。

```clojure
:ssl-client-cert, :remote-addr, :scheme, :query-params, :session, :form-params,
:multipart-params, :request-method, :query-string, :route-params, :content-type,
:cookies, :uri, :server-name, :params, :headers, :content-length, :server-port,
:character-encoding, :body, :flash
```

パラメータに名前を付けてルーティングのパターンに含めることができます。

```clojure
(GET "/hello/:name" [name] (str "こんにちは、" name))
```

正規表現を渡すことによって、各パラメータが何にマッチするかを調整することができます。

```clojure
(GET ["/file/:name.:ext" :name #".*", :ext #".*"] [name ext]
    (str "ファイル：" name ext))
```

ハンドラでクエリ・パラメータを有効化することもできます。

```clojure
(GET "/posts" []
  (fn [req]
    (let [title (get-in req [:params :title])
          author (get-in req [:params :author])]
      "titleとauthorでなにかする")))
```

あるいは、POSTとPUTのリクエストでは、クエリ・パラメータではなくフォーム・パラメータを有効化します。

```clojure
(POST "/posts" []
  (fn [req]
    (let [title (get-in req [:params :title])
          author (get-in req [:params :author])]
      "titleとauthorでなにかする")))
```

Compojureでは、フォームパラメータにアクセスする構文糖衣も提供されています。以下のとおりです。

```clojure
(POST "/hello" [id] (str "ようこそ、" id))
```

ゲストブックのアプリケーション例では、次のようなルーティング定義がありました。

```clojure
(POST "/"  [name message] (save-message name message))
```

`POST`リクエストはデフォルトでCSRFトークンを含まなければならないことに注意してください。CSRFミドルウェアを管理するより詳細な情報は[ここ](/docs/security.md#cross_site_request_forgery_protection)を参照してください。

このルーティングは、フォーム・パラメータからnameとmessageを取り出し、同名の変数にそれらを束縛します。すると、普通に宣言されたその他の変数と同様にこのnameとmessageという変数を使用することができるようになります。

ルーティングの内側では、標準のClojureの分配束縛を使用することも可能です。

```clojure
(GET "/:foo" {{foo "foo"} :params}
  (str "Foo = " foo))
```

そのうえCompojureでは、フォーム・パラメータのサブセットに分配束縛を使い、余ったパラメータのマップを作成することができます。

```clojure
[x y & z]
x -> "foo"
y -> "bar"
z -> {:v "baz", :w "qux"}
```

上記では、パラメータxとyは変数に束縛されて、パラメータvとwはzという名前のmapに残ります。結局のところ、各パラメータとともに完全なリクエストのデータも必要であれば、次のように書くことができます。

```clojure
(GET "/" [x y :as r] (str x y r))
```

ここでは、フォーム・パラメータxとyを束縛し、完全なリクエストmapをrという変数に束縛しています。

### Return values

ルーティング・ブロックは、次のうち少なくとも1つを返します。HTTPクライアントに渡すためのレスポンスボディか、あるいはringスタックにおける次のミドルウェアです。最もありふれた返り値は、上記の例のような文字列です。しかし、[レスポンスmap](https://github.com/mmcgrana/ring/blob/master/SPEC)を返すこともできます。

```clojure
(GET "/" []
    {:status 200 :body "Hello World"})

(GET "/is-403" []
    {:status 403 :body ""})

(GET "/is-json" []
    {:status 200 :headers {"Content-Type" "application/json"} :body "{}"})
```

## Static Resources

デフォルトでは、`resources/public`ディレクトリ配下に置かれた全てのリソースは、クライアントから見えるようになります。この制御は、`<app>.handler`名前空間の中にある`compojure.route/resources` ハンドラで行われています。

```clojure
(defroutes base-routes
  (route/resources "/")
  (route/not-found "Not Found"))
```

アプリケーションのクラスパス上に置かれた全てのリソースは、`clojure.java.io/resource`関数を使ってアクセスすることが可能です。

```clojure
(slurp (clojure.java.io/resource "myfile.md"))
```

規約として、ソースコードではなくリソース・ファイルは、プロジェクトの`resources`ディレクトリ内に配置することになります。

### Handling file uploads

次のようなフォームを持った`upload.html`という名前のページにおいて

```xml
<h2>ファイルのアップロード</h2>
<form action="/upload" enctype="multipart/form-data" method="POST">
    {% csrf-field %}
    <input id="file" name="file" type="file" />
    <input type="submit" value="upload" />
</form>
```

次のようにして、このページをレンダリングしファイル・アップロードを取り扱うことができます。

```clojure
(ns myapp.upload
  (:use compojure.core)
  (:require [myapp.layout :as layout]
            [ring.util.response :refer [redirect file-response]])
  (:import [java.io File FileInputStream FileOutputStream]))

(def resource-path "/tmp/")

(defn file-path [path & [filename]]
  (java.net.URLDecoder/decode
    (str path File/separator filename)
    "utf-8"))

(defn upload-file
  "ターゲットのフォルダにファイルをアップロードします。
   :create-path? フラグがtrueになっている場合、ターゲットのパスは作成されます。"
  [path {:keys [tempfile size filename]}]
  (try
    (with-open [in (new FileInputStream tempfile)
                out (new FileOutputStream (file-path path filename))]
      (let [source (.getChannel in)
            dest   (.getChannel out)]
        (.transferFrom dest source 0 (.size source))
        (.flush out)))))

(defroutes home-routes
  (GET "/upload" []
       (layout/render "upload.html"))

  (POST "/upload" [file]
       (upload-file resource-path file)
       (redirect (str "/files/" (:filename file))))

  (GET "/files/:filename" [filename]
       (file-response (str resource-path filename))))
```

`:file`リクエストのフォーム・パラメータは、アップロードされるファイルの説明を入れたmapです。上記の`upload-file`関数では、ディスクにファイルを保存するのに、このmapの`:tempfile`、`:size`、そして`:filename`キーを使います。

もしアプリケーションの手前にNginxをかましているなら、この[Upload Progress Module](http://wiki.nginx.org/HttpUploadProgressModule)を使えばファイルアップロードの進捗機能を簡単にサポートできます。

## Organizing application routes

アプリケーションのルーティングを機能毎に編成するのは良いやり方です。Compojureは`defroutes`というマクロを提供しています。これを使うと、複数のルーティングをグルーピングしシンボルに束縛することができます。

```clojure
(defroutes auth-routes
  (POST "/login" [id pass] (login id pass))
  (POST "/logout" [] (logout)))

(defroutes app-routes
  (GET "/" [] (home))
  (route/resources "/")
  (route/not-found "Not Found"))
```

`context`を使って、共通するパス要素のルーティングをまとめることもできます。以下のように`/user/:id`パスを共通で使用する一揃いのルーティングがあったとしたら、

```clojure
(defroutes user-routes
  (GET "/user/:id/profile" [id] ...)
  (GET "/user/:id/settings" [id] ...)
  (GET "/user/:id/change-password [id] ...))
```

こういうふうに書き換えることができます。

```clojure
(def user-routes
  (context "/user/:id" [id]
    (GET "/profile" [] ...)
    (GET "/settings" [] ...)
    (GET "/change-password" [] ...)))
```

いったんルーティングを定義したら、あなたのアプリケーションのメインのハンドラにそのルーティングを追加することができます。Luminusテンプレートでは`handler`名前空間の中に`app`が既に定義されていることに気付くでしょう。そのため、あなたは新しいルーティングをそこに追加するだけでOKです。

`home-routes`のときに見てきた`wrap-routes`を使えば、独自のミドルウェアを適用することができることを強調しておきます。独自のミドルウェアは、マッチしたルーティングの後に解決され、`middleware/wrap-base`の中で参照されるグローバルなミドルウェアとは違って特定のルーティングにのみ作用します。

```clojure
(def app-routes
  (routes
    (wrap-routes #'home-routes middleware/wrap-csrf)
    (route/not-found
      (:body
       (error-page {:code 404
                    :title "page not found"})))))

(def app (middleware/wrap-base #'app-routes))
```

より詳しいドキュメントは[Compojureの公式Wiki](https://github.com/weavejester/compojure/wiki)で手に入ります。

## Restricting access

いくつかのページは、特定の条件に合致した場合にのみアクセス可能にすべきです。例えば、管理者のみ管理ページを見ることができるとか、ユーザのプロフィールのページはそのユーザのセッションがある場合にのみ見ることができる、というふうにしたくなることがあるでしょう。
るように定義したいとします。

### Restricting access based on route groups

最も簡単な方法は、`restrict`ミドルウェアを適用することによって、公にはアクセスすべきではないルーティング・グループへのアクセスを制限することです。まず、次のコードを`<app>.middleware`名前空間の中に追加しましょう。

```clojure
(ns <app>.middleware
  (:require
    ...
    [buddy.auth.middleware :refer [wrap-authentication]]
    [buddy.auth.backends.session :refer [session-backend]]
    [buddy.auth.accessrules :refer [restrict]]
    [buddy.auth :refer [authenticated?]]))

(defn on-error [request response]
  {:status  403
   :headers {"Content-Type" "text/plain"}
   :body    (str (:uri request) "へのアクセスは認証されていません。")})


(defn wrap-restricted [handler]
  (restrict handler {:handler authenticated?
                     :on-error on-error}))

(defn wrap-base [handler]
  (-> handler
      wrap-dev
      (wrap-authentication (session-backend))
      ...))
```

認証のミドルウェアでラップすると、セッションがある持続している場合にリクエスト内に`:identity`キーがセットされます。セッションによる仕組みは、利用可能な最もシンプルな方法ではあります。しかしながらBuddyを使えば、[ここ](https://funcool.github.io/buddy-auth/latest/#_authentication)に書かれているような数多くの様々な認証の仕組みが提供されます。

`authenticated?`ヘルパーは、リクエスト内の`:identity`キーを確認し存在する場合にそれをハンドラに渡すのに使われています。もし`:identity`キーが存在しなかったら`on-error`関数を呼び出します。

これは、新しいプロジェクトの作成時に`+auth`profileを使って生成されるデフォルトの認証の構成です。

`<app>.handler/app`名前空間の中の `wrap-restricted`ミドルウェアを使って、隠したいルーティングのグループをラップすることができます。

```clojure
(def app
  (-> (routes
        (-> home-routes
            (wrap-routes middleware/wrap-csrf)
            (wrap-routes middleware/wrap-restricted))
        base-routes)
      middleware/wrap-base))
```

### Restricting access based on URI

[Buddy](https://funcool.github.io/buddy-auth/latest/)の`buddy.auth.accessrules`名前空間を使って、URIのパターンに基いて特定ページへのアクセスを制限するルールを定義することができます。

### Specifying Access Rules

セッション内に`:identity`キーが存在する場合にのみアクセス可能にすべき制限されたルーティングを指定するルール作成する方法を見ていきましょう。

まず、`<app>.middleware`名前空間の中にいくつかのBuddy名前空間を参照します。

```clojure
(ns myapp.middleware
  (:require ...
            [buddy.auth.middleware :refer [wrap-authentication]]
            [buddy.auth.accessrules :refer [wrap-access-rules]]
            [buddy.auth.backends.session :refer [session-backend]]
            [buddy.auth :refer [authenticated?]]))
```

次に、アクセス・ルールを作成します。このアクセス・ルールは、各ルールがmapを使って表現されたvectorを使用して定義されます。ユーザが認証されているかどうかを確認する簡単なルールは、以下のようになります。

```clojure
(def rules
  [{:uri "/restricted"
    :handler authenticated?}])
```

拒否されるあるルーティングにアクセスされたときに使用されるエラー制御の関数も定義しましょう。

```clojure
(defn on-error
  [request value]
  {:status 403
   :headers {}
   :body "Not authorized"})
```

最後に、Sessionの仕組みを使って、認証とアクセス・ルールを有効にするのに必要なミドルウェアを追加しなくてはいけません。
Finally, we have to add the necessary middlware to enable the access rules and authentication using a session backend.

```clojure
(defn wrap-base [handler]
  (-> handler
      (wrap-access-rules {:rules rules :on-error on-error})
      (wrap-authentication (session-backend))
      ...))
```

ミドルウェアの順序が重要であり、`wrap-access-rules`は`wrap-authentication`の前に配置されなければいけないことに注意してください。

Buddyのセッションは、ユーザが問題なく認証されたときにセッション内に`:identity`キーがセットされることによってできあがります。

```clojure
(def user {:id "ボブ" :pass "秘密の言葉"})

(defn login! [{:keys [params session]}]
  (when (= user params)
    (-> "ok"
        response
        (content-type "text/html")
        (assoc :session (assoc session :identity "ほげ")))))
```
