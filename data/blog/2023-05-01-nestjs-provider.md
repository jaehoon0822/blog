---
title: 'NestJS Provider 및 Module 정리'
date: '2023-05-01'
tags: ['nestJS']
draft: false
summary: 'NestJS Provider 정리'
---

# NestJS Provider 개념

`Docs` 에서는 다음처럼 설명한다.

> Many of the baseic Nest classes may be treated as a provider
> 대부분의 `Nest` 클래스들은 `Provider` 로 취급될 수 있다
>
> The main idea of a provider is that it can be `injected` as a dependency
> `Provider` 의 주 `idea` 는 `의존성으로써 주입`될수 있다는 것이다
>
> this means objects can create various relationships with each other, and the function of "wiring up" instances of objects can largely be delegated to the Nest runtime system.
>
> 이것은 객체가 서로 다양한 관계를 생성할 수 있음을 의미하며, 객체 인스턴스를 연결하는 기능은 대부분 `Nest runtime systme` 에 위임할 수 있다

위의 설명을 보면 중요한 `keyword` 는 바로 `의존성 주입` 이다.

`Provider` 에 대해서 명확히 알려면, `DI(Dependency Injection)` 에 대해서 알필요가 있다.

`DI` 에 대한 개념은 단순하다. `A 는 B 에 의존하다` 이다.
즉, `B` 가 변경되면 그에 의존하는 `A` 역시 영향을 받아 변하게 된다

그렇다면, 이러한 의존이 존재하면서, 최소화할 수 있는 방법을 생각하는것이 좋다
그러한 방법은 바로 `Interface` 로 타입을 받으며, 해당 `Interface` 로 구현된 `Strategy` 를 `Context` 에서 받아 처리하는 방법이다

다음을 보자

```ts

interface IStartegy {
  ...
}

class StartegyA extends IStartegy { ... }
class StartegyB extends IStartegy { ... }

const strategyA = new StrategyA()
const strategyB = new StrategyB()


class Context {
  constructor(private strategy: IStartegy) {}

  changeStrategy(strategy: IStartegy) {
    this.strategy = strategy
  }
  ...
}

class Context {
  this.strategy = new StrategyA()

  changeStrategy(strategy: IStartegy) {
    this.strategy = strategy
  }
  ...
}

const context = new Context(new StrategyA())
context.changeStrategy(new StrategyB())

```

위는 `Strategy` 라는 `interface` 를 생성한후, 생성된 `interface` 를 `Context` 에 넣어 사용된다

이때, `Context` 는 `Strategy` 를 받는 인자에 의해 의존하게 되지만, 내부에서 특정 `Strategy` 를 생성하여 처리하지 않으므로, `IStartegy` 로 확장한 `Strategy` 라면 얼마든지 `changeStrategy` 를 통해 변경 가능하다

즉 결합도가 낮아지며, 유지보수 및 확장이 유리한 이점을 갖게 되는 구조이다

`NestJS` 에서 `Provier` 는 바로 이러한 `DI` 를 바탕으로 만들어진 개념이다
다음을 보도록 하자

```ts
import { UserService } from 'users.service.ts

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  findUser(@Param('id') id: string) {
    return this.userService.findOne(id)
  }
}

```

위의 로직을 보면, `Controller` 에서 직접 비지니스 로직을 생성하지 않는다.
비지니스 로직은 `UserService` 에 의해 비지니스 로직을 사용한다

이때, `UserService` 는 `UserController class` 에서 `userService` 로 주입받아 처리된다.

그럼 `UserService` 를 리턴하는 `user.service.ts` 를 보도록 하자

```ts
import { Injectable } from '@nestjs/common'

@Injectable()
export class UsersService {
  ...
  fineUser(id: string) {
    ...
  }
}
```

위는 `Injectable` 데코레이터를 가져와서 `UsersService` 에 선언하는 것을 볼 수있다
위 로직은 `UsersService` 를 `Provider` 로 변경해주는 데코레이터이다.

> `@Injectable` Decorator 는 `UserService` 가 `Nest IoC` 컨테이너에 위해 관리할수 있는 class 임을 선언하는 `metadata` 를 첨부하는 역할을 한다
>
> 여기서 말하는 `IoC` 는 `Inversion of Control` 로써 `제어의 역전` 이라고 불린다.
> `IoC` 개념은 `DI` 와 밀접하게 연관되어 있다.
> `제어의 역전` 이라고 할때 이 개념이 명확하게 와 닿지 않는다... `IoC` 의 개념은 간단하다.
> 개발자가 직접 코드를 작성해서 제어하는것이 아니라, 외부 모듈 및 라이브러리에게 제어를 넘기는것이다.
> 이로써, 개발자는 원하는 로직에만 집중할 수 있으며, 외부 모듈에 덜 의존적인 형태로 코드 작성이 가능하다
>
> 더 간략하게 말하면, `IoC` 는 객체가 의존하는 다른 객체를 내부에서 생성하지 않고, 외부에서 생성된 객체를 얻어 사용한다.
> 이렇게 외부에서 생성된 객체를 사용한다면, `Lifecycle` 이 다르며, 기능을 분리할 수 있고, `low Coupling` 으로 결합이 느슨하기에 재사용성을 향상시키게 해준다.
> `NestJS` 는 내부적으로 `IoC Container` 를 사용하여 `Providers` 를 연결할 다른 컴포넌트에 주입시켜주는 역할을 한다
>
> `제어의 역전` 이라니.. 왜 한국말로 번역하면 이렇게 말이 어려워 지는거지??

이러한 방식으로, 각 `Providers Class` 를 `IoC Container` 에 의해 `Controller` 에 넘겨 처리가 이루어진다
이는 덜 의존적으로 `Controller` 가 `Service Logic` 실행이 가능하게 만들어준다.

> 이러한 부분은 개념적으로 이해는 가지만, `OOP` 관련 책을 더 읽어보면서 연구해야 할것 같다

```ts
import { Injectable } from '@nestjs/common'
import { User } from './interfaces/users.interface';

@Controller('user')
export class UsersController {
  constructor(private readonly usersService: UsersService) // DI 된 catsService
  ...
  fineUser(id: string): Pormise<User> {
    return this.usersService.findUser()
  }
}
```

`UsersController` 에 의해 만들어진 `Controller` 에 `usersService` 를 통해, `UsersService` 의 인스턴스가 만들어진다.
여기서 `UserService Class` 를 `Nest` 가 `DI` 한것이다

`Providers` 는 일반적으로 `Application lifecycle` 과 함께, 동기화된 `Scope` 의 `lifetime` 을 가진다
`Application` 이 부팅된다면, 모든 의존성들은 반드시 해결되어야 한다. 그러므로 모든 `provider` 는 인스턴스화 된다
이는 `Application` 이 중단되면, 모든 인스터스들 역시 없어진다.

여기까지는 지극히 일반적인 상황이다.
그렇지만, 이러한 일반적인 상황에서, `Provier` 의 수명을 `Application` 이 아닌 `Request-time` 으로 만들수 있는 방법역시 존재한다
즉, `Application` 이 종료되어도, `Request-time` 내에 살아있다면, `Request-time` 이 끝날때까지는 `Provider` 가 존재한다는 것이다

이러한 방법으로 인해, `Proivers` 는 동기화된 `Scope` 의 `lifetime` 을 가진다라고 표현할 수 있다.
이렇게 만들어진 `Provider` 는 등록을 통해 `NestJS` 에서 인식할 수 있도록 해주어야 한다

등록은 `users.module.ts` 의 `Provider` 배열안에 넣어주면 된다

```ts
import { Module } from '@nestjs/common'
import { UsersService } from './users.service'
import { UsersController } from './users.controller'

@Module({
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

이로써, `UsersController` 에서 사용가능한 `Provider` 등록이 완료된다
여기서 나온 `Module` 같은 경우는 내용을 따로 더 정리할 예정이다.
