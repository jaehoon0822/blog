---
title: '자료구조 Stack 에 대해서'
date: '2023-05-09'
tags: ['자료구조', 'stack', '스택']
draft: false
summary: '자료구조 Stack'
---

# Queue 를 구현해보자

> `Queue` 는 `Stack` 과 반대의 개념이다.
> `FIFO` 라고 해서 `First In First Out` 의 개념으로 만들어진 자료구조이다

`Queue` 는 실제로 매표소에 줄 서있는 사람과 똑같은 구조이다.
처음온 사람이 표를 끊고 그다음사람이 표를 끊는다. 새로운 사람이 있다면 줄의 맨끝에서 앞사람의 처리가 끝날때까지 기다린다.

또한, `Print` 할때도, 첫번째 장이 프린트되어야 그 다음 장이 프린트된다.
이것역시 `Queue` 라고 볼 수 있다

그럼 `Queue` 를 구현하기 위해 `Class` 를 구현해보자

```ts
interface IList {
  [idx: number]: any
}

abstract class AbQueue {
  protected idx: number = 0 // <-- queue 의 첫번째 index
  protected list: IList = {} // <-- queue list

  public abstract enqueue(item: any): any // <-- queue 에 item 추가
  public abstract dequeue(): any // <-- queue 의 첫번째 item 제거
  public abstract peek(): any // <-- queue 의 첫번째 item 반환
  public abstract legnth(): number // <-- queue 의 길이
  public abstract clear(): void // <-- queue tiem 전부 삭제
  public abstract isEmpty(): boolean // <-- queue 가 비었는지 확인
  public abstract toString(): string
  protected abstract sortedObj(): void // <-- equeue 이후 queue 재정렬 함수
}
```

이러한 `Queue` 의 설계도를 만들고 다음은 구현해보도록 한다

```ts
interface IList {
  [idx: number]: any
}

abstract class AbQueue {
  protected idx: number = 0 // <-- queue 의 첫번째 index
  protected list: IList = {} // <-- queue list

  public abstract enqueue(item: any): any // <-- queue 에 item 추가
  public abstract dequeue(): any // <-- queue 의 첫번째 item 제거
  public abstract peek(): any // <-- queue 의 첫번째 item 반환
  public abstract legnth(): number // <-- queue 의 길이
  public abstract clear(): void // <-- queue tiem 전부 삭제
  public abstract isEmpty(): boolean // <-- queue 가 비었는지 확인
  public abstract toString(): string
  protected abstract sortedObj(): void // <-- equeue 이후 queue 재정렬 함수
}

class Queue extends AbQueue {
  enqueue(item: any) {
    this.list[this.idx] = item
    const val = this.list[this.idx]
    this.idx += 1
    return val
  }

  dequeue() {
    if (this.idx < 0) this.idx = 0
    if (this.isEmpty()) {
      return '값이 비어있어요'
    }
    const val = this.list[0]
    delete this.list[0]
    this.sortedObj()
    this.idx -= 1
    return val
  }

  peek() {
    if (this.isEmpty()) {
      return '값이 비어있어요'
    }
    return this.list[0]
  }

  clear() {
    this.list = {}
    this.idx = 0
  }

  public isEmpty(): boolean {
    return this.legnth() === 0
  }

  public legnth(): number {
    return this.idx
  }

  public toString(): string {
    if (this.isEmpty()) return '값이 비어있어요'
    let str = ''
    for (const key in this.list) {
      if (this.list.hasOwnProperty(key)) str += this.list[key] + ' '
    }

    return str
  }

  sortedObj() {
    let idx = 0
    for (const item in this.list) {
      if (this.list.hasOwnProperty(item)) {
        this.list[idx] = this.list[item]
        idx += 1
      }
    }
    delete this.list[idx]
  }
}

const q = new Queue()
console.log(q.enqueue('jh')) // jh
console.log(q.enqueue('kim')) // kim
console.log(q.enqueue('lee')) // lee
console.log(q.toString()) // jh kim lee
console.log(q.dequeue()) // jh
console.log(q.toString()) // kim lee
console.log(q.legnth()) // 2
console.log(q.peek()) // kim
console.log(q.clear()) // undefinedk
console.log(q.isEmpty()) // true
console.log(q.dequeue()) // 값이 비어있어요
```

여기서 중요한 부분은 `sortedObj` 인데, 구현하다 보니 `dequeue` 시 첫번째 값이 빠지면서 그 값을 채워넣어 주어야만 했다.

그래서 `this.list` 를 순환한후, `idx` 값의 증가값에 맞추어 객체를 재구성한다.
그리고, 마지막 `property` 는 남아있으므로, 마지막에 증가된 값인 `idx` 를 사용하여 마지막 값을 없애주면, `dequeue` 된 `list` 가 만들어 진다

한번 구현후 내가 참고한 책에서는 다른 방법으로 구현했다.
기존의 `code` 에 `lowestCount` 를 넣어서, 위의 `sortedObj` 없이 구현했다.

이 방법이 훨씬 좋아보인다.
반복문 없이 구현가능하면 없이 구현하는것이 좋을듯 싶다

로직은 다음과 같이 생겨먹었다

```ts
interface IList {
  [idx: number]: any
}

abstract class AbQueue {
  protected idx: number = 0 // <-- queue 의 첫번째 index
  protected lowestCount: number = 0 // <-- dequeue 이후 증가된 갯수
  protected list: IList = {} // <-- queue list

  public abstract enqueue(item: any): any // <-- queue 에 item 추가
  public abstract dequeue(): any // <-- queue 의 첫번째 item 제거
  public abstract peek(): any // <-- queue 의 첫번째 item 반환
  public abstract legnth(): number // <-- queue 의 길이
  public abstract clear(): void // <-- queue tiem 전부 삭제
  public abstract isEmpty(): boolean // <-- queue 가 비었는지 확인
  public abstract toString(): string
}

class Queue extends AbQueue {
  enqueue(item: any) {
    this.list[this.idx] = item
    const val = this.list[this.lowestCount]
    this.idx += 1
    return val
  }

  dequeue() {
    if (this.idx < 0) {
      this.idx = 0
    }
    if (this.isEmpty()) {
      return '값이 비어있어요'
    }
    const val = this.list[0]
    delete this.list[0]
    this.lowestCount += 1
    this.idx -= 1
    return val
  }

  peek() {
    if (this.isEmpty()) {
      return '값이 비어있어요'
    }
    return this.list[this.lowestCount]
  }

  clear() {
    this.list = {}
    this.idx = 0
    this.lowestCount = 0
  }

  public isEmpty(): boolean {
    return this.legnth() === 0
  }

  public legnth(): number {
    return this.idx
  }

  public toString(): string {
    if (this.isEmpty()) return '값이 비어있어요'
    let str = ''
    for (const key in this.list) {
      if (this.list.hasOwnProperty(key)) str += this.list[key] + ' '
    }

    return str
  }
}

const q = new Queue()
console.log(q.enqueue('jh')) // jh
console.log(q.enqueue('kim')) // kim
console.log(q.enqueue('lee')) // lee
console.log(q.toString()) // jh kim lee
console.log(q.dequeue()) // jh
console.log(q.toString()) // kim lee
console.log(q.legnth()) // 2
console.log(q.peek()) // kim
console.log(q.clear()) // undefined
console.log(q.isEmpty()) // true
console.log(q.dequeue()) // 값이 비어있어요
```

훨씬 깔끔하고 잘 작동한다.
오늘인 `Queue` 구조에 대해서 살펴보았다

다음은 `DeQeue` 에 대해서 알아보고 구현해보도록 한다
