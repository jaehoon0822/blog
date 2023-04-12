---
title: 'tailwindcss 를 알아보자'
date: '2023-04-10'
tags: ['tailwindcss', 'next.js']
draft: false
summary: 'tailwindcss 에 대한 환경설정 및 개념'
---

# Tailwindcss 도입배경

기존에 `Emotion` 을 사용하고 있었다.  
`Emotion` 을 사용하면서 굉장히 불편했던점이, 매번 `Styling 된 Component` 를 생성하는 문제이다.  

그렇게 만든 `Component` 를 가져오고, 사용하는데, 꼭 추가적인 `css` 를 추가할 일이 생긴다.

그럴때, 그냥 `Emotion css` 를 사용하면 간단히 해결되지 않느냐 물을수 있지만, `Code` 가 굉장히 더러워지는 상황이 발생하는것이 끝내 내키지 않았다.

그래서 이를 해결하고자 했던 방식은 다음과 같다.

> 공통적으로 많이 변경되는 `css` 스타일을 미리 만들어놓고 상속하기

```ts
// commons/style/index.ts
interface ICommonProps {
  left?: string;
  right?: string;
  top?: string;
  bottom?: string;
}

interface IDivProps extends ICommonProps {
  direction?: string;
  justyfyContents?: string;
  alignItems?: string;
}
export const Div = styled(div)<IDivProps>`
  display: flex;
  padding-left: ${({ left }) => left ?? null};
  padding-right: ${({ right }) => right ?? null};
  padding-top: ${({ top }) => top ?? null};
  padding-bottom: ${({ bottom }) => bottom ?? null};
  flex-direction: ${({ direction }) => direction ?? "row"};
  justify-content: ${({ justyfyContents }) => justyfyContents ?? "flex-start"};
  align-items: ${({ alignItems }) => alignItems ?? "flex-start"};
`;
```

```ts
// component/commons/style.ts
import * as GS from '@/commons/style'

export const Wrapper = styled(GS.Div)`
  etc...
`
```

일단 저렇게 따로 상속받아서, 많이 사용되는 속성을 처리하도록 만들었다.  
하지만 저것역시 불완전하다.  
`emotion` 는 `CSS in JS` 에 매우 적합하고 좋은 툴이지만, `inline` 으로  
깔끔하게 `styling` 하고 싶기도 하다.  

> `emotion` 으로 `inline` 작성시 매우 지저분해진다...

그러던 와중, `tailwindcss` 에 대해서 알게 되었고,  
`utility-first` 라는 개념을 사용하여, `css` 사용시 `inline` 으로 작성하듯 작성하여,  
매우 쉽게 `styling` 하는것을 볼 수 있었다.  

또한 굉장히 직관적인, `class naming` 역시 사용하기 매우 좋았다.
`class naming` 이 정해져 있다면, 협업시 매우 효율적인 방식으로 사용가능할 것이다.

거기다, `tailwindcss.config` 를 사용하여 직접 `naming` 역시 가능하다.  

`Design System` 을 `tailwindcss.config` 를 사용하여 적용한다면, 손쉽게  
`theme` 을 만들어 사용 가능할것으로 느껴졌다.

다음은 `Docs` 에 나온 `tailwindcss.config.js` 의 예이다.

```js
module.exports = {
  content: ['./src/**/*.{html,js}'],
  theme: {
    colors: {
      'blue': '#1fb6ff',
      'purple': '#7e5bef',
      'pink': '#ff49db',
      'orange': '#ff7849',
      'green': '#13ce66',
      'yellow': '#ffc82c',
      'gray-dark': '#273444',
      'gray': '#8492a6',
      'gray-light': '#d3dce6',
    },
    fontFamily: {
      sans: ['Graphik', 'sans-serif'],
      serif: ['Merriweather', 'serif'],
    },
    extend: {
      spacing: {
        '8xl': '96rem',
        '9xl': '128rem',
      },
      borderRadius: {
        '4xl': '2rem',
      }
    }
  },
}
```

위의 예 및 `tailwindcss` 의 `classname` 을 보면서 `emotion` 과 같이 사용해야할 `library` 라고 확신하게 되었다.

`tailwindcss` 를 사용하면서 중요한것은 사용 환경설정을 습득하는 것이다.
지금부터 `환경설정` 관련 내용을 정리하도록 한다.

## Theme Configuration

`tailwindcss.config.js` 에는 `theme` 섹션이 존재한다.  
`theme` 에서는 `project` 의 `color`, `palette`, `type scale`, `fonts`, `breackpongs`, `border radius...` 같은 `values` 들을 정의할 수 있다.

### Theme structure

`theme` 객체는 `screens`, `colors`, `spacing` 등 사용자 지정가능한 `key` 들을 가지며,  
각 사용자 지정 가능한 핵심 플러그인들([Core Plugins](https://tailwindcss.com/docs/configuration#core-plugins))에 대한 키도 포함한다.

### Screens

`screens key`는 `project` 안에 반응형 중단점을 커스터마이징할 수 있도록 허용한다

```js
module.exports = {
  theme: {
    screens: {
      'sm': '640px',
      'md': '768px',
      'lg': '1024px',
      'xl': '1280px',
      '2xl': '1536px',
    }
  }
}
```

### Colors

`colors key` 는 전역 색상 팔레트를 커스터마이징 할 수 있도록 해준다.

```js
module.exports = {
  theme: {
    colors: {
      transparent: 'transparent',
      black: '#000',
      white: '#fff',
      gray: {
        100: '#f7fafc',
        // ...
        900: '#1a202c',
      },

      // ...
    }
  }
}

```

