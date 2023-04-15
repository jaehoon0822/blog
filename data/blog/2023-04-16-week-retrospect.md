---
title: '4월 2째주 회고'
date: '2023-04-16'
tags: ['retrospective', '회고']
draft: false
summary: '4월 2째주의 회고'
---

# 4월 2째주 회고

> 이번주 부터 매주마다 나 자신에 대한 회고를 시작하며, 내 자신을 되돌아 보는 시간을 가지려 한다.

## Facts

이번주는 주로, 내가 알고 싶어했던 여러가지 개념들을 정리하는 시간을 보낸것 같다.
주로 많이 보았던 부분은 `twin.macro` 를 사용하여, `tailwindcss` 와 `emotion` 을   
어떻게 병합하여 사용할까에 대한 고민이 많았었다.

하지만, `twin.macro` 를 사용하면서, `tailwindcss` 의 `API` 습득 및 `twin.macro` 에서 제공하는 `API` 습득을 하는데 시간을 걸린것 같다.

또한, 어제 느낀것인데, `input` 관련해서 처리하다가, 문득, `react-hook-form` 에 대해  
제대로 알고 사용하는것인가 하는 의문이 들었다.

그저 `input` 을 `hooks` 로 빼서 편하게 사용하려고 하는 목적만 있는것 같아,  
이 `library` 가 왜 등장했으며, 어떠한 개념으로 만들어졌는지 알고 싶다는 욕구가 셈솟았다.

## Feelings

### twin.macro

이전부터 `tailwindcss` 를 사용하고 싶었지만, `emotion` 을 사용하고 있어,  
나중으로 미루고 있던 상황이 었다.

그러던중, `twin.macro` 를 알게 되며, `tailwindcss` 와 `emotion` 을 같이 사용할 수 있는 방법이 있다는것을 알게 되었을때 이것을 사용해야겠다는 생각이 들었다!!

사실 `emotion` 을 사용하면서, 불편했던 점이 `inline` 으로 `css` 작업시,  
`code` 가 매우 지저분해 지는 단점이었다.

`emotion` 으로 스타일된 컴포넌트를 만드는것(CSS in JS)은 매우 좋았지만,  
해당 컴포넌트에 `inline` 으로 추가적인 `css` 작업이 이루어져야 하는 상황이 꽤나 많이 발생했다.

그래서 깔끔하게 정리하기 위해 처음 사용했던 방법은 `props` 로 공통컴포넌트를 만들고  
그 공통컴포넌트를 상속해서, 다른 컴포넌트를 만드는 방식을 사용했었다.

하지만, 마음에 들지 않는다.

그러던중, `tailwindcss` 의 존재를 알게 되고, `utility-first` 라는 개념을 통해  
미리 만들어 놓은 유틸리티 클래스를 사용하여 스타일을 적용하는 굉장히 `Awesome` 한 방식을 알게 되었다.

이러한 방식의 장점으로는, 미리 지정해놓은 유틸리티 클래스를 사용하므로,  
협업에 굉장히 도움이 된다.

또한, `css` 작성 `code` 를 줄이면서, 원하는 `css` 로 스타일링 가능하기에,  
기존에 제공되는 비슷한 프레임워크(`bootstrap`) 보다 더 유연한 디자인이 가능하다

그런데, `twin.macro` 를 사용하면, `emotion` 의 장점인 `CSS in JS` 와 함께 `Utility-first` 라는 개념을 같이 사용할 수 있게 해준다고 한다.

이것은 매우 흥분되고, 이 `tool` 을 꼭 사용해보고 말겠다는 의지가 불살라 올랐다!

### react-hook-form

어제 `4월 15일(어제)` 에 `input` 관련 `component` 를 작성하면서, 갑자기 의문이 들었다.  

`react-hook-form` 에 대한 개념을 명확하게 알고 사용은 하고 있는것인가?
내가 왜 이 `library` 를 채택해서 사용하고 있는것이지?
단지, `hooks` 로 빼서 `공통 컴포넌트` 를 만들기 위해서?  

항상, 무언가를 배울때 느낀점은 그 배경지식을 알고 접근한다면, 이후 더 수월하게 습득 가능하다는 것이다.

나는 `react-hook-form` 에 대해서 `비제어 컴포넌트` 를 사용해서 더 좋다는  
외우기 식의 개념만 박혀 있었지, 내가 `비제어 컴포넌트` 에 대해 정확하게 이해하고 있는지 확신할 수 없었다.

그렇다면, 나는 이 `react-hook-form` 에대한 개념을 머릿속에 박아 넣기 위해  
그 개념을 알아야 했다.  

## Findings

### twin.macro 를 하면서 배운점

`twin.macro` 를 사용하기 위해 알아보던준 `next.js` 의 `version12` 부터 `babel` 이 아닌 `swc` 를 사용한다는것을 알게 되었다.

덕분에, `swc` 가 얼마나 유용하며 `transpiler` 의 미래라는것도 알게되었다
역시, `next.js` 의 `docs` 를 파면서, 개념들을 정리할 시간이 필요하다는것을 깨우치는 시간이 되기도 했다.

처음부터 사용하기 위한 장벽이 생겨났지만, 어찌 저찌해서 `withTwin` 함수를 `wrapper` 로 만들어 `next.config.js` 에서 사용할 수 있도록 만들었다.  

하지만, 거기서 문제가 끝나지 않았다...
어찌된 영문인지, `tailwind.config.js` 에 설정한 값이 `twin.macro` 에 적용되지 않았다.

설정값으로 `extends` 해서 `colors` 값으로 `primary` 값을 주려고 했지만,  
`Primary is not found` 라는 `Error message` 만 반복될 뿐이었다.

이를 해결하기 위해 `twin.macro` 의 `docs` 를 보며, `pacakge.json` 에 `babelMacro` 를 주어 `config` 값으로 `tailwind.config.js` 로 주어보고, `content` 값으로 `glob` 방식의 경로도 주어보았지만 모두 헛수고였다.

내가 본 `API` 면, 방법이 맞을텐데, 적용이 안되는것이 아직도 이해가 가지 않는다.
하지만, `컴퓨터는 거짓말을 하지 않더라.. 내 잘못이겠지....`

결국, `tailwind.config.js` 를 사용하여 설정한 값으로는 사용 못하고 `default theme` 만으로 코드를 작성하기 시작했다.

사실, `default theme` 만으로도 굉장히 많은 것들을 제공해줘서, 부족함은 없더라..

`하지만, custom theme 을 사용하고 싶은 욕구는 여전하다... 다음 project 때 다시 도전해봐야지`

이렇게, 어려움을 딛고 일어나서, `code` 를 작성하기 시작하면서 몇가지 방식에 대해서  
정리할 필요성을 느꼈다.

`twin.macro` 는 `tw` 을 `import` 해서 매우 쉽게 사용가능하게 만들어 졌다.
`tw` 을 사용해 `CSS in JS` 를 구현할 수 있지만, `styled component` 가 필요한 상황이 생겨났다.

`styled component` 사용시 몇가지 방식이 존재하는데, `variants` 를 사용하여, 구현하는 방식과, `interplation` 과 `overriding` 하는 방식을 사용한다.

`variants` 를 사용하는 방법이 약간 낮설어서, 이 부분을 조금더 익숙해지는것이 좋을것 같다는 생각이 든다.

> `code`  는 아래와 같다

```tsx

import tw, { styled, TwStyle } from 'twin.macro'

type ContainerVariant = 'light' | 'dark' | 'crazy'

interface ContainerProps {
  variant?: ContainerVariant
}

// Use the `TwStyle` import to type tw blocks
const containerVariants: Record<ContainerVariant, TwStyle> = {
  // Named class sets
  light: tw`bg-white text-black`,
  dark: tw`bg-black text-white`,
  crazy: tw`bg-yellow-500 text-red-500`,
}

const Container = styled.section<ContainerProps>(() => [
  // Return a function here
  tw`flex w-full`,
  ({ variant = 'dark' }) => containerVariants[variant], // Grab the variant style via a prop
])
const Column = tw.div`w-1/2`

const Component = () => (
  <Container variant="light">
    <Column></Column>
    <Column></Column>
  </Container>
)
```

개념을 보면 어렵지는 않은 개념인데, 실제 사용하려고 하니, 조금은 헷갈리더라..

아직 `typescript` 를 사용하면서 `type` 지정에 대해서 익숙하지 않은 듯 싶다.
추후 `typescript` 를 재 공부하면서 `type` 과 `generic` 에 대해 알아야 겠다.

굉장히 유용한 `tool` 임에는 분명하지만, 기존의 `emotion` 을 쓰던 방식이  
그대로 남아 있어서 그런지, 사용하면서 익숙치 않은 개념들이 손을 덜 가게 만드는것은 사실이다.

> 예를 들면, `[]` 을 사용해서 `tw` 과 `props function` 을 넘겨준다는것 같은 것들...

이런것들은, 점차 익혀나가면 되는것들이라 그렇게 걱정은 되지 않는다.
기껏해야, 사용하기 시작한 시점은 몇일 안되었으니, 익숙하지 않은것이 당연하다.

지속 사용하면서, `tailwindcss` 의 `Docs` 를 확인하며 `code` 작성 중이지만,  
`tailwindcss` 는 정말 감탄하면서 사용하고 있다.

굉장히 잘만든 `css framework` 라는 생각이 든다.
앞으로 `tailwindcss` 는 나의 주력 `library` 가 될것 같은 생각이 든다.

또한, `tailwindcss` 를 사용하면서, `css` 에 대한 지식이 많이 부족한점을  
계속 깨닫는다.  

한가지 깨닫게 되는 부분중 하나가, `twin.macro` 를 사용하는것이 과연 괜찮은 것인가 싶다.

확실히 유용한 `tool` 임은 분명하지만, 아직 성숙치 않은 `library` 라 사용에 꺼리김이 존재하기도 하다.

일단, `tailwindcss` 의 유용함을 제대로 만끽하는 기회가 되었으니,  
이로쎠 좋은 경험이 었다고 생각한다.

### react-hooks-form 에 대해서 배운점

`react-hook-form` 의 개념을 알아가면서 `uncontrolled component` 와 `controlled component` 의 개념을 확실히 알게되었다.

또한, `State Colocation(상태융합)` 을 통해, `react` 구조 설계시 더 효율적인 구조로 작성하기 위해 고민하기 위한 지식도 습득하게 된것 같다.

`library` 하나가 탄생하기 위해 기존의 문제점을 해결하여 만들어졌다는 부분에
대해 알아가는 과정역시 중요하다는 생각을 다시한번 하게 된 좋은 공부였다.

## Future Action

앞으로 `tailwindcss` 및 `next.js` 를 조금더 자세히 공부하고,  
추가적으로 `react native` 를 사용해서 `app` 을 만드는것을 추가적 목표로 삼을 예정이다.

무엇보다 중요한것은 `취업` 관련해서 지속 노력하는 시간을 갖도록 해야한다.

현재 `side project` 진행을 하기 위해, 기존의 작업하던 `project` 역시 빠르게 마무리 지어야 하는 상황이다.

첫번째 목표는 단연 `취업` 준비를 하며 `side project` 를 빠르게 진행하여, 결과물을 내놓는 것이다.

이번주를 뒤돌아보면, 이전보다 조금 게을러진것이 느껴진다.

나 자신에 대해서 더 많이 뒤돌아보며 지속적인 회고를 진행하며, 더 나은 내일을 만들어나가는것이 중요하다는 생각이 든다.
