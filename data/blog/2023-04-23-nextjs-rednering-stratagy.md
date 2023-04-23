---
title: 'nextjs 의 렌더링전략'
date: '2023-04-23'
tags: ['nextjs']
draft: false
summary: 'nextjs 의 렌더링'
---

# nextjs 의 렌더링 전략

> next.js 를 사용하고 있지만, 개념에 대한 정리를 하는것이 좋을 듯 싶다.  
아직 알아야 할 것들이 많아 렌더링 전략에 대해서 자세히 알아본다.

`nextjs` 는 다른 `react`로 만들어진 `framework` 와는 다르게, `static page` 가 아닌 `dynamic page`를 제공한다.

어떤 페이지가 빌드 시점에 정적으로 생성되고, 실행 시점에 동적으로 제공될지 정할 수 있다.

이로 인해, `serverside rendering` 을 지원하며, 각 요청별로 페이지를 렌더링이 가능하다.

또한, 특정 컴포넌트를 클라이언트에서만 렌더링할 수 있다.

이 부분에 대해서 더 알아보자.

## SSR(ServerSideRendering)

서버측에서 `html` 페이지를 만들어서 `client` 로 보내주는 방식을 `server side rendering` 이라고 불린다. 

`nextjs` 에서도 이러한 기능을 제공하는데, 이러한 기능을 왜 제공하는지 알아야 한다.

### `SPA` 의 한계

`react` 하면 빠질 수 없는 것이 바로 `SPA` 이다.
`SPA` 는 단일페이지로 모든 페이지를 보여줄 수 있는 획기적인 방법이다.

`SPA` 는 `server` 에 의존한다기 보다는, `client` 측에서 작동된다.

`SPA` 는 비어있는 정적페이지에, `Javascript` 를 사용하여 페이지를 그려주는 방식을 사용한다.

그러므로, `server` 에서 모든 자원을 받을 필요 없어 비용을 절감할 수 있으며, `client` 측의 남는 자원을 사용하여 작동하므로, 더 빠르고 좋은 서비스를 제공할 수 있다

`SPA` 가 급발전하게 된것도 바로 이 부분이다.
하지만, 이러한 `SPA`도 반드시 `Server` 를 통해 렌더링된 페이지를 받아야만 하는 경우가 생긴다. 

이는 데이터 검증 및 주요 API 등 `Server` 에서만 처리하는 것들과 함께, `SEO` 에도 영향을 끼치기 때문이다.

`SEO` 는 `Web Bot` 이 각 `page` 를 돌면서, 각 `page` 마다 점수를 매기는데, 점수를 높게 받을수록 상위에 노출된다.

이 점수를 매기는 기준이, `SPA` 가 아닌 `SSR` 에 초점이 맞추어져 있다는것이 문제이다.

보통 `Bot` 들은 `Javascript` 를 실행시키지 못한다.
단순히, `HTML` 만을 보고 정보를 수집한다.

앞써서, `SPA` 는 `javascript` 를 통해 `page` 를 그린다고 하였다
바로 이부분이 문제이다.

`SPA` 만 제공한다고 가정하면,  `Server` 로 부터 `HTML`을 받지만, `비어있는 HTML` 을 받을 것이고, 이후 `Javascript` 를 받은 이후에, `HTML` 의 화면을 그려준다.

즉, `Bot`이 `비어있는 HTML`을 이해하게된다는 것이다.
그러므로, `SEO` 는 당연 점수가 낮을수 밖에 없다.

`SEO` 를 위해 `SSR` 이 필요할 수 밖에 없는 상황이다.

### NextJS 의 SSR 방식

`next.js` 는 이러한 부분을 해결하기 위해 `SSR` 을 지원한다.
`next.js` 는 각 요청에 따라 `HTML` 을 동적으로 렌더링하고 `client` 로 전달한다.

이렇게 만들어진 `HTML` 은 `javascript` 가 없는 `page` 이기에, 실상 어떠한 동작도 이루어지지 않는다.

단지, 정적인 `page` 만을 제공하여, `CEO`에 충족한 `page` 만을 제공한다.
이후, `server` 에서 `rendering` 한 페이지에 `script` 코드를 집어넣어서,
`Hydration` 시켜준다.

`Hydration` 이란 정적인 페이지를 동적페이지로 만들어주기위해 `Javascript` 코드를 적용해주는것을 말한다.

이렇게 `Hydration` 과정을 거치면, `Page` 이동시 새로운 `HTML` 을 `Server` 로 부터 요청하지 않고, 동적페이지로 작동 가능하게 된다.

이러한 `Hydration` 기능을 통해 `SSR` 을 지원하는 `SPA` 로써 작동이 가능하게 된다.

이러한 부분으로 인해 `next.js` 는 기본적으로 `build time` 때 정적페이지를 만든다.

### getServerSideProps

만약, 서버를 통해 받아온 데이터를 `client` 에 전달해서 사용해야 한다고 가정해보자.

이럴때, 유용하게 사용할 수 있는 `next.js` 의 내장함수가 `getServerSideProps` 이다.

```tsx
export async function getServerSideProps() {
  const requestData = await fetch('https://server/api/list')
  const data = await requestData.json()

  return {
    props: {
      data
    }
  }
}

const Page(props: IData}) {
  return (
    <div>
      <ul>
        {
          props.data.lists.map((item) => (
            return (
              <li>
                {item}
              </li>
            )
          ))
        }
      </ul>
    </div>
  )
}

```

`getServerSideProps` 함수를 `export` 한다.
`nextjs` 는 `build time` 때, 이 함수를 `export` 하는 모든 `page`를 찾는다.

그리고, `server` 에 `page` 를 요청할때 `getServerSideProps` 를 호출한다.
이때 중요한 것은 `getServerSideProps` 는 항상 `server` 에서 실행된다는 것이다.

이때 `getServerSideProps` 에서 반환하는 `props` 는 해당 `page component` 에 `props` 인자로 전달한다.

이렇게 받은 `props` 인자는 `page component` 에서 사용가능하다.
 `SSR` 을 이런식으로 지원하며, `Server` 를 통해 받은 `data`를 동적으로 받아 `HTML` 을 `redering` 해 보내준다.

> `SSR` 을 사용하여 `page` 처리할때(`getServerSideProps`안에서...), `clinet` 에서 사용하는 `API`는 사용 불가능하다. (ex: window, DOM)

## CSR(ClientSideRendering)

`CSR` 은 필요한 기본 `script` 및 `css`, 기본 `HTML` 마크업만 전송하기에, 비어있는 `page` 를 제공한다.
이렇게 비어있는 `page` 안에는  `root div` 가 존재하는데, 이 `root div`에 해당 `page` 내용을 그려준다.

이로 인해, 새로고침없이 다른 페이지로 이동 가능하다.(`page` 내용을 그려준다.)

### `CSR` 의 단점

`CSR` 은 전체 `CSS` 및 `Javascript` 를 받아야 해서 `server` 로 부터 받는 시간이 오래걸릴 수 있으며, `Javascript` 를 통해 `DOM` 을 기르는데 시간이 걸리수 있다.

그래서, `loding` 하는 부분을 수초동안 보여주는등, `page` 로딩이 느려질 수 있다.

그리고 앞서서 이야기한 `SEO` 에도 좋은 점수를 받기 힘들다.

### 생각보다 귀찮은 `CSR`

`CSR` 은 `browser` 상에서 작동한다.
이말은 `SSR` 이 끝난후, `client` 에 `rendering` 이 완료된 이후를 말한다.

이때, `SSR` 상에서 실수로 `window` 나 `document` 를 사용하여, `error` 가 발생하는 경우가 많은데, 이때는 `useEffect` 나 `dynamic import` 및, `typeof window === 'undefined'` 를 사용하여 처리한다.

이중, `dynamic` 에 대해서는 조금더 알아본다.

### dynamic

`dynamic import` 는 `module` 을 동적으로 가져올때 사용하는 `nextjs` 의 기능이다.

```tsx

import dynamic from 'next/dynamic'

const Page = () => {
  const Quill = dynamic(() => import ReactQuill from 'react-quill', { ssr: false })
  return (
    <div>
      <Quill />
    </div>
  )
}

```

이렇게, `import` 할 `module` 을 가져오면서, 옵션으로 `ssr: false`  를 주면, 마치 `useEffect` 및 `typeof window === 'undefined'` 를 통해 `client` 인지 확인한것 처럼 동작하는것 처럼, 사용가능하다

즉, `rendering` 된 이후 `CSR` 에서 작동하도록 만들어준다.
`CSR` 은 검색엔진에 노출 될 필요없는 페이지를 만드는 경우에 매우 좋다.

## SSG(StaticSiteGeneration)

`SSG` 는 정적 사이트 생성이라고 부른다.
`SSG` 는 `build time`에 `page` 를 미리 렌더링한다.
그러므로, 내용이 변하지 않는 페이지를 만들때 유용하게 사용가능하다.

이는 `SSR` 및 `CSR` 에서 처럼, 내용의 변화가 없으며, 이를 통한 `server` 쪽 `data` 를 요청할 필요가 없으므로 성능에 매우 좋다.

> 필요한 정보가 `build time` 에 `page` 로 전부 렌더링 되어 있기 때문에 그렇다.

이렇게 미리 렌더링되어 있다는것은, 정보 변경을 위해서는 매번 `rebuild` 가 필요하다는 것을 뜻한다.

이를 통해 조그마한 정보역시 리빌드를 해야하는 상황은 좋은 방법은 아니다.
이러한 불편함을 해결해주기위해 `ISR(IncrementaStaticRegeneration)` 을 제공한다.

`ISR` 은 만들어진 정적페이지에 어떤 내용을 업데이트할지, 재 랜더링할지 결정이 가능하다

### `getStaticProps`

`getStaticProps` 함수는 `build` 시, `page` 를 `rendering` 할때 호출된다
이때, `getServerSideProps` 와는 달리, 한번 `data` 를 가져오면 다음 `build` 시에는 호출하지 않는다.

즉, `getStaticProps` 를 다시 호출하려면 다시 `re-build` 해야한다. 딱 봐도 불편해보인다.

이러한 부분을 처리하기  위해 `revalidate` 라는 `option` 을 제공한다.

`revalidate` 는 주기를 지정해서, 언제 `re-build` 할지 정할 수 있다.
값은 숫자를 사용하여 지정가능하다

> `revalidate` 는 초단위로 숫자값을 지정한다. ex: 60 (1분)

이렇게, `revalidate` 를 사용해서 주기를 정하면 해당 주기마다 `getStaticProps` 가 실행되며, 헤당 `data`를 받아 `props` 로 넘겨주어 값 전달이 가능하다.

## 그래서

`SSR` 과 `SSG` 를 통한 처리를 비교 알게 되었다.

`SSR` 은 동적페이지를 생성이 가능하며, `SEO` 및 `server` 측에서 처리해야할 정보를 처리할 수 있을때 사용한다.

`SSG` 는 `data` 에 대한 변경이 많이 없는 `page` 같은경우 사용하기에 좋다.
하지만 새로운 정보를 받기 위해서는 매번 `re-build` 를 해야 해서 불편한 부분이 있다.

이러한 단점을 없애기 위해 `ISR` 을 제공하여, `re-build` 주기를 정하여, 특정 주기마다 `data` 를 새로받아 처리할 수 있도록 한다.

즉, 빌드 시점에 페이지를 만들지, 페이지 요청시점에 만들지를 지정 가능하다고 생각할 수 있다.
