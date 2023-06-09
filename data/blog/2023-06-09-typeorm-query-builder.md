---
title: 'typeorm queryBuilder 에 대해서'
date: '2023-06-09'
tags: ['typeorm', 'ORM', 'queryBuilder', 'query builder']
draft: false
summary: 'typeorm queryBuilder 에 대해서'
---

# `TypeORM queryBuilder` 에 대해서 알아본다.

`QueryBuilder` 는 `TypeORM` 의 강력한 기능중에 하나이다.  
마치 `SQL query` 와 비슷하게 우아하고, 편기한 기능을 제공한다.

다음을 보자.

```ts
const firstUser = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .getOne()

```

위를 보면 `getRepository` 를 사용한후 `createQueryBuilder` 를 호출한다. 그와 동시에 `build` 할 `user` 명을 적어주고,  
`SQL` 처럼 `where` 절을 통해 조건을 적어준다.  

그리고 `getOne` 메서드를 사용하여 검색된 결과중 하나만을 가져온다.

이 구문은 다음의 `SQL` 과 같다.

```sql

SELECT
    user.id as userId,
    user.firstName as userFirstName,
    user.lastName as userLastName
FROM users user
WHERE user.id = 1

```

그리고 리턴값은 다음과 같다.


```ts
User {
    id: 1,
    firstName: "Timber",
    lastName: "Saw"
}
```

## `QueryBuilder` 를 사용하는데 중요한것

`QueryBuilder` 를 사용할때, `where` 절에서 `unique parameter`들을 제공해야한다

다음은 `unique` 하지 못한 `parameter` 를 전달한 예이다.

```ts
const result = await dataSource
    .createQueryBuilder('user')
    .leftJoinAndSelect('user.linkedSheep', 'linkedSheep')
    .leftJoinAndSelect('user.linkedCow', 'linkedCow')
    .where('user.linkedSheep = :id', { id: sheepId })
    .andWhere('user.linkedCow = :id', { id: cowId });
```

위를 보면 `:id` 가 두번 적혀있다.  
이렇게 작성하지 말고, `Uniqued name` 을 작성을 하란 말이다.
위처럼 작성하면 제대로 작동하지 않는다고 강조하고 있다.


```ts
const result = await dataSource
    .createQueryBuilder('user')
    .leftJoinAndSelect('user.linkedSheep', 'linkedSheep')
    .leftJoinAndSelect('user.linkedCow', 'linkedCow')
    .where('user.linkedSheep = :sheepId', { sheepId })
    .andWhere('user.linkedCow = :cowId', { cowId });
```

이렇게 하면 제대로 작동한다.

## `QueryBuilder` 생성법 

`QueryBulder` 를 생성하는데 여러가지 방법이 존재한다.

### DataSource 사용

```ts

const user = await dataSource
    .createQueryBuilder()
    .select("user")
    .from(User, "user")
    .where("user.id = :id", { id: 1 })
    .getOne()

```

### EntityManager 사용

```ts
const user = await dataSource.manager
    .createQueryBuilder(User, "user")
    .where("user.id = :id", { id: 1 })
    .getOne()
```

### Repository  사용가능하용

```ts
const user = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .getOne()
```

> 그리고 사용되는 5가지 `queryBuilder` 유형이 존재한다고 한다.

### SelectQueryBuilder

이는 `Select` 쿼리를 작성하고 실행한다.

```ts
const user = await dataSource
    .createQueryBuilder()
    .select("user")
    .from(User, "user")
    .where("user.id = :id", { id: 1 })
    .getOne()
```

### InsertQueryBuilder  

이는 `Insert` 쿼리를 작성하고 실행시킨다

```ts
await dataSource
    .createQueryBuilder()
    .insert()
    .into(User)
    .values([
        { firstName: "Timber", lastName: "Saw" },
        { firstName: "Phantom", lastName: "Lancer" },
    ])
    .execute()
```

### UpdateQueryBuilder  

이는 `Update` 쿼리를 작성하고 실행시킨다.

```ts
await dataSource
    .createQueryBuilder()
    .update(User)
    .set({ firstName: "Timber", lastName: "Saw" })
    .where("id = :id", { id: 1 })
    .execute()
```

### DeleteQueryBuilder

이는 `Delete` 쿼리를 작성하고 실행시킨다.

```ts
await dataSource
    .createQueryBuilder()
    .delete()
    .from(User)
    .where("id = :id", { id: 1 })
    .execute()
```

### RelationQueryBuilder  

특정 `relation` 작업을 작성하고 실행시킨다.
`TBD` 라고 명시되어 있는데.. 
> `To Be Defined`, 문서 작성 시점에는 확정할 수 없어 나중에 확정한다’

라고 되어있다.
이부분은 `DB` 관련 공부를 하고 접근해야 할것같다.
다음은 코드이다.

```ts
await dataSource
    .createQueryBuilder()
    .relation(User,"photos")
    .of(id)
    .loadMany();
```

위는 아무리 봐도, `User` 와 `Photos` 간의 연결을 `id` 로 하고,  
불러오는 로직같은데, 나중에 확정한다는것이 이해가 가지 않는다.

일단은 넘어간다...

## 이제 `QueryBuilder` 를 시작해보자

`User` 의 `id` 혹은 `name` 을 사용하여 하나의 결과값을 얻으려면 다음의 `getOne` 을 사용한다.

```ts
const timber = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id OR user.name = :name", { id: 1, name: "Timber" })
    .getOne()
```

또는 `fail` 시에 `EntityNotFoundError` 를 던질수 있고, 존재하면 하나의 결과값을 받을수 있는 `getOneOrFail` 을 사용할수도 있다.


```ts
const timber = await dataSource
  .getRepository(User)
  .createQueryBuilder("user")
  .where("user.id = :id OR user.name = :name", { id: 1, name: "Timber" })
  .getOneOrFail()
```

`DB` 상 검색단 여러 결과값을 받고 싶다면, `getMany` 를 사용한다.

```ts
const users = await dataSource
  .getRepository(User)
  .createQueryBuilder('user')
  .getMany()

```

여러 결과값을 받는데, 2개의 타입이 있을수 있다.
`Enitites` 와 `Row Results` 이다.

대부분의 경우에는 `DB` 로 부터 `entities` 를 받겠지만,  
특정 `data` 를 선택할 필요가 있을수 있다.  

모든 `user phoots` 의 `sum` 같은 일부 특정 데이터를 말한다. 
이러한 특정 데이터는 `entity` 가 아닌 `raw data` 라고 할수 있다.
이럴때 사용하는것은 `raw data` 를 호출해야한다.

`getRowOne` 혹은 `getRowMany` 를 사용하면, 이러한 `row data` 를 얻을수 있다.

```ts

const { sum } = await dataSource
  .getRepository(User)
  .createQueryBuilder('user')
  .select('SUM(user.photosCount)', 'sum')
  .where('user.id = :id', { id: 1 })
  .getRowOne()

```

```ts
const photosSums = await dataSource
  .getRepository('user')
  .select('user.id')
  .addSelect('SUM(user.photosCount)', 'sum')
  .groupBy('user.id')
  .getRowMany()

  // result will be like this: [{ id: 1, sum: 25 }, { id: 2, sum: 13 }, ...]
```

> 여기서 말하는 `Entity` 는 현재 우리가 계속해서 만든 `Class` 를 뜻한다. `row data` 는 우리가 만든 형식인 `Entity` 와는 별개의 `DB` 연산을 통해 만들어진 `Data` 이므로, 이러한 `Data` 를 `row data` 라고 인지하는것이 좋을듯 싶다.

## 카운트를 얻자

`query` 된 `row` 의 갯수를 얻기 위해서 사용하는 메서드는 `getCount` 이다.

```ts
const count = await dataSource
  .getRepository(User)
  .createQueryBuilder('user')
  .where('user.name = :name', { name: 'Timber'})
  .getCount()
```

위는 `Timber` 의 이름을 가진 `Record` 의 갯수를 반환한다.
이는 다음의 `SQL` 과 같다.

```sql
SELECT count(*) FROM `users` as `user` where `user`.`name` = `Timber`;
```

## Aliases 는 무엇인가?

지금까지 `createQueryBuilder('user')` 형식으로 사용해 왔다.  
`user` 는 무엇인가 싶을텐데, 이는 `seleted data` 의 별칭을 뜻한다.

`SQL` 에서 사용되는 `as` 와 같다.

이러한 `Aliases` 는 어디에든 사용가능하다. 

```ts
createQueryBuilder().select("user").from(User, "user")
```

위는 다음과 같다.

```sql

SELECT ... FROM users AS user

```

보톤 이러한 `Aliases` 는 여러 `Table` 을 사용하는 쿼리에서 각 쿼리를 구분하기 위해 많이 사용된다.

## Using parameters to escape data

여태껏 `where("user.name = :name", { name: "Timber" })` 방식의 구문을 많이 사용했다.

이유는 `SQL` 에 인자값이 문자열로 주입되지 않도록, `parameter` 로써 제공하는 것이다.

만약 이렇게 사용안하면 굉장히 불편한 다음의 방식으로 구성될수도 있다.

```ts
where("user.name = '" + name + "')
```

물론 이방식대로 작성해도 제대로 동작은 하겠지만, 이는 안전하지 못하며, 실수가 발생하기 쉽다.

그러므로 이러한 부분을 방지하기 위해 새로운 특별한 문법을 사용하여 처리하는것이다.

```ts
where("user.name = :name", { name: "Timber" })
```

여기서 `:name` 은 `name` 과 매핑되어 값이 할당된다.
굉장히 편리하다.

이는 아래의 `shortcut` 이라고 한다.

```ts
.where("user.name = :name")
.setParameter("name", "Timber")
```

`Array` 배열도 받을수 있는데 다음처럼 사용하라고 한다.

```ts
.where("user.name IN (:...names)", { names: [ "Timber", "Cristal", "Lina" ] })
```

이는 다음의 `SQL` 문과 같다.

```sql
WHERE user.name IN ('Timber', 'Cristal', 'Lina')
```

## `Where` 표현식

`where` 표현식을 사용하는 방법을 살펴본다.
일단 다음의 기본 사용법이 있다.

```ts
createQueryBuilder("user").where("user.name = :name", { name: "Timber" })
```

그리고 `AND` 를 추가할수도 있다.

```ts
createQueryBuilder("user")SELECT ... FROM users user WHERE user.id IN (1, 2, 3, 4)
```

`OR` 을 사용하기 위해서는 다음의 문법을 사용한다.

```ts
createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Timber" })
    .orWhere("user.lastName = :lastName", { lastName: "Saw" })
```

이는 다음의 `SQL` 과 같다

```sql
SELECT ... FROM users user WHERE user.firstName = 'Timber' AND user.lastName = 'Saw'
```

`IN` 은 다음처럼 사용한다.

```ts
createQueryBuilder("user")
    .where("user.id IN :...ids", { ids: [1, 2, 3, 4] })
```

이는 다음의 `SQL` 과 같다.

```sql
SELECT ... FROM users user WHERE user.id IN (1, 2, 3, 4)
```

`Bracket` 을 사용하여 조금 복잡한 `Where` 절을 만들수도 있다다

```ts
createQueryBuilder("user")
    .where("user.registered = :registered", { registered: true })
    .andWhere(
        new Brackets((qb) => {
            qb.where("user.firstName = :firstName", {
                firstName: "Timber",
            }).orWhere("user.lastName = :lastName", { lastName: "Saw" })
        }),
    )
```

개인적으로 더 복잡해보이는데, 
`Brackets` 를 지원하는 이유가 분명히 존재할것이다.  
아직 그렇게 와닿지는 않는다.

일단 이러한 문법을 사용할수 있다는 정도로 이해하고 들어간다.
위의 쿼리는 다음의 `SQL` 쿼리로 만들어진다.

```sql
SELECT ... FROM users user WHERE user.registered = true AND (user.firstName = 'Timber' OR user.lastName = 'Saw')
```

또는 `NotBrackets` 를 이용한 `where` 가 존재한다.

```ts
createQueryBuilder("user")
    .where("user.registered = :registered", { registered: true })
    .andWhere(
        new NotBrackets((qb) => {
            qb.where("user.firstName = :firstName", {
                firstName: "Timber",
            }).orWhere("user.lastName = :lastName", { lastName: "Saw" })
        }),
    )
```

이는 다음의 `SQL` 문으로 만들어진다.

```sql
SELECT ... FROM users user WHERE user.registered = true AND NOT((user.firstName = 'Timber' OR user.lastName = 'Saw'))
```

필요하다면, 이렇게 많은 `AND`, `OR` 을 사용하여 구현가능하다.  
만약 `where` 을 두번이상 사용하면 앞전의 `where` 을 덮어씌어진다고 말하고 있다.

아... 그래서 `Brackets` 를 지원하는구나..
이제 이해되었다.

## `HAVING` 절

`HAVING` 은 `GROUP BY` 절로 선택된 그룹에 대한 탐색 조건을 지정한다.

이는 다음처럼 사용될 수 있다.

```ts
reateQueryBuilder("user").having("user.name = :name", { name: "Timber" })
```

이는 다음의 `SQL` 표현식으로 작성된다.

```sql
SELECT ... FROM users user HAVING user.name = 'Timber'
```

`Where` 절과 비슷하게 `AND` 와 같이 사용가능하다.

```ts
createQueryBuilder("user")
    .having("user.firstName = :firstName", { firstName: "Timber" })
    .andHaving("user.lastName = :lastName", { lastName: "Saw" })
```

이는 다음과 같다,

```sql
SELECT ... FROM users user HAVING user.firstName = 'Timber' AND user.lastName = 'Saw'
```

`OR` 은 다음과 같다.

```ts
createQueryBuilder("user")
    .having("user.firstName = :firstName", { firstName: "Timber" })
    .orHaving("user.lastName = :lastName", { lastName: "Saw" })
```

```sql

SELECT ... FROM users user HAVING user.firstName = 'Timber' OR user.lastName = 'Saw'

```

## ORDER BY 절

`ORDER BY` 절은 매우 간단하게 구현가능하다.

```ts
createQueryBuilder("user").orderBy("user.id")
```

이는 다음과 같다.

```sql
SELECT ... FROM users user ORDER BY user.id
```

또한 오름차순으로 하는 방법은 다음과 같다.

```ts
createQueryBuilder("user").orderBy("user.id", "ASC")
```

내림차순은 다음과 같다.

```ts
createQueryBuilder("user").orderBy("user.id", "DESC")
```

또한 여러개의 `ORDER BY` 추가역시 가능하다.

```ts
createQueryBuilder("user").orderBy("user.name").addOrderBy("user.id")

```

## DISTINCT ON 절 (Postgres 만 해당됨)

`DISTINCT ON` 은 `MariaDB` 에도 있는것으로 알고 있는데,  
`Docs` 에서는 `Postgres` 만 해당된다고 한다.

일단 [Adding DISTINCT ON expression](https://typeorm.io/select-query-builder#what-are-aliases-for) 에서 확인은 가능하다.

이부분은 넘어간다.

## `GROUP BY` 절

`GROUP BY` 절은 다음처럼 작성가능하다.

```ts

createQueryBuilder("user").groupBy("user.id")

```

이는 다음처럼 해석된다.

```sql
SELECT ... FROM users user GROUP BY user.id
```

또한 더 많은 그룹화 기준을 추가하려면 다음처럼 작성한다.

```ts
createQueryBuilder("user").groupBy("user.name").addGroupBy("user.id")
```

만약, `.groupBy` 를 한번보다 더 많이 사용된다면, 앞전의 `.groupBy` 를 덮어씌우게 된다.

## LIMIT 절

```ts

createQueryBuilder("user").limit(10)

```

이는 다음처럼 작성된다.

```sql

SELECT ... FROM users user LIMIT 10

```

여기서 주의점으로 다음처럼 설명한다.

> 복잡한 쿼리(조인 및 하위쿼리)를 사용하는경우  
`LIMIT` 가 예상대로 작동하지 않을 수 있다.  
이런경우 대신에 `take` 를 사용하기를 권장한다.

## OFFSET 절

```ts

createQueryBuilder("user").offset(10)

```

이는 다음과 같다.

```sql
SELECT ... FROM users user OFFSET 10
```

이 역시 위의 `LIMIT` 절과 같게, 복잡한 쿼리시 동작을 안할 수 있으니, `skip` 을 사용하라고 한다.

## JOIN 관계

```ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm"
import { Photo } from "./Photo"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @OneToMany((type) => Photo, (photo) => photo.user)
    photos: Photo[]
}
```

```ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm"
import { User } from "./User"

@Entity()
export class Photo {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    url: string

    @ManyToOne((type) => User, (user) => user.photos)
    user: User
}
```

```ts
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .where("user.name = :name", { name: "Timber" })
    .getOne()
```

이를 이러한 방식으로 `JOIN` 을 처리한다.
결과는 다음과 같다.

```ts
{
    id: 1,
    name: "Timber",
    photos: [{
        id: 1,
        url: "me-with-chakram.jpg"
    }, {
        id: 2,
        url: "me-with-trees.jpg"
    }]
}
```

`leftJoinAndSelect` 에서 보면 첫번째 인자는 `JOIN` 할  
`Column` 이고, 두번째는 해당 `Column` 의 `Alias` 이다.

또한 3번째 인자작성도 가능한데, 이는 `JOIN` 할 `Table` 의  `where` 조건문을 대신한다.

다음을 보자

```ts
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo", "photo.isRemoved = :isRemoved", {
        isRemoved: false,
    })
    .where("user.name = :name", { name: "Timber" })
    .getOne()
```

위는 다음의 `query` 와 같다

```sql
SELECT user.*, photo.* FROM users user
    LEFT JOIN photos photo 
    ON photo.user = user.id AND photo.isRemoved = FALSE
    WHERE user.name = 'Timber'
```

위는 `JOIN` 할 `Table` 은 `photo.user = user,id AND photo,isRemove = FALSE` 인 조건의 `Table` 을 가져온다.

그런후, `JOIN` 된 `Table` 에서 `user.name` 이 `Timber` 를 가져온다.

## Inner 그리고 Left Joins

`LEFT JOIN` 뿐만 아니라 `INNER JOIN` 역시 가능하다.

```ts

const user = await createQueryBuilder("user")
    .innerJoinAndSelect(
        "user.photos",
        "photo",
        "photo.isRemoved = :isRemoved",
        { isRemoved: false },
    )
    .where("user.name = :name", { name: "Timber" })
    .getOne()

```

이는 다음과 같다,

```sql

SELECT user.*, photo.* FROM users user
    INNER JOIN photos photo ON photo.user = user.id AND photo.isRemoved = FALSE
    WHERE user.name = 'Timber'

```

## Join without selection

`Select` 없이 `JOIN` 할수도 있다.
이는 `.innerJoin` 및 `.outerJoin` 만을 사용하여 선택가능하다.

```ts
const user = await createQueryBuilder("user")
    .innerJoin("user.photos", "photo")
    .where("user.name = :name", { name: "Timber" })
    .getOne()
```

이는 다음처럼 생성된다.

```sql
SELECT user.* FROM users user
    INNER JOIN photos photo ON photo.user = user.id
    WHERE user.name = 'Timber'
```

보면 알겠지만 `SELECT` 부분에 `user` 만 선택되어 있고,  
`photo` 는 없는것을 볼 수 있다.

## Talbe 혹은 Entity Join

`relations` 될때만 `Join` 이 가능하지 않다.  
관계가 없는 `Table` 및 `Entity` 도 조인이 가능하다.

```ts
const user = await createQueryBuilder("user")
    .leftJoinAndSelect(Photo, "photo", "photo.userId = user.id")
    .getMany()
```

위처럼 `Entity` 를 넣어주면 가능하다.
이는 다음과 같다.

```ts
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("photos", "photo", "photo.userId = user.id")
    .getMany()

```

물론, `Join` 가능한 조건이 있을때만 가능하지만 말이다.

## Joining and mapping functionality

아래는 `User` 에 `profilePhoto` 를 추가한것이다.  
그리고 `QueryBuilder` 를 사용해서 모든 `data` 를 해당 `property` 에 매핑할수 있다고 한다.

```ts

export class User {
    /// ...
    profilePhoto: Photo
}

```
```ts
const user = await createQueryBuilder("user")
    .leftJoinAndMapOne(
        "user.profilePhoto",
        "user.photos",
        "photo",
        "photo.isForProfile = TRUE",
    )
    .where("user.name = :name", { name: "Timber" })
    .getOne()
```

위를 보면 `user.profilePhoto` 에 `photo` 의 조건에 맞는 `Table` 을 매핑해서 넣어준다.

이후 `user.profilePhoto` 안에 해당 값이 담겨있을 것이다.
여기서 하나의 `Entity` 를 로드하고 매핑하려면 `leftJoinAndMapOne` 을 사용하라고 하며, 그렇지 않고 여러 `Entity` 를 로드하고 매핑하려고 한다면, `leftJoinAndMapMany` 를 사용하라고 한다.

## Getting the generated query

`QueryBuilder` 에 의해 생성된 `SQL Query` 를 얻고 싶다면  
`getSql` 을 사용하면 된다.

```ts
const sql = createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Timber" })
    .orWhere("user.lastName = :lastName", { lastName: "Saw" })
    .getSql()
```

`debugging` 할 목적으로 사용한다면 `printSql` 을 사용한다.

```ts
const users = await createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Timber" })
    .orWhere("user.lastName = :lastName", { lastName: "Saw" })
    .printSql()
    .getMany()

```

`printSql` 은 `console` 상에 `print` 된다.

## Getting raw results

결과값은 두개의 타입이 존재한다고 했다.
`Entities` 혹은 `Row Results`,

대부분은 `Entities` 를 사용할 것이다. 하지만 그렇지 않고,  
`SQL` 연산에 의해 새롭게 생성된 `Data` 를 반환해야 하는 경우가 생긴다.

이때 사용하는것인 `getRowOne` 혹은 `getRowMany` 이다.
다음을 보자


```ts
const { sum } = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .select("SUM(user.photosCount)", "sum")
    .where("user.id = :id", { id: 1 })
    .getRawOne()
```

위는 `user.photosCount` 가 합산된 결과를 `sum` 이라는 이름으로 반환되는 `Data` 이다.

당연하게도, 이는 우리가 생성한 `Entity` 가 아닌 다르게 생성된 `Data` 이므로 `getRawOne` 을 통해 가져온것이다.

이는 `GROUP BY` 를 통해 다르게 생성된 값이 있다면 그것도 포함된다.

```ts
const photosSums = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .select("user.id")
    .addSelect("SUM(user.photosCount)", "sum")
    .groupBy("user.id")
    .getRawMany()
```

## Streaming result data

`Data` 를 `Stream` 으로 리턴할수 있다고 한다.
`Streaming` 은 `raw data` 이므로, 직접 `Entity` 를 변환해야 한다고 말한다.

이부분은 많이 와닿는 부분은 아니다.

```ts
const stream = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .stream()
```

## Using pagination

`Application` 은 `Pagenation` 이 필요할수있다.
이때 다음처럼 구현할수 있다.

```ts
const users = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .take(10)
    .getMany()
```
위는 `10` 개까지만 제공된다.

```ts
const users = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .skip(10)
    .getMany()
```

위는 첫번째 `10` 개를 `skip` 하고 나머지를 제공한다.
이 두개를 같이 사용하면 다음과 같다.

```ts
const users = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .skip(5)
    .take(10)
    .getMany()
```

이는 첫번째 5 개를 `skip` 하고 그다음 `10` 개를 반환한다.
이러한 방식을 사용하면 쉽게 `Pagenation` 구현이 가능하다.

`server` 로 구현될시 `varTake` 변수 를 `10` 로 할당하고,  
`varSkip` 변수는 `(pageNumber - 1) * take` 로 할당하면,  
`varSkip` 변수는 해당 `pageNumber` 만큼 `skip` 하고 값을 반환할수 있을것이다.

## Set locking

이 개념에 대해서는 조금더 알고 보아야 할것같아서 일단 넘긴다.

## Use custom index

경우에 따라서 `DB` 에서 사용할 특정 `index` 를 제공할 수 있다.
이기능은 `MySQL` 에서만 제공된다고 한다.

```ts
const users = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .useIndex("my_index") // name of index
    .getMany()
```

## Max execution time

`query` 실행시 최대 실행시간을 설정한다.

```ts

const users = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .maxExecutionTime(1000) // milliseconds.
    .getMany()

```

## Partial selection

특정 `Entity` 의 특정항목을 선택하기 원한다면 다음처럼 하면 된다.

```ts

const users = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .select(["user.id", "user.name"])
    .getMany()

```

이제 `id, name` 만 선택되어 표시된다.

## Using subqueries

`TypeORM` 은 매우쉽게 `SubQuery` 를 지원한다.  
다음을 보자.


```ts
const qb = await dataSource.getRepository(Post).createQueryBuilder("post")

const posts = qb
    .where(
        "post.title IN " +
            qb
                .subQuery()
                .select("user.name")
                .from(User, "user")
                .where("user.registered = :registered")
                .getQuery(),
    )
    .setParameter("registered", true)
    .getMany()
```

위는 `qb` 라는 변수에 `QueryBuilder` 를 담고,  
`where` 절에서 `qb` 를사용해 `query` 하고 있는것을 볼 수 있다.

`subQuery` 사용시 위의 `.subQuery` 문구를 넣어주어야 한다.
`Subquery` 는 `FROM`, `WHERE`, `JOIN` 문 안에서 제공된다.

하지만, 위의 표현식을 무언가 조잡하다.
굳이 `qb` 변수를 두번사용하는것 좋은 방법은 아닌듯하다.

그래서 조금더 우아한 방법을 제공한다.

```ts
const posts = await dataSource
    .getRepository(Post)
    .createQueryBuilder("post")
    .where((qb) => {
        const subQuery = qb
            .subQuery()
            .select("user.name")
            .from(User, "user")
            .where("user.registered = :registered")
            .getQuery()
        return "post.title IN " + subQuery
    })
    .setParameter("registered", true)
    .getMany()
```

`.where` 절 안에 익명함수를 사용하여 `qb` 를 받고,  
받은 `qb` 의 결과값을 `subQuery` 에 담아서 원하는 표현식인  
`post.title IN + subQuery` 를 리턴한다.

그럼 리턴된 결과물을 받아서 `where` 절이 실행되는 방식이다.
이는 `Callback` 함수 패턴방식으로 구현된다.

또한 다른 방식이 존재하는데, 이 방식역시 굉장히 직관적이고 좋은 방법이다. 바로 `.getQuery` 를 사용하는 방식인데 다음의 `code` 를 살펴보자.

```ts
const userQb = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .select("user.name")
    .where("user.registered = :registered", { registered: true })

const posts = await dataSource
    .getRepository(Post)
    .createQueryBuilder("post")
    .where("post.title IN (" + userQb.getQuery() + ")")
    .setParameters(userQb.getParameters())
    .getMany()
```

위는 `SubQuery` 를 변수에 담고, 해당 변수를 `where` 절에  
`userQb.getQuery()` 를 사용하여 반환한다.  

이렇게 반환된 `query` 는 `subQuery` 가 되어 작동하게 된다.
앞에서 보았지만 `getQuery`는 순수하게 `SQL` 쿼리문을 반환하므로 전혀 문제 없이 사용된다.

다음은 `FROM` 절 안에 사용되는 `subQuery` 이다.

```ts
const userQb = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .select("user.name", "name")
    .where("user.registered = :registered", { registered: true })

const posts = await dataSource
    .createQueryBuilder()
    .select("user.name", "name")
    .from("(" + userQb.getQuery() + ")", "user")
    .setParameters(userQb.getParameters())
    .getRawMany()
```

굉장하다...
이거 말고 다른 방법도 존재한다.

```ts
const posts = await dataSource
    .createQueryBuilder()
    .select("user.name", "name")
    .from((subQuery) => {
        return subQuery
            .select("user.name", "name")
            .from(User, "user")
            .where("user.registered = :registered", { registered: true })
    }, "user")
    .getRawMany()
```

매우 단순하게 구현된다.
또한 추가적인 `subSelect` 를 추가하고 싶다면 다음과 같이 사용될수 있다.


```ts
const posts = await dataSource
    .createQueryBuilder()
    .select("post.id", "id")
    .addSelect((subQuery) => {
        return subQuery.select("user.name", "name").from(User, "user").limit(1)
    }, "name")
    .from(Post, "post")
    .getRawMany()
```

위를 보면 알겠지만, 새로운 `Data` 를 만들어내므로  
`getRowMany` 를 사용한것을 볼 수 있다.

## Hidden Columns

해당 컬럼이 `select` 되지 않도록 설정가능하다.

```ts

import { Entity, PrimaryGeneratedColumn, Column } from "typeorm"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @Column({ select: false })
    password: string
}
```

이제 `find` 를 사용해 `query` 하면 `password` 프로퍼티를 받을 수 없다. 하지만 다음은 가능하다.

```ts

const users = await dataSource
    .getRepository(User)
    .createQueryBuilder()
    .select("user.id", "id")
    .addSelect("user.password")
    .getMany()
```

이제 `query` 안에 `password` 얻게 된다.

## Querying Deleted rows

`@DeleteDateColumn` 을 사용한다면, 자동적으로 `soft-delete` 된다. 

```ts
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @DeleteDateColumn()
    deletedAt?: Date
}
```

이렇게 삭제된 `column` 은 보통의 `query` 로는 해당 `record` 를 받지 않는다.

 `soft-delete` 된 `row` 를 받기 위해서는 `.withDelete` 메서드를 사용 하면 된다.

```ts
const users = await dataSource
    .getRepository(User)
    .createQueryBuilder()
    .select("user.id", "id")
    .withDeleted()
    .getMany()

```

이제 `soft-delete` 된 `row` 도 함께 얻을수 있다.

## Insert using Query Builder

이전까지는 `Select` 관련 부분을 사용해왔다.  
`QueryBuilder` 는 `insert` 도 가능하므로, `Insert` 위주로 설명이 이어진다.

```ts
await dataSource
    .createQueryBuilder()
    .insert()
    .into(User)
    .values([
        { firstName: "Timber", lastName: "Saw" },
        { firstName: "Phantom", lastName: "Lancer" },
    ])
    .execute()
```

문법이 마치 `SQL` 과 동일하다.
별반 어려운것이 없다.

## Raw SQL support

`SQL` 을 사용하여 `insert` 가 이루어질때, `SQL` 쿼리를 실행하는 경우 함수 스타일을 사용하여 값을 처리한다.

```ts
await dataSource
    .createQueryBuilder()
    .insert()
    .into(User)
    .values({
        firstName: "Timber",
        lastName: () => "CONCAT('S', 'A', 'W')",
    })
    .execute()
```
이 구문은 값을 `escape` 하지 않으므로, 직접 `escape` 해야 한다고 말한다. 위의 `'` 을 사용한 문구를 말하는듯 하다.

## Update values ON CONFLICT

`ON CONFLICT` 를 제공하는데, 처리하기 위해서는 `orUpdate` 를 사용한다.

이는 만약 값이 있을경우 `orUpdate` 구문에 의해서 `update` 됨을 말한다.

```ts
await datasource
    .createQueryBuilder()
    .insert() 
    .into(User) 
    .values({
        firstName: "Timber",
        lastName: "Saw",
        externalId: "abc123",
    })
    .orUpdate(
        ["firstName", "lastName"],
        ["externalId"],
    )
    .execute()
```

## Update using Querty Builder

`QueryBuilder` 를 사용하여 `update` 구문을 만들수 있다.

```ts
await dataSource
    .createQueryBuilder()
    .update(User)
    .set({ firstName: "Timber", lastName: "Saw" })
    .where("id = :id", { id: 1 })
    .execute()
```

## Raw SQL support

원한다면 `SQL` 쿼리를 직접 사용할수 있다. 그러기 위해서는 함수를 사용하여 구현해야 한다.

```ts
await dataSource
    .createQueryBuilder()
    .update(User)
    .set({
        firstName: "Timber",
        lastName: "Saw",
        age: () => "age + 1",
    })
    .where("id = :id", { id: 1 })
    .execute()
```

## Delete

`QueryBuilder` 를 통해 `Delete` 를 구현한다.

```ts

await myDataSource
    .createQueryBuilder('users')
    .delete()
    .from(User)
    .where("id = :id", { id: 1 })
    .execute()

```

## softDelete

이는 `soft-delete` 하는 방법이다.


```ts
await myDataSource
  .createQueryBuilder('users')
  .softDelete()
  .where("id = :id", { id: 1 })
  .execute();
```

## Restore-Soft-Delete

`Soft-delete` 된 `row` 를 다시 되돌릴수 있다.

```ts
await myDataSource
  .createQueryBuilder('users')
  .restore()
  .where("id = :id", { id: 1 })
  .execute();
```

## Working with Relations

`RelationQueryBuilder` 는 `QueryBuilder` 의 특별한 타입이다.  
이는 `relations` 와 함께 동작하는것을 허용한다.

이것을 사용한다면, `Entity` 를 로드할 필요없이 `DB` 에서 `Entity` 를 서버로 바인딩할 수 있거나 관련 `Entity` 를 쉽게 로드할수 있다.

예를 들어서 `Post` `Entity` 를 가지고 있고, `Categories` 라 불리는 `Category` 와 `manyToMany` 의 관계를 가진다고 해보자.  

그리고 새로운 `category` 를 추가한다.

```ts
await dataSource
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post)
    .add(category)
```

이 코드는 다음과 같다.

```ts

const postRepository = dataSource.manager.getRepository(Post)
const post = await postRepository.findOne({
    where: {
        id: 1,
    },
    relations: {
        categories: true,
    },
})
post.categories.push(category)
await postRepository.save(post)

```

이는 최소한의 작업을 수행하기에 더 효율적인 방법이다. 그리고  
`save` 메서드를 호출하지 않고 `database` 에 `entities` 를 `bind` 한다.  

또한 다른 이점으로는 모든 연관된 `entity` 을 `push` 하기 전에 `load` 할 필요가 없어진다.

예를들어서, 만약 `post` 안에 만개의 카테고리를 가진다면, 이 목록에 새로운 `post` 들을 추가하는데 문제가 발생할수 있다.  

왜냐하면 만개의 카테고리 전부 포함된 `post` 를 `load` 해야 하고 `save` 해야하기 때문이다. 

이는 매우 무거운 성능비용을과 생산실적에는 기본적으로 적용불가하다. 


그러나 `RelationQueryBuilder` 는 이러한 문제를 해결해준다.
또한 `Entity` 를 반인딩할때 `Entity ID` 를 대신 사용할수 있으므로 실제로 `Entity` 를 사용할 필요가 없다. 

```ts
await dataSource
  .createQueryBuilder()
  .relation(Post, "categories")
  .of(1)
  .add(3)
```

위는 `id 1` 을 바인딩하여, `id 3` 인 `category` 를 추가한다. 
만약 복잡한 `Primary key` 를 사용한다면 해당 `ID` 를 매핑시킬수 있다.

```ts
await dataSource
    .createQueryBuilder()
    .relation(Post, "categories")
    .of({ firstPostId: 1, secondPostId: 3 })
    .add({ firstCategoryId: 2, secondCategoryId: 4 })
```

또한 `Entity` 를 추가하는 방식 그대로 제거도 가능하다.

```ts
// this code removes a category from a given post
await dataSource
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post) // you can use just post id as well
    .remove(category) // you can use just category id as well
```// this code unsets category of a given post
await dataSource
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post) // you can use just post id as well
    .set(null)
`.add` 및 `.remove` 는 `manyToMany` 및 `OneToMany` 관계안에서 작동한다. `OneToOne` 및 `ManyToOne` 관계는 `set` 을 사용해라.

```ts
// this code sets category of a given post
await dataSource
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post) // you can use just post id as well
    .set(category) // you can use just category id as well
```
만약 관계를 끊고 싶다면, 간단하게 `null` 로 `set` 하면 된다.

```ts

// this code unsets category of a given post
await dataSource
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post) // you can use just post id as well
    .set(null)

```

`RelationQueryBuilder` 는 관계형 `Entity` 를 로드할수도 있다.

예를 들어 `Post` 내부에 `ManyToMany` 관계와 `ManyToOne` 관계가 있다고 가정하면 다음의 코드를 수행할수 있다.

```ts
const post = await dataSource.manager.findOneBy(Post, {
    id: 1,
})

post.categories = await dataSource
    .createQueryBuilder()
    .relation(Post, "categories")
    .of(post) // you can use just post id as well
    .loadMany()

post.author = await dataSource
    .createQueryBuilder()
    .relation(Post, "user")
    .of(post) // you can use just post id as well
    .loadOne()
```

위는 `id 1` 를 가진 `post` 의 `categories` 를 가진 `post` 와 `user` 를 가진 `post` 를 가져온다.

## Caching queries

`QueryBuilder` 의 `method` 를 사용하여 `selected` 결과를 `cache` 할수 있다.

이때 사용하는 `method` 들은 다음과 같다.
`getOne`, `getRawMany`, `getRawOne`, `getCount`

또한, `Repostitory` 및 `EntityManager` 의 `find*`, `count*` 에 의해 `selected` 된 결과역시 `cache` 가능하다.

`dataSource` 안에서 `db` 설정할때, `caceh` 를 `true` 로 해주면 사용할수 있다.

```ts
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: true
}
```

`cache` 를 활성화했다면, 첫번째로 `Database` 스키마를 동기화해주어야 한다.

>동기화는 CLI 를 사용하거나 `migration` 또는 `data source` 옵션상에 `synchronous` 를 사용하라고 한다.

그런다음 `QueryBuilder` 에서 모든 `query` 에 대해 `query cache` 를 활성화 할 수 있다.

```ts

const users = await dataSource
    .createQueryBuilder(User, "user")
    .where("user.isAdmin = :isAdmin", { isAdmin: true })
    .cache(true)
    .getMany()

```

`Repository query` 도 동일하다.

```ts
const users = await dataSource.getRepository(User).find({
    where: { isAdmin: true },
    cache: true,
})
```

이렇게 하면 모든 `admin` 유저들이 `fetch` 를 통해 `query` 를 실행할것이고 그 결과는 `cahche` 된다.

기본적인 `cache` 의 `lifecycle` 은 `1000ms` 이다. 
이는 `QueryBuilder` 가 호출된 후 1초 동안만 캐시가 유효하지 않음을 의미한다.

> 위의 내용은 다음과 같다
> 
>This means the cache will be invalid 1 second after the query builder code is called.
>
> 내가 직역을 해서 그런지 뭔가 말이 앞뒤가 안맞는다.  
> 1초간의 라이프사이클을 가지면, 1초동안만 유효함을 말해야 하는데, 1초동안만 캐시가 유효하지 않음을 의미한다고 해석된다.
>
> 이는 내가 잘못해석한 부분일수도 있으니, 참고하자.


사용자가 3초 이내에 사용자 페이지를 150번 열면, 이 기간동안 3번의 쿼리만 실행된다.

또한 `cache` 된 1초동안에 `insert` 한 사용자는 1초 `cache`  가 끝날때 까지 반환되지 않는다.

`cache` 의 시간은 조절가능하며, `ms` 단위로 조정할수 있다.

```ts

const users = await dataSource
    .createQueryBuilder(User, "user")
    .where("user.isAdmin = :isAdmin", { isAdmin: true })
    .cache(60000) // 1 minute
    .getMany()

```

또는

```ts
const users = await dataSource.getRepository(User).find({
    where: { isAdmin: true },
    cache: 60000,
})

```

또는

```ts
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {
        duration: 30000 // 30 seconds
    }
}
```
이렇게 가능하며,  
`cache id` 도 설정할수 있다.

```ts
const users = await dataSource
    .createQueryBuilder(User, "user")
    .where("user.isAdmin = :isAdmin", { isAdmin: true })
    .cache("users_admins", 25000)
    .getMany()
```

또는

```ts

const users = await dataSource.getRepository(User).find({
    where: { isAdmin: true },
    cache: {
        id: "users_admins",
        milliseconds: 25000,
    },
})

```

`TypeORM` 은 `query-result-chach` 로 불리는 별도의 `table` 을 사용한다.  

그리고 그곳에 모든 `query` 와 결과를 저장한다.

`Table` 이름은 설정가능하므로, `Table` 이름과 다른 속성값을 지정하여 변경가능하다.

```ts

{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {
        type: "database",
        tableName: "configurable-table-query-result-cache"
    }
}

```
만약, `single database table` 에 `cache` 하는것이 효과적이지 않다면, `redis` 나 `ioredis` 타입의 `cache` 로 변경할 수 있다.

```ts
{
    type: "mysql",
    host: "localhost",
    username: "test",
    ...
    cache: {
        type: "redis",
        options: {
            host: "localhost",
            port: 6379
        }
    }
}
```

이부분에 대해서는 `Redis` 관련 `Options` 를 알고 처리해야 하므로 [caching](https://typeorm.io/caching) 부분을 살펴보도록 한다.

추후 `Redis` 관련 부분으로 구성하게 된다면 추가적으로 살펴보고, 해당부분에 대해서 더 작성하도록 하겠다.

## 마무리

이렇게 `QueryBilder` 까지 훑어보았다.
역시나 잠깐사이에 `Docs` 를 본것이다보니,  
많이 보고 사용해보아야 손에 익을듯 싶다.

여기까지가 기본적인 `TypeORM` 사용법들이었다.
이외에 `Transaction` 이라든지, `Loggin` 혹은 `Migration` 관련 부분은 따로 필요시 `Docs` 를 참고하며 볼 예정이다.

이제 겨우 `TypeORM` 관련 `API` 가 어떻게 생겨먹었는지 알았을 뿐이다.

이제 사용하는 일만 남았다.




























