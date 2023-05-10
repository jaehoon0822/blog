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

위의 `ConfigModule.register` 를 사용한후 제공하는 `Module` 은 다음과 같다고 한다.
