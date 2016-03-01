## Requests

フォームのPOST送信などのリクエスト・パラメータは、デフォルトでは自動でパースされてリクエストの`:params`キーにセットされます。

しかしながら、もしあなたがリクエストのボディとしてパラメータを渡しているなら、そのパラメータを正しく取り扱うための適切なミドルウェアを有効にする必要があるでしょう。Luminusは、リクエスト・ボディのパラメータをエンコードとデコードするために[ring-middleware-format](https://github.com/ngrunwald/ring-middleware-format)を使います。

リクエスト・パラメータは、リクエストの`:params`キー配下から取得できるでしょう。レスポンスに適切なMIMEタイプを設定しているとき、レスポンス・ボディのエンコードについてもこのring-middleware-formatミドルウェアが処理することに注意してください。レスポンス生成のより詳しい情報については、[response types](http://www.luminusweb.net/docs/responses.md)の節を見てください。

有効なフォーマットの一覧は以下の通りです。

* :json - :params と :body-params 内に文字列のキーを持つJSON
* :json-kw - :params と :body-params 内にキーワード化されたキーを持つJSON
* :edn - Clojureのネイティブなフォーマット
* :yaml - YAMLフォーマット
* :yaml-kw - :params と :body-params 内にキーワード化されたYAMLフォーマット 
* :yaml-in-html - HTMLページ内のyaml
* :transit-json JSONフォーマットでのやり取り
* :transit-msgpack Msgpackフォーマットでのやり取り
