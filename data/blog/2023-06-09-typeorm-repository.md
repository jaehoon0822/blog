---
title: 'typeorm repository 에 대해서'
date: '2023-06-09'
tags: ['typeorm', 'ORM', 'Repository', 'repository']
draft: false
summary: 'typeorm repository 에 대해서'
---

# `TypeORM Repository` 에 대해서 알아본다.

`TypeORM` 은 `CRUD` 등등.. `DML` 관련 부분을 처리해주는 `Collection` 을 제공한다.

이러한 `Collection` 은 3가지로 나누는데,
`EntityManager`, `Repository`, `QueryBuilderer` 이다.

여기서는 `Repository` 먼저 보고 이후 `EntityManager`, `QueryBuilder` 관련  
`API` 를 살펴보도록 할것이다.

`Repository` 를 사용하기 위해서는 `DataSource` 에서 `manager` 를 불러와 사용하거나, `getRepository` 를 사용하여 가져올 수 있다.

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

> `getRepository` 를 통한 `query`
```ts
import { User } from "./entity/User"

const userRepository = dataSource.getRepository(User)
const user = await userRepository.findOneBy({
    id: 1,
})
user.name = "Umed"
await userRepository.save(user)
```

개인적으로, `getRepository` 로 변수에 저정한후에, 해당 변수를 불러와서 사용하는것이 편할듯 싶다.

`Repository` 는 총 3개의 ` Type` 이 존재한다.

- Repository  
모든 `entity` 에 대한 일반적인 저장소

- TreeRepository  
`Tree-Entities` 로 부터 사용된 저장소의 `extensions` 이다  
`tree structures` 를 위한 특별한 `methods` 를 가진다.

- MongoRepository  
오직 `MongoDB` 를 지원하기 위해 사용되기 위한 특별한 기능들을 가진 저장소이다.

## Find Optopns

모든 `repository` 그리고 `manager` 의 `.find*` 메서드들은 특별한 `options` 를 허용한다.  

이로써 `QueryBuilder` 사용 없이 필요한 `data` 를 `query` 하는데 사용할 수 있다.

### select  

기본 `Object` 의 어떤 속성을 선택해야 하는지를 나타낸다

```ts
userRepository.find({
  select: {
    firstName: true,
    lastName: true,
  }
})
```

위는 다음과 같다.

```sql
SELECT "firstName", "lastName" FROM "user"; 
```
### relations

기본 `entity` 와 함께 `load` 되는데 필요한 관계를 정의한다.  
`sub-relations`(하위관계) 도 `load` 할수 있다.

```ts

userRepository.find({
  relations: {
    profile: true,
    photos: true,
    videos: true,
  }
});

userRepository.find({
  relations: {
    profile: true,
    photos: true,
    videos: {
      videoAttributes: true,
    }
  }
});
```

위는 아래의 `SQL` 문과 같다


```sql

SELECT * FROM "user" as "user"
LEFT JOIN "profile" as "profile"
ON "profile"."id" = "user"."profileId"
LEFT JOIN "photos" as "photos"
ON "photos"."id" = "user"."photosId"
LEFT JOIN "videos" as "videos"
ON "videos"."id" = "user"."vidoesId"

SELECT * FROM "user" as "user"
LEFT JOIN "profile" as "profile"
ON "profile"."id" = "user"."profileId"
LEFT JOIN "photos" as "photos"
ON "photos"."id" = "user"."photosId"
LEFT JOIN "videos" as "videos"
ON "videos"."id" = "user"."videosId"
LEFT JOIN "videos_attributes" as "videos_attributes"
ON "videos_attributes"."id" = "videos"."videos_attributesId"

```
위를 보면 `Camecase` 시 구분자는 `_` 로 구분되는듯 하다.

### where

`entity` 의 쿼리에 대한 간단한 조건이라고 보면된다.

```ts
userRepository.find({
    where: {
        firstName: "Timber",
        lastName: "Saw",
    },
})
```

이는 다음의 `SQL` 과 일치하다

```sql

SELECT * FROM User WHERE firstName = "Timber" AND lastName = "Saw";

```

`query` 시 `Embadded Entity` 같은경우 다음처럼 가능하다.

```ts

userRepository.find({
    relations: {
      project: true,
    },
    where: {
        project: {
          name: "TypeORM",
          initials: "TROM",
        }
    },
})
```

이는 다음과 같다

```sql
SELECT * FROM "user" as "user"
LEFT JOIN "project" as "project"
ON "project"."id" = "user"."projectId"
WHERE "project"."name" = "TypeORM" AND "project"."initials" = "TROM";
```

또한 `where` 문에서 배열로 담아 쿼리하면 `OR` 과 같은 방식으로 쿼리할 수 있다.

```ts
userRepository.find({
    where: [
        { firstName: "Timber", lastName: "Saw" },
        { firstName: "Stan", lastName: "Lee" },
    ],
})
```

이는 다음과 같다

```sql

SELECT * FROM "user" WHERE 
("firstName" = "Timber" AND "lastName" = "Saw") 
OR 
("firstName" = "Stan" AND "lastName" = "Lee");

```

## order

오름차순, 내림차순을 지정한다.

```ts
userRepository.find({
  order: {
    name: "ASC", // 오름차
    id: "DESC", // 내림차
  }
})
```

이는 다음과 같다.

```sql
SELECT * FROM "user"
ORDER BY "name" ASC, "id" DESC
```

### withDeleted

이는 `softDelete` 혹은 `softRemove` 로 삭제된 `entity` 를 포함한다. 사용하려면 `@DeleteDateColumn` 을 포함해야만 한다고 한다.  

기본적으로 일시 삭제된 `Entity` 는 포함하지 않는다

```ts
userRepository.find({
    withDeleted: true,
})
```

> 다음의 옵션들은`find*` 메서드들 즉, `find`, `findBy`, `findAndCount`, `findAndCountBy` 처럼 여러 `Entity` 를 반환할때 사용가능한 옵션들이다.

### skip

`entity` 들을 가져오는 곳 에대한 `offset` 을 지정한다

```ts
userRepository.find({
  skip: 5
})
```

이는 다음과 같다.


```sql

SELECT * FROM "user" OFFSET 5;

```

### take

가져올 `entity` 의 최대갯수를 제한한다.

```ts

userRepository.find({
    take: 10,
})

```

`sql` 문은 다음과 같다


```sql

SELECT * FROM "user" LIMIT 10;

```
보통 `skip` 과 `take` 는 같이 사용한다.

```ts

userRepository.find({
    order: {
        columnName: "ASC",
    },
    skip: 0,
    take: 10,
})

```

이는 `0` 개부터 `10` 개까지 `오름차순` 으로 가져오는 로직이다.

```sql

SELECT * FROM "user"
ORDER BY "columnName" ASC
LIMIT 10
OFFSET 0

```

### cache

결과를 `chache` 한다.
자세한건 [caching](https://typeorm.io/caching#) 에서 살펴보라고 한다.

```ts
userRepository.find({
    cache: true,
})
```

### lock

`query` 에 대한 `mechanism` 에 대한 `locking` 을 활성화한다.
이는 오직 `finxOne` 그리고 `findOneBy` 메서드에 사용가능하다.

다음처럼 사용가능하다.

```ts
{ mode: "optimistic", version: number | Date }
```

혹은

```ts
{
    mode: "pessimistic_read" |
        "pessimistic_write" |
        "dirty_read" |
        /*
            "pessimistic_partial_write" and "pessimistic_write_or_fail" are deprecated and
            will be removed in a future version.

            Use onLocked instead.
         */
        "pessimistic_partial_write" |
        "pessimistic_write_or_fail" |
        "for_no_key_update" |
        "for_key_share",

    tables: string[],
    onLocked: "nowait" | "skip_locked"
}
```

다음의 코드를 보자

```ts
userRepository.findOne({
    where: {
        id: 1,
    },
    lock: { mode: "optimistic", version: 1 },
})
```

`lock` 에 관련된 모드들에 대해서는 아직 모르는것이 많다.
이부분에 대해서 살펴보려면 다음의 [lock mode](https://typeorm.io/select-query-builder#lock-modes) 에서 보도록 하자.

## Advanced options

`TypeORM` 은 많은 `built-in` 연산자들을 제공한다.

### Not

```ts
import { Not } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    title: Not("About #1"),
})
```

이는 다음과 같다

```sql
SELECT * FROM "post" WHERE "title" != 'About #1'
```

### LessThan

```ts

import { LessThan } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    likes: LessThan(10),
})

```

이는 다음과 같다


```sql
SELECT * FROM "posts" WHERE "likes" < 10;
```

### LessThanOrEqual

```ts

import { LessThan } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    likes: LessThanOrEqual(10),
})

```

이는 다음과 같다


```sql
SELECT * FROM "posts" WHERE "likes" <= 10;
```

### MoreThan

```ts
import { MoreThan } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    likes: MoreThan(10),
})
```

```sql
SELECT * FROM "post" WHERE "likes" > 10
```

### MoreThanOrEqual

```ts

import { MoreThanOrEqual } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    likes: MoreThanOrEqual(10),
})

```

```sql
SELECT * FROM "post" WHERE "likes" >= 10
```

### Equal

```ts
import { Equal } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    title: Equal("About #2"),
})
```

```sql
SELECT * FROM "post" WHERE "title" = 'About #2'
```

### like

```ts
import { Like } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    title: Like("%out #%"),
})
```

```sql
SELECT * FROM "post" WHERE "title" LIKE '%out #%'
```

### ILike

```ts

import { ILike } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    title: ILike("%out #%"),
})

```

```sql

SELECT * FROM "post" WHERE "title" ILIKE '%out #%'

```

### Between

```ts

import { Between } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    likes: Between(1, 10),
})

```

```sql

SELECT * FROM "post" WHERE "likes" BETWEEN 1 AND 10

```

### In

```ts
import { In } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    title: In(["About #2", "About #3"]),
})

```

```sql

SELECT * FROM "post" WHERE "title" IN ('About #2','About #3')

```

### Any

> 이는 `Postgres` 문법이라고 한다.
```ts

import { Any } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    title: Any(["About #2", "About #3"]),
})

```

```sql

SELECT * FROM "post" WHERE "title" = ANY(['About #2','About #3'])

```

### IsNull

```ts

import { IsNull } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    title: IsNull(),
})

```

```sql

SELECT * FROM "post" where "title" IS NULL;

```

### ArrayContains

> 사실 이 문법은 처음본다...
>> [Operators](https://mariadb.com/kb/en/operators/) 에도 없어보이는데... 어디 문법이지?

```ts
import { ArrayContains } from "typeorm"

const loadedPosts = await dataSource.getRepository(Post).findBy({
    categories: ArrayContains(["TypeScript"]),
})

```

```sql

SELECT * FROM "post" WHERE "categories" @> '{TypeScript}'

```

이 밑의 구문들은 아직 이해가 기지 않는 구문들이다.
일단 넘긴다.

## Custom Repository

`Repository` 사용시, `Custom Method` 작성이 가능하다.
다음을 보자


```ts

// user.repository.ts
export const UserRepository = dataSource.getRepository(User).extend({
    findByName(firstName: string, lastName: string) {
        return this.createQueryBuilder("user")
            .where("user.firstName = :firstName", { firstName })
            .andWhere("user.lastName = :lastName", { lastName })
            .getMany()
    },
})

// user.controller.ts
export class UserController {
    users() {
        return UserRepository.findByName("Timber", "Saw")
    }
}

```

위를 보면 `extend` 를 사용하여 `method` 를 등록하고, 사용하는것을 볼수 있다.

## 마무리

이렇게, `Repository` 관련 내용을 살펴보았다.  
보면서 `SQL` 관련 공부도 더러 되는것 같아 많은 이해가 쌓인 느낌이다.

하지만, 취향에 따르겠지만, `Repository`, `EntityManager`, `QueryBuilder` 를 제공하는데, 어떠한것이 맞는지는 추후 지속적으로 사용해 보아야 겠다.

다음은 `EntityManager` 에 대해서 살펴보도록 한다.

