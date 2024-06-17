---
title: クエリとミューテーション
description: GraphQL - Learn.Queries and Mutations
# image: img/thambnails/go/hierarchy-config.png
keywords: [GraphQL, Learn, Reffernce, 和訳]
tags:
  - 和訳
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# クエリとミューテーション

このページでは、GraphQLサーバーへの問い合わせ方法について学びます。

## フィールド (Fields)

<Tabs>

<TabItem value="query" label="Query">

```graphql
{
  hero {
    name
  }
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

</TabItem>

</Tabs>

ご覧の通り、Queryの形とResponseの形が全く同じであることがわかると思います。GraphQLにおいて、この点は重要です。期待したものが返ってきて、サーバーもクライアントが求めているフィールドを知ることができます。  

今回のフィールド `name` は スター・ウォーズのメインヒーロー `R2-D2` の名前で `String` 型です。  

> おっと、ひとつ付け加えると、上記の Query はインタラクティブ（対話的）です。つまり、Query に変更を加えると、新たな結果を得ることになります。ぜひ、フィールド `appearsIn` を hero オブジェクトの中に付け加えてみてください。別の結果を得られることでしょう。

先ほどの例では、文字列型を返す hero の name を問い合わせただけでしたが、フィールドにはオブジェクトを指定することもできます。ここでは、オブジェクトのサブセクションを作成してみましょう。GraphQLクエリは、関連するオブジェクトをフィールドに渡すことができるため、クライアントは、RESTアーキテクチャで必要となるような数回のラウンドトリップを行う代わりに、1回のリクエストで多くの関連データを取得することができます。

<Tabs>

<TabItem value="query" label="Query">

```graphql
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

</TabItem>

</Tabs>

この例では、フィールド `friends` はアイテムの配列を返すことに注意しましょう。GraphQLクエリは、単一のアイテムでもアイテムのリストでも同じように見えますが、スキーマに示されている内容に基づいて、どちらを意図しているかがわかります。

## 引数 (Arguments)

できることがオブジェクトとそのフィールドを渡すだけでも、GraphQLはすでにデータ取得のための非常に便利な言語でしょう。しかし、フィールドに引数を渡す機能を追加すると、もっと面白いことになります。

<Tabs>

<TabItem value="query" label="Query">

```graphql
{
  human(id: "1000") {
    name
    height
  }
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 1.72
    }
  }
}
```

</TabItem>

</Tabs>

RESTのようなシステムでは、リクエストのクエリーパラメーターとURLセグメントという単一の引数セットしか渡すことができません。しかし、GraphQLでは、すべてのフィールドとネストされたオブジェクトが独自の引数セットを持つことができるため、GraphQLは複数のAPIフェッチを行うための完全な代替となります。スカラーフィールドに引数を渡すこともでき、各クライアントで個別にデータ変換を行うのではなく、サーバー上で一度だけデータ変換を実装することができます。

<Tabs>

<TabItem value="query" label="Query">

```graphql
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 5.6430448
    }
  }
}
```

</TabItem>

</Tabs>

引数には様々な型を指定することができます。上記の例では、列挙型を使用していますが、これは有限の選択肢（この場合は長さの単位、 `METER`または `FOOT`* のいずれか）の1つを表します。GraphQLにはデフォルトの型セットが存在していますが、通信形式にシリアライズできる限り、GraphQLサーバーは独自のカスタム型を宣言することもできます。  

[GraphQLの型システムについて](https://graphql.org/learn/schema/)

#### 訳註
\* METER ≒ 0.328 FOOT（フィート）

## エイリアス (Aliases)

鋭い方ならお気づきかもしれませんが、結果オブジェクトのフィールドはクエリのフィールド名と一致しますが、引数は含まれないため、同じフィールドを異なる引数で直接クエリすることはできません。そこで、エイリアスが必要なのです。エイリアスを使えば、フィールドの結果を好きな名前に変更することができます。

<Tabs>

<TabItem value="query" label="Query">

```graphql
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```

</TabItem>

</Tabs>

上記の例では、２つの `hero` フィールドが衝突してしまいますが、エイリアスで異なる名前をつけているため、両方の結果を一度に得ることができます。

## フラグメント (Fragments)

例えば２人の hero とその friends を表示する、非各区的複雑なページがアプリにあるとしましょう。このようなクエリがすぐに複雑になることが想像できると思います。２人を比較するために少なくとも一度はフィールドを繰り返す必要があるからです。  

それが、GraphQLが再利用可能なユニットである*フラグメント*を取り入れた理由です。フラグメントは、フィールドの塊を定義し、必要なクエリに組み込むことを可能にします。先ほどの状況を解決する際は、次のようにフラグメントを使用します。

<Tabs>

<TabItem value="query" label="Query">

```graphql
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        },
        {
          "name": "C-3PO"
        },
        {
          "name": "R2-D2"
        }
      ]
    },
    "rightComparison": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

</TabItem>

</Tabs>

フィールドが繰り返されると、上記のクエリーがかなり繰り返しになりますね。フラグメントの考え方は、複雑なアプリケーションのデータ要件をより小さな塊に分割するために頻繁に使用されます。特に、異なるフラグメントを持つ多くのUIコンポーネントを一度のデータフェッチで表示する必要がある際によく使われます。

### フラグメントでの変数使用 (Using variables inside fragments)

フラグメントは、クエリやミューテーション内で宣言された変数にアクセスすることも可能です。
[変数](#変数-variables)を参照。

<Tabs>

<TabItem value="query" label="Query">

```graphql
query HeroComparison($first: Int = 3) {
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  friendsConnection(first: $first) {
    totalCount
    edges {
      node {
        name
      }
    }
  }
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "friendsConnection": {
        "totalCount": 4,
        "edges": [
          {
            "node": {
              "name": "Han Solo"
            }
          },
          {
            "node": {
              "name": "Leia Organa"
            }
          },
          {
            "node": {
              "name": "C-3PO"
            }
          }
        ]
      }
    },
    "rightComparison": {
      "name": "R2-D2",
      "friendsConnection": {
        "totalCount": 3,
        "edges": [
          {
            "node": {
              "name": "Luke Skywalker"
            }
          },
          {
            "node": {
              "name": "Han Solo"
            }
          },
          {
            "node": {
              "name": "Leia Organa"
            }
          }
        ]
      }
    }
  }
}
```

</TabItem>

</Tabs>

## オペレーション名 (Operation name)

上記のいくつかの例では、`query` キーワードと query名の両方を省略する構文を使用してきましたが、実際のアプリケーションでは、コードを曖昧にしないために、省略しない方が便利でしょう。  

次の例では、*オペレーション型(operation type)*として `query` キーワードを、*オペレーション名(operation name)*として `HeroNameAndFriends` を使用しています。

<Tabs>

<TabItem value="query" label="Query">

```graphql
query HeroNameAndFriends {
  hero {
    name
    friends {
      name
    }
  }
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

</TabItem>

</Tabs>

*オペレーション型*は 操作の種類を示す*query*、*mutation*、*subscription* のいずれかです。オペレーション型の記述は、クエリの省略記法を使用している場合を除いて必須です。  

*オペレーション名*は、オペレーションのための意味のある明示的な名前です。これは複数のオペレーションに対するドキュメントでのみ必須ですが、デバッグ時やサーバーサイドのロギングの際に非常に役立つので、その使用が推奨されます。何か問題が発生したとき（ネットワークログやGraphQLサーバーのログにエラーが表示されたとき）、内容を解読しようとするのではなく、コードベース内のクエリを名前で特定する方が簡単です。これは、お好きなプログラミング言語の関数名のようなものだと考えてください。たとえば、JavaScript では匿名関数だけを簡単に扱うことができますが、関数に名前を付けると、その関数を追跡したり、コードをデバッグしたり、その関数が呼び出されたときにログを記録したりするのが簡単になります。同じように、GraphQLのクエリー名とミューテーション名は、フラグメント名とともに、サーバー側で異なるGraphQLリクエストを識別するための便利なデバッグツールになります。

## 変数 (Variables)

これまでは、すべての引数をクエリ文字列の中に記述してきました。しかし、ほとんどのアプリケーションでは、フィールドの引数は動的です。例えば、興味のあるスターウォーズのエピソードを選択するドロップダウンや、検索フィールド、フィルターのセットがあるかもしれません。  

なぜなら、クライアント側のコードは実行時にクエリ文字列を動的に操作し、それをGraphQL固有の形式にシリアライズする必要があるからです。その代わりに、GraphQLには動的な値をクエリーから除外し、別の ディクショナリとして渡す良い方法があります。これらの値は変数と呼ばれます。  

変数を使用する際には、３つのことから始めます。  

1. クエリ内の性的な値を `$variableName` に置き換える
2. `$variableName` をクエリが受けるける変数の１つとして宣言する
3. 変数名を渡す：通信に適した（通常はJSON）個別の変数ディクショナリの値

まとめると、こんなふうになります。

<Tabs>

<TabItem value="query" label="Query">

```graphql
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

</TabItem>

<TabItem value="variable" label="Variable">

```json
{
  "episode": "JEDI"
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

</TabItem>

</Tabs>

これで、クライアントコードでは、全く新しいクエリを作成する必要がなくなり、別の変数を渡すだけで済むようになります。これは一般的に、クエリのどの引数が動的であるかを示すのに良い方法です。

### 変数定義 (Variable definitions)

変数定義は、上記のクエリで `($episode: Episode)` の部分です。これは、静的型付け言語の関数の引数定義と同じように機能します。変数の前に `$` をつけ、その後に型（この場合は `Episode`）をつけます。

宣言された変数はすべて、スカラー型(scalars)、列挙型(enums)、入力オブジェクト型(input object type)のいずれかでなければなりません。したがって、複雑なオブジェクトをフィールドに渡したい場合は、サーバ上でどの入力型がマッチするかを知っておく必要があります。入力オブジェクト型の詳細については、スキーマのページを参照してください。  

変数定義には、省略可能なものと必須なものがあります。上の例では、`Episode` 型 の横に `!` がないので、省略可能です。しかし、変数を渡すフィールドが NULL ではない引数を必要とする場合、変数も必須でなければなりません。

これらの変数定義の構文について詳しく学ぶには、[GraphQLスキーマ言語](https://graphql.org/learn/schema/)を学ぶと便利です。スキーマ言語については、[スキーマと型](https://graphql.org/learn/schema/)のページで詳しく説明しています。

### デフォルト変数 (Default variables)

型宣言の後にデフォルト値を追加することで、クエリ内の変数にデフォルト値を割り当てることもできます。

<Tabs>

<TabItem value="query" label="Query">

```graphql
query HeroNameAndFriends($episode: Episode = JEDI) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

</TabItem>

</Tabs>

すべての変数にデフォルト値が指定されている場合は、変数を渡さずにクエリを呼び出すことができます。変数ディクショナリの一部として変数を渡すと、その変数がデフォルト値を上書きします。

## ディレクティブ (Directives)

上記で、動的なクエリを構築する際に、変数によって手作業による文字列の補間を回避できることを説明しました。引数に変数を渡すことで、このような問題の大部分は解決されますが、変数を使用してクエリの構造や形状を動的に変更する方法も必要になるかもしれません。例えば、要約されたビューと詳細なビューを持つUIコンポーネントを想像してみてください。  

では、そのようなコンポーネントのためにクエリを作ってみましょう。

<Tabs>

<TabItem value="query" label="Query">

```graphql
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
```

</TabItem>

<TabItem value="variable-false" label="Variable(false)">

```graphql
{
  "episode": "JEDI",
  "withFriends": false
}
```

</TabItem>

<TabItem value="response-false" label="Response(false)">

```json
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

</TabItem>

<TabItem value="variable-true" label="Variable(true)">

```graphql
{
  "episode": "JEDI",
  "withFriends": true
}
```

</TabItem>

<TabItem value="response-true" label="Response(true)">

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

</TabItem>

</Tabs>

上記の変数を編集して、代わりにwithFriendsにtrueを渡してみて、結果がどう変わるか試してみてください。*  

私たちはディレクティブと呼ばれるGraphQLの新しい機能を使う必要がありました。ディレクティブはフィールドまたはフラグメントのインクルージョンにアタッチすることができ、サーバーが望む方法でクエリの実行に影響を与えることができます。GraphQLのコア仕様には2つのディレクティブが含まれており、仕様に準拠したGraphQLサーバー実装でサポートされなければなりません。

- `@include(if: Boolean)`: 引数がtrueの場合のみ、このフィールドを結果に含める
- `@skip(if: Boolean)`: 引数がtrueの場合、このフィールドをスキップする

ディレクティブは、クエリのフィールドを追加したり削除したりするために 文字列を操作する必要があるような場合に便利です。サーバの実装では、全く新しいディレクティブを定義することによって 実験的な機能を追加することもできます。

#### 訳註
\* 公式ドキュメントでは、対話的にコードの編集が可能。本記事では代わりに編集したクエリと結果を示した

## ミューテーション (Mutations)

GraphQLのほとんどの議論はデータ・フェッチに焦点を当てているが、完全なデータ・プラットフォームには、サーバー側のデータを変更する方法も必要です。  

RESTでは、どのようなリクエストもサーバー上で何らかの副作用を引き起こすことになるかもしれませんが、慣例として、データを変更するために `GET` リクエストを使用しないことが推奨されています。これはGraphQLも同様で、技術的にはどのようなクエリでもデータの書き込みを引き起こす実装が可能です。しかし、書き込みを行う操作は、ミューテーションを通じて明示的に送信されるべきであるという慣習を確立することは有用です。  

クエリと同じように、ミューテーションフィールドがオブジェクト型を返す場合、ネストしたフィールドを求めることができます。これは、更新後のオブジェクトの新しい状態を取得するのに便利です。簡単なミューテーションの例を見てみましょう。

<Tabs>

<TabItem value="query" label="Query">

```graphql
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

</TabItem>

<TabItem value="variable" label="Variable">

```graphql
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}
```

</TabItem>

</Tabs>

`createReview` のフィールドが新しく作成された `stars` と `commentary` フィールドを返すことに注目してください。これは、たとえばフィールドをインクリメントするときなど、既存のデータを変更するときに特に便利です。  

この例では、渡した review の変数がスカラーではないことにもお気づきでしょう。これは入力オブジェクト型で、引数として渡すことができる特別な種類のオブジェクト型です。入力型についてはスキーマのページを参照してください。

### 複数フィールドのミューテーション

ミューテーションは、クエリと同じように複数のフィールドを含むことができます。クエリとミューテーションには、名前以外に重要な違いがあります。

**クエリーフィールドが並列に実行されるのに対して、ミューテーションフィールドは直列に逐次実行されます**  

これは、１つのリクエストで２つのミューテーション `incrementCredits` を送信する場合、１つ目が２つ目の開始前に終了することが保証され、自分自身との競合状態にならないことを保証することを意味します。

## インライン・フラグメント (Inline Fragments)

他の多くの型システムと同様に、GraphQLスキーマにはインターフェースとユニオン型を定義する機能が含まれています。[これからについて、詳しくはスキーマガイドを参照](schema/#interfaces)。  

インターフェイスやユニオン型を返すフィールドを問い合わせる場合、*インライン・フラグメント*を使用して、基礎となる具象型のデータにアクセスする必要があります。例で見るのが一番簡単です。

<Tabs>

<TabItem value="query" label="Query">

```graphql
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
    }
  }
}
```

</TabItem>

<TabItem value="variable" label="Variable">

```graphql
{
  "ep": "JEDI"
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "primaryFunction": "Astromech"
    }
  }
}
```

</TabItem>

</Tabs>

このクエリでは、フィールド `hero` は `Charactor` 型を返し、`episode` の引数によって `Human` か `Droid` のどちらかになります。直接選択では、`name` のような `Character` インターフェイスに存在するフィールドのみを問い合わせることができます。  

具象型のフィールドを要求するには、型条件を持つ*インラインフラグメント*を使用する必要があります。最初のフラグメントは `...on Droid` と書かれているので、`hero` から返される `Character` が`Droid` 型の場合のみ、`primaryFunction` フィールドが実行されます。`Human` 型の `height` フィールドも同様です。  

名前付きフラグメントは常に型が付いているので、同じように使うことができます。

## メタ・フィールド (Meta fields)

GraphQLサービスからどのような型が返ってくるかわからない状況があることを考えると、クライアント上でそのデータをどのように扱うかを決定する方法が必要です。GraphQLでは、クエリの任意の時点でメタ・フィールドである `__typename` をリクエストして、その時点でのオブジェクトの型の名前を取得することができます。

<Tabs>

<TabItem value="query" label="Query">

```graphql
name of the object type at that point.

{
  search(text: "an") {
    __typename
    ... on Human {
      name
    }
    ... on Droid {
      name
    }
    ... on Starship {
      name
    }
  }
}
```

</TabItem>

<TabItem value="response" label="Response">

```json
{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo"
      },
      {
        "__typename": "Human",
        "name": "Leia Organa"
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1"
      }
    ]
  }
}
```

</TabItem>

</Tabs>

上記のクエリでは、`search` は３つのオプションのうちの１つであるユニオン型を返します。`typename` フィールドがなければ、クライアントから異なる型を見分けることは不可能でしょう。  

GraphQLサービスはいくつかのメタ・フィールドを提供し、残りは(Introspection)[イントロスペクション](https://graphql.org/learn/introspection/)システムを公開するために使用されます。