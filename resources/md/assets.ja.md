アプリケーションのアセットは、`resources`フォルダ配下に配置されます。ソースコードのファイルではないがアプリケーションに含めたいものを、アセットに入れます。例えば、データベースのマイグレーションのファイルやSQLファイルなどです。`resources/public`フォルダ配下は、HTTPサーバから直接配布されるアセットを置く所と決められています。この`resources/public`は、画像やCSSなどのアセットを置く場所です。

Luminusは、以前に[Webjars](http://www.webjars.org/)のサポートを統合しています。デフォルトでは、この`Webjars`を経由して Twitter Bootstrap のような外部のアセットを取り込みます。Twitter Bootstrap は、`[org.webjars/bootstrap "4.0.0-alpha.2"]`プロジェクトの中に依存関係として含められています。この中身は、[ring-webjars](https://github.com/weavejester/ring-webjars)ミドルウェアを使うと利用可能になります。

いったん`ring-webjars`ライブラリが取り込まれるとそのアセットはクラスパスから利用可能になり、HTMLテンプレートの中で`/assets`接頭辞を付けて取得することができます。

```clojure
{% style "/assets/bootstrap/css/bootstrap.min.css" %}
{% style "/assets/font-awesome/css/font-awesome.min.css" %}
```

この`ring-webjars`ライブラリは、取り込まれた外部アセットからバージョン番号を取り除きます。`/assets/bootstrap/css/bootstrap.min.css`として参照されるアセットは、`/webjars/bootstrap/4.0.0-alpha.2/css/bootstrap.min.css`としてクラスパス上に置かれています。
