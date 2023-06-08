---
title: 'typeorm entities 에 대해서'
date: '2023-06-08'
tags: ['typeorm', 'ORM', 'entities']
draft: false
summary: 'typeorm entities 에 대해서'
---

# `typeorm entities` 에 대해서 알아본다.

`Typeorm` 의 `Docs` 를 읽어보면서, 감탄이 나오는 부분이 바로 이 `Entities` 이다.  

`Entity` 는 `database table` 및 `collection` 에 `mapping` 된 `class` 이다.

`Entities` 를 사용하는데 있어서, `Decorator` 를 사용하는데,  
매우 간편하게 만들어진다.

```ts
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm'

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number

  @Column()
  name: string

  @Column()
  isActive: boolean
}

```

매우 간편하게 만들어진다.
이렇게 만들어진 `table` 은 다음과 같다.

```sh
+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
| isActive    | boolean      |                            |
+-------------+--------------+----------------------------+
```

대단하지 않는가?
매우 명시적으로 `code` 작성이 가능하다.

이전의 `Sequelize` 를 잠깐 공부하고 있었는데, 보다 더 나은 방식으로 느껴진다.

여기서 `Entity` 를 사용하고 싶다면 앞전의 [Data source 에 대해서](https://blog-jaehoon0822.vercel.app/blog/2023-06-08-typeorm-datasource) 의 `entities options` 를 사용하여 등록해주어야 한다.

```ts
import { DataSource } from "typeorm"

const dataSource = new DataSource({
    type: "mariadb",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    entities: ["entity/*.js"], // <-- entity directory 의 경로
})
```

또한 보통 `Prefix` 를 사용하여 지정도 하니, `prefix option` 사용도 고려하는것이 좋을것 같다.  

> 추가적으로, `Docs` 에서는 `constructor arguments` 사용은 선택적이라고 말한다.  
>
> 그 이유로는, `database` 를 로딩할때 `entity class` 의 `instance` 를 생성함으로 생성자 인수를 인식하지 못한다고 말한다.
>
> 이부분은 내부동작에 대한 원리를 아직은 모르므로, `Docs` 를 읽으면서 더 자세히 보아야 겠다.  

## Entity Columns

`DB` 상의 `Columns` 를 뜻한다.  

`TypeORM` 의 각 `Entity class property` 들은 `@Column` 을 사용하여 `table column` 과 `mapping` 하도록 한다.

이러한 `Columns` 들은 여러종류가 있는데 각 종류에 대해서 보도록 한다.

### @PrimaryColumn

`Entity` 는 반드시 하나의 `Primary key` 을 갖는다.
이러한 `Primary key` 를 가진 `Column` 을 설정하는 `Decorator` 이다.

```ts
import { Entity, PrimaryColumn } from "typeorm"

@Entity()
export class User {
    @PrimaryColumn()
    id: number
}

``Object.openSync (node:fs:601:3)
12:03:49.278	    at Object.readFileSync (node:fs:469:35)
12:03:49.278	

> 보통 `modeling` 시 어떠한 `table` 에 종속된 `column` 이 있을때 정규화시켜, `parent table id` 와 `child table id` 를 합쳐서 `primary key` 로 만들기도 한다.

```ts
import { Entity, PrimaryColumn } from "typeorm"

@Entity()
export class User {
    @PrimaryColumn()
    firstName: string

    @PrimaryColumn()
    lastName: string
}
```

단순하게 이렇게, `@PrimaryColumn` 을 2개 만들어주면 된다.

이제 `id` 는 `primary key` 를 가진 `id` 가 된다.

### @PrimaryGeneratedColumn

`Primary key` 생성은 알겠다.  
하지만 보통은 `Primary key` 를 `Int` 형식으로 만들고,  
`AUTO_INCREMENT` 형식으로 자동 증분 되도록 생성하기도 한다.

이러한 기능을 넣어 만든 `decorator` 가 `@PrimaryGeneratedColumn` 이다.

```ts
import { Entity, PrimaryGeneratedColumn } from "typeorm"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number
}

```

이제 `id` 는 자동증분되는 `primary key` 이다.

### @PrimaryGeneratedColumn("uuid") 

`Primary key` 를 `AUTO_INCREMENT` 방식으로 생성하기도 하지만,  이는 단순히 `1` 부터 값이 증가하는 방식으로 구성된다.

이렇게 만들지 않고, 특별한 중복되지 않는 `key` 를 사용하여 만들수 있는데 그것이 `uuid` 이다.

`uuid` 는 중복되지 않는 `key` 이므로, `primary key` 로 작성 가능하다.

`uuid` 로 작성하고 싶다면, `@PrimaryGeneratedColumn` 의 인자로 `uuid` 문자열을 넣어주면 된다.

```ts
import { Entity, PrimaryGeneratedColumn } from "typeorm"

@Entity()
export class User {
    @PrimaryGeneratedColumn("uuid")
    id: string
}
```

### @CreateDateColumn

생성시 자동적으로 `time` 을 추가해주는 `Decorator` 이다. 

```ts
@Entity()
export class User {
    @CreateDateColumn()
    createdDate: Date
}
```

### @CreateDateColumn

`Entity` 를 `Save` 할때마다, 해당 `Column` 의 `time` 을  
업데이트 해주는 `Column` 이다.

```ts
@Entity()
export class User {
    @UpdateDateColumn()
    updatedDate: Date
}
```

### @DeleteDateColumn

`Sofe-delete` 시에 `Delete` 된 `time` 을 자동적으로 설정 해주는 `Column` 이다.

만약, `@DelateDateColumn` 이 설정되어 있다면, `TypeORM` 의 기본 `scope` 는 `non-delete` 가 될것이라고 한다. 

```ts
@Entity()
export class User {
    @DeleteDateColumn()
    deletedDate: Date
}
```

### @VersionColumn

자동적으로 `Versioning` 해주는 `Column` 이다.
여기서는 숫자를 증분해주어서 자동적으로 설정해주는듯 하다.

자동적으로 증분되는 조건은 `save` 가 호출될때, 마다 설정해준다.

```ts
@Entity()
export class User {
    @VersionColumn()
    version: number
}
```

### Spatial Columns 

거의 왠만한 `RDBMS` 들은 `spatial columns` 를 지원한다.
`TypeORM` 역시 이러한 부분을 제공해주는듯 하다.

`MySQL/MariaDB` 는 `geometires` 로 `WKT(Well-Known Text)` 를 제공한다고 한다.

그래서 `Geometry Columns` 는 `String` 타입과 함께 `tagged` 된다.

```ts
import { Entity, PrimaryColumn, Column } from "typeorm"

@Entity()
export class Thing {
    @PrimaryColumn()
    id: number

    @Column("point")
    point: string

    @Column("linestring")
    linestring: string
}

...

const thing = new Thing()
thing.point = "POINT(1 1)"
thing.linestring = "LINESTRING(0 0,1 1,2 2)"
```

반면, `PostgreSQL and CockroachDB ` 같은경우는 `GeoJSON` 포멧으로 변환되므로, `Object` 또는 `Geometry(or subclasses, eg Point)` 방식으로 제공된다고 한다.

```ts
import {
    Entity,
    PrimaryColumn,
    Column,
    Point,
    LineString,
    MultiPoint
} from "typeorm"

@Entity()
export class Thing {
    @PrimaryColumn()
    id: number

    @Column("geometry")
    point: Point

    @Column("geometry")
    linestring: LineString

    @Column("geometry", {
        spatialFeatureType: "MultiPoint",
        srid: 4326,
    })
    multiPointWithSRID: MultiPoint
}

...

const thing = new Thing()
thing.point = {
    type: "Point",
    coordinates: [116.443987, 39.920843],
}
thing.linestring = {
    type: "LineString",
    coordinates: [
        [-87.623177, 41.881832],
        [-90.199402, 38.627003],
        [-82.446732, 38.413651],
        [-87.623177, 41.881832],
    ],
}
thing.multiPointWithSRID = {
    type: "MultiPoint",
    coordinates: [
        [100.0, 0.0],
        [101.0, 1.0],
    ],
}
```

이 부분은 `Geometry` 관련해서, `data` 저장시 다시 살펴보도록 한다.

## Column Types

지금까지 `Column` 의 종류를 보았고,  
`Column` 에 사용될 `Data Type` 에 대해서 살펴본다.

`Colume Types` 를 지정하기 위해서는 `Column Decorator` 에  
인자로 `Type` 을 문자열로 넣어준다.

```ts
@Column("int")
// or
@Column({ type: "int" })
// or
@Column("varchar", { length: 200 })
// or
@Column({ type: "int", width: 200 })
```

이러한 `Type` 이 존재하지만, 이는 `DB` 의 종류에 따라  
그 `Type`  이 달라지기도 한다.

이부분에 대해서는 [Column types for mysql/mariadb](https://typeorm.io/entities#column-types-for-mysql--mariadb) 에서 보도록 한다.

### enum Column Type

`mysql` 과 `postgres` 에서는 `emnum` 타입을 지원한다.
이를 사용하기 위해서는 다음과 같은 문법을 사용한다.

```ts
export enum UserRole {
    ADMIN = "admin",
    EDITOR = "editor",
    GHOST = "ghost",
}

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column({
        type: "enum",
        enum: UserRole,
        default: UserRole.GHOST,
    })
    role: UserRole
}
```

`Column` 은 총 3가지 타입을 지원해주며, 이는 다음과 같다.
- type:   
`Column` 의 타입 지정

- enum:  
`enum` 으로 선언된 변수를 넣어준다.

- default:  
`enum` 으로 넣어준 변수에서 `default` 로 사용될 값을 지정한다.

또는, `enum option` 에 배열을 넣어주어서 사용도 가능하다. 

```ts
export type UserRoleType = "admin" | "editor" | "ghost",

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        type: "enum",
        enum: ["admin", "editor", "ghost"],
        default: "ghost"
    })
    role: UserRoleType
}
```

### set Column Type

`mariadb` 는 `set` 타입역시 지원한다.  

```ts
export enum UserRole {
    ADMIN = "admin",
    EDITOR = "editor",
    GHOST = "ghost",
}

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column({
        type: "set",
        enum: UserRole,
        default: [UserRole.GHOST, UserRole.EDITOR],
    })
    roles: UserRole[]
}
```

이 역시, `enum` 을 `Array` 로 받아서 처리가능하다.

```ts
export type UserRoleType = "admin" | "editor" | "ghost",

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
        type: "set",
        enum: ["admin", "editor", "ghost"],
        default: ["ghost", "editor"]
    })
    roles: UserRoleType[]
}
```

### simple-array Column type

`TypeORM` 은 `simple-array` 라는 특별한 `Column` 을 지원한다.  

이는 `Array` 인듯 하지만, 실제로 저장되는 데이터는 `Comma` 를 가진 문자열로 저장된다.

> 정규형 공부할때 보는, `Comma` 로 이어진 `다가속성` 처럼 생겼다.

이를 이해하기 위해 다음을 보도록 하자.

```ts

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column("simple-array")
    names: string[]
}

```

위의 예시에서 보면 `names` 는 `string[]` 타입이며,  
`Column` 은 `simple-array` 이다.  

만약 이러한 `names` 에 배열을 집어 넣어보자.

```ts
const user = new User()
user.names = ["Alexander", "Alex", "Sasha", "Shurik"]

```

이때, 저장되는 데이터는 다음처럼 저장된다.

```
Alexander,Alex,Sasha,Shurik
```

`names` 컬럼에 들어가는 값이 마치 다가속성처럼 저장되는 것이다.

이러한 부분을 많이 사용하는지는 아직 미지수이다.
하지만, `Programing` 입장에서 보면, 배열값이 한정적으로 사용된다면, 괜찮은 방식일수 있겠다는 생각도 같이 든다.

> 주의사항
>
> 작성하는 값에 `Comma` 가 없어야 제대로 동작한다.

### simple-json Column type

`simple-array` 타입도 보았는데, `json` 이 있다고 해도 별로 놀랍지는 않다.

이는 `DB` 에 `JSON.stringify` 로 저장되게 할 수 있다.
즉, 문자열로 저장된다는 것이다.

생각해보면 꽤나 간단하게 구현가능할것 같기도 하다.
머리를 잘썼다.

```ts
@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column("simple-json")
    profile: { name: string; nickname: string }
}
```
이는 다음처럼 사용된다.

```ts
const user = new User()
user.profile = { name: "John", nickname: "Malkovich" }
```

그리고 저장되는 값은 다음과 같다.

```ts
"{"name":"John","nickname":"Malkovich"}" 
```

이제 `DB` 로 부터 해당 값을 가져온다면, `JSON.parse` 를 통해서  
그 값은 `Object/array/primitive` 값으로 받게 될것이다.

### Columns with generated values

`@Generated` 데커레이터가 존재하는데, 이 데커레이터는 생성된 값으로 해당 열에 자동적으로 값 주입이 가능하다.

```ts
@Entity()
export class User {
    @PrimaryColumn()
    id: number

    @Column()
    @Generated("uuid")
    uuid: string
}
```

이제 `uuid` 컬럼은 자동적으로 생성된 `uuid` 값을 가진다.

이 외에도 `increment`, `identity(postgres)`, `rowid(cockroachDB)` 를 줄수 있는데,  
이러한 유형으로 만들어진 이유는 각 `DB` 마다 지원하는 `generated` 타입이 다르기 때문이다.


## Column options

이렇게 `Column Types` 를 보았으니, `Column options` 를 보도록 한다.

- ***type: ColumnType***  
`Column` 에 사용될 타입을 지정한다.

- ***name: string***  
`DB Table` 에 `Column` 의 이름을 지정한다.

- ***length: number***  
`Column Type` 의 `length` 값을 지정한다.  
만약, `varchar` 타입에 `200` 의 `length` 값을 지정하고 싶다면  
`@Column({type: 'varchar', length: 200})` 으로 지정가능하다. 

- ***width: number***
`Column` 타입의 `width` 를 표시한다.  
이는 `number` 타입에 해당하는 것으로, 허용 가능한 값의 범위를 뜻하는듯 하다.  
 오직 [MySQL integer types](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html) 를 위해 사용된다고 한다.  
 > 이부분은 명확하게 맞는지 모르겠다. 다른 `SQL` 들도 값의 범위가 있을텐데?

- ***onUpdate: string***  
이는 `Constraint` 의 `ON UPDATE` 를 `trigger` 한다.

- ***nullable: boolean***  
`True` 이면 `Null` 을 허용하지만, `False` 이면 `NOT NULL` 로 허용하지 않는다. `default` 로 `nullable: false` 이다.

- ***update: boolean***  
`save` 작업에 의해 `update` 되는지 허용 여부를 결정한다.  
만약 `false` 라면 `object` 에 처음 `insert` 될때만 쓰기 가능하다. (즉, `update` 가 안되다는 말이다...)   
`Default` 값은 `True` 이다. 

- ***insert: boolean***  
`Object` 에 처음 `insert` 될때, `Column` 값이 설정되어 있는지 여부를 알려준다.  
`Default` 값은 `True` 이다.

- ***select: boolean***  
`query` 를 만들때, 기본적으로 이 `Column` 을 숨길지 아닐지 결정한다. `false` 로 설정될떼, 이 `Column data` 가 표준 쿼리와 함께 표시되지 않을 것이다.  
`Default` 로 `select: true` 이다.

- ***default: string***  
`Default` 값을 할당한다.

- ***primary: boolean***  
`Primary` 로 표시한다.  
이는 `@PrimaryColumn` 과 같다.

- ***unique: boolean***  
`Column` 을 `unique`한 값으로 만든다.

- ***comment: string***  
`Column` 의 설명을 달 수 있다.  
모든 데이터베이스에서 지원하는 부분은 아니라고 한다.

- ***precision: number***  
`deciaml` 의 정밀도를 지정한다.  

- ***zerofill: boolean***  
숫자 컬럼에 `ZEROFILL` 속성을 넣는다.  
이는 `MySQL` 에서 사용가능하며, `true` 이면, `UNSIGNED` 속성의 컬럼이 되며 자동적으로 추가된다.

- ***unsigned: boolean***   
`UNSIGNED` 속성이 추가되며 숫자 컬럼이 부호없는 숫자가 된다.

- ***charset: string***   
원하는 `Character set` 지정이 가능하다.  
모든 `DB`타입에서 지원하지는 않는다고 말한다.


- ***collation: string***   
`Column` 의 `collation` 을 지정한다.  

- ***enum: string[] | AnyEnum***  
`enum` 컬럼의 타입을 지정할때 사용된다.  
지정한 `enum` 혹은 값의 `array` 를 지정할 수 있다.

- ***enumName: string***   
`enum` 을 사용한다면 그 이름을 지정한다.

- ***asExpression: string***  
`Generated column expression` 방식이라는데, 해당 문법은 현재 나도 생소해서 추가적으로 `MYSQL` 을 보고 공부한후 이해해야할 것 같다.

- ***generatedType: "VIRTUAL" | "STORED"***  
`Generated column type` 이라는데, 이부분역시 생소하다.

- ***hstoreType: "object"|"string"***  
`Postgres` 에서 사용하는 `Type` 인 `HSTORE` 컬럼을 사용한다고 한다.

- ***array: boolean***  
`Postgres` 및 `CockroachDB` 의 `array` 컬럼타입을 사용한다.  

- ***transformer***  
임의의 `EntityType` 속성을 `DB` 에서 지원하는 `DB Type` 유형으로 변환하는데 사용한다고 한다.  
(여기서 이를 `marshal` 이라고 표현하는데, 이 뜻이 변환을 말하는것 같다.. )  
이부분은 조금 더 살펴보아야 겠다.  
다른 유용한 방식으로 사용하는 것 같기도 하고 애매하다..

## Entity inheritance

`Entity` 는 `Class` 이다. 즉, `상속` 가능하다!!

```ts
export abstract class Content {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    title: string

    @Column()
    description: string
}
@Entity()
export class Photo extends Content {
    @Column()
    size: string
}

@Entity()
export class Question extends Content {
    @Column()
    answersCount: number
}

@Entity()
export class Post extends Content {
    @Column()
    viewCount: number
}
```

## Tree entities

`TypeORM` 은 `Adjacency list` 및 `Closure table` 을 `tree structure` 저장 패턴으로 지원한다.

### Adjacency list

`Adjacency list(인접 목록)` 은 그냥 자체 참조가 가능한 모델이다.  

이 방식은 단순하지만, `Join limiations` 로 인해 `Big tree` 를 한번에 `load` 하지 못하는 것이 단점이라고 설명하고 있다.

```ts
import {
    Entity,
    Column,
    PrimaryGeneratedColumn,
    ManyToOne,
    OneToMany,
} from "typeorm"

@Entity()
export class Category {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @Column()
    description: string

    @ManyToOne((type) => Category, (category) => category.children)
    parent: Category

    @OneToMany((type) => Category, (category) => category.parent)
    children: Category[]
}
```

### Closure table

`Closure table` 은 기존의 자기참조 `relationship` 에 대한  
대책으로써 만들어진 `table` 방법이다.

이는 기존의 자손과 조상이라는 개념을 사용하여, 가장 상단의 `Table` 부터 아래 마지막 자손 `Table` 까지의 관계도를  `Table` 로 만들어, 원하는 `Column` 을 더 효율적으로 검색하기 위한 방법이다.

이러한 방식을 지원하기 위해 `TypeORM` 은 다음처럼 작성하기를 원한다. 


```ts
import {
    Entity,
    Tree,
    Column,
    PrimaryGeneratedColumn,
    TreeChildren,
    TreeParent,
    TreeLevelColumn,
} from "typeorm"

@Entity()
@Tree("closure-table")
export class Category {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @Column()
    description: string

    @TreeChildren()
    children: Category[]

    @TreeParent()
    parent: Category

    @TreeLevelColumn()
    level: number
}
```

여기서 `level` 은 `Closure Table` 의 `Depth` 를 말하는 듯 하다.

이 부분에 대한 개념정도는 이해한 부분으로, 이 `Logic` 이 어떻게 이루어져 있을지 머릿속으로 잘 그려지지는 않는다.

일단, 이러한 부분을 제공하며 사용할 수 있다는 정도로 넘어가고, 더 자세한 부분은 `SQL` 방법론들을 보면서 익히도록 하는것이 좋을 듯 싶다.

## Embedded Entities

`TypeORM` 은 `Class inheritance` 뿐만 아니라,  
`Enitity` 를 내장할 수 있는 방법역시 제공한다.

다음을 보자.

```ts
import { Column } from "typeorm"

export class Name {
    @Column()
    first: string

    @Column()
    last: string
}
```

위는 `Name` 이라는 `Column` 으로 이루어진 `Class` 이다.  
중요한건 `Entity` 데커레이터를 사용하지 않았다는 점이다.

이는 `Embedded Column` 으로 사용하기 위한 초석이다.

```ts
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm"
import { Name } from "./Name"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: string

    @Column(() => Name)
    name: Name

    @Column()
    isActive: boolean
}
```

보았는가? 
위는 `Name` 클래스를 가져온후, `Column` 데커레이터에 함수로 `Name` 을 전달하여, `name` 컴럼을 만들었다.

이때, `TypeORM` 은 `colums` 를 `connect` 할 수 있다고 표현한다.

이제 해당 `Colume` 을 `Table` 로 만들어 보면 다음처럼 이루어져 있다.

```sh
+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| nameFirst   | varchar(255) |                            |
| nameLast    | varchar(255) |                            |
| isActive    | boolean      |                            |
+-------------+--------------+----------------------------+
```

이는 마치 컴포넌트를 재사용하는 방식처럼 자주 사용되는 `Column` 을 만들어 사용하기 용이한듯 싶다.


