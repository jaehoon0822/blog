---
title: '5월 1째주 회고'
date: '2023-05-08'
tags: ['retrospective', '회고']
draft: false
summary: '5월 1째주의 회고'
---

# 5월 1째주 회고

## Facts

이번주는 주로 `Backend` 관련 해서 공부해 보는 시간을 가졌다.
`express` 에 대한 개념은 금방 잡혀서 `Code` 에 대한 이해가 가는 반면, `Nest` 로 넘어가니 객체지향 관련된 내용이 많고,
`Nest` 를 쓰기위한 방식들이 많아서 개념을 잡아가는데 약간의 시간이 걸릴것 같다

## Feelings

### Express

`Node` 진영에서 빼놓을 수 없는 `Framework` 는 역시나 `Express` 이다.
`Express` 를 보며, 개념적인 부분이 많이 정리되어가는 상태이다.
하지만, 현재 대세는 `Nest Framework` 로, `Express` 보다 구조화된 방식으로 코드를 짤수 있도록 만들어진 `Framwork` 이다.

`Express` 의 훨씬 발전된 형태로써 사용가능하다
하는김에 `Express` 개념은 간단하게 익히고, `NestJS` 로 넘어간다.

### NestJS

`Express` 를 내부적으로 사용하여 처리되지만, 이러한 `Express` 의 단점을 구조화시켜 훨씬더 거대한 프로젝트에서도 쉽게 구현하고 관리하기 위해 탄생한
`Framework` 이다.

`NestJS` 를 보다보니 객체지향 관련된 개념이 많이 접목되어 있었다.
어쩔수 없이 `Docs` 와 책 그리고 강좌를 보며 익혀가고 있는데, 몇몇 개념들이 쉽게 와닿지가 않아서 익숙해지기 위해 노력할 필요가 느껴졌다

`DI` 라든지, `IoC`, `SOLID` 라든지..

무엇보다 `Decorator` 를 적극활용하여, `Class`, `Method`, `Mothod parameter` 등을 사용할 목적에 맞도록 기능을 변경하거나, 확장시킨다 한다.
확실히 `Decorator` 의 기능을 이렇게 사용하는것을 보고 굉장히 유용해 보였다.

그리고 무엇보다 `Component` 단위로 정리되는 것을 보면서, 객체지향의 힘을 조금은 느끼는바가 있었다

### Finding

### express 를 하면서 배운점

`Express` 를 통해 `Endpoint` 를 지정하고 지정된 `Endpoint` 를 사용하여 `HTTP Method` 에 따른 동작을 구현하는것은 어느정도는 이해했다
하지만, 중간에 `middelware` 라고 하는부분이 `Express` 를 쓰면서 정수가 아닐까 싶어 이부분에 대해서 조금더 공부를 해보았다

`Middleware` 는 간단하게 `Request` 및 `Respons` 를 중간에서 가공해서 처리해준다.
이 말은, 사실 `Express` 에서 제공해주는 `Method` 들이 알고보면 `Middleware` 들이라는 것을 알 수 있다

이러한 `Middleware` 에 대한 모음을 통해, 요청에 대한 것을 `Middleware` 를 거쳐서, `Router` 에서 받고, `Router` 에서 `Client` 로 응답할때도, `Middleware` 을 거쳐 응답하게 된다
`Middleware` 는 요청 응답이 거쳐가는 흐름이며, 그 흐름상에서 그 `Data` 에 대한 가공도 가능하다

이로인해, 원하는 `Middleware` 를 작성하거나, `Library` 로 받아서 `Middleware` 로 등록만 하면 쉽게 사용가능하다
또한 `HTTP` 에 대해서는 깊이 공부하지는 않앗지만, 이러한 `Method` 들이 어떠한 흐름으로 구성되어 있는지에 대해 조금이나마 알 수 있게 되었다

추가적으로 `Websocket` 이 어떠한 방식으로 이루어져 있는지도 알아보게 되었다.
`Websocket` 관련해서는 개념만 공부했으며, 추후 `Socket.io` 에 대해서 필요하다면 내용을 정리해서 구현해볼 생각이다

### NestJS 를 하면서 배운점

`NestJS` 를 보며, 객체지향에 대해 알아야 겠다는 생각이 많이 들었다.
`React` 에서 잘 사용하지 않는 `Decorator` 를 적극적으로 사용하기에 조금더 자세히 공부해 보았다.
`Decorator` 를 조금더 살펴보기는 해야할것 같다. 내가 커스텀하게 입맛에 맞도록 구성하는것은 해보면서 익혀야 할것으로 느껴졌다.

`Decoratro` 패턴을 사용하면서, `DI` 및 `IoC` 의 개념에 대해서도 알게 되었다
`DI` 는 실상 `React` 에서도 함수 컴포넌트를 `Child` 로 넘겨서 사용하는 방식과 비슷해 보였다

그래서 이해하는데 쉽게 이해는 갔지만, `IoC` 에 대한 개념은 뭔가 용어 부터가 괴랄맞았다
`제어의 역전` 이라는 단어가 정말 와 닿지 않는 용어이다.

> 영어를 한국말로 해석하면서 이런 괴랄맞은 용어들이 생겨나는것 같아서, 참 씁쓸하다. 이런것은 그냥 영어로 아는것이 좋을듯 싶다...
> 사실 한국말로 해석한 이 구문이 내용을 알고보면 맞기도 하고, 무엇보다 한국말로 번역하면서 이 용어도 얼마나 고심해서 지은것일까 생각하면 뭐라 말은 못하겠다....

`IoC` 는 주입된 컴포넌트를 개발자가 직접 제어하지 않는다.
이 주입된 컴포는터는 다른 모듈의 컴포넌트 및 `Library` 로 따로 구분되며, 개발자가 작성하는 코드와의 의존성이 현저히 낮다.

그러므로, 주입된 컴포넌트에게 현재 사용하는 `Class` 및 `Module` 의 `Controll`(제어권이라고 하기에도 모호하다. 그냥 `Controll 이라고 하겠다.`) 을 맡기는것이다.
개발자는 더이상 주입받은 컴포넌트에 신경쓰지 않아도 되며(잘 작성된 컴포넌트라면...), 현재 자신의 코드만 신경쓸 수 있도록 해주는 방법론이라고 이해했다

`Nest` 는 이러한 `IoC` 및 `DI` 를 지원하기 위해, 내부적으로 `Nest IoC Container` 에 의해 관리 되어 있다고 말하고 있다
`IoC Container` 는 제공된 `Providers` 의 의존성 주입을 통해 다른 클래스와 `relationships` 을 맺을 수 있도록 해준다.
이러한 방법으로 `Providers` 의 `metadata` 를 분석하고 이 에 따른 의존성 그래프를 생성하여, `Provier` 를 인스턴스화 시키고 주입시킨다.

이렇게 인스턴스화 시키는 과정상 필요한것이 `@Injectable` 데코레이터이다.
여기까지만 보더라도, `Service Logic` 하나 제공하는데 필요한 `Provider` 에 대한 개념을 조금 많이 알아야 겠다는 생각도 들었다

이것만으로 끝나지 않고, 이러한 `Provider` 를 커스텀하게 생성하는것도 존재하는데, 이부분은 추가적으로 더 깊이 공부해야할 필요성을 느낀다

간단하게 말하자면, `Nest` 는 `Contollers` 와 `Providers`, 그리고 이 `Controllers` 와 `Providers` 를 모은 하나의 `Module` 로 이루어진다.
이러한 `Module` 을 통해 각 부분적으로 코드가 분할되며, 마치 `React` 의 컴포넌트들 처럼 `Root Module` 에서 모듈들을 가져와 `Application` 을 만들어낸다.

이때 중요한 개념중 하나가, 모든 `Service Logic` 들은 `Module` 범의로 `encapsulate` 되어 있는것이다.
즉, 은닉화 된다. 그러므로 특정 `Module` 의 `Service Logic` 을 다른 `Module` 에서 사용해야 한다면, 해당 `Provider` 를 `export` 해주어야 한다

그런후 해당 `Module` 을 다른 `Module` 에서 `imports` 한후, 해당 `Service Logic` 을 사용하면된다.
해당 `Module` 에서 이미 `Provider` 를 `exports` 했기때문에, 다른 `Module` 에서 `imports` 된 `Module` 의 `Provider` 가 자동적으로 `Injection` 되므로
따로 `Poviders` 에 기입하지 않아도 된다

사실 `Providers` 및 `Module`, 그리고 `Controller` 관련해서 추가적으로 제공해주는 기능은 여전히 많다
이부분에 대해서는 `Docs` 의 `Fundamentals` 를 더 살펴보며 익혀야 할것 같다.

현재까지 본 기본적은 개념이 이와 같았다.
이러한 기본적인 개념과 함께, `Mongooes` 를 `Nest` 에서 사용하기 위한 `nestjs/mongoose` 및 `configuer` 등등..

`Nest` 에서 제공하는 `IoC` 를 지원하기 위해 약간의 변형된 방법으로 해당 `library` 를 설치한후, 약간의 설정들 역시 필요하다
이뿐만 아니라 `Pipe` 및 `Middleware`, `Filter`, `Guard` 같은 개념도 같이 제공해주며, 이 개념이 `Nest` 내부적으로 처리되는 순서역시 존재한다

> 마치 `React` 의 `Lifecycle` 같은느낌이다.

아직 알아야할 부분이 많기에, 공부하고 `code` 를 작성하면서 지속 정리해나갈 예정이다.
`Frontend` 가 앞단만 하면 되지, 무엇하러 `Server` 를 공부하냐고 한다면...
언제부터 `FrontEnd` 라는 직업군이 앞단만 담당했느냐며 물어보고 싶다...

뒷단을 알아야 앞단에 대한 지식도 덩달아 올라가는 직업군이다.
결국은 둘다 알아야 하며, 여러 방법론들 `OOP, FP` 의 개념도 지속 공부해야할 필요성을 많이 느끼고 있다

특히, `RXJS` 가 자꾸 눈앞에 보이기 시작한다...

## Future Action

지금 현재로써는 `Nest` 를 사용하여, `Backend API` 를 만들고, `REST` 를 통해 `FrontEnd` 를 구현하여 결과물을 만들어내는 것을 목표로 삼고 있다
이왕 하는김에 `Nest` 를 보면서 `Design Pattern` 관련 부분도 조금은 살펴보 생각이며, `RXJS` 내용을 살펴보아야 하는지 고민중에 있기는 하다.

`Nest` 상에서 `RXJS` 를 어느정도 활용하며 사용하는것이 보이며, `ApolloClinet` 에서도 `Error handling` 할때 `toPromise` 를 사용하여 `Reactive Programing` 을 구현하고 있다
온갖가지 라이브러리들이 `Reactive Programing` 을 사용하여 구현하고 있는것을 보면, 해당 부분에 대해 공부하고 익히는것이 맞을 듯 싶다

오늘부터 추가적으로 하루에 1개씩 자료구조 관련 내용을 정리하려고 한다.
자료구조를 살펴보며 구현해보고 알아보는 시간을 가져서, 내용을 정리할 생각이다.
