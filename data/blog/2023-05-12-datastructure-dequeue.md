---
title: '자료구조 dequeue 에 대해서'
date: '2023-05-11'
tags: ['dequeue', '자료구조', '덱']
draft: false
summary: '자료구조 dequeue'
---

# dequeue 를 구현해보자

`dequeue` 는 마치 `stack` 과 `queue` 두개의 자료구조를 합친것과 비슷한 구조이다
이 말은 앞, 뒤로 추가 가능한 자료구조라는 말이다

이러한 처리를 위해, 다음의 인터페이스를 만들어본다.

```ts
interface IDequeue {
  pushBack(val: any): any
  popBack(): any
  pushFront(val: any): any
  popFront(): any
  clear(): void
  isEmpty(): boolean
  toString(): string
}

interface IList {
  [idx: number]: any
}
```

그리고, 이러한 인터페이스를 기반으로 하여, `dequeue` 를 구성해서 만들어본다

```ts
class Dequeue implements IDequeue {
  private idx: number = 0
  private lowestCount: number = 0
  private list: IList = {}

  pushBack(val: any): any {
    if (this.lowestCount > 0 && this.lowestCount === this.idx) {
      this.lowestCount -= 1
    }
    this.list[this.idx] = val
    this.idx += 1
    return this.list[this.idx - 1]
  }

  popBack() {
    if (this.isEmpty()) return '값이 없어요'
    this.idx -= 1
    const val = this.list[this.idx]
    delete this.list[this.idx]
    return val
  }

  pushFront(val: any): any {
    if (this.isEmpty()) this.pushBack(val)
    else if (this.lowestCount > 0) {
      this.lowestCount -= 1
      this.list[this.lowestCount] = val
    } else {
      for (let idx = this.idx; idx > 0; idx -= 1) {
        this.list[idx] = this.list[idx - 1]
      }
      this.lowestCount = 0
      this.list[this.lowestCount] = val
      this.idx += 1
    }
    return this.list[this.lowestCount]
  }

  popFront(): any {
    if (this.isEmpty()) return '값이 없어요'
    const val = this.list[this.lowestCount]
    delete this.list[this.lowestCount]
    this.lowestCount += 1
    return val
  }

  toString(): string {
    console.log('idx', this.lowestCount, this.idx)
    let str = ''
    for (const key in this.list) {
      if (this.list.hasOwnProperty(key)) {
        str += `${this.list[key]} `
      }
    }
    return str
  }

  clear(): void {
    this.list = {}
    this.idx = 0
    this.lowestCount = 0
  }

  isEmpty(): boolean {
    return this.idx === 0
  }

  length() {
    return this.idx - this.lowestCount
  }
}

const dq = new Dequeue()

console.log(dq.pushFront(1)) // 1
console.log(dq.pushFront(2)) // 2
console.log(dq.pushFront(3)) // 3
console.log(dq.length()) // 3
console.log(dq.popFront()) // 3
console.log(dq.length()) // 2
console.log(dq.pushBack(3)) // 3
console.log(dq.length()) // 3
console.log(dq.toString()) // 2 1 3
console.log(dq.popFront()) // 2
console.log(dq.pushBack(2)) // 2
console.log(dq.pushBack(4)) // 4
console.log(dq.length()) // 4
console.log(dq.toString()) // 1 3 2 4
console.log(dq.popFront()) // 1
console.log(dq.popFront()) // 3
console.log(dq.popFront()) // 2
console.log(dq.length()) // 1
console.log(dq.toString()) // 4
console.log(dq.popFront()) // 4
console.log(dq.toString()) // ''
```

여기서는 `Queue` 자료구조에서 사용된 `lowestCount` 를 사용하여, 구성하였다.
`lowestCount` 는 앞의 값이 삭제될때마다, 시작되는 `index` 를 지정하여 처리한다.

이러한 방식을 사용하여, 반복문을 통해서 객체를 재구성할 필요없이 사용가능하다.

이렇게 만들어진 `dequeue` 는 앞과 뒤로 값추가가 가능한 구조가 되었다.
