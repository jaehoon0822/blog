---
title: '자료구조 Stack 에 대해서'
date: '2023-05-09'
tags: ['자료구조', 'stack', '스택']
draft: false
summary: '자료구조 Stack'
---
# 자료구조

일단 알고리즘을 공부하면서 다음처럼, `stdinput`, `stdOutput` 관련 부분을 알아야 겠다는 생각을 했다
사용은 다음과 같다

```ts
// 첫번째 줄은 수의 개수 N(1 <= N <= 1,000,000) 이며, 2번째 줄 부터는 N개의 숫자가 주어진다
// 오름차순으로 차례대로 출력된다

import * as readline from "node:readline";
import { stdin as input, stdout as output } from "node:process";

class Ascending {
  constructor(private _n: number = 0, private _list: number[] = []) {}

  input() {
    const read = readline.createInterface({ input, output });
    const req = read
      .on("line", (input) => {
        if (this.n === 0) {
          this._n = parseInt(input);
        } else {
          this._list = [...this._list, parseInt(input)];
        }
        if (this._n === this._list.length) {
          read.close();
        }
      })
      .on("close", () => {
        for (const item of this._list.sort((a, b) => a - b)) {
          console.log(item);
        }
      });
  }
}

const asc = new Ascending();
console.log(asc.input());
/*
(코드가 너물 길어서 한줄로 띄어쓰기한것이지 여러줄로 입력한 것이다)
input: 5 5 2 3 4 1 
output: 1 2 3 4 5
*/


```

`Typescript` 를 사용하면서 참 불편한 것이, `stdInput`, `stdOutput` 출력 관련이다
이것은 `Javascript` 문법상 어쩔 수 없는 것이기에, 불편하지만 앞으로 이렇게 입력 및 출력이 이루어질 수 있을것 같다

## Stack

`Stack` 은 `LiFO` 구조로 이루어져 있다
`LIFO`는 `Last In First Out` 의 약자로, 말그대로 마지막에 들어온 요소가 첫번째로 빠져나가는 구조이다.

단, `Stack` 구조를 만들때, `Array` 는 사용하지 않도록 한다
`Array` 는 값을 순차적으로 메모리에 저장하므로, 적은 값이면 별 영향이 없지만 값이 커지면 그만큼의 메모리를 할당하기 위해 처리비용이 많이든다고 한다.

그러므로, 이러한 메모리의 순차적 저장없이, 저장이 가능한 객체 메모리 참조를 통해 구현하고자 한다

이러한 구조를 만들기 위해 다음과 같은 `API` 를 생각해본다

```ts

interface IStack {
  push: (val: any) => any // 값을 추가한다
  pop: () => any // 값을 제거한다. (위에서 부터 제거)
  clear: () => void // 빈 객체로 만든다
  length: () => number // 객체의 lenght 값을 반환한다
  print: () => void // 객체의 요소를 print 한다
  isEmtpy: () => boolean // 객체가 비어있는지 확인한다
}

```

`Stack` 을 간단한 `Typescript` 로 구현해본다.

```ts

interface IStackList {
  [idx: number]: any;
}

class Stack {
  private idx: number = 0;
  private list: IStackList = {};

  isEmtpy() {
    return Object.keys(this.list).length === 0;
  }

  push(val: any) {
    this.list = {
      ...this.list,
      [this.idx]: val,
    };
    const returnVal = this.list[this.idx];
    this.idx += 1;
    return returnVal;
  }

  pop() {
    this.idx -= 1;
    const val = this.list[this.idx];
    delete this.list[this.idx];
    return val;
  }

  print() {
    if (this.isEmtpy()) console.log("not exists");
    for (const key in this.list) {
      if (this.list.hasOwnProperty(key)) console.log(this.list[key]);
    }
  }

  clear() {
    this.list = {};
    this.idx = 0;
  }

  length() {
    return this.idx;
  }
}

const stk = new Stack();

console.log(stk.push(1)); // 1
console.log(stk.push(2)); // 2
console.log(stk.push(3)); // 3

console.log(stk.print()); // 1 2 3

console.log(stk.pop()); // 3

console.log(stk.print()); // 1 2

console.log(stk.length()); // 2

console.log(stk.clear()); // undefined

console.log(stk.isEmtpy()); // true

```

이렇게 `Stack` 을 만들 수 있다
구조는 매우 간단하게 이루어져 있는것을 볼 수 있다

이 `Class` 에서 사용된 `list` 는 매우 간단하다.
마치 `Array` 같이 생겼으면 해서, `key` 를 `number` 값으로 주었으며(실상 `Object` 의 `key` 는 자동적으로 `string` 으로 변환된다.),
`Array` 처럼 자동적으로 `key index` 가 증가하지 않으므로, 증가값을 주기위해 `idx` 라는 `field` 를 주어 사용한다

이렇게 하여 각 메소드를 구현하여 `Stack` 구조를 갖도록 만든 로직이다.
`Stack` 까지는 워낙에 간단한 구조를 가지고 있어서, 배열만 다룰줄 안다면 쉽게 이해하고 구현가능하다.

다음은 `Stack` 의 반대인 `Queue` 를 구현하고자 한다.
