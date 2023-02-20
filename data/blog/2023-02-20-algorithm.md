---
title: 프로그래머스 알고리즘 실패율
date: '2023-02-20'
tags: ['algorithm', '프로그래머스']
draft: false
summary: '실패율을 구하라!!'
---

# 실패율

알고리즘의 명확한 내용은 [실패율 문제](https://school.programmers.co.kr/learn/courses/30/lessons/42889) 에서 확인해 보도록 하자

> 실패율 알고리즘을 푸는데 많이 막혔던 부분이 생겼다.
이러한 부분을 공부하기 위해 해당 알고리즘의 풀이를 분석해 보도록 한다.

## 문제점

풀이했던 풀이를 본다.

```js

// 풀이 도중 다 풀지 못한 코드 

function solution(N, stage) {
  let len = stage.length
  const sortedStage = stage.sort((a, b) => a - b)
  const obj = {}
  const result = {}
  for (let i = 0; i < sortedStage.length; i+=1) {
    if (obj[sortedStage[i]] == undefined) obj[sortedStage[i]] = 1
    else obj[sortedStage[i]] = obj[sortedStage[i]] += 1
  }
  let entries = Object.entries(obj)
  for (let i = 0; i < entries.length; i+=1) {
    result[entries[i][0]] = entries[i][1] / len
    len -= entries[i][1]
  }

  entries = Object.entries(obj)
  for (let i = 0; i < N; i+=1) {
    result[entries] 
  } 

  return result
  // { '1': 0.125, '2': 0.42857142857142855, '3': 0.5, '4': 0.5, '6': 1 }
}

```

위의 코드를 보면 실수했던 부분이, `N` 값에 맞추면 되는것을 `stage` 의 각 `users` 에 맞추어 생각해서 값을 뽑아냈다.

이렇게 뽑아놓고 보니, `6` 보다 작은 값일때 를 조건으로 하여 각 값들을 정렬해야 했으며, 추가적으로 `5` 라는 값을 어떻게 추가할까 고민하고 있었다.


### 알고리즘을 풀면서 느낀점 

1. 실패율을 구하는데 있어 효율적으로 각 상태값을 뽑아내는데 어려웠다.
2. 나중에는 억지로라도 각 `stage` 마다 `실패율` 을 뽑아내는데 성공은 했지만, 그 이후 `sort` 를 사용하여 정렬한후, `stage` 의 `key` 값을 뽑아내는데 어려움을 느꼈다.
3. 처음 주어진 `stage` 의 `users` 의 값을 뽑아내기위해 고민하던 차에, `etnreis` 를 사용하여 뽑아는 내었지만, 마음에 들지 않았으며, 더 좋은 방법이 있을것이라는 생각이 들었다.


## For 문을 이용한 풀이

```js

function solution(N, stages) {
  // 모든 스테이지를 오름차순 형태로 정렬
  stages.sort( (a, b) => a > b ? 1 : -1)
  const infos = []
  for( let i = 1; i <= N; i+=1) {
    infos.push({
      stage: i, // 현재 스테이지의 번호
      users: 0, // 스테이지를 플레이 하고 있는 유저의수
      fail: 0 // 현재 스테이지의 실패율
    })
  }

  let allUsers = stages.length
  for (let i = 0; i < stages.length; i+=1) {
    if (infos[stages[i] - 1]) {
      infos[stages[i] - 1].users++;

      // 현재 스테이지의 번호와 다른 스테이지의 번호가 일치하지 않는 경우
      // === 현재 스테이지의 번호의 합산이 끝나는 순간
      if ( stages[i] !== stages[i + 1]) {
        const fail = infos[stages[i] - 1].users / allUsers;
        allUsers -= infos[stages[i] - 1].users
        infos[stages[i] - 1].fail = fail
      }
    }
  }
  return infos.sort((a, b) => a.fail > b.fail ? -1 : 1).map(el => el.stage)
}

```

> `infos` 를 이용하여 각 상태값을 저장하는 배열을 만드는것이 핵심이다.

각 `infos` 에 상태값을 가진 객체를 추가해준다.
`infos` 에 들어가는 객체는 다음과 같은 상태값을 가진다

```js
const infos = []

for( let i = 1; i <= N; i+=1) {
  infos.push({
    stage: i, // 현재 스테이지의 번호
    users: 0, // 스테이지를 플레이 하고 있는 유저의수
    fail: 0 // 현재 스테이지의 실패율
  })
}
```

각 `stage` 값을 얻기 위해 `stage`에  `i` 값을 각각 넣어준다.
이때 `i` 값은 `1` 이며 `N` 보다 작거나 같아야 맞는 `stage` 값을 가진다.

만약 `N` 이 `5` 라면 각 `stage` 는 `1 ~ 5` 까지의 `stage` 를 갖는다.

이렇게 만들어진 `infos` 의 객체를 순환해서 `users` 와 `fail` 의 값을 주어야한다.

`users` 는 현재 `stage` 를 실행하고 있는 `user` 의 수이다.
`stages` 에 있는 각 `user` 의 수를 넣기 위해 다음처럼 `for` 문을 사용한다.

만약 `infos[stages[i - 1]]` 이 존재한다면, `infos[stages[i] - 1].users` 에 `+1` 연산을 해준다.

`infos` 는 `stage[i]` 보다 `-1` 한 값과 같다.
`infos` 는 배열로 `0` 부터 시작하기 때문이다.

`infos[stages[i - 1]]` 이 존재한다면, `i` 값이 `0` 보다 큰값이므로, 이후 `stages[i]` 값과 `stages[i + 1]` 값이 같은지 같지 않은지 확인가능하다

만약 `일치하지 않는다면` 해당 `stage` 의 `users` 는 더이상 존재하지 않으므로,
`stage 에 도달했으나 클리어 못한 플레이어 수 / 스테이지에 도달한 플레이어 수` 로 계산해 준다.

이때 `N` 값은 `해당 stage 를 도달했으나 클리어 못한 플레이어 수` 를 빼주어야 한다.

그래야 `남은 플레이어` 의 수에 맞추어 `실패율` 계산이 가능하다.

이미 도달하지 못한 플레이어 수역시 포함한다면 `실패율` 에 맞는 값이 나오지 않을것이다.

> 처음 명확한 내용인지가 되지 않았을때, 했던 실수이다.


```js
  const allUsers = stages.length

  for (let i = 0; i < stages.length; i+=1) {
    if (infos[stages[i - 1]]) {
      infos[stages[i] - 1].users++;

      // 현재 스테이지의 번호와 다른 스테이지의 번호가 일치하지 않는 경우
      // === 현재 스테이지의 번호의 합산이 끝나는 순간
      if ( stages[i] !== stages[i + 1]) {
        const fail = infos[stages[i] - 1].users / allUsers;
        allUsers -= infos[stages[i] - 1].users
        infos[stages[i] - 1].fail = fail
      }
    }
  }
```

이렇게 `stage 에 도달했으나 클리어 못한 플레이어 수 / 스테이지에 도달한 플레이어 수` 로 계산된 값을 `infos` 의 `fail` 에 값을 넣어준다.

그리고 계속 같은 방식을 순회한다.


이렇게 순환되었다면 `infos` 는 `stage, users, fail` 에 대한 적절한 값을 가지고 있을 것이다.

이제 `fail` 의 값을 `내림차` 순으로 정렬한후, 그 정렬에 맞추어 `infos` 의 `stage` 를 배열로 순서대로 반환한다. 


```js
  return infos.sort((a, b) => { a.fail > b.fail ? -1 : 1}).map(el => el.stage)
```

### 미처 생각못한 것들

1. 각 상태값을 객체로 저장한 `info` 배열
2. `sort` 사용하여 `stages` 를 정렬한후, 같은 값을 연속적으로 두는것   
3. `for` 문 순환시 `info` 의 첫번재 값 및 두번째 값을 같이 비교 값이 같으면 `users` 에 `+1`을, 아니면 더이상 연속되는 값이 없으니 `fail` 값 계산하여 할당 
4. 완성된 `info` 에서 `sort` 정렬시 `객체의 프로퍼티를 비교` 하여 정렬 


## Method 를 이용한 풀이

```js

function solution(N, stages) {
  let allUsers = stages.length
  stages.sort((a, b) => a > b ? 1 : -1)
  return Array.from( new Array(N), ( _, i ) => i + 1 )
    .map( stage => {
      const result = stages.slice(
        stages.indexOf( stage ),
        stages.lastIndexOf( stage ) + 1
      )
      const fail = result.length / allUsers
      allUsers -= result.length
      return {stage, fail}
    })
    .sort((a, b) => a.fail > b.fail ? -1 : 1)
    .map( el => el.stage)
}

```

`Array.from` 을 이용한다면 `new Array(N).fill(1)` 을 사용하지 않더라도,
쉽게 원하는 갯수의 원소를 가진 배열을 만들 수 있다.

만들어진 배열에서 `stage` 값을 가져와 `stages` 를 `slice` 로 잘라준다.
이때 중요한것이 `stages` 의 `시작값` 과 `끝값` 을 가져와야 한다.

이때 유용하게 사용가능한 `method` 는 `indexOf` 와 `lastIndexOf` 이다.
`slice` 는 `끝값` 의 바로 앞의 원소를 가져오므로 `lastIndexOf(elem) + 1` 한 값을 사용해야 원하는 값을 가져온다.

이렇게 자르게된다면, 해당 `stage` 를 가진 `users` 를 `length` 값으로 가져올 수 있다.

```js
Array.from( new Array(N), ( _, i ) => i + 1 )
  .map( stage => {
    const result = stages.slice(
      stages.indexOf( stage ),
      stages.lastIndexOf( stage ) + 1
    )
    const fail = result.length / allUsers
```

가져온 `length` 값에 `allUsers` 를 나누어주면, `stage 에 도달했으나 클리어 못한 플레이어 수 / 스테이지에 도달한 플레이어 수` 가 된다.

이렇게 계산된 결과를 `fail` 에 저장해준후, `allUsers` 에서 해당 `length` 값 만큼 빼주면, 다음 `stage` 의 `실패율` 을 계산할 수 있다.

이렇게 만들어진 `fail` 과 `stage` 를 객체로 반환하면, `map` 이 순환되며, 해당 객체를 가진 배열을 반환한다.

```js
반환된 배열
    .sort((a, b) => a.fail > b.fail ? -1 : 1)
    .map( el => el.stage)
```

반환된 배열을 사용하여, `fail` 값으로 오름차순으로 정렬한후, `map` 을 사용하여 해당 `stage` 를 배열로 반환하면 알고리즘이 풀어진다. 


### 미처 생각못한 것들

1. `slice` 를 사용할때 `indexOf` 및 `lastIndexOf` 를 사용하여 끝값으로 자르는것

## 마무리

배열을 사용하면서 `Method` 에 대해서는 알고 있지만, 이를 활용하는데 부족함이 많이 느껴진다. 

문법만 아는것이 아니라 이 `Method` 를 적재적소에 사용할 부분을 알고 작성할 수 있어야 한다.

`Object` 를 어떻게 활용하고, 생태값을 어떻게 활용할지 역시 생각하고 고민하면서 작성하며 생각보다 문제들이 쉽게 풀릴수 도 있을것 같다.

`Algorithm` 을 지속풀며, `원하는 로직` 을 만드는데 필요한 `사고력` 과 `활용법` 을 지속적으로 익히도록 해야 한다.