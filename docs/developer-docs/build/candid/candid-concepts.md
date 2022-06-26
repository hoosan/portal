# Candid とは？

Candid は _インターフェース記述言語_ の一種です。 Candid の主な目的は、**Service**（Internet Computer 上で動作する **Canister スマートコントラクト** としてデプロイされたプログラム）の、公開インターフェースを記述することです。 Candid の大きな利点の一つは言語にとらわれないことであり、Motoko、Rust、JavaScript などの異なるプログラミング言語で書かれた Service やフロントエンドの相互運用を可能にしています。

Candid の典型的なインターフェースの記述は次のようなものです：

```candid
service counter : {
  add : (nat) -> ();
  subtract : (nat) -> ();
  get : () -> (int) query;
  subscribe : (func (int) -> ()) -> ();
}
```

この例では、`counter` という Service が、以下のパブリックメソッドから構成されていることを表しています。

- カウンターの値を変更する `add` と `subtract` メソッド。

- カウンターの現在の値を返す `get` メソッド。

- 別の関数を呼び出す `subscribe` メソッド。例えば、カウンターの値が変化するたびに通知用のコールバックメソッドを呼び出すことができます。

上で示した例のように、すべてのメソッドには、引数と返り値の型があります。 また、この例で示した `query` という表記のように、メソッドには Internet Computer 独自のアノテーションを含めることができます。

このようなシンプルなインターフェースを採用しているため、コマンドラインや Web ベースのフロントエンドを通して直接、 あるいは Rust プログラムや他のプログラミング言語やスクリプト言語からプログラム的に、`counter` サービスを操作することが可能です。

相互運用性に加え、既存のクライアントを壊さずに変更できる部分を正確に指定することで、Candid はサービスのインターフェースのアップグレードをサポートします。 例えば、既存のクライアントとの互換性を失うことなく、新しいオプションのパラメータを Service に安全に追加することができます。

## なぜ新しい IDL を作ったか？

一見、JSON や XML、Protobuf などの他の技術で十分だと思われるかもしれません。 しかしながら、Candid はこれらの他の技術にはない、ユニークな機能を提供しています。 Candid が Internet Computer での Dapps の開発に特に適している特徴は以下の通りです：

- JSON、XML、Protobuf などの多くの言語は、個々の値をバイトや文字にマッピングする方法のみを記述しています。 これらのデータ記述言語は、サービス全体を記述しているわけではありません。 これらの言語は、そのデータ型を利用するメソッドではなく、転送したいデータの型に焦点を当てているのです。

- Candid では、Candid の値をホスト言語の型や値に直接マッピングするように実装されています。 Candid では、開発者は、抽象的な `Candid` の値を構築したり分解したりしません。

- Candid は、Service とそのインターフェースを健全かつ合成的にアップグレードするためのルールを定義しています。

- Candid は本質的に高次の言語です。 Candid では、Service やメソッドへの参照など、単なるデータ以上のものを渡すことができます。 安全なアップグレードのための Candid のサポートは、そのような高次の使用が念頭にあります。

- Candid は、`query` アノテーションのような、Internet Computer 独自の機能をビルトインでサポートしています。

## Candid の型と値

Candid は強力な型付けシステムで、以下で示すように、ほとんどの用途をカバーする型のセットがあります：

- Unbounded（制限なし）整数型 (`nat`, `int`).

- Bounded（制限付き）整数型 (`nat8`,`nat16`, `nat32`, `nat64`, `int8`,`int16`, `int32`, `int64`).

- 浮動小数点型 (`float32`, `float64`).

- ブール型 (`bool`).

- テキストデータ(`text`)とバイナリデータ(`blob`)の型

- バリアントを含むコンテナ型 (`opt`, `vec`, `record`, `variant`).

- 参照型 (`service`, `func`, `principal`).

- `null`、`reserved`、`empty` の特殊な型。

全ての型について [リファレンス](../../../../references/candid-ref)の章で詳細に記述されています。

これらの型の背景にある哲学は、データの _構造_ を記述するのに十分であり、情報をエンコードしたり、渡したり、デコードすることができる一方で、表現を記述するのに必要な以上の _セマンティクス_ に関する制約を意図的に記述しないというものです。 例えば、数が偶数であること、ベクトルが固定長であること、ベクトルの要素がソートされていることなどを表現する方法はありません。

Candid がサポートする型によって、Motoko、Rust、JavaScript、その他の言語でコードを書いていても、それぞれのホスト言語に適した合理的な選択に基づき、データ型の自然なマッピングが可能になります。

## Candid Service の説明

Candid の型に慣れてきたら、Candid を使って Service を記述することができます。 Candid の Service 記述ファイル（`.DID` ファイル）は、自分で作成するか、Service の実装から自動生成するかのどちらでも構いません。

特定のホスト言語用の Service 記述ファイルを生成する方法を説明する前に、Service 記述ファイルの構造とその構成要素について例を挙げて詳しく見てみましょう。

公開メソッドを持たない Service に対する最もシンプルな Service 記述ファイルは以下のようになります：

```candid
service : {}
```

この Service はあまり便利ではないので、`ping` という簡単なメソッドを追加しましょう：

```candid
service : {
  ping : () -> ();
}
```

この例では，`ping` という単一の公開メソッドをサポートする Sercvice を記述しています。 メソッド名は任意の文字列で構いませんが、平文でない場合は引用符で囲んでください（`"method with spaces"`）。

メソッドは、引数と返り値の型の _シーケンス_ を宣言します。 この `ping` メソッドの場合、引数はなく、返り値もないので、引数と返り値の両方に空のシーケンス `()` が使用されます。

上の例はあまりにも単純なケースでしたので、もう少し複雑な `Service` の記述を考えてみましょう。 この Service は、`reverse` と `divMod` という２つのメソッドで構成されており、それぞれ以下のような引数と返り値の型のシーケンスを持っています：

```candid
service : {
  reverse : (text) -> (text);
  divMod : (dividend : nat, divisor : nat) -> (div : nat, mod : nat);
}
```

`reverse` メソッドは、`text` 型の値を引数として１つ受け取り、`text` 型の値を１つ返します。

`divMod` メソッドは、`nat` 型の２つの値を引数とし、返り値も同じく２つの `nat` 型の値となります。

## 引数と返り値の命名

前述の例では、`divMod` メソッドのシグネチャに引数と返り値の名前が含まれています。 メソッドの引数や返り値に名前を付ける目的はドキュメント化です。 使用する名前は、メソッドの型や渡される値を変更するものではありません。 その代わり、引数や返り値は名前とは関係なく、その _場所_ によって識別されます。

特に、Candid は以下のように型を変更したり、

```candid
  divMod : (dividend : nat, divisor : nat) -> (mod : nat, div : nat);
```

最初に `mod` を返すメソッドを期待する Service に上記の `divMod` を渡すことを妨げません。

これは、意味論的に関連性を持つ _レコード_ 型フィールドとは大きく異なります。

## 複雑な型の再利用

しばしば、Service 内の複数のメソッドが同じ複合型を参照することがあります。 その場合、その型に名前を付けて複数回再利用することができます。 例えば、以下のようになります：

```candid
type address = record {
  street : text;
  city : text;
  zip_code : nat;
  country : text;
};
service address_book : {
  set_address: (name : text, addr : address) -> ();
  get_address: (name : text) -> (opt address) query;
}
```

これらの型定義は、_既存の_ 型を単に省略しているだけで、新しい型を定義しているわけではありません。 関数のシグネチャで `address` を使おうが、レコードを書き出そうが関係ありません。 また、名前が異なっていても定義が同じである２つの略語は、同じ型を表しており交換可能です。言い換えれば、Candid は _構造的_ 型付けを使用しています。

## クエリメソッドの指定

全節の最後の例で、`get_address` メソッドに `query` アノテーションを使用していることにお気づきでしょうか。 例えば、以下のような形です：

```candid
service address_book : {
  set_address: (name : text, addr : address) -> ();
  get_address: (name : text) -> (opt address) query;
}
```

このアノテーションは、`get_address` メソッドが Internet Computer の **クエリコール** として呼び出されることを示しています。

[クエリメソッドとアップデートメソッド](../../../../concepts/canisters-code.md#query-and-update-methods) で説明したように、クエリはコンセンサスを介さずに Canister スマートコントラクトから情報を取得するための効率的な方法ですので、メソッドをクエリとして識別できることは、Internet Computer とやり取りする際に Candid を用いる重要なメリットの１つです。

## エンコードとデコード

Candid のポイントは、バイナリ形式にエンコードされた引数を渡し、特定の伝達手段（Internet Computer へのメッセージや Internet Computer 内のメッセージなど）で転送し、相手側でデコードすることで、Service のメソッドをシームレスに呼び出せるようにすることです。

Candid を使えば、このバイナリ形式の詳細を気にする必要はありません。

もしあなたが自分で Candid を _実装_ しようと思っているなら（例えば、新しいホスト言語をサポートするためなど）、[Candid の仕様](https://github.com/dfinity/candid)で詳細を参照してください。 ただし、自分で実装しない場合であっても、Candid のフォーマットのいくつかの側面は知っておく価値があります。

- Candid のバイナリ形式は、`DIDL...` (16 進数では、`4449444c...`) から始まります。 もし、低レベルのログ出力でこれを見ることがあれば、それは Candid でエンコードされた値を見ている可能性が高いです。

- Candid のバイナリ形式は、メソッドの引数と返り値が型のシーケンスとなっているため、常に値の _シーケンス_ をエンコードします。

- バイナリ形式は非常にコンパクトです。125,000 個の要素を持つ `(vec nat64)` は 1,000,007 バイトです。

- バイナリは自己記述式であり、含まれている値の（凝縮された）型に関する記述を含んでいます。 これにより、メッセージが互換性のない型として送られてきたかどうかを、受信側が検出することができます。

- 受信側が期待する型の通りに送信側が引数をシリアル化している限りにおいて、デシリアライズは成功します。

## Service のアップグレード

Service は時間とともに進化し、新しいメソッドが追加されたり、既存のメソッドがより多くのデータを返したり、追加の引数を要求したりします。 通常、開発者は既存のクライアントを壊すことなくアップグレードを成功させたいのです。

Candid は、新しい Service の型が、これまでのインターフェース記述を使用している他のすべての Canister といつまで通信できるのかを示す明確なルールを定義することで、このようなアップグレードをサポートしています。 このための基本となる形式は、_サブタイプ_ です。

Service は次の場合に安全にアップグレードすることができます：

- 新しいメソッドを追加する場合には、安全にアップグレードされます。

- 既存のメソッドは、追加の値を返すことができます。つまり、一連の返り値の型を拡張することができます。古いクライアントは、追加された値を単に無視します。

- 既存のメソッドは、その引数のリストを短くすることができます。古いクライアントは元のメソッドにあった引数を送ることができますが、それらは無視されます。

- 既存のメソッドは、オプショナルの引数（`opt ...` 型）を用いることで引数のリストを拡張することができます。その引数を渡さない古いクライアントからのメッセージを読み込む際には、`null` 値として推定されます。

- 既存の引数の型は、以前の型の _スーパータイプ_ に限り、_変更_ することができます。

- 既存の返り値の型は、以前の型の _サブタイプ_ に限り、_変更_ することができます。

ある型のスーパータイプとサブタイプに関する情報は、その型に対応する [リファレンス](../../../../references/candid-ref) セクションを参照してください。

Service をどのようにアップグレードしていくのか、具体的な例を見てみましょう。 以下のような API を持つ Service を考えてみましょう：

```candid
service counter : {
  add : (nat) -> ();
  subtract : (nat) -> ();
  get : () -> (int) query;
  subscribe : (func (int) -> ()) -> ();
}
```

この Service は、以下のインターフェースにアップグレードすることができます：

```candid
type timestamp = nat;
service counter : {
  set : (nat) -> ();
  add : (int) -> (new_val : nat);
  subtract : (nat, trap_on_underflow : opt bool) -> (new_val : nat);
  get : () -> (nat, last_change : timestamp) query;
  subscribe : (func (nat) -> (unregister : opt bool)) -> ();
}
```

## Candid のテキスト値

Candid の主な目的は、Motoko、Rust、JavaScript などのホスト言語で書かれたプログラムを Internet Computer に接続することです。 そのため、ほとんどの場合、プログラムのデータを Candid の値として扱う必要はありません。 その代わりに、慣れ親しんだ JavaScript のようなホスト言語を使って作業し、Rust や Motoko で書かれた Canister のスマートコントラクトに値を透過的に転送することを Candid に任せることができます。 値を受け取った Canister は、その値を Rust や Motoko のネイティブな値として扱います。

しかしながら、ログを取ったり、デバッグしたり、コマンドラインで Service を操作するときなど、Candid の値を人間が読める形で直接見ることができると便利な場合があります。 このような場合には、Candid の値に _テキスト表現_ を使用することができます。

シンタックスは Candid の型と似ています。 例えば、Candid の値を表す典型的なテキスト表現は次のようなものです：

```candid
(record {
  first_name = "John";
  last_name = "Doe";
  age = 14;
  membership_status = variant { active };
  email_addresses =
    vec { "john@doe.com"; "john.doe@example.com" };
})
```

<div class="note">

Candid の _binary_ 形式には実際のフィールド名は含まれておらず、単に数値の _hash_ が含まれてるだけです。 そのため、求められている型を知らずに値をきれいに出力しても、レコードやバリアントのフィールド名は含まれません。上記の値は次のように表示されます：

```candid
(record {
   4846783 = 14;
   456245371 = variant {373703110};
   1443915007 = vec {"john@doe.com"; "john.doe@example.com"};
   2797692922 = "John"; 3046132756 = "Doe"
})
```

</div>

## Service 記述ファイルの生成

[前のセクションで](#candid-service-の説明)、Candid の Service 記述ファイルを自分で書く方法を学びました。 しかし多くの場合、それは必要ですらありません！ Service を実装する際に使用する言語によっては、コードから Candid Serivice の記述ファイルを生成することができます。

例えば Motoko では、Canister スマートコントラクトを以下のように書くことができます：

```motoko
actor {
  var v : Int = 0;
  public func add(d : Nat) : async () { v += d; };
  public func subtract(d : Nat) : async () { v -= d; };
  public query func get() : async Int { v };
  public func subscribe(handler : func (Int) -> async ()) { … }
}
```

このプログラムをコンパイルすると，{proglang} コンパイラが上記のインターフェイスの Candid Service 記述ファイルを自動的に生成します。

Rust や C などの他の言語でも、その言語のネイティブな型を使って Service を開発することができます。例えば、Rust のネイティブな型を使うことができます。 しかし、Rust のような言語で Service を開発した後、Candid で Service 記述ファイルを自動的に生成する方法は今のところありません。

そのため、Rust や C で Service 用のプログラムを書く場合は、[Candid の仕様](https://github.com/dfinity/candid)に記載されている規約に従って、Candid のインターフェース記述ファイルを手動で作成する必要があります。

Rust プログラムの Candid Service 記述ファイルの書き方の例については、[Rust CDK の例](https://github.com/dfinity/cdk-rs/tree/next/examples)と[Rust のチュートリアル](../rust/rust-intro)を参照してください。

使用するホスト言語に関わらず、ホスト言語の型と Candid の型の対応関係を知っておくことは重要です。 リファレンスの[サポートされている型](../../../references/candid-ref.md) の章では、Motoko、Rust、JavaScript の Candid 型との対応関係が説明されています。

<!--
# What is Candid?

Candid is an *interface description language*. Its primary purpose is to describe the public interface of a **service**, usually in the form of a program deployed as a **canister** that runs on the Internet Computer. One of the key benefits of Candid is that it is language-agnostic, and allows inter-operation between services and frontends written in different programming languages, including Motoko, Rust, and JavaScript.

A typical interface description in Candid might look like this:

``` candid
service counter : {
  add : (nat) -> ();
  subtract : (nat) -> ();
  get : () -> (int) query;
  subscribe : (func (int) -> ()) -> ();
}
```

In this example, the described service—`counter`—consists of the following public methods. - The `add` and `subtract` methods change the value of a counter. - The `get` method reads the current value of a counter. - The `subscribe` method can be used to invoke another function, for example, to invoke a notification callback method each time the counter value changes.

As this example illustrates, every method has a sequence of argument and result types. A method can also include annotations—like the `query` notation shown in this example—that are specific to the Internet Computer.

Given this simple interface description, it is possible for you to interact with this `counter` service directly from the command line or through a web-based frontend or programmatically from a Rust program or through another programming or scripting language.

In addition to interoperability, Candid supports the evolution of service interfaces by precisely specifying the changes that can be made without breaking existing clients. For example, you can safely add new optional parameters to a service without losing compatibility for existing clients.

## Why create a new IDL?

At first glance, you might think that other technologies, such as JSON, XML, or Protobuf, would suffice. However, Candid provides a unique combination of features that are not found in these other technologies. The features that make Candid particularly well-suited for developing dapps for the Internet Computer include the following:

-   Many languages like JSON, XML, and Protobuf only describe how to map individual values to bytes or characters. These data description languages do not describe services as a whole. These languages focus on the data types you want to transfer instead of the methods that make use of those data types.

-   Candid implementations map the Candid value directly to types and values of the host language. With Candid, developers do not construct or deconstruct some abstract `Candid` value.

-   Candid defines rules for how services and their interface can be upgraded in a sound and compositional way.

-   Candid is inherently a higher-order language. With Candid, you can pass more than plain data, including references to services and methods. Candid support for safe upgrades takes such higher-order use into account.

-   Candid has built-in support for specific Internet Computer features, such as the `query` annotation.

## Candid types and values

Candid is a strongly typed system with a set of types that canonically cover most uses. It has:

-   Unbounded integral number types (`nat`, `int`).

-   Bounded integral number (`nat8`,`nat16`, `nat32`, `nat64`, `int8`,`int16`, `int32`, `int64`).

-   Floating point types (`float32`, `float64`).

-   The Boolean type (`bool`).

-   Types for textual (`text`) and binary (`blob`) data.

-   Container types, including variants (`opt`, `vec`, `record`, `variant`).

-   Reference types (`service`, `func`, `principal`).

-   The special `null`, `reserved` and `empty` types.

All types are described in detail in the [Reference](../../../references/candid-ref.md) section.

The philosophy behind this set of types is that they are sufficient to describe the *structure* of data, so that information can be encoded, passed around and decoded, but intentionally do not describe *semantic* constraints beyond what’s needed to describe the representation. For example, there’s no way to express that a number should be even, that a vector has a certain length, or that the elements of a vector are sorted.

Candid supports this set of types to allow a natural mapping of data types based on reasonable, canonical choices suitable for each host language, whether you are writing your code in Motoko, Rust, JavaScript, or some other language.

## Candid service descriptions

Once you are familiar with the Candid types, you can use them to describe a service. A Candid service description file—a `.DID` file—can either be written by hand or generated from a service implementation.

Before you explore how to generate service descriptions for a specific host language, let’s take a closer look at the structure of a sample service description and its constituent parts.

The simplest service description specifies a service with no public methods and would look like this:

``` candid
service : {}
```

This service is not very useful, so let’s add a simple `ping` method:

``` candid
service : {
  ping : () -> ();
}
```

This example describes a service that supports a single public method called `ping`. Method names can be arbitrary strings, and you can quote them (`"method with spaces"`) if they are not plain identifiers.

Methods declare a *sequence* of arguments and result types. In the case of this `ping` method, no arguments are passed and no results are returned, so the empty sequence \`() \` is used for both arguments and results.

Now that you’ve seen the simplest case, let’s consider a slightly more complex service description. This service consists of two methods—`reverse` and `divMod`—and each method include a sequence of argument and result types:

``` candid
service : {
  reverse : (text) -> (text);
  divMod : (dividend : nat, divisor : nat) -> (div : nat, mod : nat);
}
```

The method `reverse` expects a single parameter of type `text` and returns one value of type `text`.

The method `divMod` expects and returns two values, all of type `nat`.

## Naming arguments and results

In the previous example, the signature for the `divMod` method includes names for the argument and result values. Naming the arguments or results for a method is purely for documentation purposes. The name you use does not change the method’s type or the values being passed. Instead, arguments and results are identified by their *position*, independent of the name.

In particular, Candid does not prevent you from changing the type to:

``` candid
  divMod : (dividend : nat, divisor : nat) -> (mod : nat, div : nat);
```

or passing the above `divMod` to a service expecting a method that returns `mod` first.

This is thus very different from named *record* fields, which are semantically relevant.

## Reusing complex types

Often, multiple methods in a service may refer to the same complex type. In that case, the type can be named and reused multiple times. For example:

``` candid
type address = record {
  street : text;
  city : text;
  zip_code : nat;
  country : text;
};
service address_book : {
  set_address: (name : text, addr : address) -> ();
  get_address: (name : text) -> (opt address) query;
}
```

These type definitions merely abbreviate an *existing* type, they do not define a new type. It does not matter whether you use `address` in the function signature, or write out the records. Also, two abbreviations with different names but equivalent definitions, describe the same type and are interchangeable. In other words, Candid uses *structural* typing.

## Specifying a query method

In the last example, you might have noticed the use of the `query` annotation for the `get_address` method. For example:

``` candid
service address_book : {
  set_address: (name : text, addr : address) -> ();
  get_address: (name : text) -> (opt address) query;
}
```

This annotation indicates that the `get_address` method can be invoked as an IC **query call**. As discussed in [Query and update methods](../../../concepts/canisters-code.md#query-and-update-methods), a query provides an efficient way to retrieve information from a canister without going through consensus, so being able to identify a method as a query is one of the key benefits of using Candid to interact with the IC.

## Encoding and decoding

The point of Candid is to allow seamless invocation of service methods, passing arguments encoded to a binary format and transferred by an underlying transportation method (such as messages into or within the Internet Computer), and decoded on the other side.

As a Candid user, you do not have to worry about the details of this binary format. If you plan to *implement* Candid yourself (for example, to support a new host language), you can consult the [Candid specification](https://github.com/dfinity/candid) for details. However, some aspects of the format are worth knowing:

-   The Candid binary format starts with `DIDL…` (or, in hex, `4449444c…`). If you see this in some low-level log output, you are very likely observing a Candid-encoded value.

-   The Candid binary format always encodes *sequences* of values, because methods parameters and results are sequences of types.

-   The binary format is quite compact. A `(vec nat64)` with 125 000 entries takes 1 000 007 bytes.

-   The binary is self-describing, and includes a (condensed) type description of type of the values therein. This allows the receiving side to detect if a message was sent at as a different, incompatible type.

-   As long as the sender serializes the arguments as the type that the receiving side expects, deserialization will succeed.

## Service upgrades

Services evolve over time: They gain new methods, existing methods return more data, or expect additional arguments. Usually, developers want to do that without breaking existing clients.

Candid supports such evolution by defining precise rules that indicate when the new service type will still be able to communicate with all other parties that are using the previous interface description. The underlying formalism is that of *subtyping*.

Services can safely evolve in the following ways:

-   New methods can be added.

-   Existing methods can return additional values, that is, the sequence of result types can be extended. Old clients will simply ignore additional values.

-   Existing methods can shorten their parameter list. Old clients may still send the extra arguments, but they will be ignored.

-   Existing methods can extend their parameter list with optional arguments (type `opt …`). When reading messages from old clients, who do not pass that argument, a `null` value is assumed.

-   Existing parameter types may be *changed*, but only to a *supertype* of the previous type.

-   Existing result types may be *changed*, but only to a *subtype* of the previous type.

For information about the supertypes and subtypes of a given type, see the corresponding [reference](../../../references/candid-ref.md) section for that type.

Let’s look at a concrete example of how a service might evolve. Consider a service with the following API:

``` candid
service counter : {
  add : (nat) -> ();
  subtract : (nat) -> ();
  get : () -> (int) query;
  subscribe : (func (int) -> ()) -> ();
}
```

This service can evolve to the following interface:

``` candid
type timestamp = nat;
service counter : {
  set : (nat) -> ();
  add : (int) -> (new_val : nat);
  subtract : (nat, trap_on_underflow : opt bool) -> (new_val : nat);
  get : () -> (nat, last_change : timestamp) query;
  subscribe : (func (nat) -> (unregister : opt bool)) -> ();
}
```

## Candid textual values

The main purpose of Candid is to connect programs written in some host language—Motoko, Rust, or JavaScript, for example—with the IC. In most cases, therefore, you do not have to deal with your program data as Candid values. Instead, you work with a host language like JavaScript using familiar JavaScript values then rely on Candid to transparently transport those values to a canister written in Rust or Motoko. The canister receiving the values treats them as native Rust or Motoko values.

However, there are some cases—for example, when logging, debugging, or interacting with a service on the command-line—where it is useful to see the Candid values directly in a human-readable form. In these scenarios, you can use the *textual presentation* for Candid values.

The syntax is similar to that for Candid types. For example, a typical textual presentation for a Candid value might look like this:

``` candid
(record {
  first_name = "John";
  last_name = "Doe";
  age = 14;
  membership_status = variant { active };
  email_addresses =
    vec { "john@doe.com"; "john.doe@example.com" };
})
```

<div class="note">

The Candid *binary* format does not include the actual field names, merely numeric *hashes*. So pretty-printing such a value without knowledge of the expected type will not include the field names of records and variants. The above value might then be printed as follows:

``` candid
(record {
   4846783 = 14;
   456245371 = variant {373703110};
   1443915007 = vec {"john@doe.com"; "john.doe@example.com"};
   2797692922 = "John"; 3046132756 = "Doe"
})
```

</div>

## Generating service descriptions

In the [section above](#candid-service-descriptions), you learned how to write a Candid service description from scratch. But often, that is not even needed! Depending on the language you use to implement your service, you can get the Candid service description generated from your code.

For example, in Motoko, you can write a canister like this:

``` motoko
actor {
  var v : Int = 0;
  public func add(d : Nat) : async () { v += d; };
  public func subtract(d : Nat) : async () { v -= d; };
  public query func get() : async Int { v };
  public func subscribe(handler : func (Int) -> async ()) { … }
}
```

When you compile this program, the Motoko compiler automatically generates a Candid service description file with the interface shown above.

In other languages, like Rust or C, you can still develop your service using the types that are native to that language, for example, using native Rust types. After you develop a service in a language like Rust, however, there’s currently no way to automatically generate the service description in Candid. Therefore, if you write a program for a service in Rust or C, you need to write the Candid interface description manually following the conventions described in the [Candid specification](https://github.com/dfinity/candid).

For examples of how to write Candid service descriptions for Rust programs, see the [Rust CDK examples](https://github.com/dfinity/cdk-rs/tree/next/examples) and the [Rust tutorials](../cdks/cdk-rs-dfinity/index.md).

Regardless of the host language you use, it is important to know the mapping between host language types and Candid types. In the [Supported types](../../../references/candid-ref.md) reference section, you’ll find Candid type mapping described for Motoko, Rust, and JavaScript.
