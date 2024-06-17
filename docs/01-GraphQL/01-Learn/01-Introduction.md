---
title: はじめに
description: GraphQL - Learn.Introduction
# image: img/thambnails/go/hierarchy-config.png
keywords: [GraphQL, Learn, Reffernce, 和訳]
tags:
  - 和訳
---

# はじめに

> GraphQLについて、その仕組みと使い方を学びます。GraphQLサービスの構築方法に関するドキュメントをお探しですか？[さまざまな言語でGraphQLを実装するためのライブラリ](https://graphql.org/community/tools-and-libraries/)があります。実践的なチュートリアルで深く学ぶには、[利用可能なトレーニングコース](https://graphql.org/community/resources/training-courses/)をご覧ください。
  
GraphQLはAPI用のクエリ言語であり、データ用に定義した型システムを使用してクエリを実行するためのサーバーサイドのランタイムです。GraphQLは特定のデータベースやストレージエンジンに縛られることなく、既存のコードとデータに支えられています。  
  
GraphQLサービスは、型とその型のフィールドを定義し、各型の各フィールドに関数を提供することで作成されます。例えば、ログインしているユーザー（`me`）が誰なのか、またそのユーザーの名前を教えてくれるGraphQLサービスは次のようになります：

```graphql
type Query {
  me: User
}
 
type User {
  id: ID
  name: String
}
```

各型の各フィールドの関数は次のようなものになります。

```js
function Query_me(request) {
  return request.auth.user
}
 
function User_name(user) {
  return user.getName()
}
```

GraphQLサービスが実行されると（通常はWebサービス上のURLで）、検証および実行するためのGraphQLクエリを受け取ることができます。サービスはまず、クエリが既に定義された型とフィールドのみを参照していることを確認するために、クエリをチェックし、次に結果を生成するために提供された関数を実行します。  

例えば、このようなクエリは

```graphql
{
  me {
    name
  }
}
```

次のようなJSONを生成します。

```graphql
{
  "me": {
    "name": "Luke Skywalker"
  }
}
```

#### 原文
[Introduction to GraphQL](https://graphql.org/learn/)