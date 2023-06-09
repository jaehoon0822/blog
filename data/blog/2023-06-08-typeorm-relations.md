---
title: 'typeorm Relations 에 대해서'
date: '2023-06-09'
tags: ['typeorm', 'ORM', 'Relations']
draft: false
summary: 'typeorm datasource 에 대해서'
---

# `TypeORM relations` 에 대해서 알아본다.

기본적으로 `DB` 라고 한다면 누구나 알게되는 단어인  `Relationship` 이 있다.  

그만큼 이 관계는 너무나 중요한 사항인데, `TypeORM` 역시 이러한 관계를 제공해준다.

## Relation options

`Relation` 은 몇가지 옵션이 제공된다.

- ***eager: boolean***   
만약 `true` 라면, 이 `Entity` 에서 `QueryBuilder` 또는 `find` 메소드를 사용할때, 항상 관계된 기본 `Entity` 와 함께 `load` 된다고 한다. 

- ***cascade: boolean | ("insert" | "update")[]***   
만약 `true` 라면, 관련된 `Object` 가 `DB` 에서 `updated` 그리고 `inserted` 될것이다.  
`cascade options` 의 배열을 지정할수도 있다.

- ***onDelete: "RESTRIC"|"CASCADE"|"SET NULL"***  
`foreign key` 와 연관된 `Object` 가 삭제될때 동작하도록 지정한다.

- ***ophanedRowAction: "nullify"|"delete"|"soft-delete"|"disable"***  
`delete` 는 `DB` 에서 `children` 을 제거한다.  
`soft-delete` 는 `children` 에 `soft-deleted` 라고 표시한다.  
`nullify` 는 `relation key` 를 제거한다.  
`disable` 은 관계를 그대로 유지한다.  
삭제하기 위해서는 기존의 `repository` 를 사용해야만 한다.

> `ophanedRowAction` 에 대해서는 아직 그렇게 와닿지는 않는다.  
해당부분은 조금더 잘 살펴볼 필요성이 있다.

## Cascades

`Cascade` 는 종속된 `Table` 을 같이 `insert`, `update`, `remove`, `soft-remove`, `recover` 할지 결정한다.

기본적으로 `Cascade` 를 `boolean` 값으로 줄 수 있는데,  
`true` 이면 `full cascade` 되며, `false` 이면 `cascade` 하지 않는다.  

`default` 로 `false` 라고 한다.

```ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany } from "typeorm"
import { Question } from "./Question"

@Entity()
export class Category {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @ManyToMany((type) => Question, (question) => question.categories)
    questions: Question[]
}
```
위의 `Category` 는 `Question` 과 `ManyToMany` 관계이다.

```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToMany,
    JoinTable,
} from "typeorm"
import { Category } from "./Category"

@Entity()
export class Question {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    title: string

    @Column()
    text: string

    @ManyToMany((type) => Category, (category) => category.questions, {
        cascade: true,
    })
    @JoinTable()
    categories: Category[]
}
```
`Question` 에는 `cascade` 가 `true` 로 되어있는것을 볼 수 있다.

```ts
const category1 = new Category()
category1.name = "ORMs"

const category2 = new Category()
category2.name = "Programming"

const question = new Question()
question.title = "How to ask questions?"
question.text = "Where can I ask TypeORM-related questions?"
question.categories = [category1, category2]
await dataSource.manager.save(question)
```

이제 해당 `Entity` 의 `instance` 를 만들고 `save` 하는 로직이다.  
이 로직 구조상, `Question` 의 `categories` 에 새롭게 만들어진 `category1` 과 `category2` 를 배열로 담고, `question` 을 `save` 한다.  

위의 로직을 보면 `category1` 과 `category2` 는 `save` 하는 로직이 없다.

하지만 제대로 둘다 생성된다.  
이유는 `Question` 에서 `question column` 에 `cascade` 를 `true` 로 주었기에, 관계로 종속된 `categories` 역시 같이 `save` 되는것이다.

> `Docs` 상에서는 강력한 힘은 큰 책임이 따른다고 명심하라고 한다.
>
> `Cascade` 는 좋고 쉬운 방법으로 `relations` 와 함께 작업한다.  
> 하지만, `database` 에 원치 않은 객체가 `save` 될때 `bugs` 와 `security issues` 가 발생할 수 있다고 한다.
>
>또한, 덜 명시적인 방법으로 새로운 `object` 를 저장하므로, 주의를 요망한다.

### Cascade options

앞에서 말한것처럼 `Cascade` 는 몇가지 옵션이 존재한다.

```
("insert" | "update" | "remove" | "soft-remove" | "recover")[].

```

이러한 옵션을 사용하여 `Cascade` 에 지정 가능한데, 다음은 이를 사용할 예시이다.

```ts
@Entity(Post)
export class Post {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    title: string

    @Column()
    text: string

    // Full cascades on categories.
    @ManyToMany((type) => PostCategory, {
        cascade: true,
    })
    @JoinTable()
    categories: PostCategory[]

    // Cascade insert here means if there is a new PostDetails instance set
    // on this relation, it will be inserted automatically to the db when you save this Post entity
    @ManyToMany((type) => PostDetails, (details) => details.posts, {
        cascade: ["insert"],
    })
    @JoinTable()
    details: PostDetails[]

    // Cascade update here means if there are changes to an existing PostImage, it
    // will be updated automatically to the db when you save this Post entity
    @ManyToMany((type) => PostImage, (image) => image.posts, {
        cascade: ["update"],
    })
    @JoinTable()
    images: PostImage[]

    // Cascade insert & update here means if there are new PostInformation instances
    // or an update to an existing one, they will be automatically inserted or updated
    // when you save this Post entity
    @ManyToMany((type) => PostInformation, (information) => information.posts, {
        cascade: ["insert", "update"],
    })
    @JoinTable()
    informations: PostInformation[]
}
```

위는 `Cascade` 되는 때를 지정하는것이라고 보면 된다.

## @JoinColumn options

`@JoinColumn` 은 관계의 어느쪽이 외래키가 있는 조인 열을 포함하는지 정의할 뿐 아니라, 조인 열 이름과 참조 열 이름을 지정할 수 있다고 한다.

`@JoinColumn` 을 설정할때, 자동적으로 `DB` 에 `propertyName + referencedColumnName` 컬럼을 생성한다.

```ts
@ManyToOne(type => Category)
//`@ManyToOne` 은 선택적이지만, `@OneToOne` 은 필수로 필요하다고 되어 있다.
@JoinColumn() 
category: Category;
```

이제 여기에 생성될 `name` 을 만들 수 있다.

```ts
@ManyToOne(type => Category)
@JoinColumn({ name: "cat_id" })
category: Category;
```
위의 `Column` 이름은 이제 `cat_id` 이다.

`@JoinColumn` 은 다른 어떤 `Columns` 와 항상 연관되어 있다.
> 이는 `Foreign key` 가 사용된다.

`Default`로 `Foreign key`는 연관된 `entity` 의 `primary column` 을 참조한다.  

만약에, 다른 `Entity` 와 연관된 `Column` 과 연관짓고 싶다면 `referencedColumnName` 옵션을 사용하여 지정할 수 있다.

```ts
@ManyToOne(type => Category)
@JoinColumn({ referencedColumnName: "name" })
category: Category;
```

이제 `Category` 의 `name Column` 을 `Foreign key` 로 갖는다.
이제 다른 `Column` 을 `Foreign key` 로 가질 수 있다는것을 알게 되었다면, 다음처럼 다중 `Column` 을 지정할 수 도 있다.

```ts
@ManyToOne(type => Category)
@JoinColumn([
    { name: "category_id", referencedColumnName: "id" },
    { name: "locale_id", referencedColumnName: "locale_id" }
])
category: Category;
```

## @JoinTable Options

`@JoinTable` 은 `ManyToMany` 관계 그리고 `Junction table` 의 `Join Columns` 을 설명할때 사용된다.

`Junction Table` 은 `Entities` 와 연관된 `Columns` 를 `TypeORM` 에 의해 자동적으로 생성되는 특별한 별도의 `Table` 이다.

이때, `Junction tables` 내부의 `Column` 이름을 바꾼다든지, `@JoinColumn` 을 이용해서 자신의 잠조된 `Colmuns` 의 이름을 바꿀수도 있다.

또한, 생성된 `Jucntion table` 의 이름역시 변경 가능하다.

```ts
@ManyToMany(type => Category)
@JoinTable({
    name: "question_categories", // table name for the junction table of this relation
    joinColumn: {
        name: "question",
        referencedColumnName: "id"
    },
    inverseJoinColumn: {
        name: "category",
        referencedColumnName: "id"
    }
})
categories: Category[];
```
만약 목적한 `table` 이 복잡한 `primary key` 를 가진다면, 반드시 `@JoinTable` 에 `Properties` 의 배열을 전해줘야 한다.

## OneToOne relations

`OneToOne` 관계는 `A` 가 `B` 의 인스턴스를 하나만 포함하는 관계를 말한다. 그리고 `B` 는 `A` 의 인스턴스를 하나만 포함한다.

즉 서로가 서로를 하나만 포함하는 관계이다.

다음의 예제를 보도록 하자..

```ts
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm"

@Entity()
export class Profile {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    gender: string

    @Column()
    photo: string
}
```

```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    OneToOne,
    JoinColumn,
} from "typeorm"
import { Profile } from "./Profile"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @OneToOne(() => Profile)
    @JoinColumn()
    profile: Profile
}
```

위의 예를 보면, `User` 가 `Profile` 을 대상으로 추가된것을 볼 수 있다.

관계의 한쪽에만 설정되어야 하는 `@JoinColumn` 을 사용했는데, `OneToOne` 관계에서는 필수적으로 필요하다.

`@JoinColumn` 을 설정한쪽에서 `relation id` 와 대상 `Entity table` 에 대한 `Foreign key` 를 가진다.

이는 다음의 `Table` 이 만들어진다.

```sh
+-------------+--------------+----------------------------+
|                        profile                          |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| gender      | varchar(255) |                            |
| photo       | varchar(255) |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
| profileId   | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+
```

이와 같이 `@JoinCloumn` 은 반드시 관계의 한쪽에 설정되어야만 한다.

그 한쪽은 `DB table` 안에 `foreign key` 를 반드시 가진다.
이로인한 `relation` 을 어떻게 `save` 하는지 보여주는 예이다.

```ts
const profile = new Profile()
profile.gender = "male"
profile.photo = "me.jpg"
await dataSource.manager.save(profile)

const user = new User()
user.name = "Joe Smith"
user.profile = profile
await dataSource.manager.save(user)
```

위의 예를 보자면, `Profile` 을 만들고, 만든 `Profile` 을  
`user.profile` 에 담아 `save` 하는 로직이다.

앞에서 이야기했지만, 만약 `save` 를 두번하기 귀찮다면,  
`Cascade` 를 `ture` 로 사용하는것도 하나의 방법이다.

이후, `profile` 과 함께 `user` 를 `load` 하고 싶다면, `FindOptions` 의 `relation` 을 지정해야 한다.

```ts
const users = await dataSource.getRepository(User).find({
    relations: {
        profile: true,
    },
})
```

혹은, `QueryBuilder` 를 사용하여 `Join` 하는 방법도 있다.

```ts
const users = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.profile", "profile")
    .getMany()
```

`relation` 에서 `eagar loading` 을 활성화하면, `relations` 를 `find commend` 안에 작성하지 않고, 항상 자동적으로 `loaded` 된다.

만약에 `QueryBuilder` 에서 `eagar relations` 를 `disabled` 로 사용했다면, `leftJoinAndSelect` 로 `relation` 을 `load` 해야 한다.

관계는 단방향(`uni-directional`) 및 양방향(`bi-directional`) 일수 있다.

```sh 
+-------------+--------------+----------------------------+
|                         photo                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| url         | varchar(255) |                            |
| userId      | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
+-------------+--------------+----------------------------+

```
```ts
    id: number

    @Column()
    gender: string

    @Column()const profiles = await dataSource
    .getRepository(Profile)
    .createQueryBuilder("profile")
    .leftJoinAndSelect("profile.user", "user")
    .getMany()
```
```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    OneToOne,
    JoinColumn,
} from "typeorm"
import { Profile } from "./Profile"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @OneToOne(() => Profile, (profile) => profile.user) // specify inverse side as a second parameter
    @JoinColumn()
    profile: Profile
}
```

이를 통해 양방향 관계를 만들었다.
추가적으로, 역관계(`inverse relation`) 은 `@JoinColumn` 을 가지지 않는다, `@JoinCloumn`은 반드시 한쪽의 관계에 하나만 존재해야 한다.

이제 `User` 에서 뿐만아니라 `Profile` 에서도 `Join` 이 가능하다.

```ts
const profiles = await dataSource
    .getRepository(Profile)
    .createQueryBuilder("profile")
    .leftJoinAndSelect("profile.user", "user")
    .getMany()
```

## ManyToOne / OneToMany relations

`ManyToOne / OneToMany` 는 `A` 가 `B` 의 인스턴스를 여러개 포함하는 관계 혹은 그 반대이다.

이러한 예시는 다음과 같다.

```ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm"
import { User } from "./User"

@Entity()
export class Photo {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    url: string

    @ManyToOne(() => User, (user) => user.photos)
    user: User
}
```

위의 `Photo` 는 `OneToMany` 중 `Many` 를 말한다.
`Photo` 는 하나의 `User` 와 관계되어있는것을 말한다.

```ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm"
import { Photo } from "./Photo"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @OneToMany(() => Photo, (photo) => photo.user)
    photos: Photo[]
}
```

위의 `User` 는 `OneToMany` 중 `One` 을 뜻한다.
하나의 `User` 는 여러 `Photo` 와 관계되어 있음을 알려준다.

이를 `Diagram` 으로 나타내면 다음과 같다.

![ManyToOneRelation](/static/images/2023/06/ManyToOneRelation.png)

여기서 중요한점은 `OneToOne` 과는 다르게 `@JoinColumn` 이 전혀  
존재하지 않다는 것이다.

`OneToMany` 는 `ManyToOne` 없이 존재할 필요가 없기 때문이다.

뭐 필요하다면, `ManyToOne` 을 가진 `Table` 에서 `@JoinColumn`을  사용하면 되겠지만, 이미 `Foreign key` 가  
보장되어 잇는 상황에서 굳이 불필요한 사용을 할 필요가 없다.

다시 한번 위의 `Diagram` 을 보면 알겠지만, 구조자체가  
`ManyToOne` 을 가진 `Table` 에서 `OneToMany` 를 가진 테이블의  
`Foreign key` 를 가질수 밖에 없는 구조이다.

그리고 `OneToMany` 테이블은 `Foreign key` 를 가지지 않는다.
너무나 명확한 관계이지 않은가?

이를 통해 나오게되는 `Table` 은 다음과 같다.

```sh
+-------------+--------------+----------------------------+
|                         photo                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| url         | varchar(255) |                            |
| userId      | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
+-------------+--------------+----------------------------+
```

이를 `code` 로 표현하면 다음과 같이 만들어진다.

```ts
const photo1 = new Photo()
photo1.url = "me.jpg"
await dataSource.manager.save(photo1)

const photo2 = new Photo()
photo2.url = "me-and-bears.jpg"
await dataSource.manager.save(photo2)

const user = new User()
user.name = "John"
user.photos = [photo1, photo2]
await dataSource.manager.save(user)
```

위는 배열을 통해 `photos` 에 `photo` 를 넣어준다.
하지만, `photo` 내부에도 `user` 가 존재하므로,  
`photo.user` 에도 `User` 의 인스턴스를 넣어주어서 `ManyToOne` 형태의 테이블을 만들 수 있다.

```ts
const user = new User()
user.name = "Leo"
await dataSource.manager.save(user)

const photo1 = new Photo()
photo1.url = "me.jpg"
photo1.user = user
await dataSource.manager.save(photo1)

const photo2 = new Photo()
photo2.url = "me-and-bears.jpg"
photo2.user = user
await dataSource.manager.save(photo2)

```

`Docs` 에서는 `Cascade`를 사용하면, 단 하나의 `save` 를 통해서  
관계있는 `Record` 가 생성될수 있음을 지속 알려준다.

`Cascades` 를 많이 권장하는 느낌이다
이렇게 연관지어진 `Record` 을 `Find` 하는 것은 다음 코드를 보도록 하자.

```ts
const userRepository = dataSource.getRepository(User)
const users = await userRepository.find({
    relations: {
        photos: true,
    },
})

// or from inverse side

const photoRepository = dataSource.getRepository(Photo)
const photos = await photoRepository.find({
    relations: {
        user: true,
    },
})
```

또는


```ts
const users = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .getMany()

// or from inverse side

const photos = await dataSource
    .getRepository(Photo)
    .createQueryBuilder("photo")
    .leftJoinAndSelect("photo.user", "user")
    .getMany()
```

여기서 다시한번 `eagar loading` 에 대해서 말해준다.
위 같은 경우 `eagar loading` 이 되어있지 않으므로,  
`leftJoinAndSelect` 를 사용하여 `Join` 시켜준것 이라고   
`Docs` 에서 강조해준다.

추가적 `Join` 을 안한다면,  
근데, 정말 괜찮은것인지는 조금더 알아보아야 할것 같다.  

`eagar loading` 이 편하기는 할것 같다..

## ManyToMany relations

`ManyToMany` 는 각 `Table` 이 서로간에 여러개의 관계가 맺어지는 것을 말한다.

간단히 말하면, 

`A` 는 여러개의 `B` 인스턴스를 포함하고, `B` 는 여러개의 `A` 인스턴스를 포함하는 관계이다.

다음을 보도록 하자.

```ts
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm"

@Entity()
export class Category {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string
}
```

```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToMany,
    JoinTable,
} from "typeorm"
import { Category } from "./Category"

@Entity()
export class Question {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    title: string

    @Column()
    text: string

    @ManyToMany(() => Category)
    @JoinTable()
    categories: Category[]
}
```

위를 보면 `@ManyToMany` 를 `Category` 와 관계를 맺고,  
`@JoinTable` 을 통해 `Category` 와 `Question` 이 `Join` 될 `Table` 을 만들어준다.

관계도를 보자.

![ManyToManyRelation](/static/images/2023/06/ManyToManyRelation.png)

아래를 보면 이러한 관계도와 똑같이 `SQL` 테이블이 만들어지는것을 볼수 있다.

```sh
+-------------+--------------+----------------------------+
|                        category                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| name        | varchar(255) |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|                        question                         |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| title       | varchar(255) |                            |
| text        | varchar(255) |                            |
+-------------+--------------+----------------------------+

+-------------+--------------+----------------------------+
|              question_categories_category               |
+-------------+--------------+----------------------------+
| questionId  | int(11)      | PRIMARY KEY FOREIGN KEY    |
| categoryId  | int(11)      | PRIMARY KEY FOREIGN KEY    |
+-------------+--------------+----------------------------+
```

여기서 중요한점은 `@JoinTable` 은 두 관계중 오직 한쪽으로만  
넣어주어야 하며, `ManyToMany` 관계에 꼭 필수적으로 사용되어야 한다.

그래야, `JoinTable` 이 만들어지기 때문이다.

이제 만들어진 `Table` 의 관계를 `save` 해보도록 한다.

```ts
const category1 = new Category()
category1.name = "animals"
await dataSource.manager.save(category1)

const category2 = new Category()
category2.name = "zoo"
await dataSource.manager.save(category2)

const question = new Question()
question.title = "dogs"
question.text = "who let the dogs out?"
question.categories = [category1, category2]
await dataSource.manager.save(question)
```

`Docs` 에서는 이러한 관계속에서 어떻게 `Delete` 하는지 코드를 통해 보여준다.

아래의 코드는 `Cascade` 가 활성화된 상태의 `delete` 라는점을 알고 보기 바란다.

```ts
const question = await dataSource.getRepository(Question).findOne({
    relations: {
        categories: true,
    },
    where: { id: 1 }
})
question.categories = question.categories.filter((category) => {
    return category.id !== categoryToRemove.id
})
await dataSource.manager.save(question)
```

위는 오직 `join table` 안의 `record` 만 삭제한다.
`question` 과 `categoryToRemove record` 는 여전히 존재한다. 

## Soft Deleting a relationshop with cascade

`Cascade` 를 통해 `Soft-Delete` 를 만들 수 있다.
다음을 보자.

```ts
const category1 = new Category()
category1.name = "animals"

const category2 = new Category()
category2.name = "zoo"

const question = new Question()
question.categories = [category1, category2]
const newQuestion = await dataSource.manager.save(question)

await dataSource.manager.softRemove(newQuestion)
```
위의 예시를 보면, `Category1` 과 `Category2` 는 `save` 를  
호출하지 않았지만, `save` 되는것을 볼수 있다.

이는 앞에서 말했지만, `Cascade` 를 활성화시켜서 가능한 일이다.

```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToMany,
    JoinTable,
} from "typeorm"
import { Category } from "./Category"

@Entity()
export class Question {
    @PrimaryGeneratedColumn()
    id: number

    @ManyToMany(() => Category, (category) => category.questions, {
        cascade: true,
    })
    @JoinTable()
    categories: Category[]
}
```

위처럼 활성화 해놓으면, 자동적으로 연관되어 있는 `Table` 전부 `save` 되는 효과를 가진다.

여기서 사용되는 `softRemove` 역시 `casecade` 가 활성화 되어야지, 동작된다는점을 잊지 말자.

## Bi-directionla relations

위는 `Uni-directional` 방식으로 `ManyToMany` 관계를 이루었다.  
하지만, 단방향이 아닌 `Bi-directional`(양방향) 으로도  
구성이 가능한데, `@ManyToMany` 데커레이터를 양쪽 모두에 적용해주면 된다.

```ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany } from "typeorm"
import { Question } from "./Question"

@Entity()
export class Category {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @ManyToMany(() => Question, (question) => question.categories)
    questions: Question[]
}
```
```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToMany,
    JoinTable,
} from "typeorm"
import { Category } from "./Category"

@Entity()
export class Question {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    title: string

    @Column()
    text: string

    @ManyToMany(() => Category, (category) => category.questions)
    @JoinTable()
    categories: Category[]
}
```
여기서 명심해야 할것은 `Bi-directional` 라고 하더라도 `@JoinTable` 은 오직 한쪽의 관계에만 연관을 지어야 한다는 점이다.

이제, 이렇게 양방향으로 관계가 이루어졌으므로,  
`Category` 에서도 `Join` 이 가능하다.

```ts
const categoriesWithQuestions = await dataSource
    .getRepository(Category)
    .createQueryBuilder("category")
    .leftJoinAndSelect("category.questions", "question")
    .getMany()
```

## ManyToMany relations with custom properties

여태껏, `ManyToMany` 관계를 맺을때, 해당 `Table` 의  
참조 `ID` 를 사용해서, 구성된 `Join Table` 을 만들었다.  

하지만, 이렇게 `@JoinTable` 을 통해 `ManyToMany` 를 구성하지 않고, 직접 `Custom JoinTable` 을 만들 수 있다.

이는 다음처럼 이루어진다.

```ts
import { Entity, Column, ManyToOne, PrimaryGeneratedColumn } from "typeorm"
import { Post } from "./post"
import { Category } from "./category"

@Entity()
export class PostToCategory {
    @PrimaryGeneratedColumn()
    public postToCategoryId: number

    @Column()
    public postId: number

    @ManyToOne(() => Post, (post) => post.postToCategories)
    @JoinColumn({name: "postId"})
    public post: Post

    @ManyToOne(() => Category, (category) => category.postToCategories)
    @JoinColumn({name: "categoryId"})
    public category: Category
}
```

이렇게 이루어진 `Table`  을 서로 연결한다. 

```ts
// category.ts
...
@OneToMany(() => PostToCategory, postToCategory => postToCategory.category)
public postToCategories: PostToCategory[];

// post.ts
...
@OneToMany(() => PostToCategory, postToCategory => postToCategory.post)
public postToCategories: PostToCategory[];
```

위를 관계도로 그려보면 다음과 같다.

![CustomManyToMany](/static/images/2023/06/CustomManyToMany.png)

이는 그저 `TypeORM` 의 `@JoinTable` 에 의해 자동적으로,  
만들어지는 `Table` 이 아니라 직접 `JoinTable` 을 만든것으로 볼수 있을듯 하다.

## Eager and Lazy Relations

드디어 앞전에 `Eager` 을 활성화하면, `Join` 할 필요없이  
자동적으로 이루어진다는 부분을 명확히 설명해주는 섹션이 나왔다.

사실 개념적으로 위의 말이 다라고 볼수 있다.

다음을 보자.

```ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany } from "typeorm"
import { Question } from "./Question"

@Entity()
export class Category {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @ManyToMany((type) => Question, (question) => question.categories)
    questions: Question[]
}
```

위의 `Entity` 는 `Qustion` 과 `ManyToMany` 관계를 가진다.

```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToMany,
    JoinTable,
} from "typeorm"
import { Category } from "./Category"

@Entity()
export class Question {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    title: string

    @Column()
    text: string

    @ManyToMany((type) => Category, (category) => category.questions, {
        eager: true,
    })
    @JoinTable()
    categories: Category[]
}
```
그리고, `Question` 역시 `ManyToMany` 관계를 가짐을 나타내고 있다.

이때, 두번째 인자값으로 `eagar: true` 를 주는것을 볼 수 있는데  
이부분이 바로 여태껏 말한 `eagar` 활성화 부분이다.

이렇게 활성화 되면, 앞부분에서 했던 `Join` 구문 없이 바로  
`Table` 이 병합된 상태로 사용가능하다.

```ts
const questionRepository = dataSource.getRepository(Question)

// questions will be loaded with its categories
const questions = await questionRepository.find()
```

위를 보면 이니 `categories` 는 `loaded` 되었다고 말하고 있다.

앞전에는 저 `find` 구문에 `{ relations: { categories: true } }` 를 같이 명시했지만, 그렇게 안해도 자동적으로 `join` 되는것을 볼 수 있다.

> 주의점으로는, 
>
> `Eager` 관계는 오직 `find*` 에서만 사용가능하며,  
`QueryBuilder` 같은 경우에는 `desabled` 되었다면,  
`leftJoinAndSelect` 를 사용하여 관계를 `load` 해야 한다.
>

## Lazy relations

여기서는 `lazy relations` 에 대해서 설명해준다.
`lazy ralations` 는 간단하게, `Promise` 라고 생각하면 된다.

즉, `Async` 하게 작동한다는 것이다.

```ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany } from "typeorm"
import { Question } from "./Question"

@Entity()
export class Category {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @ManyToMany((type) => Question, (question) => question.categories)
    questions: Promise<Question[]>
}
```

```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToMany,
    JoinTable,
} from "typeorm"
import { Category } from "./Category"

@Entity()
export class Question {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    title: string

    @Column()
    text: string

    @ManyToMany((type) => Category, (category) => category.questions)
    @JoinTable()
    categories: Promise<Category[]>
}
```

말그대로, 각 `ManyToMany` 관계에 있는 `field` 는 `Promise` 를 리턴하는 것을 볼수 있다.

보통 내가 생각하는 `Javascript Async` 는 어떠한 작업량이 많아서 나중에 실행될 목적으로 많이 사용한다고 이해하고 있다.

즉, 위의 `Lazy realtions` 의 목적은 `JoinTable` 이 대량의 정보를 가질수 있는 상황에서 나중에 `loading` 될 수 있도록 제공해주는 방식으로 이해된다.

실제로 `Docs` 상에서, `Javascript` 혹은 `NodeJS` 환경이 아닌 다른 언어들 (Java, PHP, etc) 같은 경우에는 `Promise` 사용에 조심하라고 적혀있다.

이 언어들은 `Asynchronous` 하지 않을 수 있으며, 다른 방식으로 달성될수 있으므로, 사용에 조심하라고 권고하고 있다.

## Realtions FAQ

`Docs` 상에 `Relations` 관련된 자주 묻는 문의를 정리해놓은 부분이 있는데 알아보고 가는것이 좋을듯 싶다.

### How to create self referencing relation

자기참조 관계를 구현하는 법에 대해서 묻고 있는데,  
`tree-like structure` 안에 `entities` 를 저장하는 것이 유용하다고 설명한다.

또한, `adjacency list` 패턴이 구현된다면 자기 참조 관계를 사용할 수 있다고 한다.

단순하게 구현하자면, 다음의 예제를 말해준다.
다음은 `Category` 가 자기자신을 중첩하는 `Table` 이다.

```ts

import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToOne,
    OneToMany,
} from "typeorm"

@Entity()
export class Category {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    title: string

    @Column()
    text: string

    @ManyToOne((type) => Category, (category) => category.childCategories)
    parentCategory: Category

    @OneToMany((type) => Category, (category) => category.parentCategory)
    childCategories: Category[]
}

```

이는 다음과 같은 관계를 가진다.

![SelfRefence](/static/images/2023/06/selfRefence.png)

## How to use relation id without joining relation

이 부분은 `loading` 없이, 연관된 `Object`의 `id` 를 원할때 사용하는 패턴인듯 싶다.

실제로, `find` 시 `{ cascade: true, lagar: true }` 가 아닌 비활성화 된 상태로 `query` 한다고 해보자.

그리고 `relations` 옵션 역시 지정하지 않는다.
그러한 상황에서 `query` 하게 된다면, `연관된 테이블` 의 `id` 는 나오지 않는다.

이부분에 대한 상황에서, `연관된 테이블의 id` 를 `relation` 을 통해 `Join` 없이 가져올수 있는 방법(여기서 `Join` 으로 가져오는 방식을 `load` 라고 표현하더라...)이 없는지 묻고 있는것 같다.

다음을 코드를 보자

```ts
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm"

@Entity()
export class Profile {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    gender: string

    @Column()
    photo: string
}
```

위는 `Profile` 이 있다.
그리고 이 `Profile` 과 `OneToOne` 관계를 맺는 `User` 를 작성한다.

```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    OneToOne,
    JoinColumn,
} from "typeorm"
import { Profile } from "./Profile"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @OneToOne((type) => Profile)
    @JoinColumn()
    profile: Profile
}
```

그리고 `Table` 의 `Record` 는 저장되었다고 치고, `find` 한다.

```ts

const user = await dataSource.getRepository(User).findOne({
    where: { id: 1 }
})

res.json(user)

```

이때 결과는 다음과 같다.

```ts
User {
  id: 1,
  name: "Umed"
}
```

`ProfileId`  가 같이 나오지 않는다.
위는 `Join` 하지 않았기에, `ProfileId` 는 같이 `Load` 되어 출력되지 않는다.

이러한 상황상에, `Load` 없이 `ProfileId` 를 얻을 수 있는 방법은 간단히, `ProfileId Column` 을 만드는것이다.

이는 아래와 같다.

```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    OneToOne,
    JoinColumn,
} from "typeorm"
import { Profile } from "./Profile"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    name: string

    @Column({ nullable: true })
    profileId: number

    @OneToOne((type) => Profile)
    @JoinColumn()
    profile: Profile
}
```

이렇게, 관계에 의해 생성된 `Column` 과 정확히 같은 이름을 가진 `@Column property` 를 추가하기만 하면 된다고 설명한다.

그런다음 `find` 해보면 다음과 같이 출력된다.

```ts
User {
  id: 1,
  name: "Umed",
  profileId: 1
}

```

## How to load relations in entities

많은 `Entities` 로 연관된 `Table` 이 존재한다고 해보자.  
그때, 각 `Table` 을 `Join` 해주어야 하는 상황인데 이는 다음처럼 구현 가능하다.

> findOptions
```ts
const users = await dataSource.getRepository(User).find({
    relations: {
        profile: true,
        photos: true,
        videos: true,
    },
})
```

> QueryBuilder.leftJoinAndSelect
```ts
const user = await dataSource
    .getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.profile", "profile")
    .leftJoinAndSelect("user.photos", "photo")
    .leftJoinAndSelect("user.videos", "video")
    .getMany()
```

참고로 필요하다면 `QueryBuild` 사용시 `innerJoinAndSelect` 를 대신 사용할수도 있다.  

## Avoid relation property initializers

`Docs` 에서 `reation property`를 초기화하지 말라고 권고한다.
이는 다음과 같다.

```ts
import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToMany,
    JoinTable,
} from "typeorm"
import { Category } from "./Category"

@Entity()
export class Question {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    title: string

    @Column()
    text: string

    @ManyToMany((type) => Category, (category) => category.questions)
    @JoinTable()
    categories: Category[] = [] // see = [] initialization here
}
```

위를 보면 `categiries` 가 빈배열로 초기화 되고 있다.
이때 발생할 수 있는 문제를 보도록하자.

아래는 초기화하지 않았을때 반환되는 값이다.

```ts
Question {
    id: 1,
    title: "Question about ..."
}
```

아래는 초기화하고 반환되는 값이다.
```ts
Question {
    id: 1,
    title: "Question about ...",
    categories: []
}
```

`TypeORM` 은 `Object` 를 `save` 하면 `question` 에 의해 `bind` 된 `DB` 안의 `categiries` 를 `check` 한다. 그리고 모든 `Categiries` 를 분리한다.

이때, `[]` 로 초기화한다면 `[]` 로 연관된다고 한다. 
또는 이 `[]` 로 부터 내부의 `items` 들이 지워진것으로 간주될것이라고 설명한다.

그러므로 위처럼 `[]` 로 초기화한다면, 이전에 설정한 `categories` 라 전부 제거되어, 문제가 발생한다고 한다.

이러한 동작은 문제가 발생할 수 있으니, `einties` 안에 `array` 로 초기화하지 말것을 권장한다.

`constructor` 를 통한 초기화도 하지말라고 강조한다.

## Avoid foreign key constraint creation

가끔 퍼포먼스의 이유로 `foreign key` 없는 관계를 원하기도 한다.
이럴때 생성할 수 있는 방법은, `createForeignKeyConstraints` 옵션을 `false` 로 하는것이다.

> `default` 로 `true` 라고 한다.

```ts
import { Entity, PrimaryColumn, Column, ManyToOne } from "typeorm"
import { Person } from "./Person"

@Entity()
export class ActionLog {
    @PrimaryColumn()
    id: number

    @Column()
    date: Date

    @Column()
    action: string

    @ManyToOne((type) => Person, {
        createForeignKeyConstraints: false,
    })
    person: Person
}
```

## Avoid circular import errors

`ManyToMany` 같은 경우, 각 `Class` 를 가져와서 사용하는 로직이 많다.

이때, `circular` 에러가 발생할 수 있다고 하는데, 이러한 에러를 방지하기 위해 `Type` 을 사용하여 값을 가져오도록 설명하고 있다.

```ts
import { Entity, PrimaryColumn, Column, ManytoMany } from "typeorm"
import type { Person } from "./Person"

@Entity()
export class ActionLog {
    @PrimaryColumn()
    id: number

    @Column()
    date: Date

    @Column()
    action: string

    @ManyToMany("Person", (person: Person) => person.id)
    person: Person
}
```
```ts
import { Entity, PrimaryColumn, ManytoMany } from "typeorm"
import type { ActionLog } from "./Action"

@Entity()
export class Person {
    @PrimaryColumn()
    id: number

    @ManyToMany("ActionLog", (actionLog: ActionLog) => actionLog.id)
    log: ActionLog
}
```

이렇게 하면 `circular error` 없이 사용가능하다.

## 마무리

지금까지 `relations` 에 관련된 내용을 정리해 보았다
원래 어제 끝났어야 하는데, 오늘까지 이어지는듯하다.

`relations` 관련된 부분은 어떻게 구성되고 어떻게 만들어지는지 알게 되어, `API` 구성시 지속 참고하면서 만들어보아야 할것 같다.

이제 이 다음은 `TypeORM` 을 사용하여 `query` 하는 부분을 살펴보도록 한다.



