---
title: 'NestJS Controller 정리'
date: '2023-05-01'
tags: ['nestJS']
draft: false
summary: 'NestJS 내용 정리'
---

# NestJS Controller 개념 

> `NestJS` 를 공부하기 전에 그 개념먼저 살펴보도록 한다 

## Controller

`NestJS` 에서 `Controller` 는 `Request` 를 받고 그 결과를 `Response` 하는 `Interface` 의 역할을 한다

이러한 `Controller` 는 `Routing`  매커니즘을 통해서 각 `Controller` 의 `EndPoint` 에 따라 분류가능하다

이러한 `EndPoint` 에 따른 분류를 통해 `Apllication` 을 조금더 구조적으로 작성 가능하다

`NestJS` 는 `Controller` 를 만드는 `CLI` 명령어가 존재하는데 이는 다음과 같다

```sh
yarn nest g controller [ControllerName] 
```

이렇게 만들면 자동적으로 `Controller` 파일이 만들어 진다.
이러한 `CLI` 명령어는 다음의 명령어를 치면 볼 수 있다

```sh
yarn nest -h

      ┌───────────────┬─────────────┬──────────────────────────────────────────────┐
      │ name          │ alias       │ description                                  │
      │ application   │ application │ Generate a new application workspace         │
      │ class         │ cl          │ Generate a new class                         │
      │ configuration │ config      │ Generate a CLI configuration file            │
      │ controller    │ co          │ Generate a controller declaration            │
      │ decorator     │ d           │ Generate a custom decorator                  │
      │ filter        │ f           │ Generate a filter declaration                │
      │ gateway       │ ga          │ Generate a gateway declaration               │
      │ guard         │ gu          │ Generate a guard declaration                 │
      │ interceptor   │ itc         │ Generate an interceptor declaration          │
      │ interface     │ itf         │ Generate an interface                        │
      │ library       │ lib         │ Generate a new library within a monorepo     │
      │ middleware    │ mi          │ Generate a middleware declaration            │
      │ module        │ mo          │ Generate a module declaration                │
      │ pipe          │ pi          │ Generate a pipe declaration                  │
      │ provider      │ pr          │ Generate a provider declaration              │
      │ resolver      │ r           │ Generate a GraphQL resolver declaration      │
      │ resource      │ res         │ Generate a new CRUD resource                 │
      │ service       │ s           │ Generate a service declaration               │
      │ sub-app       │ app         │ Generate a new application within a monorepo │
      └───────────────┴─────────────┴──────────────────────────────────────────────┘
```

다음의 `CLI` 를 사용하여 `Project` 를 생성한다

```sh
yarn nest new [projectName]
```

만들어진 `Project` 를 살펴보면 `src` 폴더가 존재한다
그리고 `src` 내부 안에 `app.controller.ts` 는 다음처럼 구성되어 있다

```ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}

```

`Controller` 데코레이터를 사용하여 `Class` 를 `Controller` 로 만들어준다

또한, `Get` 데코레이터를 사용하여 해당 `controller` 내부의 `method` 를 `HTTP  GET` 요청을 받는 `method` 로 만들어주는 `code` 이다

위는 `Get` 에 아무런 인자를 넣어주니 않아 기본 경로가 `/` 가 된다 
만약, 추가적인 경로를 지정하고 싶다면 `pathName` 형식으로 `Get` 데코레이터의 인수로 넘겨주면, 경로가 된다

```ts
@Get('user')
getHello(): string {
  return this.appService.getHello();
}
```

이제 위의 경로는 `/user` 가 된다
또한 해당 인수는 `string` 으로 지정가능하며, `regexp` 형식의 인수를 넘겨줄 수도 있다
이는 `*`, `?`, `+`, `()` 문자를 사용해, 정규표현식에 해당하는 와일드카드와 동일하게 동작한다

또한 `express` 에서 제공하는 `req` 및 `res` 를 사용할 수도 있다
이러한 `req`, `res` 를 사용하기 위해서는 `controller` 의 `method` 의 인자에, `Parameter Decorator` 를 통해 처리를 해주어야 한다

```ts
@Get()
getHello(@Req() req: Request): string {
  console.log(req)
  return this.appService.getHello();
}
```

`exrpess` 에서는 `req` 객체를 사용하여, `request` 받은 내용을 받아 처리했는데, `NestJS` 에서는 많이 사용하는 객체들을 따로 빼서 사용가능할 수 있도록 편의를 제공해준다

```ts
@Get()
getHello(@Body() body): string {
  console.log(body)
  return this.appService.getHello();
}
```

이 외에서, `@Param`, `@Query` 등의 추가적인 `Decorator` 를 제공한다
`@Req` 뿐만 아니라 `@Res` 역시 존재한다

기본적으로 `NestJS` 는 각 메서드의 응답값에 대한 처리를 알아서 처리한다
원시타입을 받아서 응답값으로 반환할 경우 `Stringify` 하지 않고 응답을 보내지만, `Object` 는 `JSON.stringify` 를 통해 자동 `JSON` 으로 변환한다.

이러한 처리를 원치 않는다면, `express` 에서 제공하는 `res` 처럼 직접 설정도 가능하다
그럴때, `@Req` 데코레이터를 사용하여 지정하여 처리한다

```ts

@Get("/user")
findUsers(@Res() res) {
  const users = this.usersService.findAll()
  return res.status(200).send(users)
}

```

이외에 `HTTP` 코드 역시 다른 `code` 로 보낼수 있도록 제공한다

```ts

@httpCode(202)
@Get("/user")
findUsers(@Res() res) {
  ...
}

```

`Error` 가 발생시 보낼수 있는 `Method` 역시 제공한다
`BadRequestException('내용')` 을 보내면, `Error` 를 던지며, `httpCode` 는 `400` 이 된다

이처럼 각 응답에 따른 처리가 가능하다
`

### Header

`@Header`를 사용하면, `custom header` 생성이 가능하다

```ts
@Header('Custom', 'custom')
@Get(':id')
findUsers(@param('id') id: string) {
  ...
}
```

이렇게 하면, `Header` 상에 `Custom` 의 이름은 `custom` 이 된다

### Redirection

`@Redirect` 데코레이터를 사용하면, 원하는 `Url` 로 `redirection` 될 수 있다
이렇게 `redirection` 될때, 첫번재 인자는 `Url`, 두번재 인자는 `httpCode` 전송이 가능하다

보통은 `301`, `307`, `308` 같은 상태코드를 보내 처리한다
제대로된 상태코드를 사용해야 정상동작되므로, 주의가 필요하다

또한, `Res` 값에 따라, 페이지 이동을 원한다면, 다음의 객체를 `return` 하면 해당 경로로 `redirect` 된다

```ts
{ url: '경로', statusCode: number}
```

다음처럼 사용가능하다

```ts
@Get('/path')
@Redirect('url', 302)
getUser(@Query('name') name) { // /path?name=name -> name
  return { url: `url/${name}` }
}

```

위는 대략적으로 적은 내용이므로 더 정확한건 `Docs` 를 읽어보도록 하자

### Param

`@Param` 데코레이터를 사용한다면, `express` 의 `params` 처럼 동작한다
그러므로, 동적 `routing` 이 가능하다는 이야기다

이러한 `params` 받는 방법은 다음과 같다

```ts
@Get('/:id/:postId')
findUserPost(@Param() parms: { [key: string]: string }) {
  return this.userService.findOne({ id: params.id, postId: params.postId })
}

// or

@Get('/:id/:postId')
findUserPost(@Param('id') id: string, @Param('postId') postId: string) {
  return this.userService.findOne({ id, postId  })
}

```

### SubDomain routing

`SubDomain` 을 사용하여, 각 `Domain` 마다 다른 처리를 요청할 수 있다
즉, 전혀 다른 `page` 로써 동작하며, `API` 를 분리시켜 처리하는 것이다

이러한 처리를 위해서는 새로운 `Controller` 를 생성해 처리한다

```ts
yarn nest g co SubDomain
```

새롭게 만들어진 `SubDomain Controller` 를 만든이후, `root endPoint` 로 지정하기 위해, `app.module.ts` 에서 `controllers` 의 배열의 첫번째로 `SubDoaminController` 를 넣어준다.

그런후, `subDomain.controller.ts` 에서 `@Controller` 에 인자로 `{ host: '원하는 subDomain' }`  을 넣어서 `host` 임을 알려준다

만약 해당 `subDomain` 이 `subDomain.localhost` 로 넣으면 `SubDomain` 으로써 작동한다

> 추가적으로 `localhost` 를 통한 서브도메인을 구성하고 싶다면, `/etc/hosts` 에 해당 주소를 추가해야 함을 잊지말자
>
> 127.0.0.1 subdomain.localhost
>

```ts
@Controller({ host: 'subDomain.localhost' })
export class SubDomainController {
  @Get()
  index(): string {
    return 'SubDomain'
  }
}
```

이제, `curl http://subDomain.localhost:3000` 으로 요청을 보내면, `SubDomain` 을 받을 수 있다

`@HostParam` 이라는 데코레이터가 사용될수도 있는데, 추가적으로 `subDomain` 의 `Versioning` 을 통해 다른 `version` 에 따른 `SubDomain` 을 만들때 사용된다고 한다

```ts
@Controller({ host: 'ver.subDomain.localhost' })
export class SubDomainController {
  @Get()
  index(@HostParam('var') var: string ): string {
    return `${var}-SubDomain`
  }
}
```

`curl http://v1.subDomain.localhost:3000` 으로 요청을 보내면, `v1-SubDomain` 을 받을 수 있다

### Payload(DTO)

요청시 `HTTP` 의 내용을 전달하는데 보내는 내용을 `Payload` 혹은 `HTTP Body` 라고도 부른다 

이때, `NestJS` 에서 데이터 전송 객체(Data Transfer Object: DTO) 가 구현되어 있어서 쉽게 다를수 있다고 설명하고 있다

이는 다음처럼 처리한다

```ts
export class CreateCatDto {
  name: string
  age: number
  breed: string
}

@Post()
async create(@Body() createCatDto: CreateCatDto) {
  const { name } = createCatDto
  return `This action adds a new cat ${name}`
}

```

이제 `curl` 을 통해 `data` 를 요청해보자

```sh
$ curl -X POST http://localhost:3000/cats -H "Content-Type: apllication/json" -d '{"name": "냥냥이", "age": 1, "breed": "러시안블루"}'
This action adds a new cat 냥냥이

```

위는 `@Body` 데코레이터로 처리했지만, `@Body` 뿐만 아니라 `@Query` 를 통한 `DTO` 도 가능하다

```ts
export class PageNation {
  offset: number
  limit: number
}

@Get('/pageNation?offset=0&limit=10')
async pageNation(@Query() pageNation: PageNation) {
  const { offset, limit } = pageNation
  ...
}
```

위처럼 처리도 가능하다
이렇게, `Server` 사에서 `data` 를 어떻게 받을지에 따라, 처리방법도 약간씩 다를수 있으며, `DTO` 를 사용한다면, 받을 `Payload` 에 대한 `interface` 를 미리 정의하여 받을 `data` 구조를 구조화하여 정리할 수 있다

## 이렇게 `Controller` 에 대해서 알아보았다

`Controller` 는 말그대로, `data` 를 받아 어떠한 주소로 `routing` 해줄지 컨트롤해주는 역할을 해준다

이제 다음으로, `Controller` 를 통한 요청과 응답을 가공하는 방식이 아닌 `Server` 가 제공하는 비지니스 로직을 구성하는 것인 `Provider` 라고 한다.

다음은 `Provider` 에 대해서 더 자세히 알아보도록 하겠다
