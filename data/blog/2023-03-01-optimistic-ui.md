---
title: 'Browser 주소창의 실체'
date: '2023-03-01'
tags: ['next.js']
draft: false
summary: 'Browser 주소창의 실체'
---

# 옵티미스틱 UI

> 요청하는 값이 무조건 존재한다고 가정하고 `client단` 에서 미리 값을 주는 방법이다.

`Optimistic UI` 를 사용하면 기존의 방식과 다르게 작동한다.

1. 중용한 곳에 사용 x
   2, 덜 중요하면서 , 실패해도 문제가 안되는 곳에 사용

# CSSTrigger

> css 사용시 성능 최적화를 위해 참고하기 좋은 사이트이다.

# 주소창의 실체

> 브라우저 주소창은 `REST API`의 `GET` 요청이다.
> 그러므로, `curl` 으로도 요청이 가능하다는것을 알 수 있다

```shell
curl -X GET "https://www.naver.com"
/* // 출력결과
<!doctype html><html lang="ko" data-dark="false">...
*/

```

위의 `GET` 요청을 보면, `HTML Code` 를 보내주는것을 확인할수 있다.
실제 `Address` 는 `REST GET` 을 통해 `Data` 를 받아오는 도구이다.

`koreanjson.com` 같은 주소를 사용하면, 해당 `Json` 결과를 받아온다.

하지만, 주소가 단순히 `REST API`의 `GET` 요청이라고 한다면, `JSON` 이라고 하더라도 브라우저에 결과를 보여줄수 있을까?

가능하다.
단, `HTML` 이 아니기에 `String` 방식으로 화면에 표시한다.
실상 `HTML` 은 `MarkUp Language` 이기에 `Brower` 는 특별히 구조화된 화면을 보여줄뿐, 실제 문자열이라고 봐도 된다.

실제 `koreanjson.com` 에서 받아온 `data` 는 다음과 같다

```shell

> curl -X GET 'https://koreanjson.com/posts/1'

/* // 출력결과
{"id":1,"title":"정당의 목적이나 활동이 민주적 기본질서에 위배될 때에는 정부는 헌법재판소에 그 해산을 제소할 수 있고, 정당은 헌법재판소의 심판에 의하여 해산된다.","content":"모든 국민은 인간으로서의 존엄과 가치를 가지며, 행복을 추구할 권리를 가진다. 모든 국민은 종교의 자유를 가진다. 국가는 농·어민과 중소기업의 자조조직을 육성하여야 하며, 그 자율적 활동과 발전을 보장한다. 모든 국민은 양심의 자유를 가진다. 누구든지 체포 또는 구속을 당한 때에는 즉시 변호인의 조력을 받을 권리를 가진다.","createdAt":"2019-02-24T16:17:47.000Z","updatedAt":"2019-02-24T16:17:47.000Z","UserId":1}
*/

```

최근의 `Webserver` 는 다음과 같은 역할을 한다

```
               |-  <---> jsonData/html/json [backend server]
[브라우저] --- |
               |-  <---> /html [frontend server]
```

`Frontend server` 에 `HTML` 같은 `page file` 을 요청하고, `backend server` 에는 `data` 를 요청하는 방식으로 많이 사용된다.

`Frontend server` 역시 `SSR` 이 가능하며, 이를 위한 `Server` 이므로 `Backend` 처럼 사용가능하다.
`Next.js` 의 큰 장점이 바로 이러한 `Frontend server` 환경을 만들어주어, 기존의 방식을 `Frontend` 의 방식으로 구현한 것이다.

이는 `Frontend` 전용 서버이므로 다음처럼 요청도 가능하다

```

curl -X GET '/page/test'
/* /page/test 에 요청해서 `data` 역시 받을 수 다다.*/

```

즉, `Frontend server` 역시 `REST API` 이다.

# OpenGraph

> opengraph 역시 html 메타 태그 종류중 하나이다. SNS 에서 해당 링크를 클릭하기 전에 어떠한 링크 데이터를 가지고 있는지 할수 있도록 인의적으로 만들어진 메타테그 이다

`OpenGraph` 는 `meta` 태그 설정시 `property` 를 `og:[name]` 으로 설정한다.

이렇게 설정된 `property` 에 `content` 를 통해 내용을 집어넣는 방식이다.

다음을 보자

```tsx
// /page/ogPage
<>
  <Head>
    <meta property="og:title" content="중고마켓" />
    <meta
      property="og:description"
      content="나의 중고마켓에 오신 것을 환영합니다."
    />
    <meta property="og:image" content="https://..." />
  </Head>
<>
```

위와 같이 각 `property` 와 `content` 를 설정하여 제공할 내용을 기입한다.

개념자체는 어렵지 않다.
이제 `og:[name]` 이 있는 `meta` 태그를 찾고, 해당 `content` 를 가져와 보여주면 된다.

이러한 방식을 `scraping` 이라고 한다.

## Scraping 과 Crowling

### Scraping

> `긁다` 라는 뜻을 가졌으며, 한번만 사이트의 정보를 수집하여 가져온다.

사용하는 라이브러리가 있으며 'cheerio' 라는 라이브러리를 많이 사용한다

### Crowling

> `헤엄치다` 라는 뜻을 가졌으며, 특정한 시간에 매번 사이트의 정보를 수집하여 가져온다.

사용하는 라이브러리가 있으며 'crowler' 라는 라이브러리를 많이 사용한다

위의 개념을 단순히 생각하자면, `html` 을 긁어오는 것이다.

```tsx
const onClickEnter = async (): Promise<void> => {
  // 1. 채팅data 의 주소가 들어가 있는지 찾기, (http~~ 로 시작하는 것)

  // 2. 해당 주소로 스크래핑 하기
  const result = await axios.get('http://localhost:3000/section32/32-01-opengraph-provider')
  // console.log(result.data);
  // CORS: https://www.naver.com

  // .3 메타태그에서 오픈그래프(og:) 찾기
  console.log(result.data.split('<meta').filter((el: string) => el.includes('og:')))
}
return <button onClick={wrapAsync(onClickEnter)}>채팅 입력후 엔터키!!!</button>
```

위처럼 해당 내용을 불러온다.
그리고 `og` 를 찾아 반환하면 된다.

이렇게 만들어진 `page` 라면 `scraping` 하면된다.
하지만, `page` 가 동적이라면?

## Dynamic OpenGraph

`next.js` 는 각 `page` 별로 동적으로 구성 가능하다.
즉, 이말은 `page` 의 내용이 그때그때 생성된다는 것이다.

# SSR

> `Server Side Rander` 의 줄임말,
