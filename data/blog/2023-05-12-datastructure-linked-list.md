---
title: '자료구조 linkedList 에 대해서'
date: '2023-05-12'
tags: ['linkedList', '자료구조', '링크드리스트']
draft: false
summary: '자료구조 linkedList'
---

# linkedList 를 구현해보자

`LinkedList` 는 일반적이 배열과는 다르게, `Memory 주소` 를 통해 서로가 서로를 참조하는 방식으로 이루어져있다

이러한 `Meomory Address` 를 가리키는 것을 `Pointer` 라고 하는데, `LinkedList` 는 값과 함께 다음에 참조할 `Pointer` 를 가진다.

이렇게 `LinkedList` 를 통해 참조가 이루어진다면, 배열처럼 메모리 할당시 `Overhead` 없이 사용가능하며, 무엇보다 앞에서 다룬 `Queue` 나 `Stack` 처럼 원소 추가 및 삭제시 원소를 이동할 필요없이, `Pointer` 로 다음 연결한 원소를 연결해주면 된다

이는 마치 사람이 손에 손잡은 형태로 이루어져 있으며, 중간에 빠지는 사람은 손을 놓고, 그다음 사람의 손을 다시 잡는 형태로 이루어진다.

이를 구현한 로직은 다음과 같다

```ts
interface INode {
  elm: any // <-- Node 의 값
  next: INode | null // <-- next 가 null 이면 리스트의 마지막 Node 이다
  //     그렇지 않다면 다음 Node 를 참조한다
}

interface ILinkedList {
  append(elm: any): INode // <-- Node 추가
  insert(elm: any, pos: number): INode // pos 에 해당하는 Node 추가
  remove(elm: any): INode | string // <-- 값에 해당하는 Node 삭제
  removeAt(pos: number): INode | string // <-- pos 에 해당하는 Node 삭제
  toString(): string // <-- Node 의 값들을 문자열로 나열
  indexOf(elm: INode): number // <-- Node 의 index 반환
  size(): number // <-- Node 의 갯수반환
  isEmpty(): boolean // <-- List 가 비어있으면 true, 아니면 false
  clear(): void // <-- List 를 초기화
}

class LinkedNode implements INode {
  public next = null // <-- 다음 원소의 Pointer 참조, 참조 하지 않는다면 null 이다
  constructor(public elm: any) {} // <-- LinkedNode 생성시 들어갈 원소값
}

class LinkedList implements ILinkedList {
  private head: INode | null = null // <-- LinkedList 의 첫번째 Node참조 변수
  private length: number = 0 // <-- LinkedList 의 길이

  append(elm: any) {
    let cur = null
    if (!this.head) {
      // this.head 가 없다면 아직 리스트 생성이 이루어지지 않았으므로
      // 첫번째 Node 생성
      this.head = new LinkedNode(elm)
      this.length += 1
      return this.head
    } else {
      // Node 가 있으므로. this.head 를 참조하여
      // next 값이 null 인 요소를 찾는다
      cur = this.head
      while (cur.next) {
        cur = cur.next
      }
      cur.next = new LinkedNode(elm) // next 에 새로운 LinkedNode 생성
      this.length += 1
      return cur.next
    }
  }

  insert(elm: any, pos: number) {
    let prev = null
    if (!this.head) {
      // <-- this.head 가 null 이면 this.head 에 새로운 Node 생성
      this.head = new LinkedNode(elm)
      this.length += 1
      return this.head
    }
    if (pos <= this.length) {
      // <-- pos 가 length 보다 작다면 요소를 찾는다
      let cur = null
      let idx = 1 // <-- 1 부터 시작

      cur = this.head
      while (idx < pos && cur.next) {
        prev = cur
        cur = cur.next
        idx += 1
      }
      // pos 값이 항상 idx 값보다 크지는 않다
      // 이럴경우 prev 가 존재하지 않을 수 있다.
      // 밑은 prev 값이 존재한다면 실행한다
      if (prev) {
        prev.next = new LinkedNode(elm)
        prev.next.next = cur
        this.length += 1
        return prev.next
      }
      // prev 값이 존재하지 않는다면, 첫번째 값이라고 가정한다.
      // perv 에 현재의 this.head 를 참조하고
      // this.head 에 새로운 Node 를 만든후 next 값을 prev 로 연결한다.
      prev = this.head
      this.head = new LinkedNode(elm)
      this.head.next = prev
      this.length += 1
      return this.head
    } // <--- pos 가 length 보다 크다면, linkedNode 끝에 추가
    prev = this.head
    while (prev.next) {
      prev = prev.next
    }
    prev.next = new LinkedNode(elm)
    this.length += 1

    return prev.next
  }

  // 이 다음 부터는 로직들이 거의 동일하므로 생략한다.

  remove(elm: any) {
    let cur = null
    let prev = null
    if (!this.head) return '삭제할 값이 없습니다'
    cur = this.head
    while (cur.elm !== elm && cur.next) {
      prev = cur
      cur = cur.next
    }

    if (cur.elm !== elm) {
      return '삭제할 값이 없습니다.'
    }

    if (prev) {
      prev.next = cur.next
      this.length -= 1
      return cur
    }

    this.length -= 1
    this.head = cur.next
    return cur
  }

  removeAt(pos: number) {
    if (!this.head) return '삭제할 값이 없습니다'
    if (pos <= this.length) {
      let cur: INode | null = null
      let prev: INode | null = null
      let idx = 1
      cur = this.head
      while (idx !== pos && cur.next) {
        prev = cur
        cur = cur.next
        idx += 1
      }
      if (prev) {
        prev.next = cur.next
        this.length -= 1
        return cur
      }
      this.head = this.head.next
      this.length -= 1
      return cur
    }
    return '삭제할 값이 없습니다'
  }

  indexOf(elm: any): number {
    let idx = 1 // <-- idx 값은 1부터 시작하는것으로 한다
    if (!this.head) return -1

    let cur = this.head
    while (cur.elm !== elm && cur.next) {
      cur = cur.next
      idx += 1
    }
    return idx
  }

  size(): number {
    return this.length
  }

  isEmpty(): boolean {
    if (!this.head) return true
    return false
  }

  clear(): void {
    this.head = null
    this.length = 0
  }

  toString(): string {
    if (!this.head) return '값이 없습니다.'
    let cur = this.head
    let str = ''
    while (cur.next) {
      str += `${cur.elm} `
      cur = cur.next
    }
    return str + cur.elm
  }
}

const lk = new LinkedList()

console.log(lk.append(1)) // LinkedNode { elm: 1, next: null }
console.log(lk.append(2)) // LinkedNode { elm: 2, next: null }
console.log(lk.toString()) // 1 2
console.log(lk.insert(3, 1))
/*
LinkedNode {
  elm: 3,
  next: LinkedNode { elm: 1, next: LinkedNode { elm: 2, next: null } }
}
*/
console.log(lk.toString()) // 3 1 2
console.log(lk.remove(3))
/*
LinkedNode {
  elm: 3,
  next: LinkedNode { elm: 1, next: LinkedNode { elm: 2, next: null } }
}
*/
console.log(lk.toString()) // 1 2
console.log(lk.remove(5)) // 삭제할 값이 없습니다
console.log(lk.toString()) // 1 2
console.log(lk.removeAt(1)) // LinkedNode { elm: 1, next: LinkedNode { elm: 2, next: null } }
console.log(lk.toString()) // 2
console.log(lk.insert(3, 1))
console.log(lk.size()) // 2
console.log(lk.toString()) // 3 2
console.log(lk.indexOf(3)) // 1 <-- index 는 0 부터가 아닌 1부터 시작
```

이렇게 `LinkedList` 를 구현해 보았다.
중요한 포인트는 `next` 를 어떻게 연결하고 어떻게 끊느냐이다.

`next` 를 통해 `Node` 를 연결하는데, 따로 삭제처리하는 로직은 없다.
단순히 `Node` 의 연결을 다른 `Node` 로 연결만한다

이렇게 해도 괜찮을까 싶지만, 이렇게 해도 `Javascript Garbage Collector` 가 사용하지 않는 객체는 `Memory 참조` 에서 자동적으로 제거하므로, 따로 메모리 해제 처리를 해지 않아도 된다

실제로, `C` 에서는 위 로직에서 매번 `Memory 해제` 처리를 해주어야 한다.
안그러면, 사용하지 않는 객체가 쌓이면서, `Memory` 를 점유하는 형태가 이루어진다.

`Javascript` 는 이러한 불편함없이, 알아서 사용하지 않는 객체의 `Memory` 를 해제해주므로, 위와 같은 처리가 가능하다

`LinkedList` 에 대해서 알아보았으며, 다음은 `Doubly Linked List` 에 대해서 구현해도록 한다
