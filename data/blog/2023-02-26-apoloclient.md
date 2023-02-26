---
title: Apollo Client Queries
date: '2023-02-26'
tags: ['ApolloClient']
draft: false
summary: 'Queries'
---

> Frontend 공부를 하면서 `graphQL` 사용시 사용되는 `ApolloClient` 의 각 `Method` 에 대한 이해가 밑바탕이 되어야 한다는 생각이 들었다. </br></br>
그러한 이유로 Docs 내용을 정리해 보도록 한다.

# Executing a query

`useQuery` 는 `appolo application` 쿼리를 위한  `API` 이다.

`useQuery` 는 `React hook` 이므로, 함수로써 호출해야 하며, 실행할 `GrapyQL query` 를 `string` 으로 넘겨 사용한다.

> ApolloClient Docs 의 ex<br/>
> GraphQL query 
```ts
// queries.ts
import { gql, useQuery } from '@apollo/client`

export const GET_DOGS = gql`
  query GetDogs {
    dogs {
      id
      breed
    }
  }
`
```

`useQuery hook` 을 실행한 `component` 가 `render` 될때, `Appolo client` 로 부터 `loading`, `error`, `data` 를 가진 객체를 반환한다.

이 반환된 객체의 프로퍼티를 `render` 된 `component` 에서 사용가능하다.

> ApolloClient Docs 의 EX<br/>
> useQuery
```ts
import { GET_DOGS } from queries
import { useQuery } from '@apollo/client'

const Dogs = ({ onDogSelected }) => {
  const { loadeing, error, data } = useQuery(GET_DOGS)

  if (loading) return 'Loading...'
  if (error) return `Error! ${error.message}`

  return {
    <select name='dog' onChange={onDogSelected}>
      {
        data.dogs.map((dog) => {
          <option key={dog.id} value={dog.breed}>
            {dog.breed}
          </option>
        })
      }
    </select>
  }
}

export default Dogs

```

여기서 중요한건 각 상태에 따라, 표시할 UI 를 개별적으로 설정가능하다는 것이다.
위에서  `loading` 중일때는 `loading...` 이라는 문구를 `return` 하고, `loading` 이 끝난후, `error` 일때는 해당 `Error message`를 `return` 한다.

그 외에는 `loading` 도 아니며 `error`  도 아니므로, 정상적으로 `data` 를 `return` 해 주면 된다.

이러한 분기처리로 인해 각 상태에따른 `render`가 가능하다.

## Caching query results

`Apollo Client` 는 `query` 를 `server` 로 부터 `fetch` 할때 마다, 자동적으로 그 결과를 로컬에 `cache` 한다.

`cache` 된 `data` 는 이후 같은 `qeury` 를 실행할때 빠르게 실행할 수 있도록 만든다.
> `Apollo Client` 는 `local` 상에 저장해 놓았다가, 같은 `qeury` 가 발생하면 `server` 로 부터 `fetch` 하지 않고, `local` 상에 미리 `caching` 한 `data` 를 사용하는것으로 이해되었다.

`ApolloClient Docs` 에서의 내용처럼 `DogPhoto` 를 받는다는 가정하고, 예시를 살펴보자.

```ts

const GET_DOG_PHOTO = gql`
  query Dog($breed: String!) {
    dog(breed: $breed) {
      id
      displayImage
    }
  }
`

const DogPhoto = ({ breed }) => {
  const { loading, error, data } = useQuery(GET_DOG_PHOTO, {
    variables: { brees }
  })

  if (loading) return null
  if (error) return `Error! ${error}`

  return (
    <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
  )
}

```
위의 예시는 이전 `select` 예시와 연관선상에 있으며, `select` 로 선택된 `breed` 가 `DogPhoto` 의 `props` 로 전달된다고 가정한다.

위의 예시를 보면 `useQuery` 를 통해 `GET_DOG_PHOTO` 를 `query` 하고 있는것을 볼 수 있다.
이때, `variables` 를 사용하고 있는데, 이는 `useQuery hook` 에 제공되는 설정 옵션이다.
> `variables` 옵션은 `GraphQL query`시 원하는 변수를 전달해주는 `object` 이다.<br/>
> 여기서는 `GraphQL query` 의 `dog(breed: $breed)` 에서 `breed` 에 해당 값을 전달해주는 역할을 한다.


만약 `Dogs` 컴포넌트에서 `bulldog` 을 선택했다면, `bulldog` 이 `DogPhotp` 의 `props` 로  전달되며, `bulldog` 사진이 약간의 시간이 걸리며 표시될것이다.

그런 다음, 다른 `breed(품종)` 으로 전환한 다음 다른 `dogPhoto` 에 사진을 로드한이후 다시 `bulldog` 으로 전환해보자. `bulldog` 사진이 즉각적으로 로드 되는것을 볼 수 있다.

이것이 바로 `chach` 된 데이터를 가져온 것이다.

이에 대한 예시는 `Docs` 에서 다음의 [codepen](https://codesandbox.io/s/queries-example-app-final-nrlnl?file=/src/index.js) 사이트를 알려주므로 어떻게 작동되는지 실제 코드로 확인 가능하다.

한번 선택했던 사진은 `loading` 될때 시간이 걸리지 않고, 즉각적으로 불러오는것이 확인되었다.

이러한 `cache` 에 대한 부분을 아는것은 추후 `application` 에 대한 비용 절감은 물론, `사용자 경험 개선` 역시 고려해서 만들기 용이해 보인다.

다음은 `cache` 된 데이터를 `refresh(최신상태)` 하기 위한 몇가지 `technique` 을 알려준다고 한다. 

## Updating cached query results

우리는 `cached data` 를 `server data` 로 업데이트 하기를 원할때가 있다. 
그러한 상황에 맞추어 `Apollo Client` 는 2가지 전략을 제공한다.

그 전략 두가지를 `polling` 그리고 `refetching` 이라 명한다.

### Polling

`Polling` 은 `일정한 주기` 에 위한 `query` 라고 보면된것 같다.
마치 `javascript` 에서 `setInterval` 과 같이, 지정된 `간격(interval)` 에 따라 `query` 가 실행되어, `server` 로 부터 `near-time synchronization(실시간 동기화)` 가 제공된다.

간단히 말해서 지정한 시간에 따라 반복적으로 `server` 로 부터 `data` 를 요청해서 `server` 의 `data` 로 지속 업데이트하는 것을 말한다.

> Polling 이란?<br/>
>
> 충돌 회피 또는 동기화 처리등을 목적으로 다른 장치의 상태를 주기적으로 검사해서 일정한 조건을 만족할때 송수신등의 자료처리를 하는 방식을 말한다.
>
> 여기서는 동기화 목적에 위해 주기적으로 서버로 부터 데이터를 받는 방식을 뜻하는 듯 하다.

`useQuery hook`  와 함께 `miliseconds` 로 `interval` 되는 `pollInterval` 설정옵션이 제공된다.

다음을 보자

```ts

const DogPhoto = ({ breed }) => {
  const { loading, error, data } = useQuery(GET_DOG_PHOTO, {
    variables: { breed },
    pollInterval: 500, // miliseconds 로  0.5 초 이다
  })
}

```

위의 설정대로라면 `0.5` 초 마다 서버로 부터 `breed` 에 대한 `GET_DOG_PHOTO` 쿼리가 실행되어 매번 `images` 는 `update` 된다.

만약, `pollInterval` 을 `0` 으로 설정하면, `poll(주기적으로 query)` 하지 않는다.

> 추가적으로 `polling` 을 `start` 하고 `stop` 을 `dynamic` 하게 하기 위해 [startPoliing 과 stopPolling](https://www.apollographql.com/docs/react/data/queries/#startpolling) 이라는 `functions` 를 제공한다고 한다. 


### Refetching

`refetch` 는 특정 사용자의 `어떠한 동작(Button 을 클릭하는 등...)` 에 의해 `query` 결과를 새로고침한다.  

이는 지속 반복하는 `interval` 과 반대되는 역할을 한다고 볼 수 있다.

`refetch` 는 `useQuery hook` 의 `method` 로써 `refetch` 함수를 가져올 수 있다.

여기서 `refetch` 를 사용할때 `Object` 를 `argument` 로 줄 수 있는데, 이 인자는 `useQuery`  의 `variables` 의 역할을 한다.

만약, `refetch` 할때, 따로 `variables object` 를 주지 않는다면, 이전 `query` 시 실행했던 `variables` 로 `query` 한다.

```tsx

function DogPhoto({ breed }) {
  const { loading, error, data, refetch } = useQuery(GET_DOG_PHOTO, {
    variables: { breed },
  });

  if (loading) return null;
  if (error) return `Error! ${error}`;

  const refetchQuery = (): void => {
    void refetch({ breed: 'new_dog_breed' }) // refetch  의 object 는 variables object 역할을 한다.
  }

  return (
    <div>
      <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
      <button onClick={refetchQuery}>
        Refetch new breed!
      </button>
    </div>
  )
}

```

버튼 클릭시 `UI` 는 새로운 `dog photo` 에 위해 업데이트 된다.
`refetch` 는 `data` 를 새로고침하는 것을 보장해주는 좋은 방법이지만, 상태를 `loading` 하는데 약간의 복잡한부분이 존재한다고 한다.

이러한 부분은 `Docs` 에서 `complex loading(복잡한 로딩)` 과 `error state` 로 부터 핸들링하는 전략을 다룬다고 한다. 

> complex loading 이라고 하는데, `복잡한 로딩`  이라는 해석이 맞는지 의문이다..

### Providing new variables to refetch

```tsx 
  const { loading, error, data, refetch } = useQuery(GET_DOG_PHOTO, {
    variables: { 
      breed, 
      etc: ....
    },
  });

<button
  onClick={() =>
    refetch({
      breed: 'dalmatian', // 위의 variables 는 breed 와 etc 가 있지만, refetch 시 breed 만
                          // 설정해 주었다. 이러한 경우 variables 는 breed  만 `dalmatian`
                          // 으로 변경하고, etc 는 이전값 그대로 사용한다. 
    })
  }
>
  Refetch!
</button>

```

원래 `query` 에 모든 값이 변경되지 않고, 일부만 새로운 값으로 할당해준다면, `refetch` 는 변경된 값을 제외한 생략된 각 변수의 원래 값을 사용한다는점을 주의하자.

## Inspecting loading states

`useQuery` 시 `state` 가 `loading` 중일때, `loading` 을 사용하여 처리했다. 이는 `query` 를 처음 로드할때 유용하게 사용된다. <br/>
하지만 `reretching` 되거나 `polling` 될때의 상태를 나타낼수 있을까? 

답은 ***존재한다.***

`refetch` 상태인지, `polling` 상태인지 확인하는 방법은 `networkStatus` 프로퍼티를 통해 확인가능하다.

> `Docs` 에서는 `networkStatus` 를 `fine-grained information` 이라 칭한다. <br/>
번역을 해보면 `세분화된 정보` 를 뜻하는데, `status` 를 알아보기 쉬운 정보로 제공하는 방식을 말하는 듯 하다.<br/>실제로 아래의 `code` 를 보면 각 상태를 `property` 로 제공하여 알기 쉽게 만들었다.

`networkStatus` 를 활용하기 위해 `useQuery` 시 `notifyOnNetworkStatusChange` 옵션을 `true` 로 설정한다. 이는 `refetch` 가 진행되는 동안 `query component` 가 리렌더링 되도록 한다.

```tsx

import { NetworkStatus } from '@apollo/client';

function DogPhoto({ breed }) {
  const { loading, error, data, refetch, networkStatus } = useQuery(
    // useQurey 시 networkStatus property 를 사용하여 각 상태를 알수 있다.
    GET_DOG_PHOTO,
    {
      variables: { breed },
      notifyOnNetworkStatusChange: true,
    }
  );

  if (networkStatus === NetworkStatus.refetch) return 'Refetching!';
  // networkStatus 가 NetworkStatus.refetch 라면 `Refetching!' 을 리턴하는 로직이다.
  // NetworkStatus.refetch 로 refetch 상태라는 것을 잘 알수 있다.
  if (loading) return null;
  if (error) return `Error! ${error}`;

  return (
    <div>
      <img src={data.dog.displayImage} style={{ height: 100, width: 100 }} />
      <button onClick={() => refetch({ breed: 'new_dog_breed' })}>
        Refetch!
      </button>
    </div>
  );
}

```

`NetworkStatus` 의 프로퍼티는 `emum` 으로 되어 있다.
이에대한 것은 [source code](https://github.com/apollographql/apollo-client/blob/main/src/core/networkStatus.ts) 에서 각 상태에 따른 값을 확인할 수 있다.


## Inspecting erropr states

`error` 를 커스텀하게 처리 위해 `useQuery` 에서 `errorPolicy` 설정 옵션을 제공하고 있다
`default value` 는 `none` 으로 되어있다. 

이 옵션은 `Apollo Client` 에게 모든 `GraphQL 오류들` 을 `runtime` 에러로 처리하라고 알려준다.

이러한 경우, `Apollo Client` 는 `server` 로 부터 받은 모든 응답 `data` 를 버리고, `useQuery` 결과 객체에 `error` 프로퍼티를 설정하도록 한다.  

만약, `errorPolicy` 를 `all` 로 설정한다면, `useQuery` 는 `query` 응답 데이터를 버리지 않으므로, 부분적인 결과를 `render` 할 수 있다. 

이에 대해서는 다음의 [ Handling operation errors ](https://www.apollographql.com/docs/react/data/error-handling/) 를 참고하자.

> Handling operation errors 는 추후 지속적으로 공부하면서 보도록 한다.

## Manual execution with useLazyQuery

`useQuery` 는 `React` 가 `component` 를 렌더할때 호출한다.
그러나 만약 `button` 클릭 같은 특정한 이벤트시 `query` 를 실행하기 원한다면 어떻게 해야 할까?

이러한 상황에서 처리가능한 것이 바로 `useLazyQuery hook` 이다.

`useLazeQuery` 는 `component rendering` 을 제외한 `event` 응답으로 `query` 를 실행한다.

`useQuery` 와 달리 연관된 `query` 를 즉각적으로 실행시키지 않는다. 대신 `query` 를 실행할 준비가 될때마다 호출할 수 있는 결과 `tuple` 을 반환한다.

> Tuple 은 주어진 목록과 관계있는 속성값의 모음이다. `Array` 와 비슷하지만, `Tuple`  은 값을 생성, 삭제, 변경될 수 없다.

```tsx

import React from 'react';
import { useLazyQuery } from '@apollo/client';

function DelayedQuery() {
  const [getDog, { loading, error, data }] = useLazyQuery(GET_DOG_PHOTO);

  if (loading) return <p>Loading ...</p>;
  if (error) return `Error! ${error}`;

  return (
    <div>
      {data?.dog && <img src={data.dog.displayImage} />}
      <button onClick={() => getDog({ variables: { breed: 'bulldog' } })}>
        Click me!
      </button>
    </div>
  );
}


```

`useLazeQuery` 의 tuple 중 첫번째 `item` 으로 `query function`인 `getDog` 를 리턴한다. 그리고 두번째 `item` 은 `useQuery` 에 의해 `return` 된 `object` 결과와 비슷하다. 

여기서 `useLazyQuery` 자체에 `option` 값을 전달할수 있으며, 위처럼 `query function` 자체에 `option` 전달도 가능하다.

만약 `useLazyQuery` 와 `query function` 둘다 옵션값을 주었을대, `query function` 에 주어진 옵션이 우선시한다.

이렇게 된다면, `useLazyQuery` 에 `default option` 을 설정해주고, `query function` 에 새 옵션을 덮어씌워 간편한 사용이 가능하다.

이에 대해서 모든 제공가능한 옵션을 다음의 [API reference](https://www.apollographql.com/docs/react/api/react/hooks/#uselazyquery) 에서 확인 가능하다

## Setting a fetch policy

기본적으로 `useQuery hook` 는 요청한 모든 데이터가 이미 로컬에서 사용가능한지 확인하기 위해 `Apollo Client cache` 를 확인한다. 

만약 모든 `data` 가 로컬상에 존재한다면, `useQuery`  는 `GraphQL server` 에 `query` 하지 않고 로컬상의 `data` 를 반환한다.

이러한 `cache-first` 정책은 `Apollo Client` 의 default `fetch policy(fetch  정책)` 이다.

다른 `fetch policy` 를 지정할 수도 있다.
그러기 위해서는 `fetchPolicy` 옵션을 `useQuery` 에 포함해야 한다

```ts

const { loading, error, data } = useQuery(GET_DOGS, {
  fetchPolicy: 'network-only', // network 요청을 하기전에 cache 를 확인하지 않도록 하는 policy
});

```

## nextFetchPolicy

`nextFetchPolicy` 는 처음에 `fetchPolicy` 가 사용되고, 그이후 `cache update` 를 어떻게 처리할지 결정하는 `option` 이다

```ts

const { loading, error, data } = useQuery(GET_DOGS, {
  fetchPolicy: 'network-only', // Used for first execution
  nextFetchPolicy: 'cache-first', // Used for subsequent executions
});

```

위의 상황은 처음에 `network-only policy` 로 실행한다.  
그 이후, `cache` 로 부터 읽도록 한다.

이러한 `policy` 는 굉장히 유용하게 사용될 수 있을것 같다

### nextFetchPolicy functions

만약 기본값으로 모든 `query` 에 `nextFetchPolicy` 를 적용하기 원할수 있다.
이러한 경우 `ApolloClient` 에 `defaultOptions.watchQuery.nextFetchPolicy` 에 설정할 수 있다. 

```ts

new ApolloClient({
  link,
  client,
  defaultOptions: {
    watchQuery: {
      nextFetchPolicy: 'cache-only',
      // 모든 client.watchQuery 및 useQuery 호출시 다른 `nextFetchPolicy` 를 설정하지 않는다면 기본적으로 적용된다
    },
  },
});


```

> watchQuery 란? <br/>
> mutation 이 일어날때 혹은 update 가 일어날때 데이터를 보고 있다가 자동적으로 query 를 해주는 method 이다.

이렇게 설정하면 해당 `nextFetchPolicy` 를 설정할 수 있다
그렇지만 이러한 설정도 마음에 들지 않으며, 더 커스텀하게 만들고 싶다면 `nextFetchPolicy` 를 함수로써 제공하면 된다.

```ts

new ApolloClient({
  link,
  client,
  defaultOptions: {
    watchQuery: {
      nextFetchPolicy(currentFetchPolicy) { // currentFetchPolicy 는 fetchPolicy 이다 
        if (
          currentFetchPolicy === 'network-only' ||
          currentFetchPolicy === 'cache-and-network'
        ) {
          // 만약 currentFetchPolicy 가 network-only 또는 `cache-and-network` 가 맞다면
          // nextFetchPolicy 를 caceh-first 로 한다.
          return 'cache-first';
        }
        //  그외의 것이라면 nextFetchPolicy 를 currentFetchPolicy 로 한다.
        return currentFetchPolicy;
      },
    },
  },
});

```

위처럼 하면 처음 `fetchPolicy` 가 `network-only` 및 `cache-and-network` 를 통해, `data` 를 `grapyQL server` 에서 가져온이후  `cache` 를 업데이트한다.

이렇게 `fetchPolicy` 가 처리된 이후 `nextFetchPolicy` 에 의해 `cache-first` 정책이 실행된다.

만약 처음 시작하는 `fetchPolicy` 가 `network-only` 혹은 `cache-and-network` 가 아니라면, `nextFetchPolicy`는 기존의 `fetchPolicy` 를 지속 유지한다.

> `network-only` 는 무조껀 server 에 query 해서 data를 가져온다. 그런이후 cache 를 update 한다.
<br/>이는 server 와 일관성있게 유지하는데 도움이 되지만 cache 된 data 를 사용하지 않으므로 즉각적인 응답이 이루어지기 어렵다

> `cache-and-network` server 와 cache 모두에 query 를 실행한다. 만약 server 쪽 결과가 cache 된 필드를 수정하면 쿼리가 자동적으로 업데이트된다. <br/>
즉각적인 응답을 제공하는 동시에 캐시된 데이터를 서버 데이터와 일관성 있게 유지하는데 도움이 된다.

각 요청이후에 호출되는 것 이외에도, `nextFetchPolicy` 함수는 `variables` 가 변경될때도 호출된다.

이때 문제가 생긴다. 새로운 `variables` 가 발생했다면, `query` 역시 변경될수 있다.
그러므로 `fetchPolicy`를 기본값으로 재설정한다 

즉 처음에 `cache-and-network` 또는 `network-only` `fetch policy` 로  시작된 `query` 로 새로운 `network` 요청을 실행 할수 있도록 함수 구성을 다시 만든다.

이렇게 구성된 `fetchPolicy` 이후, 다시 `nextFetchPolicy` 가 이루어지도록 만들어진다고 한다.

이때 해당 구성을 변경할 수 있는데 이때 사용되는 것이 `NextFetchPolicyContext` 이다.
`NextFetchPolicyContext` 는 `variables-changed` 일 경우 가로채서 기본 동작을 변경 할 수 있다

`NextFetchPolicyContext` 의 구성은 다음과 같다

```ts

new ApolloClient({
  link,
  client,
  defaultOptions: {
    watchQuery: {
      nextFetchPolicy(
        currentFetchPolicy,
        {
          // nextFetchPolcy 함수가 호출될때 두가지 경우이다.
          // after-fetch(fetch된 이후) 혹은 variables-changed(variables 가 변경) 될때,
          // reason 은 nextFetchPolcy 가 어떠한 사유로 호출되었는지 나타낸다
          // reason 은"after-fetch" 또는 "variables-changed" 둘 중하나를 가리킨다 
          reason,
          // 나머지 옵션이라고 한다. 
          // options.fetchPolicy 가 있으며, 이 fetchPolicy 가 현재 실행되는 fetchPolicy 인듯하다. 이 부분은 더 찾아봐야겠다.
          options,
          // nextFetchPolcy 가 처음으로 적용되기 전에, 원래 값인 options.fetchPolicy 이다.
          // 즉 fetchPolicy 인듯하다
          initialPolicy,
          // client.watchQuery 호출과 함께 연관된 observableQuery 이다
          observable,
        }
      ) {
        // variables 가 변경될때, 기본적으로 option.fetchPolicy 를 context.initialPolicy
        // 로 재설정한다. 
        // 만약 이 로직이 무시된다면, nextFetchPolcy 함수가 기본동작을 덮어씌워버린다 
        // 역시서 해석상 헷갈리는게, 밑의 로직이 기본적으로 구성되어 있는 로직인지, 아니라면
        // 변경한 로직인지 해석이 잘 안된다.
        // 이부분은 나중에 더 자세히 파고 들어야 겠다.
        //  
        if (reason === 'variables-changed') {
          return initialPolicy;
        }

        if (
          currentFetchPolicy === 'network-only' ||
          currentFetchPolicy === 'cache-and-network'
        ) {
          // Demote the network policies (except "no-cache") to "cache-first"
          // after the first request.
          return 'cache-first';
        }

        // Leave all other fetch policies unchanged.
        return currentFetchPolicy;
      },
    },
  },
});

```

## Supported fetch policies

| name | description |
| :--- | :--- |
| cache-first | Apollo client 는 먼저 cache 에 대해 쿼리를 시작한다.<br/> 만약 모든 데이터가 요청되었다면 `cache` 안에서 제공된다. 그렇지 않다면 `Apollo client` 는 `server` 로 `query` 를 보낸이후 `data` 를 `cache` 한후 반환한다 <br/> 이는 default policy 이다
| cache-only | Apollo Client 는 오직 cache 에서만 query  한다. 적대로 server 에 요청하지 않는다. <br/> 만약 data 가 없다면 error 를 발생시킨다
| cache-and-network | Apollo Client 는 cache 와 server 에 모두 query 를 실행한다.<br/>만약 server query 가 cache 된 field 와 다르다면 자동적으로 query 를 업데이트한다<br/>이는 빠른 응답과 함께 server data 와 cached data 를 일관성있게 유지시켜준다
| network-only | Apollo Client 는 server 에만 query 를 실행한다. <br/>그 이후 cache 를 업데이트한다. <br/>server data 와 cahced data 간의 일관성을 유지하지만 매번 server 에 요청하여 data 를 받으므로 즉각적인 응답이 어렵다|
| no-cache | `network-only` 와 비슷하지만, cache 하지 않는다.
| standby | `cache-first` 와 비슷한 로직으로 사용되지만 기본 필드값이 변경될 때 자동적으로 업데이트 하지 않는다. 이는 refetch 혹은 updateQueries 를 통해 수동으로 업데이트 가능하다