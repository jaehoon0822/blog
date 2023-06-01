---
title: 'mongodb structure 에 대해서'
date: '2023-06-01'
tags: ['mongodb', '몽고DB', 'index']
draft: false
summary: 'index에 대해서'
---

# mongodb index


`Index` 는 간단하게 이야기하면 책의 색인 목록이다.  
우리는 책에서 어떤 특정페이지를 찾을때, 이러한 색인 목록에서  
페이지를 찾고, 그 페이지를 펼쳐서 한번에 원하는 주제를 찾는다.

`mongodb` 및 여러 `DBs` 들 역시 이러한 `Index` 를 제공한다.  
`Index` 를 만들면, 원하는 `Query` 를 더 효율적으로 검색할 수 있도록 만든다.

이때, `Index` 를 만들기 위해서 그 기준으로 삼은 필드의 값들을  
`Index key` 라고 부른다.

사실, `Mongodb` 는 모든 `Document` 마다 고유의 `Index key` 를  
가지는데, 바로 `_id` 이다.

`_id` 는 `Primary key` 로써, 해당 `Document` 의 `Unique key`(고유한 키) 역할을 한다.

이러한 `Primary key` 를 사용하여, 우리는 원하는 `Document` 를  
정확하게 찾아서, 읽고 쓰기 작업을 한다.

이말은, `Mongodb` 의 `Document` 생성시, `_id` 필드는 자동적으로 `Index` 로 생성된다는 것이다.

`Index` 를 이해하기 위해서는 `B-Tree` 구조에 대해서 이해하는것이 좋다.

## B-Tree

`B-Tree` 는 자료구조중 하나이다.  
이 구조는 루트 노드와 자식 노드로 나누어지는데, 각 노드가 마치  
나뭇가지가 갈라져서 이루어져있는것 처럼 생겨서 `Tree` 구조라고 부른다.

이렇게 갈라져서 나누어져 있는 구조에서 하나의 노드당 2개 이상의 자식노드를 가지는것이 `B-Tree` 의 특징이다.  

각 노드는 특정 `data` 의 범위를 가진다.
이때, `특정 Node` 의 갯수에 따라서 `child node` 가 달라지는데,  

만약, `Node` 의 `data`갯수가 `1` 이면, `child node` 는 `2`개가 된다.
만약, `Node` 의 `data`갯수가 `2` 이면, `child node` 는 `3` 개가 된다.

즉, `Node 의 data + 1` 개의 `child node` 를 가진다. 

이때, `child node` 가 생성될때, 가장 왼편의 `child node` 는  
`parent node` 의 `data` 보다 작은 값들이며, 가장 오른쪽의 `child ndoe` 는 `parent node` 의 `data` 보다 큰 값이다.

만약 중간 `child node` 가 있다면, `parent node` 의 `data` 보다 크거나 작은 값이다.

이렇게 각 `child node` 는 `parent node` 보다 작거나 큰값을 가지므로,  
`parent node` 에서 `child node` 순으로 각 `data` 를 찾아서 내려가는 구조를 띄고 있다.

이는 마치 책의 원하는 `page` 를 찾기 위해, 책을 반으로 쪼개고,
원하는 페이지가 없다면, 쪼갠 부분에서 다시 반으로 쪼개면서,  
원하는 페이지가 나올때까지 개속 쪼개서 찾아가는 방식과 비슷하다.

`Mongodb` 는 이러한 방식으로 값을 찾은 뒤 자료를 가리키는 포인터를  이용하여 도큐먼트를 찾아낸다.

구조상 보면 알겠지만, `child node` 의 수가 굉장히 많아지기 시작하거나,

> 책으로 비유하면, `목차` 를 어마어마한 수준으로 많이 만드는것이다. 
>
> `목차` 는 특정 `주제` 에 맞는 `page` 를 알려주지만,  
> 그 `주제` 가 너무 세분화되어 `목차` 를 구성하면, `목차` 의 제대로된 역할을 하지 못한다. 

그렇지않고, `node`의 수가 너무 적으면 그 효율이 떨어진다.
> `page` 수가 적은데, `목차` 를 만들 이유가 없다.

그렇기에, `query`시 적은 횟수로 조회할 수 있도록 정해진 규칙을 만드는것이 중요하다.

`Index` 는 매번 빈번히 만들어지는것이 효율적인 방법은 아니다.  
읽기 작업이 많은 `data` 라면 유용하게 사용가능하겠지만,  
쓰기 작업이 많은 `Document` 라면, `Index` 에 맞지 않다.

그러므로 `Index` 를 생성시, 잘 생각하고 만드는것이 중요하다.
`Index` 는 이러한 여러 사항을 고려하기에, 몇 가지 종류로 만들 수 있다.

## Single-key Index

`Index` 생성시 한가지 종류의 `Index` 값을 갖는것을 `Single-key Index` 라고 부른다.  

`Mongodb` 에서 `default` 로 `Document` 생성시, `_id` 를 `Indexing` 한다고 했는데 여기서 `_id` 를 `Single-key Index` 라고 부른다.

이렇게 `_id` 값을 가진 `key` 를 `index` 로 가지면, `_id` 의 값  
범위에 해당하는 `Document` 를 불러올 수 있다.

특정 `field` 를 `Single-key Index` 로 등록한다고 가정해보자.

```sh
> db.posts.createIndex({ rating: 1 })
```

이`Index` 는 `rating field` 를 오름차 순으로 생성한다.

> 위의 `1` 값은 오름차순, `-1` 값은 내림차순이다.

이때, `B-Tree` 구조로 이해하면 대략적인 그림이 그려진다.
`rating` 의 `Node` 구성시 작은값을 왼쪽으로, 그리고 큰값을 오른쪽으로 구성된 `Tree` 를 만든다.

즉, 작은값에서 큰값으로 구성된 트리가 생성된다는 것이다.
그리고, `query` 하는 `data` 를 이렇게 구성된 트리에서 분기되면서 찾아 나간다.

즉, `Index` 된 `rating` 을 사용하여 정렬 및 값 범위에 해당하는 값들을 `query` 할때 매우 유용하게 사용가능하다

그럼, 만약, `rating` 을 오름차순으로 하고, `title` 을 내림차순으로 정렬한다고 가정해보자. 

이때, `Single-key Index` 특성상 `Index` 처리가 불가능하다.
`rating` 은 당연히 `Index` 를 통해 처리하지만, `title` 은 `index` 로 처리되지 않는다.

`title` 이 `Single-key Index` 로 되어있다고 가정하더라도, `rating` 과 같은 `Index` 로 묶여 있지 않기에 의미가 없다.

생성한 `Index` 사용하여 한가지 `field` 의 `Document` 를 찾고, 다른 `field` 값은 하나씩 도큐먼트를 대조해서 검색하게 될것이다.

이러한 여러 `field` 를 `query` 할때 필요한것이 `Compound Index` 이다.

## Compound Index

이는 `Single-key Index` 와는 다르게, `여러개의 field` 를 사용하여 `Index` 를 생성한 방법이다.

`Compound Index` 설정시 필요한 부분은 각 `field` 의 정렬을 어떻게 할지 정해주어야 한다.

또한, 어떠한 `field` 를 우선할지도 같이 정해야 한다.

```sh
> db.posts.createIndex({ rating: 1, title: -1 })
```

`rating` 은 오름차순으로, `title` 은 내림차순으로 `Index` 를 생성했다.

이때, `정렬` 에 대한 `Index` 탐색과, `값의 범위` 에 대한 `Index` 탐색이 `Single-key Index` 와는 약간 다르다.

### Compound Index 의 정렬에 대해서

`Single-key Index` 는 `Compound Index` 와는 다르게,  
하나의 값만을 사용하여 `query` 한다.

하지만 `Compound Index` 같은 경우 `field` 가 여러개라서  
`정렬` 시 문제가 있을 수 있다.

위의 `Index` 는 `rating`, `title` 순으로 `Index` 를생성했으며,
`rating` 은 오름차순, `title` 은 내림차순으로 되어있다.

이때, 만약 위의 정렬방식대로 원한다면, `query` 시 `field` 순서가 중요하다.

즉, `rating`, `title` 순으로 `query` 해야 각 정렬방식에 맞게 `query` 가능하다.

만약, `rating` 을 오름차순, `title` 을 오름차순으로 한다면, `rating` 은 `index` 를 사용하여 잘 정렬되지만, `title` 은 `index` 를 이용하지 못하고 일일이 순서를 맞춰서 정렬한다.

만약 이 순서에 맞게 하지 않고 `title`, `rating` 순으로 `query` 하면, `정렬` 이 제대로 작동하지 못한다.

이렇게 `정렬` 은 `field` 순서와, 정렬 방향을 명확하게 해야만 잘 잘동한다.

### 값의 범위에 대한 Compound Index

`값의 범위` 에 대한 `Index` 는 다르다.
생각해보자, `값의 범위` 는 `정렬` 할 필요가 전혀 없다.  

`정렬` 은 오름차, 내림차 순이라는 부분이 존재하지만, `값의 범위` 는 이러한 순서가 존재하지 않는다.

앞에서 `Compound Index` 에서 두개의 `field` 인 `rating` 과 `title` 을 `index` 로 생성했다.

`title` 이 `공지` 이고, `rating` 이 `3` 라는 포스트를 조회한다고 가정해보자.

이때 우선순위는 `index` 적용순서와 다르게 `title` 로 한다.
이는 제대로 동작한다.

`title` 에서 `공지` 를 찾고, `rating` 을 찾는다.

이때, `rating` 을 먼저 조회하든 `title` 을 먼저 조회하든 사실 둘다 `index` 를 사용하여 조회하게된다.

하지만, 그렇다고 아무런 생각없이 `field` 우선순위를 지정하는것은 좋지 않다.

`Index` 를 더 효율적으로 검색하기 위해서는, 중복되는 조회가 많은 `field` 를 우선으로 두어야 좋다.

`Compound Index` 는 `field` 가 여러개이므로, 두개의 `field` 를 이용한다.

이때, 중복이 적은 `field` 를 우선에 두어 검색한다면, 나중에 중복이 많은 `field` 를 또다시 검색해서 처리하므로, 이러한 동작이 일어나지 않도록 하기 위해서는 처음부터 `중복이 많은 field` 를 우선순위로 검색하는것이 좋다.

어차피 조건에 부합하는 `query` 에 의해 `중복이 적은 field` 는 `중복이 많은 field` 에 포함관계를 형성하고 있기 때문이다.

간단하게 말하면, 불필요한 검색을 하지 않는다는 것이다.

### Multikey Index

`Mongodb` 는 `Array` 를 지원한다.

이때, `Array` 는 `Object` 와는 다르게, `Property` 가 아니라  
`value(값)` 를 가진다.

`Mongodb` 는 실제로 `Array` 의 값을 `query` 할수 있다.

```sh

> db.posts.find({ tags: '잡담' })

```

위의 `tags` 가 `Array` 라 가정하고, `tags` 내부에 `잡담` 이라는 값을 가진 모든 `Document` 를 검색한다.

이말은 `Array 의 값` 을 `Index` 로 저장할 수 있다는 것이다. 

```sh
> db.posts.createIndex({ "review.title": 1 })

```

위는 `review` 라는 `Array` 에 있는 `Document` 의 `title` 값을 인덱싱한것이다.

이러한 `값` 을 이용한 `Index` 를 `Multikey Index` 라 부른다.

### 그외 

이외에 `Text Index` 및 `Hash Index` 가 존재한다.

실제로 `Text Index` 는 언어의 형태소를 기반으로 해서 어원을 분석하여 단어 검색이 이루어지도록 해주는것 같은데, 한국어는 제대로 지원하지 않는것으로 알고 있다.

사용하려면, `Elastic search` 를 사용하라고 하는것 같다.

`Hash Index` 는 `Hash 함수` 를 이용해서 작은 크기로 변형된 `B-Tree` 를 만든다고 한다.

```sh

# title 에 아무런 Index 가 없다고 가정한다.
> db.posts.createIndex({ title: "hashed" })

```

이 `Index` 는 `Compound Index`,  `Array 를 값으로 가진 field` 에 설정할 수 없다고 한다.

기존의 `값의 범위` 및 `정렬` 에 대한 인덱싱은 사용하지 못하고, 오직 검색을 위한 방식으로 사용한다고 한다.

## Index Properties

인덱스 생성시 여러가지 `index type` 들을 지원한다.

### name

`Index` 의 이름을 지정할 수 있다

```sh

> db.posts.createIndex(
  { title: "hashed" }, 
  { name: "hashed title"}
)

```
### TTL

`TTL(Time To Live) Index` 생성역시 가능하다.  

`TTL Index` 는 기간을 정해놓고 해당 기간이 지나면 자동적으로 삭제되는 `Index` 를 말한다. 

```sh

> db.posts.createIndex(
  { expire: 1 }, 
  { expireAfterSeconds: 60 * 60 * 24}
)

```

주의할 점은, `field` 의 값이 시간값이어야 하며, `key` 를 하나만 갖는 `Single-key Index` 이어야한다.

### Unique Index

`field` 의 값을 고유하게 만들기 위한 `Index` 를 생성한다.

`Mongodb` 는 `Unique` 할수 있도록 각 `field` 를 비교하지 않는다. `Index` 를 통해 `Unique key` 를 생성하도록 만든다.

```sh

> db.users.createIndex(
  { email: 1 }, 
  { unique: true }
)

```

위는 등록된 `email` 을 오름차순으로 설정하며, 같은 `email` 이 없도록 고유한 값으로 만들어준다.

### Sparse Index

흔히, `Javascript` 에서 `undefined` 나 `null` 값이 포함된 `Array` 를 `Sparse Array` 라고 부른다.

`Sparse` 의 뜻을 `희소` 라고 하는데, 다른 뜻으로는 `부족한`, `드문드문난` 으로 해석가능하다.

> `희소배열` 이라고 했을때, 와 닿지 않는 부분이지만, 다른 뜻인 `드문드문난 배열` 이라고 하면 그 뜻이 명확히 이해가 된다.
>
> 한자어인거 같은데, 이러한 용어들이 정말 별로다...

이말은 배열의 요소가 연속적으로 이어지지 않고, 중간이 비어있는(`undefined`) 값을 가진 배열이라고 생각하면 된다.

```ts

const sparseArr = [1, 2, undefined, 3, 4, undefined]

```

`Index` 역시 `field` 값이 `null` 일때, `Index` 에서 제외하게 만드는 기능을 제공한다.

```sh

> db.images.createIndex(
  { title: 1 },
  { sparse: true, unique: true }
)

```

위는 `images` 의 `title` 값을 `unique` 하게 만들고,  
`title` 의 값이 `null` 일때, `index` 생성이 안된다.

### Partial Index

일부 `Document` 에만 `Index` 처리한다.

```sh

> db.users.createIndex(
   { username: 1 },
   { unique: true, partialFilterExpression: { age: { $gte: 21 } } }
)

```

이는 `age` 값이 `21` 보다 크거나 같다면 `unique index`를 생성 하지 않는다.

```sh
> db.users.insertMany( [
   { username: "david", age: 27 },
   { username: "amanda", age: 25 },
   { username: "rajiv", age: 32 }
] )
```

위는 `index` 생성되지 않는다.

```sh
db.users.insertMany( [
   { username: "david", age: 20 },
   { username: "amanda" },
   { username: "rajiv", age: null }
] )

```

이는, `age` 값이 `21` 작으므로 `qniuqe index` 가 생성된다.

`Partial Index` 는 특정 `field value` 를 조회할때만 `index` 를 사용하고자 할때 유용하게 사용가능하다고 한다.

## getIndexes

어떠한 인덱스들이 있는지 확인 가능한 함수이다.

```sh

> db.movies.getIndexes()

```

## dropIndex

`Index` 를 제거한다.
`Index` 의 이름 및 설정을 같이 넣어주어야 한다.

```sh

> db.movies.dropIndex({ title: 1 })

```

## dropIndexes

해당 `collection` 의 모든 `Index` 를 제거한다.

```sh

> db.movies.dropIndexes()

```

`Index` 는 `DB` 공부하면서 그 개념에 대해서 알아야 겠다는 생각에 내용을 정리한 것이다.

현재는 `MongoDB` 관련해서 공부하고 있지만, 추후 `SQL` 에 대한 공부할때 이러한 개념은 좋은 공부가 될것이다.
