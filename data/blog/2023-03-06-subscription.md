---
title: 'Apollo Client Subscription'
date: '2023-03-06'
tags: ['apollo-client']
draft: false
summary: 'RefreshToken 처리관련 해서 용어를 알아본다'
---

# ApolloClient Subscription

> [ApolloClient](https://https://www.apollographql.com/docs/react/data/subscriptions) 의 `Docs` 를 참고하여 작성한 내용이다.<br/> 번역하고 이해하는 과정에서 잘못된 정보가 있을수 있다.

> `queries`, `mutations` 외에도 `GraphQl` 을 지원하기 위한 3번째 작업이 바로 `subscription` 이다.

`subscription` 은 `queries` 와 비슷하며, `qeries` 처럼 `sbuscription` 을 통해 `data` 를 가져올 수 있다.

하지만 `quries` 는 한번 처리하고 끝이라면, `subscription` 은 `오랫동안 지속되는 작업` 이라고 볼수 있다.

`subscription` 은 `오랫동안 지속되는 작업` 이므로 `시간이 지남에 따라 그 결과를 변경` 할수 있다.

이를 통해 `GraphQL server` 와 `active(동적)` 으로 연결하며, `server` 로 부터 받은 `subscription result(구독결과)`를 넣어 `update` 할수 있게 된다.

즉, `subscription` 은 `backend data` 가 변경되는 것에 대해서 `real time` 으로 `client` 에 알리는데 유용하게 사용된다.

## Subscription 을 사용해야할 때

대부분의 경우 `backend data` 와 최신정보를 `subscription`하여 사용하지 말아야 한다.

이러한 경우 [`poll intermittently(간헐적 polling)`](https://www.apollographql.com/docs/react/data/queries/#polling), 혹은 유저를 통해 어떠한 동작이 일어나도록 하는 `re-execcute queries on demand(요청을 통한 쿼리 재실행)` 을 통해 이루어져야한다.

`subscription` 는 다음과 같은 상황에서 사용해야 한다.

1. 객체에서 작은 일부분이 점진적으로 변화될때,

   > 대부분의 객체 필드는 변경되지 않고, 어떠한 작은부분의 필드만 변경된다고 가정하자.<br/>
   > 이때 지속적으로 `polling` 하는것은 굉장히 비효율적이다.<br/>
   > 필드 하나만 변경되기 위해 새로운 전체 `data` 를 가져와야 하는것이다.<br/> > `subscription` 은 `query` 를 보내 `initial state(초기값)` 을 가져오고, `server` 는 사전에 `update` 될때 마다 능동적으로 각 부분의 `fields` 를 넣는다.

2. 대기 시간이 짧은, `realtime update`

> 채팅 어플리케이션 같은 경우 `client` 에서 새로운 메시지를 곧바로 받기를 원할것이다. 이때 사용된다.

> _주의점_<br/><br/> `subscription` 은 `cache` 를 변경하는 `local client event` 를 `listen` 받아 사용 할 수 없다.<br/> > `sbuscription` 은 `external data` 가 변경될때, 사용되도록 의도 되었다.<br/> 그리고, 수신받은 변경사항을 `cache` 안에 저장한다.<br/>
> 이렇게 저장된 `cache` 를 `client.watchQuery` 와 `useQuery` 를 사용하여 `cache` 의 변경을 관찰하여 사용할 수 있도록 되어있다고 한다.

## Defining a subscription

> `subscription` 은 `qureis` 및 `mutations` 를 사용하는것처럼, `server side` 와 `client side` 양쪽 모두에 `subscription`을 적용 가능하다.

## Server Side

> `server` 쪽에서는 `Subscription` 타입의 필드로 이미 존재하는 `GraphQL Schema` 의 `subscription` 로 정의한다.
> 다음의 `commentAdded subscription` 은 특정 블로그 게시물에 새로운 `comment` 가 추가될때 마다, `subscription client` 에게 알린다.

```tsx
type Subscription {
  commentAdded(postID: ID!): Comment
}
```

## Client side

> `client` 쪽에서는 `Apollo Client` 에서 실행을 원하는 각 `subscription` 의 `shape` 를 정의할 수 있다.

```tsx
const COMMENTS_SUBSCRIPTION = gql`
  subscription OnCommentAdded($postID: ID!) {
    commentAdded(postID: $postID) {
      id
      content
    }
  }
`
```

`query` 나 `mutation` 대신 `subscription` 만 넣으면 되는 간단한 구조이다.
위의 `onCommentAddes` `subscription` 을 실행할때, `GraphQL server` 와 `connection` 을 수립하고 응답데이터를 전달해준다.

이는 `query` 와 달리, `server` 의 응답을 즉각적으로 받길 기대하지 않는다.
그 대신에, `backend` 에서 특정 `event` 가 발생할때, `client` 에 `data` 를 넣는다.

즉, `GraphQL server` 에서 `subscribing client` 에 `data` 를 넣을때마다, 실행된 `subscription` 의 `data` 의 구조를 따른다.

다음을 보자.

```tsx

{
  "data": {
    "commentAdded": {
      "id": "123",
      "content": "What a thoughtful and well written post!"
    }
  }
}

```

이전의 `subscription OnCommentAdded` 와 동일한 구조를 가졌다.
