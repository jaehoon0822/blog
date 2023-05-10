---
title: 'NestJS module 정리'
date: '2023-05-07'
tags: ['nestJS']
draft: false
summary: 'NestJS module 정리'
---

# NestJS Module 개념

`NestJS` 에서는 복잡도를 낮추기위해 하나의 개념을 추가적으로 넣었는데, 그것이 바로 `Module` 이다

`NestJS` 의 `Module` 은 여러 컴포넌트를 조합해서 더 큰 작업을 수행할 수 있도록 만든 단위라고 보면 된다

이렇게 큰 작업을 나누고, 그 모듈을 모으면 하나의 `Application` 이 된다
이는 `Service` 를 구조화하여 처리하는 `MicroService Architecture(MSA)` 의 관점에서 각 모듈을 마이크로 서비스로 분리할 수도 있다

의존관계가 복잡하게 되면, `Application` 역시 알수없는 의존성으로 스파게티 코드가 탄생할 수 있다

그러므로, 이러한 `Module` 을 통해 응집도를 높혀 구조를 짜는 것은 매우 중요하다

`@Module` 은 `ModlueMetadata` 를 받는다

```ts
export interface ModuleMetadata {
  /**
   * Optional list of imported modules that export the providers which are
   * required in this module.
   */
  imports?: Array<Type<any> | DynamicModule | Promise<DynamicModule> | ForwardReference>
  /**
   * Optional list of controllers defined in this module which have to be
   * instantiated.
   */
  controllers?: Type<any>[]
  /**
   * Optional list of providers that will be instantiated by the Nest injector
   * and that may be shared at least across this module.
   */
  providers?: Provider[]
  /**
   * Optional list of the subset of providers that are provided by this module
   * and should be available in other modules which import this module.
   */
  exports?: Array<
    | DynamicModule
    | Promise<DynamicModule>
    | string
    | symbol
    | Provider
    | ForwardReference
    | Abstract<any>
    | Function
  >
}
```

- `imports`: 이 모듈에서 사용하기 위해 `Provider` 를 가지는 다른 모듈을 가져온다. 기본적으로 `encapsulate` 되어 있어서, 이렇게 `imports` 과정을 생략하면, 다른 모듈의 `Provier` 를 가져올 수 없다

- `controllers / provider`: 해당 모듈에서 사용되는 `Controller`들과 `Provider` 들을 배열로 정의한다

- `exports`: 이 모듈에서 제공하는 컴포넌트들을 다른 모듈에서 가져올 수 있도록 내보내는 역할을 한다.

이 밀접하게 관련되어 있는 `Controller` 와 `Services` 가 같은 `Domain` 이라고 느끼기 쉽게 만드는 것이 바로 `Module` 이다.
`Module` 의 특정기능과 관련된 코드를 구조화 시키며, 구조화된 코드를 유지시키고, 명확한 `boundaries` 를 만들어준다.

이 `Module`에 의해 `Application` 및 팀 규모의 크기가 커짐에 따라 `SOLID` 원칙에 대한 개발 그리고 복잡성 관리에 도움을 준다.

폴더구조는 다음과 같다.

```sh
src
  |- cats
  |    |- dto
  |    |- create-cat.dto.ts
  |    |- interfaces
  |    |      |- cat.interface.ts
  |    |-  cats.controller.ts
  |    |-  cats.module.ts
  |    |-  cats.service.ts
  |- app.module.ts
  |- main.ts

```

`Docs` 에서 제공하는 `Module` 에 대한 예시이다.

`Docs` 에서 간단한 예시를 통해 제공하는 `cats` 폴더안에 `CatsController` 와 `CatsService` 가 있다고 가정한다.
이때, `CatsController` 와 `catsService` 는 `Application` 상에 같은 `Domain` 에 속한다.

```ts
// cats/cats.module.ts

import { Module } from '@nestjs/common'
import { CatsController } from './cats.controller'
import { CatsService } from './cats.service'

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

> 위 `Module` 을 생성할때 `CLI` 를 활용한다면, `$nest g modlue cats` 를 통해 생성가능하다

`cats.module.ts` 안에 `CatsModule` 이 정의되어 있고, 연관된 모든 것들이 `cats` 폴더안에 있다.
그리고 `root 디렉터리` 안에는 `app.module.ts` 와 `main.ts` 가 존재한다

이때, `app.module.ts` 는 아래와 같은 `code` 가 존재한다.

```ts
import { Module } from '@nestjs/common'
import { CatsModule } from './cats/cats.module'

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

`AppModule` 은 모든 모듈들의 `root` 로써 매우 중요한 역할을 한다.
`root module` 은 `Nest` 에서 사용하는 내부적인 자료구조(`application graph`)에 의해 `Build` 되는 시작점 역할을 한다
`application graph` 는 자료구조로써, 관계 및 종속성 제공자 및 모듈들의 해결하는데 사용된다.

이때, `module` 들은 각 분리되어진 모듈들이 서로 `service logic`을 공유할 수도 있다.

```ts
// cats.module.tsJS

import { Module } from '@nestjs/common'
import { CatsController } from './cats.controller'
import { CatsService } from './cats.service'

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

여기서 `exports` 를 통해 원하는 `Service Logic` 을 내보내서 다른 모듈에서 사용가능하도록 내보내기가 가능하다
이러한 `exports` 를 다음처럼 사용도 가능하다

```ts
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}
```

위의 코드는 `core.module.ts` 에서 모든 모듈들에서 공통으로 쓰이는 `common.module.ts` 의 `CommonModule` 을 가져온 예이다.
이렇게 `CommonModule` 을 `imports` 하고 바로 `exports` 한다면, `CoreModule` 을 사용하는 모든 `Module` 들은
자동적으로 `CommonModule` 을 `Imports` 하여 사용하는 것과 같다.

또한 `Module` 역시 `class` 이므로, `constructor` 를 통한 `Provider DI` 역시 가능하다

```ts
// cats.module.tsJS

import { Module } from '@nestjs/common'
import { CatsController } from './cats.controller'
import { CatsService } from './cats.service'

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```

> `Docs` 에서는 `Circular dependency` 로 인해 자기 자신을 `injected` 할 수 없다고 강조하고 있다.

## Global modules

`NestJS` 에서는 `providers` 가 `module scope` 안에 `encapsulate` 되어있다.
이말은 `encapsulating module` 을 먼저 `import` 하지 않고는 다른 곳에서 `module providers` 를 사용할 수 없다는 것이다.

모든 모듈에서 `import` 해야 한다면, 일일히 다른 모듈을 가져와야만 할까? 이건 너무나 비효율적이다.
이렇게 모든 모듈에서 사용가능한 `module providers` 가 존재한다면, 전역적으로 설정하는것이 좋을 것이다.

> 예를 들어서, `database connections`, `helpers` 같은 `providers` 들이 있을수 있다

이렇게, `global module` 을 만들 수 있는데, `@Global() decorator` 를 사용하면 된다.

```ts
import { Module, Global } from '@nestjs/common'
import { CatsController } from './cats.controller'
import { CatsService } from './cats.service'

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

이렇게 하면, 해당 모듈은 `Global` 모듈이 된다.
여기서 중요한 것이 잇는데 `Global module` 은 오직 한번만 등록해야만 한다.
일반적으로, `root` 혹은 `core module` 에 등록하는경우가 많다

위의 예를 보자면, `CatsService` 제공자는 어느 모듈에서 전부 사용가능하다.
만약 다른 모듈에서 `CatService` 를 사용한다고 할때, 그 모듈에서 `inject` 하기 위해 따로 `import` 할 필요도 없이 `inject` 할수 있다.

`전역적 설정` 은 매우 유용하기도 하지만, 좋은 방식은 아니다. 그러므로 최소한의 꼭 필요할때 작성해야만 한다

## DynamicModules

`Nest module` 시스템은 `dynamic modules` 이라 불리는 강력한 기능을 포함한다
이 기능은 쉽게 맞춤형 모듈을 생성할 수 있으며, `Providers` 를 동적으로 등록가능하게 설정할 수 있다

```ts
import { Module, DynamicModule } from '@nestjs/common'
import { createDatabaseProviders } from './database.providers'
import { Connection } from './connection.provider'

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities)
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    }
  }
}
```

`forRoot` 는 동적 모듈을 리턴할때 사용하는 `static method` 이다.
여기서 해당 `static` 메서드의 이름은 사용자에 따라 지정 가능한데, `Convention` 으로 `forRoot` 및 `register` 둘중 하나로 지정하여 사용한다
위의 로직이 기존의 `Module metadata` 를 재정의 하는 대신 확장한다고 하는데, 로직 자체가 그렇게 이해가 가지는 않는다.

`createDatabaseProviders` 에 `entities` 를 받으면 해당 `entities` 를 받은 `Providers` 를 반환받고, `module metadata` 에 새롭게 지정한 `metadata` 를 할당하여 반환하는 로직같다.
이 부분은 조금더 깊이 공부해보아야 할 부분인것 같다.

일단, 이렇게 해당 모듈의 `static method` 를 사용하여, `module metadata` 를 동적으로 생성이 가능하다는 부분을 알고는 있도록 하자

모듈에 대해서 현재까지 `Docs` 를 보면 알아보고는 있는데, 아직 나의 개발지식이 짧아서 이해가 가지 않는 부분이 많다
객체지향 과 연관된 내용이 많기에, 모르는 지식을 조금더 채워가며 순차적으로 알아가는 방법밖에는 없을 것 같다

현재까지는 모듈을 통해 `Providers` 를 받아 `Controller` 에 `Service logic` 들을 주입해주며, 각 모듈은 각자의 `domain` 을 가지고 있다는 것을 알게 되었다
이는 `IoC` 개념에 의해, 각 객체를 따로 관리하며, 쉽게 확장 및 교체 가능하도록 만든 디자인패턴을 적용한 것이며, 각 `Module` 이 `export` 를 통해 각 `service logic` 들을 서로
공유하여 사용하는것에 대해서는 잘 알게 된것 같다
