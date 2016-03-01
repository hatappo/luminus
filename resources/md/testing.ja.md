Luminusは、デフォルトのテストのサンプル一式をプロジェクトの`test`ディレクトリにセットアップします。

データベースのテストは、`:test`profileを使って実行します。このprofileは、`project.clj`と`profiles.clj`の、`:project/test`と`:profiles/test`のそれぞれを合成してなっています。

デフォルトのテストはアプリケーションのハンドラに対して作成されます。

```clojure
(ns myapp.test.handler
  (:require [clojure.test :refer :all]
            [ring.mock.request :refer :all]
            [<app>.handler :refer :all]))

(deftest test-app
  (testing "main route"
    (let [response (app (request :get "/"))]
      (is (= 200 (:status response)))))

  (testing "not-found route"
    (let [response (app (request :get "/invalid"))]
      (is (= 404 (:status response))))))
```

このテストでは、`ring.mock.request/request`関数を使って生成されたモックのリクエストが、`<app>.handler/app`関数に渡されます。レスポンスは、そのルーティングの期待されるレスポンスに反してテストされます。サンプルのテストは、有効なルーティングへのリクエストがステータス200がレスポンスされ、かつ有効でないルーティングについては404が返る
ことを単にテストしています。


例えば`+postgres`などのRDBのprofileを使うと、1セットのデータベース・テストのコードがプロジェクトに追加されます。

```clojure
(ns myapp.test.db.core
  (:require [clojure.test :refer :all]
            [myapp.db.core :refer :all]
            [myapp.db.migrations :as migrations]))

(deftest test-users
  ;; Make sure the user with id 1 doesn't exist.
  ;; You can also use transactions around tests to ensure that.
  (delete-user! {:id "1"})  
  (is (= 1 (create-user! {:id         "1"
                          :first_name "Sam"
                          :last_name  "Smith"
                          :email      "sam.smith@example.com"
                          :pass       "pass"})))
  (is (= (get-user {:id "1"})
         [{:id         "1"
           :first_name "Sam"
           :last_name  "Smith"
           :email      "sam.smith@example.com"
           :pass       "pass"
           :admin      nil
           :last_login nil
           :is_active  nil}])))

(use-fixtures :once (fn [f] (migrations/migrate ["migrate"]) (f)))
```

テスティング環境のデータベースの接続設定は、`profiles.clj`の`:profiles/test`profileの中で定義しなければいけません。
