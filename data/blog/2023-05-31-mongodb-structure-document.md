---
title: 'mongodb structure 에 대해서'
date: '2023-05-31'
tags: ['mongodb', '몽고DB', 'document structure']
draft: false
summary: 'document structure 에 대해서'
---

# mongodb structure

> 지속적으로 `backend` 지식을 습득하기 위해 공부중이다. <br/> 현재 `DB` 를 `mongodb` 사용하며, 그에 따른 `API` 를 만들기 위해서는 `Document structure` 에 대한 지식이 필요할것 같다. 

## Schema-less Or not

`Mongodb` 는 사실 `Schema` 가 존재하지 않으며, `Document` 라는 개념으로 존재한다.

> `Schema` 은 `DB` 에서 구조설계할때, 개념을 구조적으로 만든것이다. 그러므로 한번 구조를 정해 놓으면, 이후 변경하기기 어렵다.

`Document` 의 특징은 매우 유연하게 구조를 만들 수 있다는것이다.
마치 `Javascript` 의 객체에 `Property` 를 추가하거나, 삭제하는등 마음대로 작성 가능하다.

이러한 특징은 장점이기도 하며, 단점이기도 하다

> 구조적으로 만들기 위해서는 `Schema` 의 개념이 필요한건 사실이다. `Document` 구조가 유연하다면, 예상치 못한 `Data` 가 삽입될 수도 있기에, 어느정도 강제하는것 역시 좋은 방법이다.
>
> 이로 인해 `Mongoose` 라는 `ORM` 이 탄생했으며, `Schema` 를 지원해준다. 
>
> 사실 무엇이 맞고 아니라고 생각하기는 어려운 것 같다.

```sh
> use shop

> db.products.insertOne({ name: "A book", price: 12000})

> db.products.insertOne({ title: "Keyboard", price: 50000, available: true})
```

위처럼 `porducts collection` 에 `document` 삽입시 정해진 형태없이, 원하는 값을 넣을 수 있다. 

위처럼 작성된다면, 혼돈의 카오스로 입장하는 지름길이다.
정해진 틀에서, 추가적인 `field` 가 작성되도록 하는 형태가 중요한듯 싶다.

```sh

> use shop

> db.products.insertOne({ name: "A book", price: 12000})

> db.products.insertOne({ name: "Keyboard", price: 50000, "details": { slider: "kail red" }})

```

이렇게, `name` 과 `price` 는 공통이며, `details` 만 추가된 형태로 작성된다.

만약 `Scehma` 로 작성되었다고 가정하고 만들어보면 이러한 형태를 띌것이다.

```sh

> use shop

> db.products.insertOne({ name: "A book", price: 12000, details: null})

> db.products.insertOne({ name: "Keyboard", price: 50000, details: { slider: "kail red" }})

```

상단에는 `details` 가 존재하지만, 없기때문에 `null` 값을 준다.

정해진 형태라기 보다는, 유연하게 새로운 `field` 를 줄수 있는것이 `mongodb` 의 `document` 구조이다.

## data types
data 구조를 알기 전에 `Mongodb` 에서 사용되는 `Type` 들만 살펴본다.

- Text
```sh
> db.products.insertOne({name: "MacBook"}) // Text
```
- Boolean 
```sh
> db.products.insertOne({aviliable: true}) // Boolean
```
- Number 
> `mongosh` 에서 저장되는 숫자의 `type` 은 `default` 로 `double` 이다.
>
> 이는 `javascript` 가 `64bit floating point` 로 작동되는것에 기인한 것으로 생각한다. 

Internger (int32) </br>
```sh
> db.products.insertOne({quantity: Int32(4)}) // Int
```
NumberLong (int64) </br>
```sh
> db.products.insertOne({quantity: Long("1")}) // Long
```
NumberDecimal </br>
```sh
> db.products.insertOne({price: 12.99}) // Double

> db.products.insertOne({price: Decimal128(12.99)}) // Decimal128
```
- ObjectId
```sh
> db.products.insertOne({_id: ObjectId("6114216907d84f5370391919")}) // ObjectId
```
- ISODate
```sh
> db.products.insertOne({insertedAt: new Date()}) // ISODate
```
- Timestampe
```sh
> db.products.insertOne({insertedAt: new Timestampe()}) // Timestampe
```
- Embedded Document
```sh
> db.products.insertOne({detailes: { ... } }) // Embedded Document
```
- Array
```sh
> db.products.insertOne({nums: [1, 2, 3]}) // Array
```

`Mongodb` 는 위의 `BSON` 타입을 사용하여 `Document` 를 구성한다.

### Nested / Embedded Document VS References 

> Nested / Embedded document
```ts
{
  _id: ObjectId("6114216907d84f537039191a")
  username: "Jhon",
  address: {
    _id: ObjectId("6114216907d84f537039191b"),
    street: "N Street",
    city: "Seoule",
  }
}
```

> Refernces

```ts

{
  _id: ObjectId("6114216907d84f537039191a")
  username: "Jhon",
  addressId: ObjectId("6114216907d84f537039191b"), 
}
```

```ts
address: {
  _id: ObjectId("6114216907d84f537039191b"),
  street: "N Street",
  city: "Seoule",
}
```

`SQL` 에서는 다른 `Table` 과 연관시키기 위해서 그 `Table` 을 `Join` 한다

`Document model` 은 `SQL` 처럼 `Join` 하는 구조가 아니다.
`Document` 는 위의 `Embedded document` 를 사용하여, `Nested` 형태를 만들수도 있으며, 다른 `Document`의 `ObjectId` 를 할당하는 구조로도 사용가능하다. 

`ObjectId` 로 참조하는 방식을 `Refernces` 라고 하는데, 이는 `Document` 를 `ObjectId` 로 연관시키는 방법이다.

> ***주의할점***
>
> 위의 `addressId` 는 `ObjectId` 를 가진 `Document` 가 아닌
> 정말 `ObjectId` 만을 가진다.

현재 내가 알기로는 `Mongodb` 차체에서, 해당 `ObjectId` 의 `Document` 를 넣기 위해서는, `ObjectId` 를 사용하여 `Query` 한후, 재할당해주는 방식으로 처리하는걸로 알고 있다.

> 이러한 불편함을 제거하기 위해 `Mongoose` 에서는 `Populate` 라는 기능을 제공해준다.

그럼에도 불구하고 `ObjectId` 를 넣는 이유는 무엇일까?
첫째로, `Embedded document` 는 `ObjectId` 보다 크기가 클수 밖에없다.

만약, `Embadded document` 를 가진 배열을 가진 `field` 가 있다고 가정하자.


```ts
{
  _id: ObjectId("6114216907d84f537039191a")
  username: "Jhon",
  addresses: [
    {
      _id: ObjectId("6114216907d84f537039191b"),
      street: "N Street",
      city: "Seoule",
    },
    {
      _id: ObjectId("6114216907d84f537039191c"),
      street: "N Street",
      city: "Busan",
    },
  ],
}
```

위는 `Embadded document` 를 가져도 별 문제가 되지는 않는다.
배열의 수가 많지 않기 때문이다.

하지만, `document` 수가 기하급수적으로 많은 상황이라면 어떻게 될까?
`mongodb` 에서 `document` 의 사이즈는 `default` 로 `16MB` 인것으로 알고 있다.

많은 `docuemnt`들을 전부 담았을시 `16MB` 를 초과하게 되면 더이상 해당 `document` 사용이 불가능한것이다. 

그럴때는 `ObjectId` 를 담는것이 현명하다.
> `ObjectId` 를 담은 `addresses` 배열
```ts
{
  _id: ObjectId("6114216907d84f537039191a")
  username: "Jhon",
  addresses: [
      _id: ObjectId("6114216907d84f537039191c"),
      _id: ObjectId("6114216907d84f537039191d"),
      _id: ObjectId("6114216907d84f537039191e"),
    ...
  ],
}
```

이렇게 하면, 각 `Object` 를 담았을때 보다 사이즈가 많이 줄어들수 있다.

바로 이러한 이유 (`내가 알고있는 이유일 뿐이다...`) 로 인해,  
더 효율적인 관리를 할 수 있도록 `ObjectId` 를 담는 상황이 발생할 수 있다.

실제로, 데이터수가 많지 않으면, `Embadded Document` 방식으로 하는것을 추천하기도 한다.

이는 어떠한 상황인지에 따라 달라질 수 있는 부분으로, 설계시 많은 경험을 바탕으로 만들어지는 부분인것 같다.

## One to One relationship

한 사람당, 하나의 직업을 가진다고 가정해보자.
그럼 `1:1 relationship` 이 형성된다

이를 `Mongodb` 에서 구현해보면 다음과 같은 구조가 된다.

> Persons Collection
```sh
> db.persons.insertOne({
  _id: ObjectId("6114216907d84f537039180a"),
  name: "Jhon",
  email: "Jhon@email.com",
  job: ObjectId("6114216907d84f537039191d"),
})

```

> Jobs Collection
```sh
> db.jobs.insertOne({
  _id: ObjectId("6114216907d84f537039191d"),
  name: "Programmer",
})

```

위는 `Person` 이 하나의 `Job` 을 가진 구조로 볼 수 있다.
위는 `ObjectId` 로 참조한 것이다. 

만약 `Person` 에서 `Job document` 를 조회하려면 어떻게 할까? 
이를 `shell` 에서 처리하려면 다음처러 처리 가능하다

```sh
> var jobId = db.persons.findOne({_id: "6114216907d84f537039180a"}).job

> db.jobs.findOne({_id: jobId})
{
  _id: ObjectId("6114216907d84f537039191d"),
  name: "Programmer",
}

```

이는 `ObjectId` 를 통해 사용한 `References` 방법이며, 
이렇게 사용안하고 `Embadded` 시켜도 상관없다.


> Persons Collection & Embadded Document
```sh
> db.persons.insertOne({
  _id: ObjectId("6114216907d84f537039180a"),
  name: "Jhon",
  email: "Jhon@email.com",
  job: {
    _id: ObjectId("6114216907d84f537039191d"),
    name: "Programmer",
  }
})

```

위의 예시는 `Embadded` 시킨 예시이다.
위 같은경우(`값이 많지 않은 객체 및 참조 jobs 의 종류가 많지 않은경우`)는 `Embadded` 시키는것이 더 나은 방법일수 있지만,이는 어떻게 사용되느냐에 따라 다른 부분이기에 경험을 통해 만들어주는게 중요한듯 싶다.

> 위는 가상의 상황을 만든것으로 1:1 관계를 생각하고 가정한것이다.

## One to Many

`One to Many` 의 상황은 게시판에서 볼 수 있을듯 싶다.

간단한 상황을 가정하고 생각한다면, 1개의 `Post` 안에 여러개의 `comments` 들이 있을 수 있다.

> Post document
```sh
> db.posts.insertOne({
  title: "Awasome!!",
  author: "Jhoon",
  description: "...",
  comments: [
    ObjectId("6114216907d84f537039195a"),
    ObjectId("6114216907d84f537039195b"),
  ]
})
{
  _id: ObjectId("6114216907d84f537039191a"),
  title: "Awasome!!",
  author: "Jhoon",
  description: "...",
  comments: [
    ObjectId("6114216907d84f537039195a"),
    ObjectId("6114216907d84f537039195b"),
  ],
}
```

> Comments document
```sh
> db.comments.insertOne({
  author: "Mark",
  comment: "wow!!"
})
{
  _id: ObjectId("6114216907d84f537039195a"),
  author: "Mark",
  comment: "wow!!"
}

> db.comments.insertOne({
  author: "Jim",
  comment: "powerful!!"
})
{
  _id: ObjectId("6114216907d84f537039195b"),
  author: "Jim",
  comment: "powerful!!"
}
```

여기서 `Comments` 가 얼마나 참조될지 예상이 되지 않아서, 일단 `Comments Collection` 을 만들고, 해당하는 `Document id` 를 가져오는 방식으로 작성했다.

이 역시, `Comments` 의 갯수가 많지 않다면, `Embadded` 형태로 만들어도 상관은 없다.

만약, `References` 방식으로 작성한다면 
`City` 안에 있는 `Citizen` 의 형태가 적절해 보인다.

`City` 는 1개이고, `Citizen` 은 수만명이기 때문이다.
이렇게 많은 수의 사람을 `City` 에 담아야 한다면 당연하게도 `References` 방식을 채택하여 구조를 만들어야만 할 것이다.

### Many to Many

`Many to Many relationship` 은 보통 `구매자` 와 `상품` 의 관계에 비유할 수 있다.  

`Customer` 는 여러개의 `Product` 를 가질 수 있고, `Product` 는 여러 `Customer` 를 가질수 있다. 

> Customer Document
```sh
> db.customers.insertOne({
  name:  "Jhoon",
})
{
  _id: ObjectId("6114216907d84f537039195a"),
  name: "Jhoon",
}

```

> Product Document
```sh
> db.customers.insertOne({
  title: "Keyboard"
})
{
  _id: ObjectId("6114216907d84f537039198a"),
  title: "Keyboard",
}
```

이렇게 각 `Customer` 와 `Product` `Document` 를 만들고,  
다음의 주문에 넣어주면, `Many to Many relationship` 을 만들 수 있다. 

```sh
db.orders.insertOne({
  customerId: ObjectId("6114216907d84f537039195a"), 
  products: [
    {
      productId: ObjectId("6114216907d84f537039198a"), 
      quantity: 1
    }
  ]
})
```

과연 이것이 옳은것일까?
`Mongodb` 에서는 `Embedded Document` 가 존재한다.

굳이 `Orders` 를 빼서 처리할 필요없이 `customer` 에 `oders` 라는 배열을 만들고, 각 객체를 담아주는 방식으로도 구현 가능하다.

```sh
> db.customers.updateOne({_id: ObjectId("6114216907d84f537039195a"), {
    $set: { 
      orders: [
        { 
          productId: ObjectId("6114216907d84f537039198a"), 
          quantity: 1
        },
        ... and so on
      ] 
    }
  }
})
{
  _id: ObjectId("6114216907d84f537039195a"),
  name: "Jhoon",
  orders: [
    { 
      productId: ObjectId("6114216907d84f537039198a"), 
      quantity: 1
    },
    ... and so on
  ]
}
```

이렇게 `Embedded` 형태로 구현할수도 있다,
`References` 방식으로 구현할지 `Embedded` 방식으로 구현할지는 선택사항인듯 싶다.

여기서 추가로 덧붙혀서 이야기 하지면,  
위의 로직상 `Embedded` 로 만드는데, `productId` 를 사용하여 `References` 할 필요성이 있는가 하는점이다. 

`References` 방식으로 처리하는것은, `Javascript` 의 객체 메모리 참조와 같은 형태로 구현되는 방식이라고 생각하면 된다.

즉, `ObjectId` 값을 사용하여, 만들어진 `Document` 를 공유한다는 것이다.

`Many to Many relationship` 같은 경우, `ObjectId` 를 통한 `References` 방식을 사용하는것이 좋다.

서로가 서로를 지속 참조하는 형태이므로, 중간에 `Product` 의 정보가 변경되었을시, `References` 로 참조가 아닌경우에는 모든 `Product` 를 조회하고 값을 수정해야 할것이다.

하지만, `References` 를 통해 참조하므로, 간단하게 해당 `Product` 의 정보만 수정하면, 해당 `ObjectId` 를 가진 `Document` 는 수정된 `Product` 를 `Query` 하면 간단하게 문제가 해결되는 장점이 있다.

> 물론, `data` 변경이 전혀 없는, 확정적인 `data` 같은 경우 `References` 를 통해 만들지 않아도 될것이다.

보통 이러한 `Structure` 를 만들때, 일반적으로 많이 사용되는 패턴은 다음과 같다고 이야기한다.

- `One to One` <br/>
-> Embedded

- `One to Many` <br/>
-> Embedded

- `One to Many` <br/>
-> References

위처럼 작성하는 것이 일반적이기는 하지만, `Database` 구조상 언제든지 변경도리수 있으며, 상황에 따라 변경되어야 하는 부분이므로, 절대적인 상황은 아니다.

이러한 설계는 많은 공부와 경험이 축적되어야 좋은 설계를 할 수 있을듯 싶다.

