---
title: 'typeorm EntityManager 에 대해서'
date: '2023-06-09'
tags: ['typeorm', 'ORM', 'EntityManager', 'entity manager']
draft: false
summary: 'typeorm EntityManager 에 대해서'
---

# `TypeORM EntityManager` 에 대해서 알아본다.

`EntityManager` 는 `Repository` 와는 비슷하면서도 약간 다르다.  

`Repository` 는 `단일 Entity` 위주라면, `EntityManager` 는 모든 `Entity` 를 관리하는 역할을 하는것 같다.

> `manager` 를 통한 `query`
```ts
mport { DataSource } from "typeorm"
import { User } from "./entity/User"

const myDataSource = new DataSource(/*...*/)
const user = await myDataSource.manager.findOneBy(User, {
    id: 1,
})
user.name = "Umed"
await myDataSource.manager.save(user)
```

위의 로직을 보면 알겠지만, 특정된 `Enitity` 로 고정되지 않는다.
반면 `Repository` 는 `getRepository(User)` 방식으로 특정 `Entity` 에 한정되어 `query` 가 이루어진다.

이러한 부분에서 약간은 다른 개념으로 만들어졌다고 볼 수 있다.

다음의 `API` 를 살펴보자


### dataSource

```ts
const dataSource = manager.dataSource 
```

위의 구문은 `EntityManager` 에 의해 `dataSource` 를 가져온다.

### queryRunner

```ts
const queryRunner = manager.queryRunner
```

`EntityManager` 의 `transaction instance` 에서만 사용된다고 한다. 
이부분은 [transaction](https://typeorm.io/transactions#) 부분을 보아야 겠다.

### transaction

```ts
await manager.transaction(async (manager) => {
    // NOTE: you must perform all database operations using the given manager instance
    // it's a special instance of EntityManager working with this transaction
    // and don't forget to await things here
})
```

이는 단일 `DB transaction` 에서 여러 `DB` 요청이 실행되는 트렌젝션을 제공한다.

이부분은 [transaction](https://typeorm.io/transactions#) 부분을 보아야 겠다.

### query

`SQL` 쿼리문을 실행시킨다.

```ts

const rawData = await manager.query(`SELECT * FROM USERS`)

```

### createQueryBuilder

`QueryBuilder` 를 생성한다.
문법은 [QueryBuilder](https://typeorm.io/select-query-builder#) 에서 참고할수 있다.

```ts
const users = await manager
    .createQueryBuilder()
    .select()
    .from(User, "user")
    .where("user.name = :name", { name: "John" })
    .getMany()
```

### hasId 

`Property` 에 정의된 `PrimaryColumn` 을 가졌는지 확인한다.
결과는 `Boolean` 값이다.

```ts

if (manager.hasId(user)) {
    // ... do something
}
```

### getId 

`hasId` 와 비슷히지만, `Primary Key` 값을 반환한다.

```ts
const userId = manager.getId(user) // userId === 1
```

### create

새로운 `Instance` 를 생성한다.  
선택적으로 객체 리터럴을 제공하여,`Properties` 의 값을 정의할 수 있다.

```ts
onst user = manager.create(User) // same as const user = new User();
const user = manager.create(User, {
    id: 1,
    firstName: "Timber",
    lastName: "Saw",
}) // same as const user = new User(); user.firstName = "Timber"; user.lastName = "Saw";
```

### merge

여러 `Entity` 들을 하나의 `Entity` 로 합친다.

```ts
const user = new User()
manager.merge(User, user, { firstName: "Timber" }, { lastName: "Saw" }) // same as user.firstName = "Timber"; user.lastName = "Saw";
```

### preload

`plain javascript Object` 를 사용하여 새로운 `Entity` 를 생성한다.

만약 `entity` 가 이미 존재한다면, 이것은 `load` 되고난 다음 새로운 값으로 대체된다.

그리고 새로운 `entity` 를 리턴한다.

```ts
const partialUser = {
    id: 1,
    firstName: "Rizzrak",
    profile: {
        id: 1,
    },
}
const user = await manager.preload(User, partialUser)
// user will contain all missing data from partialUser with partialUser property values:
// { id: 1, firstName: "Rizzrak", lastName: "Saw", profile: { id: 1, ... } }
```

### save

주어진 `entity` 또는 `entity` 의 배열을 저장한다.  
만약 이미 존재한다면, `updated` 될것이다.  
만약 존재하지 않다면 `inserted` 될것이다.

모든 주어진 `entity` 들은 하나의 `transaction` 안에 저장된다.
또한, 정의되지 않은 `properties` 는 전부 건너뛰기 때문에 부분 업데이트도 지원한다.

`NULL` 을 만들기 위해서는, `NULL` 과 같도록 수동으로 지정해주어야 한다.

```ts
await manager.save(user)
await manager.save([category1, category2, category3])
```

### remove

주어진 `entity` 혹은 `entity` 배열을 삭제한다.
`remove` 는 `single transaction` 안에 주어진 모든 `entity` 들을 전부 삭제처리한다.

```ts
await manager.remove(user)
await manager.remove([category1, category2, category3])
```

### insert

새로운 `entity` 를 `insert` 한다.
이는 배열일수도 있다.

```ts
await manager.insert(User, {
    firstName: "Timber",
    lastName: "Timber",
})

await manager.insert(User, [
    {
        firstName: "Foo",
        lastName: "Bar",
    },
    {
        firstName: "Rizz",
        lastName: "Rak",
    },
])
```

### update``

`entity` 를 부분적으로 업데이트 한다.

```ts
await manager.update(User, { age: 18 }, { category: "ADULT" })
// executes UPDATE user SET category = ADULT WHERE age = 18

await manager.update(User, 1, { firstName: "Rizzrak" })
// executes UPDATE user SET firstName = Rizzrak WHERE id = 1
```

### upsert

`entity` 가 존재한다면, `update` 하고, 없다면, `insert` 한다.

```ts
await manager.upsert(
    User,
    [
        { externalId: "abc123", firstName: "Rizzrak" },
    ],
    ["externalId"],
)
/** executes
 *  INSERT INTO user
 *  VALUES
 *      (externalId = abc123, firstName = Rizzrak),
 *  ON DUPLICATE (externalId) KEY UPDATE firstName = "Rizzrak"
 **/
```

> 위는 `Docs` 와 예시가 약간다르다.
>
> `MariaDB` 는 위처럼 사용하는듯 싶어 바꾸었다.

### delete  

`Entity` 의 `id`, `ids` 가 주어지면 삭제한다.  
`remove` 는 `entity` 를 제공하여 삭제하지만,  
`delete` 는 `ids` 혹은 주어진 조건에 맞는 `entity` 를 삭제한다

```ts
await manager.delete(User, 1)
await manager.delete(User, [1, 2, 3])
await manager.delete(User, { firstName: "Timber" })

```

### increment  

주어진 `entity` 와함께 옵션이 주어지면, 주어진 값 만큼 `increment` 된다.

```ts

await manager.increment(User, { firstName: "Timber" }, "age", 3)
```

### decrement

주어진 `entity` 와함께 옵션이 주어지면, 주어진 값 만큼 `decrement` 된다.

```ts

await manager.decrement(User, { firstName: "Timber" }, "age", 3)

```

### count

`FindOptions` 와 `match` 되는 `entites` 의 갯수를 가져온다.  
`pagenation` 할때 유용하다.

```ts
const count = await manager.count(User, {
    where: {
        firstName: "Timber",
    },
})
```

### countBy

`FindOptionsWhere` 절에 `match` 되는 `entities` 들의 갯수를 가져온다.

```ts

const count = await manager.countBy(User, { firstName: "Timber" })

```

### find

`FindOptions` 에 `match` 되는 `entites` 들을 가져온다

```ts
const timbers = await manager.find(User, {
    where: {
        firstName: "Timber",
    },
})
```

### findBy

`FindOptionsWhere` 에 맞는 `entities` 들을 가져온다

```ts

const timbers = await manager.findBy(User, {
    firstName: "Timber",
})

```

### findAndCount

`FindOptions` 에 맞는 `entities` 와 갯수를 가져온다.

```ts
const [timbers, timbersCount] = await manager.findAndCount(User, {
    where: {
        firstName: "Timber",
    },
})
```

### findAndCountBy

`FindOptionsWhere` 에 맞는 `entities` 들과 갯수를 가져온다.

```ts
const [timbers, timbersCount] = await manager.findAndCount(User, {
    firstName: "Timber",
})
```

### findOne는

`FindOptions` 에 맞는 `entity` 를 하나만 가져온다.  
중복된다면 아마도, 첫번째 값을 가져오지 않을까 싶다.

```ts

const timber = await manager.findOne(User, {
    where: {
        firstName: "Timber",
    },
})

```

### findOneBy  

`FindOptionsWhere` 에 맞는 `entity` 를 하나만 가져온다.  

```ts
const timber = await manager.findOneBy(User, { firstName: "Timber" })
```

### findOneOrFail  

`FindOptions` 에 맞는 `entity` 를 가져오거나, 맞는 값이 없다면 `promise` 가 `reject`을 리턴한다.

```ts
const timber = await manager.findOneOrFail(User, {
    where: {
        firstName: "Timber",
    },
})
```

### findOneByOrFail

`findOneOrFail` 과 같지만, `FindOptionsWhere` 을 통해 처리된다. 

```ts

const timber = await manager.findOneByOrFail(User, { firstName: "Timber" })

```

### clear  

주어진 `table` 의 모든 `data` 를 지운다.

```ts
await manager.clear(User)
```

### getRepository  

지정된 `entity` 의 작업을 수행하기 위해 `Repository` 를 가져온다.

```ts
const userRepository = manager.getRepository(User)
```

### getTreeRepository  

지정된 `entity` 의 작업을 수행하기 위해 `TreeRepository` 를 가져온다.

```ts
const categoryRepository = manager.getTreeRepository(Category)
```

### getMongoRepository  

지정된 `entity` 의 작업을 수행하기 위해 `MongoRepository` 를 가져온다.

```ts
const categoryRepository = manager.getMongoRepository(User)
```

### withRepository  

`transaction` 안에서 `custom repository instance` 를 사용하기 위해 가져온다.

```ts

const myUserRepository = manager.withRepository(UserRepository)

```

### release  

`EntityManager` 의 `queryRunner` 를 해제한다.  
현재로써 정확하게는 아직 어떠한 기능인지 감이 안잡힌다.

```ts
await manager.release();
```





