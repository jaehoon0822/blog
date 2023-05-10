---
title: 'nestjs custom provider 에 대해서'
date: '2023-05-10'
tags: ['nestjs', 'provider', 'custom provider', 'cutomProvider']
draft: false
summary: 'provider 를 조금더 자세히 살펴보고 custom provider 를 이해하자'
---

# Custom Provider

> `Docs` 를 살펴보면서, `Dynamic Module` 을 같이 보고 있는데, 역시나 모든것이 연관되어 있다.
> `Dynamic Module` 을 조금이나마 더 자세히 이해하기 위해서는 `Custom Provider` 를 더 알 필요성이 느껴졌다.
> `Nest` 를 더 많이 이해하고, 더 능숙하게 다루기 위해서는 기본 개념에 대해서 잘 알아야 한다.
> 그런 의미로 `Provider` 에 대해서 더 자세히 공부하고자 한다.

## Provider 의 작동되는 개념

`Provider` 의 `DI` 는 `Nest IoC Container` 에 의해 작동된다
다음은 `Docs` 의 예이다

> `Service Logic` 생성

```ts
// cats.service.ts

import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  findAll(): Cat[] {
    return this.cats;
  }
}
```

> `Servcie Logic` 을 사용할 `Controller` 생성 이후 `Service` 주입

```ts
// cats.controller.ts

import { Controller, Get } from '@nestjs/common';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

> `Nest IoC Container` 에 `Provider(Service logic)` 주입

```ts
// cats.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

위를 보면 알겠지만 여기서 말하는 `Nest IoC Container` 는 `Module` 을 말하는것을 볼 수 있다.
여기서 중요한 부분은 `Service Logic` 을 실행할 `Class` 에 `Injectable` 데코레이터를 사용한점이다.

`Provider` 에서 보아서 알겠지만, `Injectable` 데코레이터를 사용한 로직은 `Nest IoC Contianer( Module )` 에서 관리할( `or 주입가능한` ) `Class` 를 선언하는것이다.
이렇게 `Injectable` 데코레이터가 선언된 `Class` 를 `Nest IoC Container` 에서 `Provider` 로 넣어주면, 등록된 `Controller` 에서 해당 `Class` 의 `Instance` 를  
`Constructor` 의 인자로 주입받아 사용가능하게 된다.

이렇게, `Class` 를 `Provider` 로 등록하기만 하면, 그 `Class` 를 실행시킨 `Instance` 를 등록된 `Controller` 에게 주입해주는 것은 `Nest IoC Container` 에서 알아서 처리해준다,
`Class` 를 직접 주입하는 복잡한 과정없이 해당 기능에만 집중 가능하게 만든다.

> 물론 위와 같은 과정에서 `Controller` 는 주입받을 `Parameter` 에 `DI Token`(`Provier`에 주입된 `Service`) 을 미리 선언해주어야, 해당 `Parameter` 에 `Service Instance` 주입이 가능하다
>
> ```ts
> 
>  constructor(private catsService: CatsService) {} // <--  이렇게 주입가능하도록 해야만 한다
>
> ````

그렇다면, `Provider` 의 주입은 이해를 했고, `Controller` 는 어떻게 생성되고 작동하는지 그 개념적 이해가 있어야지, 앞으로 `Custom Provider` 를 이해하는데 편리할 것이다
`Nest IoC Container` 에서 `Controller` 를 `Instance` 화 할때, 첫번째 하는 일은 종속성을 찾는것이다.

위의 `Logic` 상 `catsService` 를 주입받도록 `parameter` 로 선언되어 있다.
`Instance` 화 과정에서 `CatsService` 의 종속성을 찾으면 `CatsService` 의 `Token` 에 대해 조회를 수행한다

이 조회과정은 일반 `Cache` 와 비슷하다.
`Provider Service` 를 `Instance` 화하고 캐시하거나, 이미 해당 `Provider` 의 `Instance` 가  `Cache` 되어 있다면, 그 `Cache` 된 `Instance` 를 반환한다

이부분은 `Nest` 가 알아서 처리해주는 굉장히 편리한 기능이다.
> 실제 종속성을 관리하는 알고리즘인 `dependancy Graph` 는 훨씬더 복잡하고 정교하게 이루어져 있다고 `Docs` 에서 설명하고 있다
> 즉 위의 동작은 굉장히 단순화 시켜 설명한것이라는 점을 강조한다

여기서 `Token` 이라는 말이 굉장히 중요하다.

이 `Token` 은 `DI` 진행시 `IoC Container` 에 등록된 `key` 값이라고 생각하면 쉽다
앞에서 이미 언급했지만, `Controller` 를 `Instance` 할때,

> `Instance` 화 과정에서 `CatsService` 의 종속성을 찾으면 `CatsService` 의 `Token` 에 대해 조회를 수행한다

라고 말했다.
즉, `IoC Container` 는 종속성에 대해(`provider` 를 말한다.. ) `Token` 을 통해, `Cache` 된 `Instance` 를 찾거나, 없으면 `Token` 을 `key` 값으로 생성해 `Instance` 를 `Cache` 한다는 것이다

이러한 과정을 통해, `token` 이 `provider` 에 존재한다는 것을 아는것은 중요하다.
다음의 로직을 살펴보면, 이를 더 쉽게 이해할 수 있다.

> 기존의 `cats.module.ts`

```ts
// cats.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService], // <-- token 없이 값만 입력한것으로 보인다.
})
export class AppModule {}
```

위의 로직은 실제로 다음과 같다

```ts
// cats.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [{
    provide: CatsService, // <-- Token 값
    useClass: CatsService, // <-- Provider 의 value
  }],  // <-- 위의 providers: [CatsService] 와 같다
})
export class AppModule {ㅣㅣ
```

즉, `Providers: [CatsService]` 는 위의 로직처럼 일일히 작성하는 불편함을 줄이기 위해, `Nest` 에서 알아서 `token` 값을 지정해주는것이라 볼 수 있다.
아무래도 위의 로직이 더 명식적이기는 하지만, 작성해야할 `Code` 양이 많아져 불편할 수 있기는 하다..

이렇게, `Module` 에서 `providers` 선언시, `Nest` 내부에서 `token` 이 작성되는것을 알 수 있게 되었다.
이제부터, `Custom Provider` 를 작성하는 개념을 살펴보도록 하자.

## Custom Provider

`Docs` 에서는 `Custom Provider` 사용하는 경우의 몇가지 예를 말해주고 있다.

1. `Nest` 가 `Class Instance` 화 하는 대신 사용자 정의 `Instance` 를 생성하고자 할때

> `Class Instance` 를 자동적으로 제공될 `Class` 를 `Cache` 하거나, `Cache` 된 `Instance`  가 있다면, 그 `Instance` 를 주입한다고 앞에서 설명했다. 그러므로, 위에서는 기본적으로 생성된 `Instance`를 사용안하고, 이름은 같지만, 개발자가 직접 생성한 다른 `Instance` 를 주입하고 싶을때 사용하는것을 말하는것 같다.

2. 두번째 종속성에서 기존 클래스를 재사용하려고 할때

3. 테스트를 위해 `Mock` 버전으로 클래스를 재정의할때

### Value Provider

`useValue` 구문을 상요하면 상수 값을 주입할 수 있다
`Docs` 에서는 `Mock` 객체를 반환할 `Object Value` 값으로 작성하는 예시로 설명한다.

```ts
import { CatsService } from './cats.service';

const mockCatsService = {
  /* mock implementation
  ...
  */
}; //< --  이때, 이 객체는 `CatsService` 의 Type Compatibility 에 맞도록 
   //      작성되어야만 한다

@Module({
  providers: [
    {
      provide: CatsService,
      useValue: mockCatsService,
    },
  ],
})
export class catsModule {}
```

위처럼 지정하면 `provider token` 은 `CatsService` 가 되지만, 그 값은 `mockCatsService` 가 된다.

위에서 `mockCatsService` 는 `CatsService` 의 `instance` 의 `Type` 호환되는 객체이므로, 적용가능하다.

> `Object literal` 역시 `Object` 이므로, `Type Compatibility` 만 된다면, 그 값으로 사용가능하다. 위의 예에서 주입시, `CatsService` 의 `Class` 타입을 가졌을때를 가정하므로, 타입역시 호환되어야 한다

하지만, `Provider` 는 `Token` 이므로, 언제든지 `Token` 값 변경이 가능하다.
이전에는 `Class Name` 이 사용된 `Provider token` 을 사용했었다. 이는 `Constructor based injection` 이 사용된 기본 패턴에 의해 `match` 된다.

하지만 꼭 `Class Name` 일 필요는 없지 않은가? `String` 혹은 `Symbol` 로 사용가능하다

이는 `name` 값을 `return` 하는 `Custom Provider` 이다.

```ts
import { UsersService } from './users.service';
  
@Module({
  providers: [
    UsersService,
    {
      provide: 'NAME',
      useValue: 'JH'
    }
  ],
})
export class UsersModule {}

```

이를 통해 `Service Logic` 을 통해 `Injection` 하는 방법은 다음과 같다

```ts
@Injectable()
export class UserServide {
  @Inject('NAME') private  name: // name = 'JH"
}

```

기본의 방식과는 다르게, 직접 `Token` 을 작성했다면, 위처럼 `@Inject` 데코레이터를 사용하여 처리해야 한다.

또한, `main.ts` 에서는 `App.module.ts` 에서 이렇게 `Providers` 를 통해 주입된 값(`useValue`)을 `app.get('token')` 을 통해 접근도 가능하다고 하니 참고하자.

### Class Provider

`useCalss` 를 사용하면 `Token` 이 해결해야할 `Class` 를 동적으로 결졍 가능하다

`Docs` 에서는 다음처럼 예시를 보여준다

```ts
const configServiceProvider = {
  provide: ConfigService,
  useClass:
    process.env.NODE_ENV === 'development'
      ? DevelopmentConfigService
      : ProductionConfigService,
};

@Module({
  providers: [configServiceProvider],
})
export class AppModule {}
```

위의 로직을보면, 환경변수에 따라 선택되는 `configService` 가 달라지는것을 볼 수 있다.

이렇게 `configService` 에 따라, 주입되는 `Service Logic` 이 달라진다.

### Factory Providers

위는 이미 만들어놓은 `Service` 를 동적으로 선택하여 만들어지지만, 만약 동적으로 `Service logic` 을 만들어 `Provider` 에 넣어야 하는 상황이 있다면 어떻게 해야 할까?

이럴때 사용하는 `Provider` 가 `Factory Provider` 이다.
말 그대로 주어지는 `Factory function` 을 통해 만들어진 `Service` 로 주입할 `Instance` 를 만들어낸다

`Factory Provider` 의 함수는 선택적 인자를 받는다
이렇게 선택적 인자를 제공하는 속성이 `inject` 속성으로, 만들어진
`Factory Function` 에 인자에 값을 전달하는 공급자 배열이다.

```ts
const connectionProvider = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider, optionalProvider?: string) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider, { token: 'SomeOptionalProvider', optional: true }],
  //       \_____________/            \__________________/
  //        This provider              The provider with this
  //        is mandatory.              token can resolve to `undefined`.
};

@Module({
  providers: [
    connectionProvider,
    OptionsProvider,
    // { provide: 'SomeOptionalProvider', useValue: 'anything' },
  ],
})
export class AppModule {}

```

위의 선택적 인자를 넣어서 처리한다.
`inject` 속성을 상용하여, `OptionsProvider` 를 주입하여 인자값으로 넘기며, 2번째 인자로 객체를 넘기는것을 볼 수 있다

2번째 인자의 객체는 `token` 값으로 `SomeOptionalProvider` 를 사용하며, `optional` 값을 `true` 로 설정하여, 선택적으로 받을 수 있도록 한다.

위의 로직에서 `@Module` 을 보면 알겠지만 `SomeOptionalProvider` 는 주석처리되어있어서, 현재로써는 `undefined` 로 주입되지 않았다.

### useExistsing

`Provider` 에 `useExisting` 을 사용하면, 이미 존재하는 `Service` 에 새로운 별칭을 부과할 수 있다

```ts
@Injectable()
class LoggerService {
  /* implementation details */
}

const loggerAliasProvider = {
  provide: 'AliasedLoggerService',
  useExisting: LoggerService,
};

@Module({
  providers: [LoggerService, loggerAliasProvider],
})
export class AppModule {}

```

이렇게 위의 `@Module` 은 `LoggerService` 에 대해서 2개의 이름으로 주입될 수 있다 

`Provider` 처리하는데도 여러가지 방식으로 처리가 가능하다
`useFactory` 는 필요시 많이 사용되는 패턴으로 보이므로, 지속 사용하면서 알아보아야 겠다.

다음은 `Dynamic Module` 에대해서 더 공부해볼 예정이다.
