---
title: '비밀지도[1차]'
date: '2023-02-24'
tags: ['algorithm', '프로그래머스']
draft: false
summary: '프로그래머스 비밀지도[1차]'
---

# 비밀지도

> 비밀지도 [문제설명](https://school.programmers.co.kr/learn/courses/30/lessons/17681) 는 프로그래머스에서 확인 가능하다.

문제의 조건은 다음과 같다.

---

1. 지도는 한 변의 길이가 n인 정사각형 배열 형태로, 각 칸은 "공백"(" ") 또는 "벽"("#") 두 종류로 이루어져 있다.

2. 전체 지도는 두 장의 지도를 겹쳐서 얻을 수 있다. 각각 "지도 1"과 "지도 2"라고 하자. 지도 1 또는 지도 2 중 어느 하나라도 벽인 부분은 전체 지도에서도 벽이다. 지도 1과 지도 2에서 모두 공백인 부분은 전체 지도에서도 공백이다.

3. "지도 1"과 "지도 2"는 각각 정수 배열로 암호화되어 있다.

4. 암호화된 배열은 지도의 각 가로줄에서 벽 부분을 1, 공백 부분을 0으로 부호화했을 때 얻어지는 이진수에 해당하는 값의 배열이다.

---

> 나의 문제풀이

```js
function solution(n, arr1, arr2) {
  return arr1.map((row, idx) => {
    const row1 = row.toString(2).padStart(n, 0).split('')
    const row2 = arr2[idx].toString(2).padStart(n, 0).split('')
    // const baseRow = row1.length > row2.length ? row1.split("") : row2.split("")
    // 실수로 포함한 필요없는 코드
    const resultRow = row1.map((col, idx) => {
      if (row1[idx] === '1' || row2[idx] === '1') {
        return '#'
      }
      return ' '
    })
    return resultRow.join('')
  })
}
```

위 문제를 어떻게 풀었는지 확인해보자.

첫번째로 `arr1` 과 `arr2` 는 같은 `index` 를 갖는다고 가정했다.
조건으로 보면 `n` 의 정사각형 배열이기에, 각 값이 같은 `index` 를 갖는다고 가정했다.
그럼 `arr1` 과 `arr2` 의 `index` 를 서로 비교할 생각으로 `map`을 사용하여 각 원소를 순회한다.

```js
function solution(n, arr1, arr2) {
  return arr1.map((row, idx) => {})
}
```

`toString(2)` 를 사용하여 해당 원소를 `이진수`로 변경한다.
하지만 전달되는 숫자값이 항상 `n` 값의 길이를 가진 `이진수`가 아니었다.
그러므로 `arr1` 의 원소와 `arr2` 원소를 `n` 의 길이로 맞추기 위해 다음과 같은 로직을 만든다

이때, `n` 값에 맞는 값을 만들기 위해 `pasStart` 메서드를 사용하여 `n` 의 길이를 갖도록 한다.

```js
function solution(n, arr1, arr2) {
  return arr1.map((row, idx) => {
    const row1 = row.toString(2).padStart(n, 0)
    const row2 = arr2[idx].toString(2).padStart(n, 0)
  })
}
```

이제, 원하는 `이진수`의 값을 가지게 되었다.
그럼 각 `이진수`의 각 문자위치를 각각 비교하기위해 `이진수 문자열` 을 배열로 순회한다.
순회시 `row1` 과 `row2` 의 길이는 `n` 이므로, 각 비교 `index` 는 같다.

그러므로 두 배열중 하나를 순회한후, 해당 `index` 로 각값을 참조할 수 있다.
이것을 사용하여 각 값을 비교할 생각이다.

그렇기에 둘중 하나인 `row1` 을 `map` 으로 순회하며, `index` 를 같이 가져온다.

```js
function solution(n, arr1, arr2) {
  return arr1.map((row, idx) => {
    const row1 = row.toString(2).padStart(n, 0).split('')
    const row2 = arr2[idx].toString(2).padStart(n, 0).split('')
    const resultRow = row1.map((col, idx) => {})
  })
}
```

`이진수의 각문자열` 은 문제 조건에 따라 각 비교값이 `1` 이면 `#`, `0` 이면 `" " ` 이다.<br/>
이에 따라 조건을 정리해본다.

`1` 과 `0` 을 비교하면 값은 `#` 이 된다.<br/>
`1` 과 `1` 을 비교하면 값은 `#` 이 된다.<br/>
`0` 과 `1` 을 비교하면 값은 `#` 이 된다.<br/>
`0` 과 `0` 을 비교하면 값은 `" "` 이 된다.<br/>

여기서 정확한 조건이 성립된다.
두 문자열 비교시 둘중 하나만 `1` 이라면, 무조건 `#` 이 된다.

이 방법을 사용한다.
이러한 방법을 사용하기 위해 다음처럼 비교문을 사용한다.

```js
function solution(n, arr1, arr2) {
  return arr1.map((row, idx) => {
    const row1 = row.toString(2).padStart(n, 0).split('')
    const row2 = arr2[idx].toString(2).padStart(n, 0).split('')
    const resultRow = row1.map((col, idx) => {
      if (row1[idx] === '1' || row2[idx] === '1') {
        return '#'
      }
      return ' '
    })
    return resultRow.join('')
  })
}
```

여기서 사용된 비교 조건식은 `row1` 또는 `row2` 의 원소가 `1` 이면 무조건 `#` 을 반환한다.
그렇지 않으면 둘다 `0` 이기에 `" "` 을 반환한다.

이렇게 하면 해당 조건에 맞는 `#` 또는 `" "` 을 가진 배열을 `return` 하여 `resultRow` 에 넣어준다

이렇게 반환된 결과값은 배열이다.
조건에 맞는 문자열의 나열로 변경하기 위해 `resultRow` 에 `join("")` 을 사용하여 문자열의 나열로 변경한다.

이렇게 각각의 `row` 를 순환한 결과값을 반환하면, 조건에 맞는 배열을 반환하게 된다.

## FOR 문을 이용한 다른 문제 풀이

```js
function solution(n, arr1, arr2) {
  const answer = []

  for (let i = 0; i < arr1.length; i += 1) {
    answer[i] = ''
    const map1 = arr1[i].tostring(2).padstart(n, 0)
    const map2 = arr2[i].tostring(2).padstart(n, 0)

    for (let j = 0; j < map1.length; j += 1) {
      answer[i] += map1[j] === '1' || map2[j] === '1' ? '#' : ' '
    }
  }
  return answer
}
```

> 이 문제를 보면서 `삼항연산자` 를 사용하여, 각 값을 비교한후 원하는 값을 반환한다.
> `if` 문을 쓸때, 2개의 값을 비교한다고 하면, `삼항연산자` 를 쓰는것이 더 간단해 보인다.

## Method 를 이용한 다른 문제 풀이

```js

function solution(n, arr1, arr2) {
  return arr1.map(map1, i) => {
    map1 = map1.toString(2).padStart(n, '0')
    const map2 = arr2[i].toString(2).padStart(n, '0')
    const map1.split("").reduce((acc, cur, j) => {
      return acc + ( cur === '1' || map2[j] === '1' ? '#' : " ")
    }, "");
  }
}

```
