---
title: 'Decorator 에 대해서'
date: '2023-04-30'
tags: ['nodejs', 'design pattern', '디자인패턴', 'typescript', 'javascript']
draft: false
summary: 'Decorator 에 대해서'
---

# Decorator

> 사람의 욕심은 끝이 없다고 했던가... 단순하게 `Frontend` 만으로 부족함을 느끼고, `Backend` 의 동작과정을 보고싶고, 단순한 `API` 를 구현하여 `ToyProject` 를 만들가 했던것이 어느새 `express` 가 아닌 `nest` 까지 알고 싶어졌다..

## 그래서 왜 갑자기 `Decorator` 야?

`NestJS` 를 보며 공부를 하고 있는 상황에서, 문제가 한가지 발생했다.
잘 알지 못하는 `Decorator` 를 사용하고 있다는 것이다...

`Typescript` 에서 `Decorator` 가 존재하고 사용하는것에 대해서는 알았지만, 그렇게 많이 사용할 여건이 되지 않았다.

당연, 그로인해 사용법도 익숙치 않은 상황이다.

`NestJS` 는 기본적으로 이러한 `Typescript Decorator` 방식과 함께, 객체지향에서 주로 다루는 `Desting Pattern` 들을 종합하여 만들어진 `Framework` 라는것을 알게 되었다.

한번 살짝 맛만 보았는데, 확실히 `Code` 가 정리되어가는것을 보며, 왜 `Design Pattern` 이 중요하고, 왜 사용하는지 느낄수 있는 계기가 되었다.

`NestJS` 는 기본적으로 `Decorator` 를 사용하여 이미 만들어져 있는 코드의 재사용성을 높이는 방식으로 만들어져 있다.

이후, 여건이 된다면 `Java` 혹은 `Kotlin` 을 공부하며, `Design Pattern` 및 객체지향에 대해서 조금더 자세히 공부해볼 생각이었는데, 이렇게 된것 `Decorator` 에 대해서는 조금 깊이 공부를 해보아야 겠다.

## 일단 `Decorator pattern` 먼저 살펴볼까?

`Decortor pattern` 은 한글로 하면 `장식자` 자로도 불릴수 있다.
말그대로, 기존에 있는 녀석을 장식해주는 방식이라고 생각하면 편하기는 하다.

`Decorator pattern` 은 왜 탄생했는지 그 개념먼저 살펴보고자 한다.

### `Class` 를 확장하고 싶다면 어떻게 할까?

보통 `Class` 를 생성하고, 확장하고자 한다면 `extends` 를 사용하여 `SubClass` 를 만들어 확장하는 방식으로 사용했다.

흔히, 객체지향에서 말하는 `Inherit(상속)` 이다.
유전적인 영향을 받아 부모의 개성을 자식이 물려받는다고 생각하자.

다음을 보자

```ts

// 부모의 성은 'Skywalker' 이다.

class Parent {
  protected firstName = 'Anakin'
  protected lastName = 'Skywalker'
  protected force: booelan : true

  getName() {
    console.log(`parent: ${this.firstName} ${this.lastName}`)
  }
}

// 자식의 성역시 'Skywalker' 이다.
// 그리고, `firstName` 만 변경한다.

class Child extends Parent {
  constructor(firstName: string) {
    super()
    this.firstName = firstName
    this.force = true
  }

  getName() {
    console.log(`
      child: ${this.firstName} ${this.firstName} 
      force: ${this.force}
    `)
  }
}


const p = new Parent().getName() // child: Anakin Skywalker 
                                 // force: true
const c = new Child('Luke').getName() // Luke Skywalker
                                      // force: ture

```

이러한 방식으로 상속을 구현할 수 있다.
하지만, 이러한 상속으로 구현하기 어려운 상황이 발생하기도 한다.

> 밑의 예시는 [Software Design Patterns With TypeScript Examples: Decorator](https://javascript.plainenglish.io/software-design-patterns-with-typescript-examples-decorator-cb6160ddbeb9) 의 글을 참고하여 만들어본 예시이다. 더 좋은내용은 위 블로그가 잘 설명 되어 있다.

예를 들어, `Cafe` 에서 커피가 있다고 하자,
`커피에 들어가는 토핑` 은 그 종류수가 많으며, 매번 변화되기 싶다.

이때, 각 `토핑` 별로 `Coffee` 의 종류가 달라지는데, 매번 `SubClass` 에 상속을 통해 구현하기에는 불편한점이 많을 수 있다

```ts
// abstract class 를 사용하여, class 의 기본 토대를 만든다
abstract class Base {
  protected desc!: string;
  protected cost!: number;
  public getCost() {
    return this.cost
  }
  public getDesc() {
    return this.desc
  }
}

// 기본 들어가는 Topping 을 Base 로 상속받아 만들어보자
class Milk extends Base {
  desc = 'milk'
  cost = 1000
} 

class Chocolet extends Base {
  desc = 'chocolet'
  cost = 500
}

class Calamel extends Base {
  desc = 'caramel'
  cost = 500
}

class Espresso extends Base {
  desc = 'espress'
  cost = 1000
}

class Wather extends Base {
  desc = 'water'
  cost = 500
}

// 위의 Topping 을 조합하여 Americano 를 만든다고 가정한다

class Americano extends Base {
  protected base1: Base
  protected base2: Base

  constructor(wather: Wather, espresso: Espresso) {
    super()
    this.base1 = wather
    this.base2 = espresso
    this.desc = 'americano'
    this.cost = 1500
  }

  getDesc() {
    return `${this.desc}: ${this.base1.getDesc()} + ${this.base2.getDesc()}`
  }

  getCost() {
    return this.base1.getCost() + this.base2.getCost() + this.cost
  }
}

const w = new Wather()
const e = new Espresso()
const americano = new Americano(w, e)

console.log(americano)
/*
Americano: {
  "base1": {
    "desc": "water",
    "cost": 500
  },
  "base2": {
    "desc": "espress",
    "cost": 1000
  },
  "desc": "americano",
  "cost": 1500
} 
*/
console.log(americano.getDesc()) // "americano: water + espress" 
console.log(americano.getCost(), '원') // 3000원 

// 다른 종류의 Coffee...

```

몇개 까지는 매번 해당 `class` 를 만들어도 상관은 없다.

하지만 그 종류가 기하급수적으로 늘어나며, 사용되는 `Topping` 마저 기하급수적으로 늘어난다고 가정해보자.

위의 방법은 적절하지 않으며 매번 `class`작성시 `base1`, `baseN..` 으로 `Topping` 이 늘어날 것이다.
이때, 사용하는 유용한 방식이 `Decorator` 이다.

`Decorator` 패턴은 다음의 구조를 따른다.

![DecoratorUML](/static/images/2023/04/DecoratorUML.png)

기본적으로 `Component` 를 구성하여 만든다.
이렇게 만든 `Concrete Component` 는 `Decorator Pattern` 에서 `wrapping` 될 객체를 말하며, 이 `Concrete Component` 를 기반으로 `Concreate Decorator` 클래스를 구현하도록 한다.

`Concrete Component` 는 `Concrete Decorator` 클래스를 구현하는 공통 `interface` 를 제공한다고 생각하면 쉽다.

이렇게 각 만들어진 `Concrete Component` 를 `Coffee` 들이고,
`Decorator pattern` 을 통해 조합해야할 `Topping` 들이 `Concrete Docorato` 라고 생각하면 된다.

이제 `Decorator pattern` 을 통해 구성해보도록 한다.

```ts

// 여기서는 간단하게 이해를 위한 목적으로 만들예정이므로
// Americano 와 Moca 만 만들어 본다.

abstract class Base {
  protected desc!: string;
  protected cost!: number;
  public getCost() {
    return this.cost
  }
  public getDesc() {
    return this.desc
  }
}

// ConcreteComponent
class Coffee extends Base {
  desc = 'americano'
  cost = 1000
}

// DecoratorComponent
class ToppingDecorator extends Base {
  private topping: Base

  constructor(topping: Base) {
    super()
    this.topping = topping
    this.desc = topping.getDesc()
    this.cost = topping.getCost()
  }

  getDesc() {
    return `${this.topping.getDesc()} + ${this.desc}`
  }
  getCost() {
    return this.topping.getCost() + this.cost
  }
}


// Concrete Decorator
class Milk extends ToppingDecorator {
  desc = 'milk'
  cost = 1000
} 

class Chocolet extends ToppingDecorator {
  desc = 'chocolet'
  cost = 500
}

class Calamel extends ToppingDecorator {
  desc = 'caramel'
  cost = 500
}

class Espresso extends ToppingDecorator {
  desc = 'espress'
  cost = 1000
}

class Wather extends ToppingDecorator {
  desc = 'water'
  cost = 500
}

const coffee = new Coffee()
const toppingWather = new Wather(coffee)
const americano = new Espresso(toppingWather)
// == new Espresso(new Water( new Coffee() ))

console.log(americano)
/*
Espresso: {
  "topping": {
    "topping": {
      "desc": "coffee",
      "cost": 1000
    },
    "desc": "water",
    "cost": 500
  },
  "desc": "espress",
  "cost": 1000
}
*/
console.log(americano.getCost()) // 2500 
console.log(americano.getDesc()) // "coffee + water + espress" 

const toppingWatherWithEspresso = new Espresso(toppingWather)
const moca = new Chocolet(toppingWatherWithEspresso)
// == new Chocolet(new Espresso(new Water( new Coffee() )))

console.log(moca)
/*
Chocolet: {
  "topping": {
    "topping": {
      "topping": {
        "desc": "coffee",
        "cost": 1000
      },
      "desc": "water",
      "cost": 500
    },
    "desc": "espress",
    "cost": 1000
  },
  "desc": "chocolet",
  "cost": 500
}
*/
console.log(moca.getCost()) // 3000 
console.log(moca.getDesc()) // "coffee + water + espress + chocolet" 

```

이렇게 `Decorator` 를 사용하면, 각 `Class`를 조합하여, 원하는 값을 얻을 수 있도록 구현 가능하다.

이는 기하급수적으로 늘어나는 `SubClass` 를 방지하고, 다수의 `SubClass` 를 조합하여 원하는 구성으로 조립할 수 있도록 만들어진 `pattern` 이다.

> 즉, `Class` 에 대한 조합이 많아질 경우, 일일히 `SubClass` 로 만들기 보다는 `Decorator` 를 조합하여 원하는 `Class` 로 변경하는 용도로 사용한다.

위의 말이 바로 `NestJS` 에서 `Decorator` 를 사용하는 용도이기도 하다.
이제 `Decorator Pattern` 에 대해서 알아보았으니, `Typescript` 에서 제공하는 `Decorator` 를 알아보도록 하자.

### Typescript Decorator

앞에서도 말했지만 `Decorator` 는 `NestJS` 에서 엉청나게 활용한다.
이러한 `Decorator` 는 `Typescript` 에서 제공해주는 문법을 통해 이루어져 있다

`Decorator` 를 사용하기 위해서는 문법적으로 `@expression` 형식의 구문을 사용해야 한다.

여기서, `Custom` 하게 `Decorator`생성이 가능한데, 생성하기 위해서는 함수를 사용하여 만든다.

```ts
function d(target: any, propertyKey: string, description: PropertyDescriptor) {
  console.log('decorator')
}

class Target {
  @d
  execConsole() {
    console.log('test')
  }
}

const t = new Target()
t.execConsole()
// decorator
// test

```

다음처럼 `argument` 를 넘겨 처리하도록 만들 수도 있다

```ts
function d(arg: string) {
  console.log(arg)
  return function (target: any, propertyKey: string, description: PropertyDescriptor) {
    console.log('decorator')
  }
}

class Target {
  @d('arg')
  execConsole() {
    console.log('test')
  }
}

const t = new Target()
t.execConsole()
// arg
// decorator
// test

```

이렇게 만들어진, `Decorator` 는 함수처럼 동작하므로,  
여러개의 `Decorator` 를 만들 수도 있다

```ts
function d1 (target: any, propertyKey: string, description: PropertyDescriptor) {
  console.log('decorator1')
}

function d2 (target: any, propertyKey: string, description: PropertyDescriptor) {
  console.log('decorator2')
}

class Target {
  @d1
  @d2
  execConsole() {
    console.log('test')
  }
}

const t = new Target()
t.execConsole()
// decorator2
// decorator1
// test

```

이는 마치, `d1(d2())` 처럼 자동한다
그러므로, `decorator2` 먼저 출력하고 그다음 `decorator1` 을 출력한다.

> 이는 마치 `Decorator Pattern` 에서 보여주었던 `Americano` 를 조합할때,  
`Class` 가 `Class` 를 포함하여 원하는 `Instance` 를 만드는것과 비슷하다.

```ts
const coffee = new Coffee()
const toppingWather = new Wather(coffee)
const americano = new Espresso(toppingWather)
// == new Espresso(new Water( new Coffee() ))
```

`Typescript` 의 `Decorator` 역시 이러한 `Decorator` 특성을 가지고 문법적으로 만든것으로 볼 수 있다

`Typescript` 는 문법적으로 지원하는 `Decorator` 가 여러종류로 존재한다.
각 종류를 살펴보도록 하자.

#### Class Decorator

해당 `Decorator` 는 `Class` 에서 사용할 수 있는 `Decorator` 이다.

기본적으로 `Class Decorator` 는 클래스의 생성자를 통해 입맛에 맛게 정의 및 수정할 수 있도록 해주는 `Decorator` 이다.

`Class` 의 `Decorator` 를 생성할때, 들어갈 인자는 `Class` 타입을 받아야 하므로,  
`new (...args: any[]): {}`  타입을 갖는다.

이부분은 `Class` 타입이라는 것을 조금만 살펴보면 이해할 수 있다.
`new` 연산자를 통해 `argument` 를 받으며, `{}` 객체를 반환하는 타입은 `Class` 밖에 없다.

그러므로, 해당 `Class Decorator` 는 `Class` 타입을 지정하기 위해 해당 타입을 확장하여 `Generic` 으로 인자에 넘겨주어야 한다

```ts
function classDeco<T extends { new (args: any[] ): {}}>(constructor: T) {
  ...
}
```

위의 타입을 선언한후, 내용을 넣어주어야 한다.
내용을 넣어본다.

```ts
function classDeco<T extends { new (args: any[] ): {} }>(constructor: T) {
  return class extends constructor {
    name = 'test'
  }
}

@classDeco
class Test {
  constructor(desc: string) {
    this.desc = desc
  }
}  

console.log(new Test('이건 실험적으로 만들어본거야'))
/*
{
  "desc": "이건 실험적으로 만들어본거야",
  "name": "test"
} 
*/
```

굉장히 흥미로운 코드이다.

내가 이 `Logic` 을 이해한 바로는 다음과 같이 호출된다고 본다.

1.`classDeco` 가 실행되며, `constructor` 로 `Test` 를 받는다.
2. `return` 되는 `class` 는 인자로 받은 `Test constructor` 를 확장한 `class` 이다.
3. 이렇게 `return` 된 `class` 에 인자로 `이건 실험적으로 만들어본거야` 라는 문자열을 넣는다.
4. 그렇게 생성된 `Instance` 는 새롭게 만들어진 `class` 의 `name` 값을 포함한 객체가 된다

> 여기서 `extends` 를 했으면 `super` 를 왜 안불러오는지 궁금할 것이다.
> 위의 코드에서 명시적으로 `super` 를 통해 `desc` 를 전달하지 않았지만,
> `super` 는 생략가능하며, 생략하면 `typescript` 가 알아서 `desc` 값에 대한 처리를해준다.  

이렇게, 만들어진 `Class` 를 사용한다.

즉, 더 간단히 말하자면,
`TypeScript` 에서 데코레이터를 사용할 때, 데코레이터 함수가 반환하는 클래스가 원래의 클래스를 대체하게 된다는 것이다

#### Method Decorator

`Method` 에 사용되는 `Decorator` 이다.

`Method Decorator` 는 3개의 인자를 받는데 다음과 같다.

```ts
function methodDeco (target: any, propertyKey: string, description: PropertyDescriptor) {
  ...
}

/*
target => class Prototype
propertyKey => 메서드의 key
description => javascript property descriptor
*/

```

여기서 `property descriptor` 는 `javascript spec` 에서 해당 `property` 에 대한 `access` 관련 설정을 해주는 프로퍼티 서술자이다.

이러한 `spec` 이나온이유는 기존의 자바스크립트에서 `property` 에 대한 `access modifier` 를 제공하고 있지 않기 때문에, 이러한 처리를 위해 추후에 제공된 `spec` 이다.

그래서 다른 객체지향 문법과는 다르게 약간의 수고스러운 작업이 필요한것이 사실이다.

`method decorator` 에대해 알기 이전에 `property descriptor` 를약간 살펴보가 지나가도록 한다

`property descriptor` 에는 몇가지 기본 `flag` 가 존재한다.
이는 다음과 같다

1. `enumerable`: 반복문을 통해 나열이 가능한지 설정
2. `wraitable`: 프로퍼티값 쓰기가 가능한지 설정
3. `configurable`: 프로퍼티 설정 및 삭제가 가능한지 설정

위의 3가지 `flag` 는 `property` 의 가장 기본이 되는 flag 이다.
`default` 로 `flag` 는 전부 `true` 이다.

```ts

interface IObj {
  name: string
}

const obj: IObj = {
  name: “jh”
}

Object.getOwnPropertyDescriptor(obj, ‘name’)
/*

descriptor:
{
  "value": "jh",
  "writable": true,
  "enumerable": true,
  "configurable": true
}
*/

```

`javascript` 에서제공하는 `Object.getOwnPropertyDescriptor` 를 사용하면 해당 `property` 에대한 `propertyDescriptor flag` 를 확인할 수 있다.

그럼, `propertyDescriptor` 를 수정하기 위해서 사용되는 메서를 통해 `flag` 수정을 해보도록 한다.

```ts

Object.defineProperty(obj, ‘name’, {
  value: ‘jh’
})

Object.getOwnPropertyDescriptor(obj, ‘name’)
/*
{
  "value": "jh",
  "writable": false,
  "enumerable": false,
  "configurable": false
}
 */

```

이번에는 `javascript` 에서제공하는 `defineProperty` 메서드를 사용하여 `propertyDescriptor` 에 대해 수정했다.

여기서 흥미로은 것은 `default` 로 이루어진 `flag` 의 `true` 값이 전부 `false` 로 처리되었다는 것이다.

이렇게 별도의 처리가 없는 `flag` 는 전부 `false` 처리가 되므로 주의가 필요하다.

`writable` 및 `enumerable` 의 설정되는 조건은 어느정도 이해가 되었지만, `configurable` 에 대한 설정된후 처리 되는 방식은 약간의 이해가 필요할 듯싶다.

`configurable: fasle` 일때 해당 조건은 다음과 같다

1. `writable` 이 `false`일때 수정불가 (단, true 일때 false로 변경가능)
2. `enumarable` 수정 불가
3. `configurable` 수정 불가
4. 접근자 프로퍼티 변경 불가(단, 새로만드는것은 가능)

이러한 `flag` 를사용하여 객체 프로퍼티에 대한 각 설정이 가능하다.

실제로 객체 프로퍼티의 접근관련해서 설정해주는 `Object`의 내장 메서드들(`preventExtensions`, `seal`, `freaze` ) 은 전부 이러한 `property descriptor` 를 설정해주어 객체 접근을 못하게 막는것이다.

이렇게 `property descriptor` 에 대한 설명이 이루어졌다.

다시 `Decorator` 로 돌아가서, 위의 설명을 통해 `method decorator` 에 대한 부분을 이해할 수 있게 되었다.

즉, 해당 `method` 에 대한 접근을 제어할 수 있도록 하는 `parameter` 인 것이다.

만약, 반복문을 통한 열거를 원치 않는다고 가정해보자.

```ts

function enumerable(val: boolean) {
  return function (target: any, key: string, desc: PropertyDescriptor) {
    desc.enumerable = val
  }
}

class obj  {
  @enumerable(false)
  test() {…}
}

```

이제 `test` 는 반복문을 통해 열거 되지 않는다.
지금 느끼는 것이지만, 알고보니까 정말 멋진 방법이라는 생각이 든다.

기능을 분리해서 훨씬더 명시적으로 코드가 변경되는 것을 볼 수 있다.

#### Accessor Decorator

지금까지의 `property` 는 `javascript` 에서  `data property` 라고 부른다.

말 그대로 `data` 를통한 설정을 위한 `property` 라고생각하면 된다.

하지만 이러한 `data property` 로는 부족한지, 새로운 `property` 가 만들어지는데 이 새로운 종류의 프로퍼티는 함수이며, 값을 설정하고, 값을 가져오는 역할만을 담당하는 함수 프로퍼티이다.

> 간단하게 `getter` 와 `setter` 함수이다.

이러한 새로운 종류의 프로퍼티를 `Accessor property` 라고부른다.
이러한 `Access property` 를 다루는 `Decorator` 역시 따로 존재하는데 이는 다음과 같다.

```ts
function configurable(val: boolean) {
  return function(target: any, key: string, desc: PropertyDescriptor) {
    desc.configurable = val
  }
}

class User {
  private _name: string
  constructor (name: string, email: string) {
    this._name = name
  }

  @configurable(false)
  get name() {
    return _name
  }

}
```

위를 통해 보면 `configurable` 을 `false` 로 설정하는 `Access Decorator` 를 만들었다.
참고로 `Accessor Property` 를 다루는데 `PropertyDescriptor` 는 약간 다른 `flag` 를 갖는다.

- get: `property` 를 읽을때 사용
- set: `property` 쓰기할때 사용
- enumerable: 반복문에 의해 나열될지 설정
- configurable: `property` 설정 및 삭제 가능한지 설정

`Data Property` 와는 다르게, `writable` 이 없어지고, `get` 과 `set` 이 존재할뿐 나머지는 동일하다

#### property decorator

`Property Decorator` 는 다음처럼 작동 가능하다

```ts
function form(str: string) {
  return function(target: any, propertyKey: string) {
    let val = target[propertyKey]
    
    const getter = () => {
      return `${str} ${val}`
    }
    const setter = (newVal: string) => {
      this.val = newVal
    }
    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter,
      configurable: true,
      enumarable: true,
    })
  }
}

class Welcom {
  @form('welcome')
  name: string

  constructor(name: string) {
    this.name = name
  }
}

const welcome = new Welcom('jh')
consoe.log(welcom.name) // welcome jh
welcome.name = 'JH'
console.log(welcom.name) // welcome JH
```

여기서 보면, `name` 프로퍼티에 `property decorator` 를 사용하여, 기본값을 다른 값으로 출력되도록 만들었다.

`value` 값을 받기 위해, `target` 에서 `propertyKey` 값을 받은이후에, `val` 값으로 해당 `property` 의 값을 받는다.

이때, 처음 `property decorator` 가 시작될때는, 적용된 값이 없어서 `undefined` 로 작동되지만, `instance` 생성이후 값을 전달하게 되면, 그 다음부터 `val` 값에 `value` 값이 들어가게 된다.

이후부터, `welcom.name` 을 통해 값을 가져오면, `name` 은 `welcom ${name}` 값을 반환하며, `welcome.name = value` 값을 주면 `name` 이 `value` 로 변경되어 출력된다.

#### Parameter Decorator

이 `decorator` 는 함수의 `parameter` 에 적용하는 `decorator` 이다.
이때 받는 `decorator` 의 인자는 다음과 같다

1. target: class 의 Proptotype
2. propetyKey: 인자의 `key` 값
3. parameterIndex: 인자의 `index` 값

다음의 예제는 `NestJS 로 배우는 백엔드 프로그래밍` 에 나오는 예제이다.
굉장히 찰떡같은 예제라 `code` 를 참고한다

```ts
function MinLength(min: number) {
  return function (target: any, propertyKey: string, parameterIndex: number) {
    target.validators = {
      minLength: function(args: string[]) {
        return args[parameterIndex].length >= min
      }
    }
  }
}

function Validate(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const method = descriptor.value

  descriptor.value = function(...args) {
    Object.keys(target.validators).forEach(key => {
      if (!target.validators[key](args)) {
        throw Error('error')
      }
    })
    method.apply(this. args)
  }
}

class User {
  private name: string

  @Validate
  setName(@MinLength(3) name: string) {
    this.name = name
  }
}

const t = new User()
t.setName('Dexter')
console.log('---')
t.setName('De')
```

더 자세한 예제는 [NestJS 를 배우기전에](https://velog.io/@kshjessica/2%EC%9E%A5-NestJS%EB%A5%BC-%EB%B0%B0%EC%9A%B0%EA%B8%B0-%EC%A0%84%EC%97%90) 를 보도록 하자.

굉장히 잘 설명되어 있다

## 이렇게 `Decorator` 를 알아보았는데...

아직 `객체지향` 을 다루는데 어려운점이 많은 느낌이다.
익숙하지 않아서 그런가, 사용하고 이해하는데 한참을 들여다보고 살펴보고 있다.

대략적인 흐름은 이해하고 있는 상황이라, `NextJS` 를 사용하면서 조금더 익숙해지기 위한 노력이 필요할듯 보인다.

`Design Pattern` 관련 책도 보면서, 공부할 필요성을 많이 느끼게 된것 같다.
