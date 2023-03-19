---
title: '신규 아이디 추천'
date: '2023-02-27'
tags: ['algorithm', '프로그래머스']
draft: false
summary: '프로그래머스 신규 아이디 추런'
---

# 신규 아이디 추천

> 1단계 new*id의 모든 대문자를 대응되는 소문자로 치환합니다.
> 2단계 new_id에서 알파벳 소문자, 숫자, 빼기(-), 밑줄(*), 마침표(.)를 제외한 모든 문자를 제거합니다.
> 3단계 new_id에서 마침표(.)가 2번 이상 연속된 부분을 하나의 마침표(.)로 치환합니다.
> 4단계 new_id에서 마침표(.)가 처음이나 끝에 위치한다면 제거합니다.
> 5단계 new_id가 빈 문자열이라면, new_id에 "a"를 대입합니다.
> 6단계 new_id의 길이가 16자 이상이면, new_id의 첫 15개의 문자를 제외한 나머지 문자들을 모두 제거합니다.

     만약 제거 후 마침표(.)가 new_id의 끝에 위치한다면 끝에 위치한 마침표(.) 문자를 제거합니다.

7단계 new_id의 길이가 2자 이하라면, new_id의 마지막 문자를 new_id의 길이가 3이 될 때까지 반복해서 끝에 붙입니다.

```js
const filter = 'qwertyuiopasdfghjklzxcvbnm1234567890-_.'

function solution(new_id) {
  // 1 단계
  new_id = new_id.toLowerCase()

  // 2단계
  let answer = ''
  for (let i = 0; i < new_id.length; i += 1) {
    if (filter.includes(new_id[i])) {
      answer += new_id[i]
    }
  }

  while (answer.includes('..')) {
    answer = answer.replace('..', '.')
  }
  if (answer[0] === '.') answer = answer.substring(1)
  if (answer.length >= 16) answer = answer.substring(0, 15)
  if (answer[answer.length - 1] === '.') {
    answer = answer.substring(0, answer.length - 1)
  }
  if (answer.length === 0) answer = 'a'
  while (answer.length <= 2) {
    answer = answer += answer[answer.length - 1]
  }
  console.log(answer)
  return answer
}
```

```js
// 새로운 아이디에 들어갈 수 있는 문자열
const filter = 'qwertyuiopasdfghjklzxcvbnm1234567890-_.'

function solution(new_id) {
  // 1단계 : 대문자를 소문자로 치환
  new_id = new_id.toLowerCase().split('')

  // 2단계 : 알파벳 소문자, 숫자, 빼기, 밑줄, 마침표를 제외한 모든 문자 제거
  let answer = new_id.filter((str) => filter.includes(str))

  // 3단계 : 마침표가 2번 이상 연속된다면, 1개의 마침표로 치환
  answer = answer.filter((str, i) => {
    return str !== '.' || (str === '.' && answer[i + 1] !== '.')
  })

  // 4단계 : 마침표가 처음이나 끝에 위치한다면 제거
  if (answer[0] === '.') {
    answer = answer.slice(1)
  }

  const removeLastDot = () => {
    if (answer.at(-1) === '.') {
      // === answer[ answer.length - 1 ]
      answer = answer.slice(0, answer.length - 1) // answer.pop();
    }
  }
  removeLastDot()

  // 5단계 : 빈 배열일 경우 문자열 "a"를 추가
  if (!answer.length) {
    answer.push('a')
  }

  // 6단계 : 길이가 16자 이상이라면, 15자까지 제거
  //        제거 한 후에 문자열 끝에 마침표가 있다면 제거
  if (answer.length >= 16) {
    answer = answer.slice(0, 15)
    removeLastDot()
  }

  // 7단계 : 길이가 2글자 이하라면, 마지막 글자를 3글자가 될 때까지 뒤에 붙여준다.
  if (answer.length <= 2) {
    const add = Array.from(new Array(3 - answer.length), () => answer.at(-1))
    answer = [...answer, ...add] // === answer.concat( add )
  }
  return answer.join('')
}
```
