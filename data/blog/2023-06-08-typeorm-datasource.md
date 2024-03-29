---
title: 'typeorm datasource 에 대해서'
date: '2023-06-08'
tags: ['typeorm', 'ORM', 'Data Source']
draft: false
summary: 'typeorm datasource 에 대해서'
---

# `typeorm` 에 대해서 알아본다.

`typeorm` 을 본격적으로 알아보기 위해서 공부하는 이유는 단순하다.  
`sequelize` 는 `typescirpt` 로 사용하는데, 불편함이 많았다.

`sequelize` 를 사용하려고 했더니, `typescript` 사용시 추가적인 `typescript-sequelize` 를 설치하고 사용하는것이 좋을듯 싶었다.

그렇게 하지 않고 사용한다면, 사용은 가능하지만 `relationship` 으로 만들어진  
관계상에서, 만들어져야할 `Methods` 를 `class` 생성시 미리 구현하고 짜 맞추는  
과정이 필요했다.

```ts
import {
  Association, DataTypes, HasManyAddAssociationMixin, HasManyCountAssociationsMixin,
  HasManyCreateAssociationMixin, HasManyGetAssociationsMixin, HasManyHasAssociationMixin,
  HasManySetAssociationsMixin, HasManyAddAssociationsMixin, HasManyHasAssociationsMixin,
  HasManyRemoveAssociationMixin, HasManyRemoveAssociationsMixin, Model, ModelDefined, Optional,
  Sequelize, InferAttributes, InferCreationAttributes, CreationOptional, NonAttribute, ForeignKey,
} from 'sequelize';

class User extends Model<InferAttributes<User, { omit: 'projects' }>, InferCreationAttributes<User, { omit: 'projects' }>> {
  // id can be undefined during creation when using `autoIncrement`
  declare id: CreationOptional<number>;
  declare name: string;
  declare preferredName: string | null; // for nullable fields

  // timestamps!
  // createdAt can be undefined during creation
  declare createdAt: CreationOptional<Date>;
  // updatedAt can be undefined during cr Subscribersate>;

  // Since TS cannot determine model association at compile time
  // we have to declare them here purely virtually
  // these will not exist until `Model.init` was called.
  declare getProjects: HasManyGetAssociationsMixin<Project>; // Note the null assertions!
  declare addProject: HasManyAddAssociationMixin<Project, number>;
  declare addProjects: HasManyAddAssociationsMixin<Project, number>;
  declare setProjects: HasManySetAssociationsMixin<Project, number>;
  declare removeProject: HasManyRemoveAssociationMixin<Project, number>;
  declare removeProjects: HasManyRemoveAssociationsMixin<Project, number>;
  declare hasProject: HasManyHasAssociationMixin<Project, number>;
  declare hasProjects: HasManyHasAssociationsMixin<Project, number>;
  declare countProjects: HasManyCountAssociationsMixin;
  declare createProject: HasManyCreateAssociationMixin<Project, 'ownerId'>;

  // You can also pre-declare possible inclusions, these will only be populated if you
  // actively include a relation.
  declare projects?: NonAttribute<Project[]>; // Note this is optional since it's only populated when explicitly requested in code

  // getters that are not attributes should be tagged using NonAttribute
  // to remove them from the model's Attribute Typings.
  get fullName(): NonAttribute<string> {
    return this.name;
  }

  declare static associations: {
    projects: Association<User, Project>;
  };
}

class Project extends Model<
  InferAttributes<Project>,
  InferCreationAttributes<Project>
> {
  // id can be undefined during creation when using `autoIncrement`
  declare id: CreationOptional<number>;

  // foreign keys are automatically added by associations methods (like Project.belongsTo)
  // by branding them using the `ForeignKey` type, `Project.init` will know it does not need to
  // display an error if ownerId is missing.
  declare ownerId: ForeignKey<User['id']>;
  declare name: string;

  // `owner` is an eagerly-loaded association.
  // We tag it as `NonAttribute`
  declare owner?: NonAttribute<User>;

  // createdAt can be undefined during creation
  declare createdAt: CreationOptional<Date>;
  // updatedAt can be undefined during creation
  declare updatedAt: CreationOptional<Date>;
}

class Address extends Model<
  InferAttributes<Address>,
  InferCreationAttributes<Address>
> {
  declare userId: ForeignKey<User['id']>;
  declare address: string;

  // createdAt can be undefined during creation
  declare createdAt: CreationOptional<Date>;
  // updatedAt can be undefined during creation
  declare updatedAt: CreationOptional<Date>;
}

```

위에 보면 `accociation` 상에 이러한 구문이 있다.  
```
  // Since TS cannot determine model association at compile time
  // we have to declare them here purely virtually
  // these will not exist until `Model.init` was called.

```
대략적으로 해석해보자면, 

> `TS` 는 `compile` 때 `model association` 을 가리키지 못한다. 그러므로, 가상으로 선언해야 한다. 이는 `Model.init` 이 호출된다면 존재하지 않을것이다.

이말은 `TS` 에서 `Type` 을 알기 위해 가상으로 존재함을 알려야 한다는것 같다.
그외에도, `TS` 사용을 위해 제공되는 `Type` 들이 어마어마하게 늘어난다..

자세한건 [sequelize Other topics: typescirpt](https://sequelize.org/docs/v6/other-topics/typescript/) 에서 살펴보자.

이러한 불편함으로 인해 탄생한 라이브러리가 [sequelize-typescript](https://www.npmjs.com/package/sequelize-typescript) 인데, 문법이 `Typeorm` 과 비슷하다? 

이럴것이리면, 그냥 `Typeorm` 으로 갈아타도록 하자는 마음을 먹게 되었다.  

나중에 `nestjs` 관련해서 더 공부하고자 하는데, `Typeorm` 을 기본 내장하고 있는것으로 알고 있으니, 금상첨화 일것 같다.

## Datasource

`Typeorm` 에서는 `Datasource` 를 사용하여, `DB` 를 연결한다.

```ts
import { DataSource } from "typeorm"

const AppDataSource = new DataSource({
    type: "mariadb",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
})
```
여기서 사용된는 각 `options` 는 다음과 같다.

- type  
단순하게 `RDBMS` 의 종류를 정한다.

- extra  
사용할 `RDBMS` 의 기본 드라이브를 정하는 옵션이다.

- entities  
`Entities` 또는 `Entities Schemas` 를 가져올 경로를 말한다.  
사용시, `entity classes`, `entity schema classes`, `directory paths` 를 배열에  
넣는 구조로 만들어진다.  
`directory paths` 설정시에는 `Glob` 패턴을 사용하여 지정 가능하다.
```ts 
import Post from 'src/entities/Post'
import Category from 'src/entities/Category'

const AppDataSource = new DataSource({
    type: "mariadb",
    host: "localhost",
    port: 3306,
    username: "test",
    password: "test",
    database: "test",
    /* 이것들 중 하나 */
    entities: [Post, Category] // entities
    entities: ["src/entities/*.js"] // directory paths
    entities: [Post, Category, "src/entities/*.js"] // entities and directory paths
})
```

- subscribers  
각 `Entity Class` 마다 `Custom Logic` 을 가질 수 있다.  
이때, 어떤 `Event` 에 따라 지정된 `Custom Logic` 을 실행할지 정할 수 있다.  
마치 `DOM` 의 `EventListener` 와 비슷한 개념이다.  
  
  어떠한 특정 `Events` 가 발생한다면, 어떠한 행동을 할지 미리 등록해두는  
  것으로 보아도 될것 같다.
  
  이러한 만들어진 `Subscribers` 를 등록하는 `options` 로 생각하면 될듯하다.
  등록 방법은 `entities` 와 유사하다.

- migrations  
`Migrations` 는 할당된 `data source` 로 부터 로드되어 `Migration` 에 사용된다.  
등록 방법은 `entities` 와 유사하다.

- logging  
`logging` 할지 안할지 가리키는 `option` 이다.  
이 `logging` 은 `boolean` 으로 설정도 가능하지만, 원하는 `Logging Types` 를  
지정할수도 있다.
```ts
const AppDataSource = new DataSource({
  ...,
  /* 이 중 하나 선택 */
  logging: true // or false
  logging: ["query", "error", "schema"] // Loggin Types
});
```
- maxQueryExecutionTime  
이 `option` 은 `query` 실행시 허용하는 최대 시간을 정한다.  
`time` 은 `miliseconds` 로 지정한 숫자값이다. 이 시간이 넘어가면 `logger` 가   
이 시간을 기록한다.

- poolSize  
이 `option` 은 `connection pool` 의 활성 `connections` 의 최대 숫자를 설정한다.  

- nameStrategy  
이 `option` 은 `database` 안의 `columns`, `tables` 의 이름을 지정하는데 사용된다고 한다.

- entityPrefix  
`entity` 생성시, 해당 `entity` 이름의 앞에 붙을 `prefix` 를 정한다.

- entitySkipConstructor  
해당 `entity` 를 역직렬화할때, `constructor` 를 `skip` 한다고 한다.  
이부분은 어느때 사용할지 감이 잡히지 않아서 추가적으로 공부해야 겠다.

- dropSchema  
매번 `data source` 가 초기화될때마다, `schema` 를 `drop` 한다.  
이 `option` 은 `develop` 일때 사용해야만 한다.

- synchronous  
만약 `db` 가 매번 `application` 시작할때마다 `schema` 가 자동 생성하는지 여부를 나타낸다.  
이 옵션은 `production mode` 에서는 `data` 를 잃어버릴수 있으므로, 사용되지 않아야 한다.   
이 옵션은 `debug` 치 `develop mode` 일때 유용하게 사용가능하다.  

- migrationRun
매번 `application` 시작할때마다 자동적으로 `migration` 하는지 여부를 나타낸다.

- migrationTableName  
`migrations` 가 실행됨에 의해 그 정보를 포함할 데이터베이스 테이블 이름을 말한다.  
기본값으로는 `migrations` 로 불리는 `table` 로 명명지어진다.

- migrationsTransactionMode  
`migration` 을 위한 `transaction` 을 `controll` 한다.  
기본값으로는 `all` 이며, `all` | `none` | `each` 중 하나를 선택할 수 있다.

- metadataTableName  
`table metadata` 에 대한 정보를 포함할 데이터베이스의 테이블 이름이다.

- cache  
`entity` 결과 캐싱을 활성화한다.  


이것 이외에 각 `DB` 에 따른 추가 `options` 들이 존재하는데,  
이부분은 `Docs` 에서 살펴보도록하자.

내가 사용할 `mariadb` 는 [mysql/mariadb data source options](https://typeorm.io/data-source-options) 에서 필요할때 찾아보도록 하겠다.

## 마무리

요새 `Database` 관련 설계가 잠깐 멘붕으로 다가오다보니, 공부에 대한 필요성이 느껴져 짧은 시간에 `one to one`, `one to many`, `many to many` 에 대한 개념 및 `Modeling` 내용을 다시 보고, `sql` 문 공부 및 `express` 를 통핸 `server` 구현, `docker` 를 공부하느라, 따로 블로그 작성에 소홀했던것 같다.

이렇게 `Docs` 를 보면서 공부하는 김에 `Typeorm` 관련해서 보고 내용 정리하며, 내용을 이해하기 위해 다시 `Blog` 를 작성하도록 한다.
