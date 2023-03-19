---
title: 'Groomeong 1week 회고'
date: '2023-03-19'
tags: ['project', 'groomeong', 'design', '회고']
draft: false
summary: 'Project 진행하며 1주 동안의 회고'
---

# Groomeong 1주차 회고

> Groomeong Service <br/>
> 강아지 미용실 `Gather`

강아지 미용실을 찾기 위한 서비스이다. <br/>
이러한 서비스를 생각하기 위한 이유는 다음과 같다

### 기존 시장의 문제점 및 개선 필요성

강아지 미용샵 검색시, 소비자가 원하는 정보보다 `홍보 사이트`가 대부분이다.<br/>
그러므로, 해당 서비스 검색을 위한 `커뮤니티` 검색이 필수적이다.

이러한 부분을 파고들어 `강아지 미용실` 을 찾을 수 있는 전용 서비스를 만들어 보았다.

## 사용자 요구사항 정의

> 요구사항으로 `회원` 및`관리자` 서비스가 필요했다.<br/>
> 현 서비스는 `관리자` 가 아닌 `회원` 을 위한 서비스를 준비한다.

회원

> `회원가입` `로그인` `로그아웃` `마이페이지`

반려견

> `반려견 등록` `반려견 삭제` `반려견 페이지`

상점 Detail Page

> `상점 Detail`

리뷰

> `리뷰작성`

예약

> `날짜` `시간` `시간예약`

지도

> `미용샵 검색` `미용샵 선택`

## Flow Chart 작성이후 디자인 작업

`Front End` 에서 작업이전에 어떠한 방법을 통해 `Design` 작업이 들어갈지 상의를 했다.
이전부터 `Atomic Design System` 에 관심이 있어왔고, `React` 에서 재사용성을 위해 이러한 시스템을 사용하도록 결졍했다.

> 현제 해당 `Service` 를 구현하기 위해서는 `Designer` 없이 진행되므로, 직접 `Design` 작업이 들어가야만 하는 상황이 되었다.

### Logo Design

`그루멍` 의 이름은 `고양이의 그루밍` 에서 따왔다.
`강아지 미용샵` 과 자신의 몸을 가꾸는 `그루밍` 은 잘 어울리는 조합이다.
거기에 `개` 를 뜻하는 `멍` 을 붙혀 `Naming` 되었다.

이러한 `강아지 미용샵` 과 `털` 그리고 `고양이의 그루밍` 을 조합할때 이 3가지의 공통점은 바로 `털` 이었다.

`고양이는 그루밍` 을 한후 `털 뭉치` 를 내뱉으며, 반려견 미용시 뭉치는 `털` 이 같이 연상되었다.

만들어진 `Logo` 는 다음과 같다.

![Logo](/public/static/images/2023-03/Logo-120-120.svg)

### Primary Color 선정

`그루멍` 은 `예약시스템` 과 함께 고객들께 미용샵을 제공해 드리는 `Gather` 의 특징을 갖고 있다.

이러한 부분은 `고객의 신뢰` 가 밑바탕이 되어야 한다.
`Blue` 는 심리학적으로 `평온함과 평화로움의 상징` 으로 대표된다.

이는 `신뢰` 와 `안전`, `관리` 와 연관이 깊은 색상으로, 단연 `Blue` 가 제격이라는 생각이 들었다.

그러므로 우리가 쓰는 `Primary` 색상은 `Blue` 가 된다.

### Token

가장 먼저 `Design Token` 을 통해 가장 작은 단위로 생각되는 `Typograph`, `Color`, `Spacing` 를 지정하였다.

지정된 `Token` 은 다음과 같다.

> Token / Heading
> ![Token/Heading](/static/images/2023-03/Heading.png)

> Token / Label
> ![Token/Label](/static/images/2023-03/Label.png)

> Token / Paragraph
> ![Token/Paragraph](/static/images/2023-03/Paragraph.png)

> Token / Space
> ![Token/Space](/static/images/2023-03/Space.png)

> Token / Color
> ![Token/Color](/static/images/2023-03/Color.png) >

이렇게 만든 `Token` 은 기본적으로 이 `Service` 를 구축하는데 만들어질 가장 기본적인 틀이다.

`Programing` 언어에서 비교하자면 `Primary Type` 과 같은 역할을 한다.

> `Token` 을 설정하면서, `Grid` 를 추가적으로 넣었지만, 구현시 `Mobile` 까지 가능한지는 조금더 상의를 해보아야 할 것같다.<br/>
> 현재 `Service` 에 들어갈 `Stack` 들을 같이 공부하며 정해진 기간안에 완성해야 하므로, 시간이 허락한다면 추가적으로 만들 생각이다.

이제 `Token` 을 지정했다면, 다음은 `Atomic Design` 을 정할 차례이다.

## Atomic Design: 재사용 가능한 컴포넌트 지정하기

`Figma` 를 사용하여 재사용 가능한 컴포넌트 먼저 정하면서 작업이 들어갔다.
먼저 `Wire Frame` 을 사용하여 어떠한 `Desing` 이 나올지, 대략적인 작업이 들어가게 되었다.

`Wire Frame` 이후 대략적인 틀이 완성되었다.
`Design Token` 생성후 재사용가능한 컴포넌트를 분리하기 시작했다.

Atomic Design 이란? <br/>
`Design` 시 `System` 적으로 만들 수 있도록 제시하는 방법론중 하나이다.<br/><br/>

> 이는 각 `Design System` 을 `화학` 을 통해 영감을 받은 `Brad Frost` 에 의해 만들어졌다.<br/>

이 이론을 간략하게 이야기 하면 다음과 같다<br/><br/>

`Design` 상 가장 작은 단위를 `Atoms(원자)` 로 구성한다.<br/>
`Atoms` 를 조합하여 `Molecules(분자)` 를 구성한다 <br/>
`Molecules(분자)` 를 조합하여 `Organisms(유기체)` 를 구성한다 <br/>
`Organisms(유기체)` 를 조합하여 `Template` 를 구성한다 <br/>
`Template` 를 조합하여 `Page` 를 구성한다 <br/>
`Page` 를 조합하여 `Web Service` 를 구성한다 <br/><br/>

현재 우리 `Service` 는 그렇게 큰 `Service` 가 아니므로, 쪼개는 상황에서 오히려 불편이 발생할 수 있다.

`Atoms` 를 `Molecules` 와 합쳐서 `Atoms` 라 명한다.
그런후 `Organisms` 로 바로 넘어가도록 단계 구성했다.

즉 다음과 같다.

```
`Atoms` ->  `Organisms` -> `Template` -> `Page`
```

이렇게 단계별로 구성한후 하나의 `page` 를 구성한다.
이는 `재사용 가능한 컴포넌트` 를 만드는데 굉장히 좋은 방법이라는 생각이 들었다.

다행이도, `Figma` 는 재사용 가능하도록 `Component` 화 시켜 처리가 가능하며, `Variables` 를 통해 각 상태 지정이 가능하다.

이러한 부분을 적극적으로 사용하여 `Design` 시 `Component` 단위로 구성해본다.

이렇게 구성된 `Atoms` 는 다음처럼 분류되었다.

---

`Button` `Input` `Modal` `SearchBar` `MapSideList / Card` `TopBar` `PageHeader` `Comments / Comment` `Comments / Header` `Comments / TextArea` `Rate` `Images` `Badge`

---

### Atoms / Buttons

![Atoms/Color](/static/images/2023-03/Atoms-Button.svg)

### Atoms / Input

![Atoms/Input](/static/images/2023-03/Atoms-Input.svg)

### Atoms / Modal

![Atoms/Modal](/static/images/2023-03/Atoms-Modal.svg)

### Atoms / SearchBar

![Atoms/SearchBar](/static/images/2023-03/Atoms-SearchBar.svg)

### Atoms / MapSideList

![Atoms/MapSideList](/static/images/2023-03/MapSideList.svg)

### Atoms / TextArea

![Atoms/TextArea](/static/images/2023-03/Atoms-TextArea.svg)

### Atoms / CommentsList

![Atoms/CommentsList](/static/images/2023-03/Atoms-CommentsList.svg)

### Atoms / Rate

![Atoms/Rate](/static/images/2023-03/Atoms-Rate.svg)

### Atoms / Images

![Atoms/Images](/static/images/2023-03/Atoms-Image.svg)

### Atoms / Badge

> `Badge` 를 `Radio` 방식으로 처리하는데 `의미론적으로` 맞지는 않다.
> 이부분은 상의를 통해서 `Badge` 를 `Radio Button` 처럼 작동하도록 미리 상의를 통해 처리한 부분이다.

![Atoms/Badge](/static/images/2023-03/Atoms-Badge.svg)

해당 `Componets` 는 [Components Figma](https://www.figma.com/file/JngYoKqeHWFb6bwwlCqfZR/GROOMEONG?node-id=0-1) 에서 확인가능하다

이렇게 만들어진 각 `Component` 를 재사용하여 `Template` 를 구성한다.

`Template` 역시 재구성할때 필요한 `Page` 영역을 구성한 것이므로, `Next.js` 사용시 `Dynamic Routing` 할때 사용할때 이러한 `Template` 를 사용하여 처리한다.

재사용되지 않는 `Page` 는 `Template` 를 거치지 않고, 바로 `Page` 로 구성할 것이다.

이러한 `Atoms` 를 사용하여, 만들어진 페이지는 [Figma](https://www.figma.com/file/JngYoKqeHWFb6bwwlCqfZR/GROOMEONG?node-id=33-3214) 에서 확인가능하다.

## 회고

이렇게 `Design` 작업을 지속 진행하면서 느낀것은, `Component` 의 분리를 더 정밀하게 했으면 좋지 않았을까 하는 아쉬움이다.

`Atomic Design System` 을 적용하기로 한 시점에서, 처음 적용해보고 부딪치며 어떻게 `Component` 화 시킬지 고민하며 작업은 하였지만, 여전히 부족한 부분이 많이 보인다.

`Badge` 를 `Radio Button` 으로 따로 `Button Component` 로 추가했어야 했던점 및 `Organisms` 로 전환되는 `Comments` 및 `MapSideList` 같은 부분을 좀더 편리하게 변환이 가능하지 않을까 하는 아쉬움이 있다.

또한, `Wire Frame` 으로 먼저 대략적인 부분을 지정한후 `Component` 를 만들었음에도 불구하고, `Programing` 적 관점이 아닌 `Design` 관점에서는 `Atomic System` 이 적절하지는 않다는 생각이 들었다.

`Programing` 관점에서는 이미 완성된 `Design` 을 토대로 만드므로 `Atomic Design` 적용이 가능하지만, 새로운 `Layout` 을 짜내고, `Design` 하는 (무 에서 유를 만들어내는...) 작업에서는 `Component` 단위로 만들고 `Page` 로 조합하는 과정에서 지속 새로운 `Component` 를 생성해 내야 하는 상황이 부딪친다.

이렇게 새롭게 만들어진 `Component` 는 예상치 못하게 발생한 상황에서 발생했다.

이러한 예상치 못한 `Component` 는 재사용성 좋게 만들기는 어려운 상황으로 번지기도 하며, 이 `Component` 를 추가하기 위해 기존의 만들어진 `Atoms` 를 물갈이 해야 하는 상황이 발생하기도 했다.

사실 `Badge` 형태의 `button` 을 만들게 되었는데, `Button Component` 에 추가하지 못했다.

또한, 지속적으로 새로운것이 추가되는 작업이 발생하게 되어, 좀더 세밀한 회의와 처리 프로세스가 진행되어야 함을 느끼게 된 부분이 크다.

여러 고난의 과정 끝에 `Design` 은 완성되었으며, `Component` 단위로 할당량을 정해 작업 분배가 되었다.

이렇게 만들어진 `Atoms` 를 적용하고, 처리 가능하도록 `React` 를 통해 짜 넣는과정을 만드는건 이번 `2 주차` 에 진행되는 과제이다.

`Backend API` 역시 작업중이므로, 현재로써 할 수 있는 부분은 `Atoms` 를 만들고 조합하여, `Front` 단에서 기능없는 `Page` 를 먼저 만드는것이 먼저이다.

그런 후 `API` 가 만들어지면, 이를통해 해당 `API` 로 처리가능한 서비스를 구현할 생각이다.

사실 이러한 부분이 앞으로의 걱정이기는 한데, 해당 부분은 부딪치면서 많은 부분에 대해 배워볼 수 있는 기회가 될것으로 예상되어 한편으로는 기대가 크다.

이러한 회고는 `2주차` 회고에 남기도록 하겠다.
