# Candid リファレンス

このドキュメントは、 Candid のサポートしている型に関する詳細なリファレンス情報と、Candid の仕様と Candid Rust crate のドキュメントへのリンクを提供しています。

- [サポートされている型](#サポートされている型)

- [Candid の仕様](https://github.com/dfinity/candid)

- [Candid Rust crate](https://docs.rs/candid)

Candid を使い始める前にさらなる情報をお探しの場合は、以下の関連リソースをご確認ください。

- [How Candid provides a common language for application interfaces (video)](https://www.youtube.com/watch?v=O2KaWRtsqHg)

## サポートされている型

この章では、Candid でサポートされているすべての型をリストアップしています。 それぞれの型について、以下の情報が記載されています：

- 型の構文と、型のテキスト表現の構文。

- それぞれの型に対する Candid のアップグレードに関するルールは、型の _サブタイプ_ と _スーパータイプ_ によって決まります。

- それぞれの型に対応する Rust、Motoko、Javascript の型。

既存メソッドの _返り値_ の型は以前の型のサブタイプに変更できます。 既存メソッドの _引数_ の型は以前の型のスーパータイプに変更できます。

このリファレンスには、それぞれの型に関連する特定のサブタイプとスーパータイプのみが記載されていることに注意してください。 どの型にも適用可能なサブタイプやスーパータイプは記載していません。 たとえば、`empty` は他のどの型のサブタイプにもなりうるので、このリファレンスではサブタイプとして挙げていません。 同様に、`reserved` と `opt t` 型 は、任意の型のスーパータイプであるため、特定の型のスーパータイプとして挙げていません。 `empty`、`reserved`、`opt t` 型のサブタイプ化のルールに関する詳細については、以下の節を参照してください。

- [`opt t`](#opt-t-型)

- [`reserved`](#reserved-型)

- [`empty`](#empty-型)

## text 型

`text` 型は、人が読めるテキストが必要な場面で使われます。より正確には、unicode の符号位置のシーケンスです（サロゲート部分を除く）。

型の構文  
`text`

テキスト表現の構文

```candid
""
"Hello"
"Escaped characters: \n \r \t \\ \" \'"
"Unicode escapes: \u{2603} is ☃ and \u{221E} is ∞"
"Raw bytes (must be utf8): \E2\98\83 is also ☃"
```

対応する Motoko の型  
`Text`

対応する Rust の型  
`String` または `&str`

対応する JavaScript の値  
`"String"`

## blob 型

`blob` 型は、バイナリデータ（バイトのシーケンス）に用いることができます。 `blob` 型を用いて書かれたインターフェースは、`vec nat8` を用いて書かれたインターフェースと互換性があります。

型の構文  
`blob`

テキスト表現の構文  
`blob <text>`

ここで、`<text>` は、すべての文字が UTF8 エンコーディングで表現された文字列リテラルと任意のバイトシーケンス(`"\CA\FF\FE"`)を表します。

Text 型に関して詳しく知りたい場合は、[Text 型](#text-型)を参照してください。

サブタイプ  
`vec nat8` および `vec nat8` の全てのサブタイプ。

スーパータイプ  
`vec nat8` および `vec nat8` の全てのスーパータイプ。

対応する Motoko の型  
`Blob`

対応する Rust の型  
`Vec<u8>` または `&[u8]`

対応する JavaScript の値  
`[ 1, 2, 3, 4, ... ]`

## nat 型

`nat` 型は、すべての（非負の）自然数を含みます。 値の大きさに制限はなく、任意の大きな数字を表すことができます。 通信時のエンコーディングは LEB128 なので、小さな数字も効率よく表現できます。

型の構文  
`nat`

テキスト表現の構文

```candid
1234
1_000_000
0xDEAD_BEEF
```

スーパータイプ  
`int`

対応する Motoko の型  
`Nat`

対応する Rust の型  
`candid::Nat` または `u128`

対応する JavaScript の値  
`BigInt(10000)` または `10000n`

## int 型

`int` 型はすべての整数を含みます。 大きさに制限がなく、任意の大小の数値を表現することができます。 通信時のエンコーディングは SLEB128 なので、小さな数字も効率的に表現できます。

型の構文  
`int`

テキスト表現の構文

```candid
1234
-1234
+1234
1_000_000
-1_000_000
+1_000_000
0xDEAD_BEEF
-0xDEAD_BEEF
+0xDEAD_BEEF
```

サブタイプ  
`nat`

対応する Motoko の型  
`Int`

対応する Rust の型  
`candid::Int` または `i128`

対応する JavaScript の値  
`BigInt(-10000)` または `-10000n`

## natN 型と intN 型

`nat8`、`nat16`、`nat32`、`nat64`、`int8`、`int16`、`int32`、`int64` の型は、そのビット数の表現を持つ数値を表し、より低レベルなインターフェースで使用することができます。

`natN` の範囲は `{0 …​. 2^N-1}` であり、`intN` の範囲は `-2^(N-1) …​ 2^(N-1)-1` となります。

通信時の表現は、ちょうどその長さのビット数になります。そのため、小さな値に対しては、`nat64` よりも `nat` の方が容量の効率が良いです。

型の構文  
`nat8`, `nat16`, `nat32`, `nat64`, `int8`, `int16`, `int32` または `int64`

テキスト表現の構文  
`nat8`, `nat16`, `nat32`, `nat64` は `nat` と同じです。

`int8`, `int16`, `int32`, `int64` は `int` と同じです。

型アノテーションを使って、異なる整数型を区別することができます。

```candid
100 : nat8
-100 : int8
(42 : nat64)
```

対応する Motoko の型  
`natN` はデフォルトでは `NatN` に翻訳されますが、必要に応じて `WordN` にも翻訳されます。

`intN` は `IntN` に翻訳されます。

対応する Rust の型  
同サイズの符号付き整数と符号なし整数に対応します。

| ビット長 | 符号付き | 符号なし |
| -------- | -------- | -------- |
| 8-bit    | i8       | u8       |
| 16-bit   | i16      | u16      |
| 32-bit   | i32      | u32      |
| 64-bit   | i64      | u64      |

対応する JavaScript の値  
8-bit, 16-bit, 32-bit は number 型に翻訳されます。

`int64` と `nat64` は JavaScript の `BigInt` プリミティブに翻訳されます。

## float32 型と float64 型

`float32` 型および `float64` 型は，IEEE 754 の浮動小数点数を、単精度（32 ビット）および倍精度（64 ビット）で表したものです。

型の構文  
`float32`, `float64`

テキスト表現の構文  
`int` と同じ構文で、次のように浮動小数点リテラルが加わります：

```candid
1245.678
+1245.678
-1_000_000.000_001
34e10
34E+10
34e-10
0xDEAD.BEEF
0xDEAD.BEEFP-10
0xDEAD.BEEFp+10
```

対応する Motoko の型  
`float64` は `Float` に対応します。

`float32` は、現在、Motoko での表現はありません。`float32` を使った Candid インターフェースは、Motoko のプログラムからは生成できませんし、利用することもできません。

対応する Rust の型  
`f32`, `f64`

対応する JavaScript の値  
float number

## bool 型

`bool` 型は論理値を示すデータ型で、`true` または `false` の値のみを持つことができます。

型の構文  
`bool`

テキスト表現の構文  
`true`, `false`

対応する Motoko の型  
`Bool`

対応する Rust の型  
`bool`

対応する JavaScript の値  
`true`, `false`

## null 型

`null` 型は値 `null` の型であり、全ての `opt t` 型のサブタイプです。また、[バリアント](#variant--n--t---型)を使用して列挙型をモデル化する際に慣例的に使用されます。

型の構文  
`null`

テキスト表現の構文  
`null`

スーパータイプ  
全ての `opt t` 型。

対応する Motoko の型  
`Null`

対応する Rust の型  
`()`

対応する JavaScript の値  
`null`

## vec t 型

`vec` 型はベクター（シーケンス、リスト、配列）を表します。 `vec t` 型の値は、`t` 型の 0 個以上の値のシーケンスを含みます。

型の構文  
`vec bool`, `vec nat8`, `vec vec text` など。

テキスト表現の構文  
``` candid
vec {}
vec { "john@doe.com"; "john.doe@example.com" };
```

サブタイプ

- `t` が `t'` のサブタイプであるときはいつでも、`vec t` は `vec t'` のサブタイプです。

- `blob` は `vec nat8` のサブタイプです。

スーパータイプ

- `t` が `t'` のスーパータイプであるときはいつでも、`vec t` は `vec t'` のスーパータイプです。

- `blob` は `vec nat8` のスーパータイプです。

対応する Motoko の型  
`[T]` となります。ここで、Motoko 型の `T` は `t` に対応しています。

対応する Rust の型  
`Vec<T>` または `&[T]` となります。ここで、Rust 型の `T` は `t` に対応しています。

`vec t` は `BTreeSet` または `HashSet` に翻訳されます。

`vec record { KeyType; ValueType }` は、`BTreeMap` または `HashMap` に翻訳されます。

対応する JavaScript の値  
`Array` 例えば `[ "text", "text2", …​ ]`

## opt t 型

`opt t` 型は、`t` 型のすべての値と、特殊な値である `null` を含みます。 これは、ある値が任意であることを表現するのに使われます。つまり、データは `t` 型の値として存在するかもしれないし、`null` という値として存在しないかもしれない、ということです。

`opt` 型は入れ子にすることができ（例：`opt opt text`）、値 `null` と `opt null` は別の値です。

`opt` 型は、Candid インターフェース のアップグレードにおいて重要な役割を果たしており、以下のような特別なサブタイプのルールを持っています。

型の構文  
`opt bool`, `opt nat8`, `opt opt text` など。

テキスト表現の構文  
``` candid
null
opt true
opt 8
opt null
opt opt "test"
```

サブタイプ  
`opt` を使ったサブタイプの規範的なルールは次の通りです：

- `t` が `t'` のサブタイプであるときはいつでも、`opt t` は `opt t'` のサブタイプです。

- `null` は `opt t'` のサブタイプです。

- `t` は `opt t` のサブタイプです（`t` 自体が `null` でない限り、`opt …​` または `reserved` ）。

加えて、アップグレードや上位のサービスに関する技術的な理由から、 _every_ 型は `opt t` のサブタイプであり、型が一致しない場合には `null` が生成されます。ただし、ユーザーはこのルールを直接利用しないようにしてください。

スーパータイプ

- `t` が `t'` のスーパータイプであるとき、`opt t` は `opt t'` のスーパータイプです。

対応する Motoko の型  
`?T` となります。ここで、Motoko 型の `T` が `t` に対応しています。

対応する Rust の型  
`Option<T>` となります。ここで、Rust 型の `T` が `t` に対応しています。

対応する JavaScript の値  
`null` は `[]` に翻訳されます。

`opt 8` は `[8]` に翻訳されます。

`opt opt "test"` は `[["test"]]` に翻訳されます。

## record { n : t, … } 型

`record` 型はラベル付けされた値の集まりです。例えば、以下のコードはテキストフィールドの `street`、`city`、`country` と数値フィールドの `zip_code` を持つ record の型に `address` という名前を与えています。

``` candid
type address = record {
  street : text;
  city : text;
  zip_code : nat;
  country : text;
};
```


record 型宣言のフィールドの順序は重要ではありません。 各フィールドは異なる型を持つことができます（同じ型のみを持つことができる vector とは異なります）。 record フィールドのラベルは、以下の例のように 32 ビットの自然数にすることもできます。

``` candid
type address2 = record {
  288167939 : text;
  1103114667 : text;
  220614283 : nat;
  492419670 : text;
};
```

実際のところテキストラベルはその _ハッシュ値_ として扱われますし、さらに言えば `address` と `address2` は Candid にとって同じ型です。

ラベルを省略すると、Candid は自動的に順次昇順のラベルを割り当てます。この挙動により，以下のような短縮された構文になり、通常ペアやタプルを表現するのに使われます。`record { text; text; opt bool }` は、`record { 0 : text; 1: text; 2: opt bool }` と同等です。

型の構文  
``` candid
record {}
record { first_name : text; second_name : text }
record { "name with spaces" : nat; "unicode, too: ☃" : bool }
record { text; text; opt bool }
```

テキスト表現の構文  
``` candid
record {}
record { first_name = "John"; second_name = "Doe" }
record { "name with spaces" = 42; "unicode, too: ☃" = true }
record { "a"; "tuple"; null }
```

サブタイプ  
record のサブタイプとは、（任意のタイプの）フィールドが追加されたり、フィールドの型がサブタイプに変更されたり、選択型のフィールドが削除されたりした record 型のことです。ただし、メソッドの返り値で選択型のフィールドを削除するのはバッドプラクティスです。フィールドの型を `opt empty` に変更することで、そのフィールドがもう使われていないことを示すことができます。

例えば、次のような record を返す関数があったとします：

``` candid
record {
  first_name : text; middle_name : opt text; second_name : text; score : int
}
```

上の record は、次のような record に更新することができます：

``` candid
record {
  first_name : text; middle_name : opt empty; second_name : text; score : nat; country : text
}
```

ここでは、`middle_name` フィールドを非推奨とし、`score` の型を変更し、`country` フィールドを追加しています。

スーパータイプ  
record のスーパータイプとは、一部のフィールドが削除された record 型、一部のフィールドのタイプがスーパータイプに変更された record 型、または選択型のフィールドが追加された record 型のことです。

後者は、引数の record を追加フィールドで拡張することができるものです。古いインターフェースを使用しているクライアントは、 record にフィールドを含めることができず、アップグレードされたサービスで期待される `null` としてデコードされます。

例えば、レコード 型を期待する関数があるとします。

``` candid
record { first_name : text; second_name : text; score : nat }
```

以下の record を受け取る関数に更新することができます。

``` candid
record { first_name : text; score: int; country : opt text }
```

対応する Motoko の型  
record 型がタプル（例えば、0 から始まる連続したラベル）を参照している場合は、Motoko のタプル型（例えば `(T1, T2, T3)`）が使用されます。それ以外の場合は、Motoko の record `({ first_name :Text, second_name : Text })` が使用されます。

フィールド名が Motoko の予約語の場合は、アンダースコア が付加されます。つまり、`record { if : bool }` は、`{ if_ : Bool }` となります。

フィールド名が Motoko の有効な識別子でない場合は、代わりに _フィールド_ のハッシュが使われます。例えば、`record { ☃ : bool }` は `{ 11272781 : Boolean }` となります。

対応する Rust の型  
`derive(CandidType, Deserialize)]` というトレイトを持つ、ユーザ定義の `構造体` となります。

フィールド名を変更するには、`#[serde(rename = "DifferentFieldName")]` 属性を使用します。

record 型がタプルの場合は、`(T1, T2, T3)` のようなタプル型に変換されます。

対応する JavaScript の値  
record 型がタプルの場合、配列に変換されます。例えば、`["Candid", 42]` のようになります。

それ以外の場合は、record オブジェクトに翻訳されます。例えば、`{ "first name": "Candid", age: 42 }` のようになります.

フィールド名がハッシュの場合は、フィールド名として `_hash_` を使用します。例えば、`{ _1_: 42, "1": "test" }` のようになります。

## variant { n : t, … } 型

`variant` 型は、定義された値の組み合わせ（あるいは _タグ_）のうちの 1 つの値を表します。つまり、以下の variant 型は、dot、circle（半径が与えられる）、rectangle（寸法が与えられる）、吹き出し（テキストが与えられる）のいずれかです。なお、吹き出しは、ユニコードのラベル(💬)の使用が可能であることを例示しています。

``` candid
type shape = variant {
  dot : null;
  circle : float64;
  rectangle : record { width : float64; height : float64 };
  "💬" : text;
};
```

`variant` 型のタグは、record 型のラベルと同様、実際には数字であり、文字列のタグはそのハッシュ値を指します。

しばしば、タグの一部（または全部）がデータを持たないことがあります。このような場合、上記の `dot` のように、`null` 型を使用するのが慣例です。実際、Candid はこのような使い方を推奨しており、variant では `: null` 型のアノテーションを省略することができます。つまり、

``` candid
type season = variant { spring; summer; fall; winter }
```

は以下と等価であり、

``` candid
type season = variant {
  spring : null; summer: null; fall: null; winter : null
}
```

となります。これは列挙を表現するのに使われます。

`variant {}` 型は構文上問題ありませんが、値を持っていません。値がないことを意図するのであれば、[`empty` 型](#empty-型)の方が適切かもしれません。

型の構文  
``` candid
variant {}
variant { ok : nat; error : text }
variant { "name with spaces" : nat; "unicode, too: ☃" : bool }
variant { spring; summer; fall; winter }
```

テキスト表現の構文  
``` candid
variant { ok = 42 }
variant { "unicode, too: ☃" = true }
variant { fall }
```

サブタイプ  
variant 型のサブタイプは、一部のタグを削除し、一部のタグの型をサブタイプに変更した variant 型です。

メソッドの返り値の variant に新しいタグを _追加_ できるようにしたい場合、variant 自体が `opt …​` でラップされていれば可能です。これには事前の計画が必要です。インターフェースを設計する際には、次のように書く代わりに：

``` candid
service {
  get_member_status (member_id : nat) -> (variant {active; expired});
}
```

以下のように書くのが良いでしょう：

``` candid
service {
  get_member_status (member_id : nat) -> (opt variant {active; expired});
}
```

このようにすることで、後に `名誉` 会員ステータスを追加する必要が生じた場合に、ステータスのリストを拡張することができます。古いクライアントは未知のフィールドを `null` として受け取ります。

スーパータイプ  
variant 型のスーパータイプは、タグが追加された variant です。一部のタグの型がスーパータイプに変更されている場合もあります。

対応する Motoko の型  
variant 型は、以下のように Motoko の variant 型として表現されます：

``` motoko
type Shape = {
  #dot : ();
  #circle : Float;
  #rectangle : { width : Float; height : Float };
  #_2669435721_ : Text;
};
```

列挙型を variant としてモデル化する際、Candid と Motoko それぞれの慣例の対応付けを行う必要があるため、タグの型が `null` の場合は Motoko では `()` に対応することに注意してください。

対応する Rust の型  
`#[derive(CandidType, Deserialize)]` トレイトを持つユーザー定義の `enum` となります。

フィールド名を変更するには、`#[serde(rename = "DifferentFieldName")]` 属性を使用することができます。

対応する JavaScript の値  
1 つの要素を持つ record オブジェクトとなります。例えば、`{ dot: null }` のようになります。

フィールド名がハッシュ値の場合には、フィールド名として `_hash_` を用います。例えば、`{ _2669435721_: "test" }` のようになります。

## func (…) → (…) 型

Candid は、上位のユースケースをサポートするように設計されており、あるサービスが他のサービスやそのメソッドへの参照を受け取ったり、提供したりすることができます（例：コールバック関数）。 `func` 型はこの目的において中心的な役割を果たします。これは、関数の _シグネチャ_ (引数や返り値の型、アノテーション)を示しており、この型の値は、そのシグネチャを持つ関数への参照となります。

サポートされているアノテーションは以下の通りです：

- `query` は、Canister のステートを変更せず、安価なクエリコールのメカニズムを使用して呼び出すことができることを意味しています。

- `oneway` は、この関数が何のレスポンスも返さないことを示します。これは、Fire and Forget シナリオ（訳註：イベントハンドラなど、非同期呼び出しで関数を投げ放す場合）を想定しています。

引数の命名について詳しく知りたい方は、[引数と返り値の命名](../developer-docs/build/languages/candid/candid-concepts#引数と返り値の命名)を参照してください。

型の構文  
``` candid
func () -> ()
func (text) -> (text)
func (dividend : nat, divisor : nat) -> (div : nat, mod : nat);
func () -> (int) query
func (func (int) -> ()) -> ()
```

テキスト表現の構文  
現在、プリンシパルによって識別されるサービスのパブリックメソッドのみサポートされています。

``` candid
func "w7x7r-cok77-xa".hello
func "w7x7r-cok77-xa"."☃"
func "aaaaa-aa".create_canister
```

サブタイプ  
[Service のアップグレード](../developer-docs/build/languages/candid/candid-concepts#service-のアップグレード)のルールで説明されているように、以下の修正は、ある func 型をそのサブタイプに変更します：

- 返り値の型のリストを拡張することができます。

- 引数の型のリストを短くすることができます。

- 引数の型のリストを、オプションの引数（`opt …​` 型）で拡張することができます。

- 既存の引数の型を _スーパータイプ_ に変更することができます。言い換えれば、関数の型は引数の型に _反変_ であるということです。

- 既存の返り値の型をサブタイプに変更することができます。

スーパータイプ  
以下の修正は、ある func 型をそのスーパータイプに変更します：

- 返り値の型のリストを短くすることができます。

- 返り値の型のリストはオプションの引数（`opt …​` 型）で拡張することができます。

- 引数の型のリストは拡張さすることができます。

- 既存の引数の型を _サブタイプ_ に変更することができます。言い換えれば、関数の型は引数の型に _反変_ であるということです。

- 既存の返り値の型をスーパータイプに変更することができます。

対応する Motoko の型  
Candid の関数型は、Motoko の `shared` 関数型に対応しており、返り値の型は `async` でラップされています（`oneway` でアノテーションされていない限り、返り値の型は単に `()` となります）。引数と返り値はタプルになりますが、1 つだけ指定されている場合はタプルにならず、直接使用されます：

``` candid
type F0 = func () -> ();
type F1 = func (text) -> (text);
type F2 = func (text, bool) -> () oneway;
type F3 = func (text) -> () oneway;
type F4 = func () -> (text) query;
```

は、Motoko では以下に対応します：

``` Motoko
type F0 = shared () -> async ();
type F1 = shared Text -> async Text;
type F2 = shared (Text, Bool) -> ();
type F3 = shared (text) -> ();
type F4 = shared query () -> async Text;
```

対応する Rust の型  
`candid::IDLValue::Func(Principal, String)` となります。詳しくは、 [IDLValue](https://docs.rs/candid/0.6.15/candid/parser/value/enum.IDLValue.html) を参照ください。

対応する JavaScript の値  
`[Principal.fromText("aaaaa-aa"), "create_canister"]`

## service {…} 型

サービスは、それぞれの関数（[`func` 型](#func----型)を使用）だけでなく、サービス全体への参照を渡したい場合があります。このような場合には、Candid の型はサービスの（完全な）インターフェースを宣言するために使うことができます。

service 型の構文に関する詳細は、[Candid Service の記述](../developer-docs/build/languages/candid/candid-concepts#service-記述ファイルの生成)を参照してください。

型の構文  
``` candid
service {
  add : (nat) -> ();
  subtract : (nat) -> ();
  get : () -> (int) query;
  subscribe : (func (int) -> ()) -> ();
}
```

テキスト表現の構文  
``` candid
service "w7x7r-cok77-xa"
service "zwigo-aiaaa-aaaaa-qaa3a-cai"
service "aaaaa-aa"
```

サブタイプ  
service 型のサブタイプとは、追加のメソッドが付与されたり、既存のメソッドの型がサブタイプに変更されている service 型です。

これは、[Service のアップグレード](../developer-docs/build/languages/candid/candid-concepts#service-のアップグレード)内のルールにて説明されているのと同じ原理に基づくものです。

スーパータイプ  
service 型のスーパータイプとは、一部のメソッドが削除されたり、既存のメソッドの型がスーパータイプに変更されている service 型です。

対応する Motoko の型  
Candid の Service 型は Motoko の `actor` 型に直接対応します：

``` motoko
actor {
  add : shared Nat -> async ()
  subtract : shared Nat -> async ();
  get : shared query () -> async Int;
  subscribe : shared (shared Int -> async ()) -> async ();
}
```

対応する Rust の型  
`candid::IDLValue::Service(Principal)` に対応します。詳しくは、 [IDLValue](https://docs.rs/candid/0.6.15/candid/parser/value/enum.IDLValue.html) を参照してください。

対応する JavaScript の値  
`Principal.fromText("aaaaa-aa")`

## principal 型

Internet Computer では、Canister やユーザーや他のエンティティを識別するための共通の方式として、_principal_ を使用しています。

型の構文  
`principal`

テキスト表現の構文  
``` candid
principal "w7x7r-cok77-xa"
principal "zwigo-aiaaa-aaaaa-qaa3a-cai"
principal "aaaaa-aa"
```

対応する Motoko の型  
`Principal`

対応する Rust の型  
`candid::Principal` または `ic_types::Principal`

対応する JavaScript の値  
`Principal.fromText("aaaaa-aa")`

## reserved 型

`reserved` 型は、1 つの（情報を持たない）値 `reserved` を持つ型で、他のすべての型のスーパータイプです。

メソッドの引数を削除するのに `reserved` 型を使用することができます。次のようなシグネチャを持つメソッドを考えてみましょう：

``` candid
service {
  foo : (first_name : text, middle_name : text, last_name : text) -> ()
}
```

ここで、`middle_name` をもはや使わなくなったと仮定します。ところが、Candid はあなたが関数シグネチャを以下のように変更することを妨げません：

``` candid
service {
  foo : (first_name : text, last_name : text) -> ()
}
```

これは非常に危険です。なぜなら、クライアントが古いインターフェースを使ってコールした場合、この関数は黙って `last_name` を無視し、`middle_name` を `last_name` として受け取ることになるからです。メソッドの引数名は単なる慣例であり、メソッドの引数はその位置によって識別されることを思い出してください。

代わりに、以下のようにすることができます：

``` candid
service {
  foo : (first_name : text, middle_name : reserved, last_name : text) -> ()
}
```

これは、`foo` は以前は第 2 引数を使用していたものの、現在は使用していないということを示しています。

将来引数が変わることが予想される関数や、型ではなく位置でしか区別できない引数を持つ関数は、1 つの record を取るように宣言するというパターンを採用することで、この落とし穴を回避することができます。 例えば以下のようになります：

``` candid
service {
  foo : (record { first_name : text; middle_name : text; last_name : text}) -> ()
}
```

ここで、関数シグネチャを以下のように変更します：

``` candid
service {
  foo : (record { first_name : text; last_name : text}) -> ()
}
```

これは正しく動作します。このようにすることで、削除された引数に関する記録を残す必要もありません。

<div class="note">

一般的に、メソッドから引数を削除することは推奨されません。通常は、引数を省略した新しいメソッドを導入することが望ましいです。

</div>

型の構文  
`reserved`

テキスト表現の構文  
`reserved`

サブタイプ  
全ての型

対応する Motoko の型  
`Any`

対応する Rust の型  
`candid::Reserved`

対応する JavaScript の値  
任意の値

## empty 型

`empty` 型は、値を持たない型で、他のどの型のサブタイプでもあります。

`empty` 型の実用的なユースケースは比較的まれです。 例えば、`empty` 型は、あるメソッドが「決して正常にリターンしない」ことを示すために使用することができます：

``` candid
service : {
  always_fails () -> (empty)
}
```

型の構文  
`empty`

テキスト表現の構文  
この型には値がないため、テキスト表現はありません。

スーパータイプ  
全ての型

対応する Motoko の型  
`None`

対応する Rust の型  
`candid::Empty`

対応する JavaScript の値  
この型には値がないため、対応する JavaScript の値はありません。

<!--
# Candid Reference

This document provides detailed reference information about Candid supported types and links to the Candid specification and documentation for the Candid Rust crate.

-   [Supported types](#supported-types)

-   [Candid specification](https://github.com/dfinity/candid)

-   [Candid Rust crate](https://docs.rs/candid)

If you are looking for more information before you start working with Candid, check out the following related resources:

-   [How Candid provides a common language for application interfaces (video)](https://www.youtube.com/watch?v=O2KaWRtsqHg)


## Supported types

This section lists all the types supported by Candid. For each type, the reference includes the following information:

-   Type syntax and the syntax for the textual representation of the type.

-   Upgrade rules for each type are given in terms of the possible *subtypes* and *supertypes* of a type.

-   Corresponding types in Rust, Motoko and Javascript.

Subtypes are the types you can change your method *results* to. Supertypes are the types that you can change your method *arguments* to.

You should note that this reference only lists the specific subtypes and supertypes that are relevant for each type. It does not repeat common information about subtypes and supertypes that can apply to any type. For example, the reference does not list `empty` as a subtype because it can be a subtype of any other type. Similarly, the types `reserved` and `opt t` are not listed as supertypes of specific types because they are supertypes of any type. For details about the subtyping rules for the `empty`, `reserved`, and `opt t` types, see the following sections:

-   [`opt t`](#type-opt)

-   [`reserved`](#type-reserved)

-   [`empty`](#type-empty)

## Type text

The `text` type is used for human readable text. More precisely, its values are sequences of unicode code points (excluding surrogate parts).

Type syntax  
`text`

Textual syntax  
``` candid
""
"Hello"
"Escaped characters: \n \r \t \\ \" \'"
"Unicode escapes: \u{2603} is ☃ and \u{221E} is ∞"
"Raw bytes (must be utf8): \E2\98\83 is also ☃"
```

Corresponding Motoko type  
`Text`

Corresponding Rust type  
`String` or `&str`

Corresponding JavaScript values  
`"String"`

## Type blob

The `blob` type can be used for binary data, that is, sequences of bytes. Interfaces written using the `blob` type are interchangeable with interfaces that are written using `vec nat8`.

Type syntax  
`blob`

Textual syntax  
`blob <text>`

where `<text>` represents a text literal with all characters representing their utf8 encoding, and arbitrary byte sequences (`"\CA\FF\FE"`).

For more information about text types, see [Text](#type-text).

Subtypes  
`vec nat8`, and all subtypes of `vec nat8`.

Supertypes  
`vec nat8`, and all supertypes of `vec nat8`.

Corresponding Motoko type  
`Blob`

Corresponding Rust type  
`Vec<u8>` or `&[u8]`

Corresponding JavaScript values  
`[ 1, 2, 3, 4, ... ]`

## Type nat

The `nat` type contains all natural (non-negative) numbers. It is unbounded, and can represent arbitrary large numbers. The on-wire encoding is LEB128, so small numbers are still efficiently represented.

Type syntax  
`nat`

Textual syntax  
``` candid
1234
1_000_000
0xDEAD_BEEF
```

Supertypes  
`int`

Corresponding Motoko type  
`Nat`

Corresponding Rust type  
`candid::Nat` or `u128`

Corresponding JavaScript values  
`+BigInt(10000)` or `10000n`

## Type int

The `int` type contains all whole numbers. It is unbounded and can represent arbitrary small or large numbers. The on-wire encoding is SLEB128, so small numbers are still efficiently represented.

Type syntax  
`int`

Textual syntax  
``` candid
1234
-1234
+1234
1_000_000
-1_000_000
+1_000_000
0xDEAD_BEEF
-0xDEAD_BEEF
+0xDEAD_BEEF
```

Subtypes  
`nat`

Corresponding Motoko type  
`Int`

Corresponding Rust type  
`candid::Int` or `i128`

Corresponding JavaScript values  
`+BigInt(-10000)` or `-10000n`

## Type natN and intN

The types `nat8`, `nat16`, `nat32`, `nat64`, `int8`, `int16`, `int32` and `int64` represent numbers with a representation of that many bits, and can be used in more “low-level” interfaces.

The range of `natN` is `{0 …​ 2^N-1}`, and the range of `intN` is `-2^(N-1) …​ 2^(N-1)-1`.

The on-wire representation is exactly that many bits long. So for small values, `nat` is more space-efficient than `nat64`.

Type syntax  
`nat8`, `nat16`, `nat32`, `nat64`, `int8`, `int16`, `int32` or `int64`

Textual syntax  
Same as `nat` for `nat8`, `nat16`, `nat32`, and `nat64`.

Same as `int` for `int8`, `int16`, `int32` and `int64`.

We can use type annotation to distinguish different integer types.

``` candid
100 : nat8
-100 : int8
(42 : nat64)
```

Corresponding Motoko type  
`natN` translates by default to `NatN`, but can also correspond to `WordN` when required.

`intN` translate to `IntN`.

Corresponding Rust type  
Signed and unsigned integers of corresponding size.

| Length | Signed | Unsigned |
|--------|--------|----------|
| 8-bit  | i8     | u8       |
| 16-bit | i16    | u16      |
| 32-bit | i32    | u32      |
| 64-bit | i64    | u64      |

Corresponding JavaScript values  
8-bit, 16-bit and 32-bit translate to the number type.

`int64` and `nat64` translate to the `BigInt` primitive in JavaScript.

## Type float32 and float64

The types `float32` and `float64` represent IEEE 754 floating point numbers in single precision (32 bit) and double precision (64 bit).

Type syntax  
`float32`, `float64`

Textual syntax  
The same syntax as `int`, plus floating point literals as follows:

``` candid
1245.678
+1245.678
-1_000_000.000_001
34e10
34E+10
34e-10
0xDEAD.BEEF
0xDEAD.BEEFP-10
0xDEAD.BEEFp+10
```

Corresponding Motoko type  
`float64` corresponds to `Float`.

`float32` does *not* currently have a representation in Motoko. Candid interfaces using `float32` cannot be served from or used from Motoko programs.

Corresponding Rust type  
`f32`, `f64`

Corresponding JavaScript values  
float number

## Type bool

The `bool` type is a logical data type that can have only the values `true` or `false`.

Type syntax  
`bool`

Textual syntax  
`true`, `false`

Corresponding Motoko type  
`Bool`

Corresponding Rust type  
`bool`

Corresponding JavaScript values  
`true`, `false`

## Type null

The `null` type is the type of the value `null`, thus a subtype of all the `opt t` types. It is also the idiomatic choice when using [variants](#type-variant) to model enumerations.

Type syntax  
`null`

Textual syntax  
`null`

Supertypes  
All `opt t` types.

Corresponding Motoko type  
`Null`

Corresponding Rust type  
`()`

Corresponding JavaScript values  
`null`

## Type vec t

The `vec` type represents vectors (sequences, lists, arrays). A value of type `vec t` contains a sequence of zero or more values of type `t`.

Type syntax  
`vec bool`, `vec nat8`, `vec vec text`, and so on.

Textual syntax  
``` candid
vec {}
vec { "john@doe.com"; "john.doe@example.com" };
```

Subtypes  
-   Whenever `t` is a subtype of `t'`, then `vec t` is a subtype of `vec t'`.

-   `blob` is a subtype of `vec nat8`.

Supertypes  
-   Whenever `t` is a supertype of `t'`, then `vec t` is a supertype of `vec t'`.

-   `blob` is a supertype of `vec nat8`.

Corresponding Motoko type  
`[T]`, where the Motoko type `T` corresponds to `t`.

Corresponding Rust type  
`Vec<T>` or `&[T]`, where the Rust type `T` corresponds to `t`.

`vec t` can translate to `BTreeSet` or `HashSet`.

`vec record { KeyType; ValueType }` can translate to `BTreeMap` or `HashMap`.

Corresponding JavaScript values  
`Array`, e.g. `[ "text", "text2", …​ ]`

## Type opt t

The `opt t` type contains all the values of type `t`, plus the special `null` value. It is used to express that some value is optional, meaning that data might be present as some value of type `t`, or might be absent as the value `null`.

The `opt` type can be nested (for example, `opt opt text`), and the values `null` and `opt null` are distinct values.

The `opt` type plays a crucial role in the evolution of Candid interfaces, and has special subtyping rules as described below.

Type syntax  
`opt bool`, `opt nat8`, `opt opt text`, and so on.

Textual syntax  
``` candid
null
opt true
opt 8
opt null
opt opt "test"
```

Subtypes  
The canonical rules for subtyping with `opt` are:

-   Whenever `t` is a subtype of `t'`, then `opt t` is a subtype of `opt t'`.

-   `null` is a subtype of `opt t'`.

-   `t` is a subtype of `opt t` (unless `t` itself is `null`, `opt …` or `reserved`).

In addition, for technical reasons related to upgrading and higher-order services, *every* type is a subtype of `opt t`, yielding `null` if the types do not match. Users are advised, however, to not directly make use of that rule.

Supertypes  
-   Whenever `t` is a supertype of `t'`, then `opt t` is a supertype of `opt t'`.

Corresponding Motoko type  
`?T`, where the Motoko type `T` corresponds to `t`.

Corresponding Rust type  
`Option<T>`, where the Rust type `T` corresponds to `t`.

Corresponding JavaScript values  
`null` translates to `[]`.

`opt 8` translates to `[8]`.

`opt opt "test"` translates to `[["test"]]`.

## Type record { n : t, … }

A `record` type is a collection of labeled values. For example, the following code gives the name `address` to the type of records that have the textual fields `street`, `city` and `country` and a numerical field of `zip_code`.

``` candid
type address = record {
  street : text;
  city : text;
  zip_code : nat;
  country : text;
};
```

The order of fields in the record type declaration does not matter. Each field can have a different type (unlike vectors). The label of a record field can also be a 32-bit natural number, as in this example:

``` candid
type address2 = record {
  288167939 : text;
  1103114667 : text;
  220614283 : nat;
  492419670 : text;
};
```

In fact, textual labels are treated as their *field hash*, and incidentally, `address` and `address2` are—to Candid—the same types.

If you omit the label, Candid automatically assigns sequentially-increasing labels. This behavior leads to the following shortened syntax, which is typically used to represent pairs and tuples. The type `record { text; text; opt bool }` is equivalent to `record { 0 : text; 1: text; 2: opt bool }`

Type syntax  
``` candid
record {}
record { first_name : text; second_name : text }
record { "name with spaces" : nat; "unicode, too: ☃" : bool }
record { text; text; opt bool }
```

Textual syntax  
``` candid
record {}
record { first_name = "John"; second_name = "Doe" }
record { "name with spaces" = 42; "unicode, too: ☃" = true }
record { "a"; "tuple"; null }
```

Subtypes  
Subtypes of a record are record types that have additional fields (of any type), where some field’s types are changed to subtypes, or where optional fields are removed. It is, however, bad practice to remove optional fields in method results. You can change a field’s type to `opt empty` to indicate that this field is no longer used.

For example, if you have a function returning a record of of the following type:

``` candid
record {
  first_name : text; middle_name : opt text; second_name : text; score : int
}
```

you can evolve that to a function returning a record of the following type:

``` candid
record {
  first_name : text; middle_name : opt empty; second_name : text; score : nat; country : text
}
```

where we have deprecated the `middle_name` field, change the type of `score` and added the `country` field.

Supertypes  
Supertypes of a record are record types with some fields removed, some fields’ types changed to supertypes, or with optional fields added.

The latter is what allows you to extend your argument records with additional fields. Clients using the old interface will not include the field in their record, which will decode, when expected in the upgraded service, as `null`.

For example, if you have a function expecting a record of type:

``` candid
record { first_name : text; second_name : text; score : nat }
```

you can evolve that to a function expecting a record of type:

``` candid
record { first_name : text; score: int; country : opt text }
```

Corresponding Motoko type  
If the record type looks like it could refer to a tuple (that is, consecutive labels starting at 0), a Motoko tuple type (for example `(T1, T2, T3)`) is used. Else, a Motoko record `({ first_name :Text, second_name : Text })` is used.

If the field name is a reserved name in Motoko, an undescore is appended. So `record { if : bool }` corresponds to `{ if_ : Bool }`.

If (even then) the field name is not a valid Motoko identifier, the *field* hash is used instead: `record { ☃ : bool }` corresponds to `{ 11272781 : Boolean }`.

Corresponding Rust type  
User defined `struct` with `#[derive(CandidType, Deserialize)]` trait.

You can use the `#[serde(rename = "DifferentFieldName")]` attribute to rename field names.

If the record type is a tuple, it can be translated to a tuple type such as `(T1, T2, T3)`.

Corresponding JavaScript values  
If the record type is a tuple, the value is translated to an array, for example, `["Candid", 42]`.

Else it translates to a record object. For example, `{ "first name": "Candid", age: 42 }`.

If the field name is a hash, we use `_hash_` as the field name, for example, `{ _1_: 42, "1": "test" }`.

## Type variant { n : t, … }

A `variant` type represents a value that is from exactly one of the given cases, or *tags*. So a value of the type:

``` candid
type shape = variant {
  dot : null;
  circle : float64;
  rectangle : record { width : float64; height : float64 };
  "💬" : text;
};
```

is either a dot, or a circle (with a radius), or a rectangle (with dimensions), or a speech bubble (with some text). The speech bubble illustrates use of a unicode label name (💬).

The tags in variants are, just like the labels in records, actually numbers, and string tags refer to their hash value.

Often, some or all of the the tags do not carry data. It is idiomatic to then use the `null` type, as in the `dot` above. In fact, Candid encourages this by allowing you to omit the `: null` type annotation in variants, so:

``` candid
type season = variant { spring; summer; fall; winter }
```

is equivalent to:

``` candid
type season = variant {
  spring : null; summer: null; fall: null; winter : null
}
```

and used to represent enumerations.

The type `variant {}` is legal, but has no values. If that is the intention, the [`empty` type](#type-empty) may be more appropriate.

Type syntax  
``` candid
variant {}
variant { ok : nat; error : text }
variant { "name with spaces" : nat; "unicode, too: ☃" : bool }
variant { spring; summer; fall; winter }
```

Textual syntax  
``` candid
variant { ok = 42 }
variant { "unicode, too: ☃" = true }
variant { fall }
```

Subtypes  
Subtypes of a variant type are variant types with some tags removed, and the type of some tags themselves changed to a subtype.

If you want to be able to *add* new tags in variants in a method result, you can do so if the variant is itself wrapped in `opt …`. This requires planning ahead! When you design an interface, instead of writing:

``` candid
service {
  get_member_status (member_id : nat) -> (variant {active; expired});
}
```

it is better to use this:

``` candid
service {
  get_member_status (member_id : nat) -> (opt variant {active; expired});
}
```

This way, if you later need to add a `honorary` membership status, you can expand the list of statuses. Old clients will receive unknown fields as `null`.

Supertypes  
Supertypes of a variant types are variants with additional tags, and maybe the type of some tags changed to a supertype.

Corresponding Motoko type  
Variant types are represented as Motoko variant types, for example:

``` motoko
type Shape = {
  #dot : ();
  #circle : Float;
  #rectangle : { width : Float; height : Float };
  #_2669435721_ : Text;
};
```

Note that if the type of a tag is `null`, this corresponds to `()` in Motoko, to preserve the mapping between the respective idiomatic ways to model enumerations as variants.

Corresponding Rust type  
User defined `enum` with `#[derive(CandidType, Deserialize)]` trait.

You can use the `#[serde(rename = "DifferentFieldName")]` attribute to rename field names.

Corresponding JavaScript values  
A record object with a single entry. For example, `{ dot: null }`.

If the field name is a hash, we use `_hash_` as the field name, for example, `{ _2669435721_: "test" }`.

## Type func (…) → (…)

Candid is designed to support higher-order use cases, where a service may receive or provide references to other services or their methods, for example, as callbacks. The `func` type is central to this: It indicates the function’s *signature* (argument and results types, annotations), and values of this type are references to functions with that signature.

The supported annotations are:

-   `query` indicates that the referenced function is a query method, meaning it does not alter the state of its canister, and that it can be invoked using the cheaper “query call” mechanism.

-   `oneway` indicates that this function returns no response, intended for fire-and-forget scenarios.

For more information about parameter naming, see [Naming arguments and results](../developer-docs/build/candid/candid-concepts.md#service-naming).

Type syntax  
``` candid
func () -> ()
func (text) -> (text)
func (dividend : nat, divisor : nat) -> (div : nat, mod : nat);
func () -> (int) query
func (func (int) -> ()) -> ()
```

Textual syntax  
Currently, only public methods of services, which are identified by their principal, are supported:

``` candid
func "w7x7r-cok77-xa".hello
func "w7x7r-cok77-xa"."☃"
func "aaaaa-aa".create_canister
```

Subtypes  
The following modifications to a function type change it to a subtype as discussed in the rules for [Service upgrades](../developer-docs/build/candid/candid-concepts.md#upgrades):

-   The result type list may be extended.

-   The parameter type list may be shortened.

-   The parameter type list may be extended with optional arguments (type `opt …`).

-   Existing parameter types may be changed to to a *supertype* ! In other words, the function type is *contravariant* in the argument type.

-   Existing result types may be changed to a subtype.

Supertypes  
The following modifications to a function type change it to a supertype:

-   The result type list may be shortened.

-   The result type list may be extended with optional arguments (type `opt …`).

-   The parameter type list may be extended.

-   Existing parameter types may be changed to to a *subtype* ! In other words, the function type is *contravariant* in the argument type.

-   Existing result types may be changed to a supertype.

Corresponding Motoko type  
Candid function types correspond to `shared` Motoko functions, with the result type wrapped in `async` (unless they are annotated with `oneway`, then the result type is simply `()`). Arguments resp. results become tuples, unless there is exactly one, in which case it is used directly:

``` candid
type F0 = func () -> ();
type F1 = func (text) -> (text);
type F2 = func (text, bool) -> () oneway;
type F3 = func (text) -> () oneway;
type F4 = func () -> (text) query;
```

corresponds in Motoko to

``` Motoko
type F0 = shared () -> async ();
type F1 = shared Text -> async Text;
type F2 = shared (Text, Bool) -> ();
type F3 = shared (text) -> ();
type F4 = shared query () -> async Text;
```

Corresponding Rust type  
`candid::IDLValue::Func(Principal, String)`, see [IDLValue](https://docs.rs/candid/0.6.15/candid/parser/value/enum.IDLValue.html).

Corresponding JavaScript values  
`[Principal.fromText("aaaaa-aa"), "create_canister"]`

## Type service {…}

Services may want to pass around references to not just individual functions (using the [`func` type](#type-func)), but references to whole services. In this case, Candid types can be used to declare the complete interface of such a service.

See [Candid service descriptions](../developer-docs/build/candid/candid-concepts.md#candid-service-descriptions) for more details on the syntax of a service type.

Type syntax  
``` candid
service {
  add : (nat) -> ();
  subtract : (nat) -> ();
  get : () -> (int) query;
  subscribe : (func (int) -> ()) -> ();
}
```

Textual syntax  
``` candid
service "w7x7r-cok77-xa"
service "zwigo-aiaaa-aaaaa-qaa3a-cai"
service "aaaaa-aa"
```

Subtypes  
The subtypes of a service type are those service types that possibly have additional methods, and where the type of an existing method is changed to a subtype.

This is exactly the same principle as discussed for upgrade rules in [Service upgrades](../developer-docs/build/candid/candid-concepts.md#upgrades).

Supertypes  
The supertypes of a service type are those service types that may have some methods removed, and the type of existing methods are changed to a supertype.

Corresponding Motoko type  
Service types in Candid correspond directly to `actor` types in Motoko:

``` motoko
actor {
  add : shared Nat -> async ()
  subtract : shared Nat -> async ();
  get : shared query () -> async Int;
  subscribe : shared (shared Int -> async ()) -> async ();
}
```

Corresponding Rust type  
`candid::IDLValue::Service(Principal)`, see [IDLValue](https://docs.rs/candid/0.6.15/candid/parser/value/enum.IDLValue.html).

Corresponding JavaScript values  
`Principal.fromText("aaaaa-aa")`

## Type principal

The Internet Computer uses *principals* as the common scheme to identify canisters, users, and other entities.

Type syntax  
`principal`

Textual syntax  
``` candid
principal "w7x7r-cok77-xa"
principal "zwigo-aiaaa-aaaaa-qaa3a-cai"
principal "aaaaa-aa"
```

Corresponding Motoko type  
`Principal`

Corresponding Rust type  
`candid::Principal` or `ic_types::Principal`

Corresponding JavaScript values  
`Principal.fromText("aaaaa-aa")`

## Type reserved

The `reserved` type is a type with one (uninformative) value `reserved`, and is the supertype of all other types.

The `reserved` type can be used to remove method arguments. Consider a method with the following signature:

``` candid
service {
  foo : (first_name : text, middle_name : text, last_name : text) -> ()
}
```

and assume you no longer care about the `middle_name`. Although Candid will not prevent you from changing the signature to this:

``` candid
service {
  foo : (first_name : text, last_name : text) -> ()
}
```

it would be disastrous: If a client talks to you using the old interface, you will silently ignore the `last_name` and take the `middle_name` as the `last_name`. Remember that method parameter names are just convention, and method arguments are identified by their position.

Instead, you can use:

``` candid
service {
  foo : (first_name : text, middle_name : reserved, last_name : text) -> ()
}
```

to indicate that `foo` used to take a second argument, but you no longer care about that.

You can avoid this pitfall by adopting the pattern any function that is anticipated to have changing arguments, or whose arguments can only be distinguished by position, not type, is declared to take a single record. For example:

``` candid
service {
  foo : (record { first_name : text; middle_name : text; last_name : text}) -> ()
}
```

Now, changing the signature to this:

``` candid
service {
  foo : (record { first_name : text; last_name : text}) -> ()
}
```

does the right thing, and you don’t even need to keep a record of the removed argument around.

<div class="note">

In general, it is not recommended to remove arguments from methods. Usually, it is preferable to introduce a new method that omits the argument.

</div>

Type syntax  
`reserved`

Textual syntax  
`reserved`

Subtypes  
All types

Corresponding Motoko type  
`Any`

Corresponding Rust type  
`candid::Reserved`

Corresponding JavaScript values  
Any value

## Type empty

The `empty` type is the type without values, and is the subtype of any other type.

Practical use cases for the `empty` type are relatively rare. It could be used to mark a method as “never returns successfully”. For example:

``` candid
service : {
  always_fails () -> (empty)
}
```

Type syntax  
`empty`

Textual syntax  
None, as this type has no values

Supertypes  
All types

Corresponding Motoko type  
`None`

Corresponding Rust type  
`candid::Empty`

Corresponding JavaScript values  
None, as this type has no values

-->