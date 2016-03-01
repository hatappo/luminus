Luminusは、デフォルトのバリデーション・ライブラリとして[Bouncer](https://github.com/leonardoborges/bouncer)を使います。Bouncerは、Clojure/Scriptライブラリであり、クライアントとサーバとの間でバリデーション・ロジックを共有することを可能にしてくれます。

Bouncerは、バリデーションを扱うための`bouncer.core/validate`と`bouncer.core/valid?`という関数を提供します。この2つの関数は、パラメータを保持したマップと種々のバリデータを受け取ります。

Bouncerは、最初から次のようなバリデータを用意しています。

* `required` args: `[v]` - 値が存在することを検証します
* `number` args: `[v]` - 値が数値であることを検証します
* `positive` args: `[v]` - 値が正の数値であることを検証します
* `member` args: `[v coll]` - 値が任意のコレクションに含まれるかどうかを検証します
* `custom` args: `[v pred]` - 独自のバリデータを使って値を検証します
* `every` args: `[coll pred]` - コレクションの全ての要素が任意の述語を満たすかをチェックします
* `matches` args: `[v regex]` - 値が与えられた正規表現を満たすかをチェックします
* `email` args: `[v]` - 値がEメールのアドレスかをチェックします
* `datetime` args: `[v & [format]]` - 値が、フォーマットを満たす日付（時刻）かをチェックします。フォーマットのしては任意です。
* `max-count` args: `[coll n]` - コレクションの要素数が多くともn個であることを検証します
* `min-count` args: `[coll n]` - コレクションの要素数が少なくともn個であることを検証します

バリデーションがどのように動作するかを見る前に、名前空間に`bouncer.validators`とともに`bouncer.core`を取り込みましょう。

```clojure
(ns myapp.home
  (:require
    ...
    [bouncer.core :as b]
    [bouncer.validators :as v]))
```

次に、パラメータを表すmapを定義します。

```clojure
(def user {:id nil :pass "secret"})
```

これで、いくつかのvalidatorsを使ってバリデーションができます。

```clojure
(b/valid? user
  :id v/required
  :pass v/required)
```

`valid?`関数は、検証したデータが有効か無効かを示す真偽値を返します。エラーの内容が見たい場合には、`valid?`はななく`validate`関数を使用しなければいけません。

```clojure
(b/validate user
  :id v/required
  :pass v/required)
```

この`validate`関数は、最初の要素がエラーのmapになったvectorを返します。2番目の要素は、元のパラメータのmapになります。ただし、
エラーの場合には元のパラメータのmapに`:bouncer.core/errors`キーが追加されます。

```clojure
[{:id ("id must be present")}
 {:bouncer.core/errors {:id ("id must be present")}
  :id nil
  :pass "secret"}]
```

バリデータをvectorに入れることで、1つの値に複数のバリデータを適用することができます。

```clojure
(b/validate user
  :id v/required
  :pass [v/required [v/min-count 8]])
```

`min-count`のような追加の引数を取るバリデータは、その引数とともにvectorに入れることに注意してください。そうすると、内部的にはそのバリデータの最初の引数として渡されます。

入れ子になったmapをバリデートすることもできます。

```clojure
(def person
  {:address
   {:unit 10
    :street nil
    :country "Canada"}})

(b/validate person
    [:address :street] v/required
    [:address :unit]   v/number
    [:address :phone]  [[v/matches #"^\d+$"]])

```

それから、`bouncer.validators/defvalidator`を使えば独自のバリデータが容易に作成できます。

```clojure
(v/defvalidator valid-password
  {:default-message-format "%s must be at least 7 characters long"}
  [p]
  (and p (> (count p) 7)))

(b/validate user
  :id v/required
  :pass valid-password)
```

より多くの例は、[プロジェクトの公式ページ](https://github.com/leonardoborges/bouncer)を参照してください。

----
# 訳注

* 述語(predicate): true/falseの値を返す関数。つまり真偽を判定する関数。
