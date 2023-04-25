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
  left?: string
  right?: string
  top?: string
  bottom?: string
}

interface IDivProps extends ICommonProps {
  direction?: string
  justyfyContents?: string
  alignItems?: string
}
export const Div = styled(div)<IDivProps>`
  display: flex;
  padding-left: ${({ left }) => left ?? null};
  padding-right: ${({ right }) => right ?? null};
  padding-top: ${({ top }) => top ?? null};
  padding-bottom: ${({ bottom }) => bottom ?? null};
  flex-direction: ${({ direction }) => direction ?? 'row'};
  justify-content: ${({ justyfyContents }) => justyfyContents ?? 'flex-start'};
  align-items: ${({ alignItems }) => alignItems ?? 'flex-start'};
`
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

> `emotion` 역시 `theme` 을 제공하지만, `theme.provider` 를 사용하는둥, 뭔가 석연찮게 설정해야 하는 부분이 많았다...<br/>오죽하면, 이전 `Project` 에서 그냥 `객체` 로 빼서 작업했다.

다음은 `Docs` 에 나온 `tailwindcss.config.js` 의 예이다.

```js
module.exports = {
  content: ['./src/**/*.{html,js}'],
  theme: {
    colors: {
      blue: '#1fb6ff',
      purple: '#7e5bef',
      pink: '#ff49db',
      orange: '#ff7849',
      green: '#13ce66',
      yellow: '#ffc82c',
      'gray-dark': '#273444',
      gray: '#8492a6',
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
      },
    },
  },
}
```

위의 예 및 `tailwindcss` 의 `classname` 을 보면서 `emotion` 과 같이 사용해야할 `library` 라고 확신하게 되었다.

`tailwindcss` 를 사용하면서 중요한것은 사용 환경설정을 습득하는 것이다.
지금부터 `환경설정` 관련 내용을 정리하도록 한다.

> 이는 `Tailwindcss Docs` 를 보며 개인적으로 정리한 내용이다.

## Theme Configuration

`tailwindcss.config.js` 에는 `theme` 섹션이 존재한다.  
`theme` 에서는 `project` 의 `color`, `palette`, `type scale`, `fonts`, `breackpoints`, `border radius...` 같은 `values` 들을 정의할 수 있다.

### Theme structure

`theme` 객체는 `screens`, `colors`, `spacing` 등 사용자 지정가능한 `key` 들을 가지며,  
각 사용자 지정 가능한 핵심 플러그인들([Core Plugins](https://tailwindcss.com/docs/configuration#core-plugins))에 대한 키도 포함한다.

### Screens

`screens key`는 `project` 안에 반응형 중단점을 커스터마이징할 수 있도록 허용한다

```js
module.exports = {
  theme: {
    screens: {
      sm: '640px',
      md: '768px',
      lg: '1024px',
      xl: '1280px',
      '2xl': '1536px',
    },
  },
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
    },
  },
}
```

`Docs` 에서는 기본값으로, 이 색상들이 `Color` 와 연관된 `Core Plugins` 에 상속된다고 한다.
이부분의 더 자세한 부분은 [Customizing Colors](https://tailwindcss.com/docs/customizing-colors) 에서 확인 가능하다

### Spacing

`spacing` 은 전역적으로 크기관련 해서 `Customizing` 가능하다.

이 값은 크기에 영향을 받는 `padding`, `margin`, `width`, `height`, `maxHeight`, `flex-basis`, `gap`, `inset`, `sapce`, `translate`, `scrollMargin`, `scrollPadding`, `textIndent` 같은 `Core Plugins` 에 상속된다.

```js
module.exports = {
  theme: {
    spacing: {
      px: '1px',
      0: '0',
      0.5: '0.125rem',
      1: '0.25rem',
      1.5: '0.375rem',
      2: '0.5rem',
      2.5: '0.625rem',
      3: '0.75rem',
      3.5: '0.875rem',
      4: '1rem',
      5: '1.25rem',
      6: '1.5rem',
      7: '1.75rem',
      8: '2rem',
      9: '2.25rem',
      10: '2.5rem',
      11: '2.75rem',
      12: '3rem',
      14: '3.5rem',
      16: '4rem',
      20: '5rem',
      24: '6rem',
      28: '7rem',
      32: '8rem',
      36: '9rem',
      40: '10rem',
      44: '11rem',
      48: '12rem',
      52: '13rem',
      56: '14rem',
      60: '15rem',
      64: '16rem',
      72: '18rem',
      80: '20rem',
      96: '24rem',
    },
  },
}
```

이부분에 대한 자세한 설명은 [spacing customization documentation](https://tailwindcss.com/docs/customizing-spacing) 에서 살펴보도록 하자.

### Core Plugins

`theme` 섹션의 나머지 부분은 각 `core plugins` 의 값을 설정하는데 사용된다.
`Docs` 에 나온 예시는 다음과 같다.

```js
module.exports = {
  theme: {
    borderRadius: {
      none: '0',
      sm: '.125rem',
      DEFAULT: '.25rem',
      lg: '.5rem',
      full: '9999px',
    },
  },
}
```

위는 `borderRadius` 를 `customize` 한 것이다.

각 `key` 들을 `suffix(접미사)` 로 가진 `class` 가 생성되며, 그 값은 실제 `CSS` 의 값으로 선언된다.

이는 다음과 같다.

```scss
.rounded-none {
  border-radius: 0;
}
.rounded-sm {
  border-radius: 0.125rem;
}
.rounded {
  border-radius: 0.25rem;
}
.rounded-lg {
  border-radius: 0.5rem;
}
.rounded-full {
  border-radius: 9999px;
}
```

여기서 `DEFAULT` 는 `rounded` 이며, 각 `key` 들은 `rounded-[key]` 형태로 `class` 가 만들어진것을 볼 수 있다.

만약, 기본적으로 설정되어 있는 `default Theme` 을 보고싶다면, [여기](https://github.com/tailwindlabs/tailwindcss/blob/master/stubs/config.full.js)를 클릭하면 어떻게 설정되어 있는지 볼 수 있다.

## Customizing the default theme

`Tailwindcss` 는 기본 `theme` 의 설정을 자동적으로 상속한다.  
그런데 만약 `default theme` 에서 약간의 다른 `options` 들로 커스터마이징 하고 싶다면 가능할까?

이는 객체의 상속개념과 같이 `Overriding` 하거나 `Extending`으로 가능하다.

### Extending the default theme

만약 `default theme` 를 그대로 유지하면서, 새로운 `Options` 를 추가하고 싶다고 해보자.
이때 사용할 수 있는 방법이 `extend key` 이다.

사용법은 간단하다.

`theme` 에서 `extend key` 를 가진 객체를 만들고, 해당 객체내에 `Section` 을 선언한다.

선언된 `Section` 내애 추가할 옵션을 넣어주면 된다.
이는 다음과 같다

```js
module.exports = {
  theme: {
    extend: {
      // Adds a new breakpoint in addition to the default breakpoints
      screens: {
        '3xl': '1600px',
      },
    },
  },
}
```

이렇게 하면, 기존 `default theme` 의 `screens` 에 `3xl` 가 추가된다.

### Overriding the default theme

객체의 상속에서 `Overriding` 이라는 개념이 있다.
이는 기존의 값을 `덮어씌운다`고 보면된다.

바로 이 개념과 똑같이 기존의 `default theme` 의 값을 다른 값으로 `덮어씌우기` 위해 사용되는 개념이다.

이는 위의 `extend` 처럼 `key` 값을 줄 필요 없고, 그저 `section` 을 선언한후, 해당 하는 `options` 에 새로운 값을 주면 된다.

굉장히 직관적이고 간편해보인다.

```js
// 기존의 opacity

 opacity: {
      0: '0',
      5: '0.05',
      10: '0.1',
      20: '0.2',
      25: '0.25',
      30: '0.3',
      40: '0.4',
      50: '0.5',
      60: '0.6',
      70: '0.7',
      75: '0.75',
      80: '0.8',
      90: '0.9',
      95: '0.95',
      100: '1',
    },

// overriding 된 opacity

module.exports = {
  theme: {
    // Replaces all of the default `opacity` values
    opacity: {
      '0': '0',
      '20': '0.2',
      '40': '0.4',
      '60': '0.6',
      '80': '0.8',
      '100': '1',
    }
  }
}
```

이 밖에도 몇가지 개념이 추가적으로 존재한다.

### Referencing order value

만약, `theme` 안에 다른 값을 참조해야 하는경우, 정적 값 대신에 `Closure` 를 통해 참조할 수 있다.

이 `Closure` 는 `theme()` 함수를 포함한 객체를 받으며, `theme()` 을 사용해서 다른 `value` 값을 조회해 볼수 있다고 설명한다.

`Docs` 에서 `background-size` 라는 `utilites` 는 `sapcing` 의 모든 값을 참조한다.

```js
module.exports = {
  theme: {
    spacing: {
      // ...
    },
    backgroundSize: ({ theme }) => ({
      auto: 'auto',
      cover: 'cover',
      contain: 'contain',
      ...theme('spacing'),
    }),
  },
}
```

이제 `backgroundSize` 함수는 `default theme` 뿐 만 아니라, 사용자 지정값을 참조할 수 있게 된다.

재귀적으로 작동한다고 하니, 해당 정적 값이 있다면 해당 값을 사용할 수 있다고 한다.

> :boom: 내가 이해한 개념이 맞다면,  
> 위의 `sapcing` 은 사용자가 지정한 `spacing` 이고, `core plugins` 의 `backgroundSize` 는 `Colsuer` 가 제공하는 `theme` 을 사용하여, 사용자가 지정한 `spacing` 을 상속받는다고 이해가 된다.
>
> 해석하면서, 이상한 부분이 `theme 안에 다른 값을 참조해야 하는경우` 이다.
> 이부분은 조금 모호해서 좀더 확인해 보아야 할 것같다.

추가적으로, 주의사항에 대해서 이야기 하는데, `section` 안의 `key` 가 아닌, `theme` 안의
`top level key` 를 통해서만 `closure` 를 사용할 수 있다고 설명하고 있다.

```js
// 틀림
module.exports = {
  theme: {
    fill: {
      gray: ({ theme }) => theme('colors.gray'),
    },
  },
}
```

```js
// 맞음
module.exports = {
  theme: {
    fill: ({ theme }) => ({
      gray: theme('colors.gray'),
    }),
  },
}
```

### Referencing the default theme

만약, 어떠한 이유로 인해 `default theme` 을 사용해야 하는 상황이 생기기 마련이다.
이미, 변경한 `theme` 에서는 당연히 `default theme` 을 사용할 수 없다.

이러한 경우 `default theme` 을 다음처럼 가져올 수 있다.

```js
const defaultTheme = require('tailwindcss/defaultTheme')1

module.exports = {
  theme: {
    extend: {
      fontFamily: {
        sans: [
          'Lato',
          ...defaultTheme.fontFamily.sans,
        ]
      }
    }
  }
}

```

위는 `tailwindcss/defaultTheme` 에서 `sans`를 사용한 예이다.
이처럼 `tailwindcss/defaultTheme` 에서 언제든지, 기본 설정값을 가져와 적용가능하다.

### Disabling an entire core plugin

만약, 특정 `Core Plugin` 의 `class` 를 생성하고 싶지 않다고 가정해보자.
그러한 상황에 `Core Plugins Section` 에서 `false` 값만 준다면, 해당 `Core Plugin` 을 사용하지 않아도 된다.

```js
module.exports = {
  corePlugins: {
    opacity: false,
  },
}
```

이제 `opacity class`는 생성되지 않는다.
추가로 `theme.extend` 가 가능한 `key` 들은 [여기](https://tailwindcss.com/docs/theme#configuration-reference)에서 보도록 하자.

이렇게 `Configuration` 관련 부분에 대해서 살펴보았다.
하지만 이렇게 끝나지 않는다.

목표는 `emotion` 과 `tailwindcss` 를 같이 적용하여 `BolierPlate` 를 만드는 것이다.

# Twin.macro

`twin.macro` 는 `Tailwind` 와 `CSS-in-js` 를 같이 사용하도록 만들어주는 `library` 이다.
딱, 내가 원하는 방식이다.

2개의 `library` 를 하나로 만들어 처리하려면 `Twin.macro` 설정법도 같이 알아야 한다.

## Twin.macro 의 작동방식

컴파일시, `bebel` 을 실행시킬때, `twin` 은 `classes` 를 `CSS Object` 로 변환시킨다.
이렇게 변환된 `CSS Object` 는 선택된 `CSS-in-js` 라이브러리로 전달되어 처리하는 방식이라고 되어 있다.

```js

//Awesome 한데?
import tw from 'twin.macro'

tw`text-sm md:text-lg`

// ↓ ↓ ↓ ↓ ↓ ↓

{
  fontSize: '0.875rem',
  '@media (min-width: 768px)': {
    fontSize: '1.125rem',
  },
}
```

> 찾아보던 도중에 `next.js` 가 `Twin.macro` 를 사용하는데 문제가 있음을 발견했다.
>
> `next.js` 의 `12 version` 부터 `bable` 이 아닌 `swc` 를 사용하여 처리한다고 한다.
> 사용하려면, `babel` 설정을 따로 해야 하는 상황인듯 하다.

## Twin.macro 설정

찾던중, [nextjs 12 에서 emotion과 함께하는 tailwindcss](https://www.inflearn.com/blogs/2858) 을 참고하여, 설정해 보려고 한다.

해당 블로그에서 중요한 부분은 이 부분인듯 하다.

```sh
yarn add twin.macro babel-loader babel-plugin-macros @babel/plugin-syntax-typescript @babel/preset-react
```

> `withTwin` 함수를 분석해 본다.

```js
// withTwin
const path = require('path') // path 를 가져온다.

const includedDirs = [
  path.resolve(__dirname, 'components'),
  path.resolve(__dirname, 'pages'),
  path.resolve(__dirname, 'styles'),
] // 포함할 dir 들의 경로를 배열에 담는다.

module.exports = function withTwin(nextConfig) {
  // nextConfig 를 감싸서 실행시킬 wrapper 함수를 작성한다.
  return {
    ...nextConfig, // 인자로 받은 nextConfig 를 전개구문을 사용하여 할당한다.
    webpack(config, options) {
      // webpack 설정을 위한 함수를 작성한다.
      // config 는 설정관련 부분을, options 는 nextjs 의 webpack 설정에 2번째 인자로 들어가는 Object 이다.
      // options object 는 다음의 properties 를 갖는다.
      // buildId, dev, isServer, nextRuntime, defaultLoaders, webpack
      // 자세한 설명은 docs 를 보도록 한다.
      const { dev, isServer } = options // options 에서  dev, isServer 를 구조분해할당한다.
      // dev 는 개발단계에서 컴파일되는지 확인하는 boolean 값이다.
      // isServer 는 client 가 아닌 server 에서 실행되는지 확인하는 boolean 값이다.
      config.module = config.module || {}
      // config.module 은 기존의 config.module 이 있으면 그대로 사용하고, 없으면 {} 을 사용한다.
      config.module.rules = config.module.rules || []
      // config.module.rules 은 기존의 config.module.rules 가 있으면 그대로 사용하고, 없으면 [] 을 사용한다.
      config.module.rules.push({
        // webpack rules 를 추가한다.
        // webpack 의 rules 는 배열이므로, push 를 사용하여 추가한다.
        test: /\.(tsx|ts)$/,
        include: includedDirs, // rules 를 사용할 dir path 를 배열로 넣는다
        use: [
          options.defaultLoaders.babel,
          // nextjs 에서 기본적으로 사용할 loader 를 babel 로 한다.
          {
            loader: 'babel-loader', // babel loader 를 구성한다.
            options: {
              sourceMaps: dev, // dev 환경에서만 sourceMap을 사용한다.
              presets: [
                // babel preset 설정
                [
                  '@babel/preset-react',
                  { runtime: 'automatic', importSource: '@emotion/react' },
                  // runtime 으로 automatic 을 사용한다.
                  // automatic 은 React New JSX transfrom 을 통해 자동적으로 JSX 를 변환시켜주는 기능이다
                  // 이는 `import React from 'react'` 구문을 생략해도 되고,
                  // 기존의 `React.createElement` 방식을 `import {jsx as _jsx} from 'react/jsx-runtime'`
                  // 으로 변경하면서, build 시점에 자동적으로 주입시켜주는 방식으로 변경되었다.

                  // JSX 컴파일러를 @emotion/react 의 JSX 컴파일러로 변경한다.
                  // emotion 내부의 JSX 는 props 로 `css`가 포함되면, emotionElement 를 포함한
                  // JSX 로 변경하지만, 그렇지 않다면 기본 JSX 컴파일러를 사용한다.
                ],
              ],
              plugins: [
                require.resolve('babel-plugin-macros'),
                // plugins 로 bable-plugin-macros 를 사용한다.
                require.resolve('@emotion/babel-plugin'),
                // emotion babel plugin 을 사용한다.
                [
                  require.resolve('@babel/plugin-syntax-typescript'),
                  // typescript plugin 을 설정한다.
                  { isTSX: true }, // TSX 를 true 로 설정
                ],
              ],
            },
          },
        ],
      })

      if (!isServer) {
        // server 에서 동작하지 않을때,
        config.resolve.fallback = {
          ...(config.resolve.fallback || {}),
          // 모듈 resolve 가 실패하면 polyfill 실행할 수 있도록 모아놓은 Object 이다.
          // 해당 Object 가 있다면 사용하고, 없으면 {} 을 사용한다.

          // webpack5 에서는 더 이상 Node.js 핵심 모듈을 자동으로 폴리필하지 않는다.
          // 그러므로, client 에서는 Node.js 에서 사용하는 module 사용이 불가하다.
          // 만약 원한다면, npm 에서 호환되는 모듈을 설치해야 하지만,
          // 현 설정에서는 필요 없는듯하다.
          fs: false,
          module: false,
          path: false,
          os: false,
          crypto: false,
        }
      }

      if (typeof nextConfig.webpack === 'function') {
        // 만약 function 으로 config 가 되어 있다면,,
        return nextConfig.webpack(config, options)
        // 설정한 config 와 options 를 인자값으로 넘겨준다
      } else {
        return config
        // 아니면 config 를 반환한다.
      }
    },
  }
}
```

> 나중에 시간되면 `webpack` 은 다시한번 살펴봐야 겠다.

이렇게 만들어진 `withTwin` 을 `next.config.js` 에 `wrapping` 처리한다.

```js
// next.config.js

const withTwin = require('./withTwin')

const nextConfig = withTwin({
  // <<- `withTwin` 함수 적용
  reactStrictMode: true,
  swcMinify: true, // terser 대신 swc 의 minification 을 사용한다.
  // docs 에서는 Terser 보다 7배 더 빠르다고 소개하고 있다.
})

module.exports = nextConfig
```

## 마무리

해당 블로그에서 굉장히 친절하게 설명 되어 있으며, 참고한 `reference` 들도 같이 제공하고 있다.
이제 이러한 설정을 기반으로 `setting` 해보면서, `tailwindcss` `docs` 와 함께 적용해 보려 한다.

이후 추가적인 사항이 생긴다면 더 적을 예정이다.
이쪽 세계를 알면 알수록 재미있으면서, 알아야 할것들이 많구나!
