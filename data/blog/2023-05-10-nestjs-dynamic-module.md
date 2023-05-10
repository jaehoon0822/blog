---
title: 'nestjs dynamic module 에 대해서'
date: '2023-05-10'
tags: ['nestjs', 'dynamic module', 'module']
draft: false
summary: 'module 를 조금더 자세히 살펴보고 dynamic module 을 이해하자'
---

# `Dynamic Module` 에 대해서 알아보자

`Module` 은 말그대로 그룹을 형성해주는 역할을 한다
내부에는 `Controller` 와 `Providers` 를 받아서 자신만의 실행 컨텍스트를 만든다고 보면 된다

만약 `Module` 내부에서 `Service logic` 을 `exports` 하지 않는다면 `encapsulate` 되기에, 해당 `Module` 범위에서만 사용되도록 한정시킨다.

이러한 방식을 `nestjs` 에서는 `static module binding` 이라고 한다
만약 `exports` 에 `Service` 가 포함되어 있으며, `Module` 을 `imports` 한다면 다음과 같이 작동한다.

가져오는 `Module`을 사용하는 `Module` 에서 가져와 모든 종속성을 해결하고 `Module` 을 `Instace` 화 시킨다

그리고 사용하는 `Module` 에서 가져온 `Module` 이 내보낸 `Providers` 를 구성요소에서 사용할 수 있도록 하며, 사용하는 `Module` 을 `Instance` 화 한다.

그런다음, 사용해야할 `Service` 를 주입시키는 방식으로 이루어진다

`Docs` 에서는 사용하는 `Module` 을 `Host Module` 이라 칭하고, 사용되는 모듈을 `Consuming Module` 이라 명한다

## Dynamic Module 을 사용하는 이유

`Module` 을 사용자 입맛대로 동적으로 생성이 가능하다

이전의 `Module` `export` 및 `import` 방식은 `consuming module` 에 의해 항상 정의되고 만들어져야 한다는 점이다

이렇게 정적으로 제공되어지기 보다는, 사용자가 해당 모듈을 가져올때 원하는 방식의 `API` 를 만들어, `Module` 로 받아올 수 있다면 정말 편할것이다.

`Dynamic Module` 은 `consuming module` 에서 `API` 를 구성하여 `Host Module` 로 가져올때, 사용자가 정의한 방법에 따라 동적으로 모듈을 구성하는 방법이다

`Docs` 에서는 `ConfigModule` 을 사용할때, 사용자가 정의한 옵션에 맞추어 `Module` 을 가져오는것을 보여준다

```ts
// 정적으로 module 을 가져오는 로직

import { Module } from '@nestjs/common'
import { AppController } from './app.controller'
import { AppService } from './app.service'
import { ConfigModule } from './config/config.module'

@Module({
  imports: [ConfigModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

```ts
// 동적으로 module 을 가져오는 로직

import { Module } from '@nestjs/common'
import { AppController } from './app.controller'
import { AppService } from './app.service'
import { ConfigModule } from './config/config.module'

@Module({
  imports: [ConfigModule.register({ folder: './config' })],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

위의 로직을 살펴보면 `Dynamic module` 은 `register static method` 를 사용하여 객체로 인자를 전달하고 있다

보통 이렇게 `Dynamic module` 로 만들어지면, `static method` 를 통해 `forRoot` 혹은 `register` 를 만들어 사용하는것이 `Convention` 이라고 한다

위의 로직은 흥미롭게도, `Module` 의 `static method` 에 인자로 객체를 보낸다.
그러면 이 `Module` 는 제공하는 `Service` 에 해당 인자의 객체를 제공하여, 설정한 객체에 맞는 `Service` 를 주입해준다

일단 `register()` 가 반환하는 `DynamicModule` 은 `runtime` 에 생성된 모듈에 불과하다.

모듈을 간단하게 아무렇게나 빠르게 선언하면 다음과 같이 선언도 가능하다

```ts

@Module({
  imports: [DogsModule],
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})

```

실제로, 우리가 여태껏 작성한 `Module` 역시 동일하게 작성해서 처리한다
그냥 객체에 `Module` 에 사용할 `API` 에 맞춰서 값을 넣은것 뿐이다.

`Dynamic Module` 역시 동일한 인터페이스 와 함께 `module` 이라고 불리는 `property` 를 추가한 형태를 가진다

이 `module property` 는 `Dynamic module` 의 이름을 정의해주는 `property` 로 생각하면 된다

이제, `register` 가 어떻게 작동하는지 확인하는것이 좋을 듯 하다

```ts
import { DynamicModule, Module } from '@nestjs/common';
import { ConfigService } from './config.service';

@Module({})
export class ConfigModule {
  static register(): DynamicModule {
    return {
      module: ConfigModule,
      providers: [ConfigService],
      exports: [ConfigService],
    };
  }
```

위는 `Docs` 에서 제시해주는 예시이다.
이 예를 보면 `ConfigModule` 의 `register` 가 어떻게 생겼는지 볼 수 있다
정말 간단하게도, 일반 모듈과 같지만, `module` 프로퍼티가 존재하는것이 보인다.

이제 이 `register` 에 인자를 넣으면 다음처럼 만들 수 있다

```ts
import { DynamicModule, Module } from '@nestjs/common'
import { ConfigService } from './config.service'

@Module({})
export class ConfigModule {
  static register(options: Record<string, any>): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'CONFIG_OPTIONS',
          useValue: options,
        },
        ConfigService,
      ],
      exports: [ConfigService],
    }
  }
}
```

굉장히 흥미로운 상황이다.
여기서 인자값으로 받은 값을 받는데, 위의 `Record Utility` 는 다음의 `type` 으로 만든다.

```ts

  static register(options: { [key: string]: any }): DynamicModule {

```

즉, 어떠한 값이라도 받을수 있는 `options` 객체를 뜻하지만, 실제로 `Interface` 로 만들어 `options` 를 지정하는것이 더 현명한 방법이다.

위는 `Docs` 에서 보여주기식으로 만든 코드라는점 알아두자
즉, 어떠한 값이라도 받을수 있는 `options` 객체를 뜻하지만, 실제로 `Interface` 로 만들어 `options` 를 지정하는것이 더 현명한 방법이다.

위는 `Docs` 에서 보여주기식으로 만든 코드라는점 알아두자

일단, 위의 로직은 `custom Provider` 를 사용하여 새로운 `Servide Logic` 을 만든다.

```ts

providers: [
  {
    provide: 'CONFIG_OPTIONS',
    useValue: options,
  },
  ConfigService,
],
```

위 로직은 간단하게, `value` 값이 `options` 이며, `Provider token` 은 `CONFIG_OPTIONS` 라는것을 뜻한다

이렇게, 주입된 `Provider` 를 어떻게 커스텀하게 사용할까?
만약 `DI` 에 대한 이해가 있다면 쉽게 유추가능하다.

바로, `Service` 로직인 `configService` 에서 주입받아서 처리하면, 동적으로 설정에 따라 `Logic` 변경이 가능하다.

`ConfigService` 를 보도록 하자

```ts
import * as dotenv from 'dotenv'
import * as fs from 'fs'
import * as path from 'path'
import { Injectable, Inject } from '@nestjs/common'
import { EnvConfig } from './interfaces'

@Injectable()
export class ConfigService {
  private readonly envConfig: EnvConfig

  constructor(@Inject('CONFIG_OPTIONS') private options: Record<string, any>) {
    const filePath = `${process.env.NODE_ENV || 'development'}.env`
    const envFile = path.resolve(__dirname, '../../', options.folder, filePath)
    this.envConfig = dotenv.parse(fs.readFileSync(envFile))
  }

  get(key: string): string {
    return this.envConfig[key]
  }
}
```

위 로직을 보면, `Custom Provider` 를 제공하기 위해, `@inject` 를 사용하여 `Constructor` 에서 주입받는것을 볼 수 있다.

이제 이 `Service` 로직은 `Host Module` 에서 `Dynamic module` 에 넣은 `options` 주입받아서 동적으로 처리된다.

굉장히 멋진 방법으로 `Module` 을 조작한다.
`Dynamic Module` 에 대해서 알아보았다.

`Docs` 에는 `Dynamic Module` 을 `build` 할수 있는 새로운 `API` 를 제시하지만 이부분은 조금씩 차근차근 공부해 나가볼 예정이다.
