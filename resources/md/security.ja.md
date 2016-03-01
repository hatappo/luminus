## Restricting Route Access

ルーティングの構築のやり方については[ルーティングの節](/docs/routes.md#restricting_access)を参照してください。

## Password Hashing

パスワードのハッシュ化と照合は、[Buddy](https://github.com/funcool/buddy)が提供する`buddy.hashers`名前空間で取り扱われます。

`buddy.hashers`では2つの関数が提供されます。`encrypt`と`check`です。`encrypt`は、パスワードにsaltを付加し暗号化します。`check`は、`encrypt`により生成された平文のパスワードと暗号化された文字列を照合します。デフォルトの暗号化アルゴリズムとしてBCryptが採用されています。

```clojure
(ns myapp.auth
  (:require
   ...
   [buddy.hashers :as hashers]))

(def raw "secret")
(def encrypted (hashers/encrypt raw))
(hashers/check raw encrypted)
```

`encrypt`関数は、第一引数に指定する平文以外に追加のパラメータを指定することができます。暗号化アルゴリズムや（暗号化の）繰り返しの回数などです。

```clojure
(hashers/encrypt "secretpassword" {:algorithm :pbkdf2+sha256})
(hashers/encrypt "secretpassword" {:algorithm :pbkdf2+sha256
                                   :salt "123456"})
(hashers/encrypt "secretpassword" {:algorithm :pbkdf2+sha256
                                   :salt "123456"
                                   :iterations 200000})
```

次の暗号化アルゴリズムに紐付くオプションとそのデフォルト値は以下のとおりです。

* `:algorithm :bcrypt+sha512`
    * `:iterations` 12
    * `:salt` random
* `:algorithm :pbkdf2+sha256`
    * `:iterations` 100000
    * `:salt` random
* `:algorithm :pbkdf2+sha3_256`
    * `:iterations` 100000
    * `:salt` random
* `:algorithm :pbkdf2+sha1`
    * `:iterations` 100000
    * `:salt` random
* `:algorithm :scrypt`
    * `:salt` random
    * `:cpucost` 65536
    * `:memcost` 8
    * `:parallelism` 1
* `:algorithm :sha256`
    * `:salt` random
* `:algorithm :md5`
    * `:salt` random

特定のルートのアクセスを制限する情報は[ルーティングの節](/docs/routes.md#marking_routes_as_restricted)を参照してください。

別のセキュリティ・ソリューションとしては、[Friend](https://github.com/cemerick/friend)ライブラリをチェック・アウトすると良いかもしれません。

## LDAP Authentication

TODO:

### Cross Site Request Forgery Protection

CSRF攻撃は、ログイン状態にあるユーザのクリデンシャルを使って第三者があなたのサイト上で任意の動作をさせることを引き起こします。CSRF攻撃は通常、あなたのサイトに良くないリンクやフォーム・ボタン、あるいはなんらかのJavaScriptがある場合に引き起こされる可能性があります。

[Ring-Anti-Forgery](https://github.com/ring-clojure/ring-anti-forgery)は、このCSRF攻撃を防ぐために使用されます。対偽造リクエスト防御（anti-forgery protection）はデフォルトで有効になっています。

いったんCSRFミドルウェアを有効にすると、ランダムに生成された文字列が*anti-forgery-token*varとして割り当てられます。サーバに届く全てのPOSTリクエストは、`__anti-forgery-token`と呼ばれるパラメータにこのCSRFトークンを保持していなければなりません。

あなたのアプリケーションの`<app>.layout`名前空間に、ページにこのCSRFトークンを埋め込むことができる`csrf-field`タグを生成します。

```clojure
(parser/add-tag! :csrf-field (fn [_ _] (anti-forgery-field)))
```

HTMLテンプレートの中でこのタグを次のようにして利用できます。

```xml
<form name="input" action="/login" method="POST">
  {% csrf-field %}
  Username: <input type="text" name="user">
  Password: <input type="password" name="pass">
  <input type="submit" value="Submit">
</form>
```

GETでもHEADでもなく、かつCSRFトークンを持たない全てのリクエストは、CSRFミドルウェアによって拒否されます。このときサーバは「Invalid anti-forgery token」というメッセージ付きで403エラーを返答します。

Luminusでは、anti-forgeryミドルウェアは、`<app>.handler`名前空間の`app`を定義している場所で`home-routes`にラップされています。`wrap-csrf`ラッパーは`home-routes`に明示的に適用されていることに注意してください。外部のクライアントによって使用されることがなさそうなその他のルーティングのグループについても、`wrap-csrf`は適用すべきです。

```clojure
(def app
  (-> (routes
        (wrap-routes home-routes middleware/wrap-csrf) ;; CSRF protection でラップ
        base-routes)
      middleware/wrap-base))
```

なんらかの理由でこの機能を無効にしたい場合は、単に`app`の定義を次のように変えればOKです。

```clojure
(def app
  (-> (routes
        home-routes ;; CSRF protection 無し
        base-routes)
      middleware/wrap-base))
```

アプリケーションの中でルートによって選択的にCSRF機能を有効にするやり方の詳細は、[ここ](/docs/services.md#csrf)を見てください。
