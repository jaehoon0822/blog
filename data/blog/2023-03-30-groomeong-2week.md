---
title: 'Groomeong 2week 회고'
date: '2023-03-30'
tags: ['project', 'groomeong', '회고']
draft: false
summary: 'Project 진행하며 2주 동안의 회고'
---

# Groomeong 2주차 회고

> 이번 `Project` 를 진행하면서 느낀 회고를 작성해본다.

기존의 `Atomic Design` 을 토대로 각 컴포넌트들을 나누고 쪼개서 작업하기 위해 생각을 많이 하였다.

각 컴포넌트는 재사용의 목적에 맞추어 작업을 해야만 했다.
`재사용` 목적을 위해 만들어진 `Component` 중, `Input` 을 공통컴포넌트로 만드는 작업을 주로하였다.

`Input` 구현은 `React-hooks-form` 을 사용하여 만들어야 하므로, 일반적인 `useState` 를 사용하여 만드는 방식과는 차이가 있다.

다행히도, `React-hooks-form` 은 명칭 그대로 `Hooks` 로써 사용가능하기에, `Atomic Desing` 에 맞추어 `Component` 로 쪼갤수 있다.

`Input` 컴포넌트를 만들기 위해서는 다음의 조건이 성립되어야 한다.

1. `Input` 사용시 어떠한 값이라도 받을 수 있도록 해야 한다.
2. `Input` 은 `Type` 을 받으며, 받은 `type` 이 없다면 `default` 값으로 `text` 이어야 한다.
3. `Input` 은 `React-hooks-form register` 처리를 위해 반드시 `name` 을 받아야 한다.
4. `Placeholder` 를 받아 처리할 수 있어야 한다.
5. 각 `State` 마다 다른 `Input` 을 보여주어야 한다.
6. `Input` 은 `Label` 을 통해 어떠한 `Input` 인지 나타내야 한다.

이후 값들이 더 추가한 부분이 있지만, 최소한 위의 조건을 사용하여 공통적으로 사용가능하도록 만들어야 한다.

재사용의 목적은 간단한 사용에 있다.
그러므로 해당 `Input` 은 마치 `Element input` 처럼 규칙이 비슷하며, 다른 설정이 많이 필요없게 만드는 것이 중요했다.

## FormProvider

`React-hooks-form` 에서 제공하는 `FromProvider` 는 `Hooks` 로써 편리한 방법을 제공한다.

`FormProvider` 는 `Props` 로 `useForm` 을 실행시킨후 반환되는 객체를 받는다.

이러한 `FormProvider` 안에 `form` 을 사용한후, `useFrom` 을 통해 반환된 객체에서 `handleSubmit` 을 가져와 `onSubmit Event Handler` 를 등록해준다.

그후 재사용 목적에 의해 만들어진 `Input` 을 넣어주기만하면, 바로 사용가능하도록 만들었다.

이 `code` 는 대략적으로 처럼 생겨먹었다.

> 대략적인 회원가입 form

```ts
const method = useForm<IMutationCreateUserArgs>({
  mode: 'onChange',
  resolver: yupResolver(Schema),
})

;<FormProvider {...method}>
  <SignUpForm onSubmit={method.handleSubmit(withPromiseVoidFunc(onClickSignUp))}>
    <SignUpValidateInput setValid={setValid} />
    <InputMiddle label="닉네임" name="name" placeholder="닉네임을 입력해주세요." />
    <InputMiddle
      label="비밀번호"
      name="password"
      placeholder="비밀번호를 입력해주세요."
      type="password"
    />
    <Buttons
      label="회원가입하기"
      type="submit"
      size="large"
      border="none"
      state={valid ? undefined : 'disabled'}
      variation="primary"
    />
  </SignUpForm>
</FormProvider>
```

이는 대략적인 틀로써 만들어졌으며, 이를 구현한 `Input` 은 대략 다음과 같다.

> 재사용 가능한 Input

```ts
const {
  register,
  formState: { errors },
} = useFormContext()

return (
  <S.InputWrapper onFocus={onFocusTest} onBlur={onBlurTest}>
    <S.Label focus={focus} error={errors[props.name]?.message as string}>
      {props.label}
    </S.Label>
    <S.InputTag
      type={props.type ?? 'text'}
      placeholder={props.placeholder}
      error={errors[props.name]?.message as string}
      {...register(props.name)}
      defaultValue={props.defaultValue}
      maxLength={props.maxLength ?? undefined}
    />
    <S.Error>{errors[props.name]?.message}</S.Error>
  </S.InputWrapper>
)
```

위의 `InputTag` 은 `emotion` 을 사용하여 구현했으며, 이는 팀원이 만든 `style` 에 `react-hooks-form` 을 연결시켜준 예이다.

위의 `code` 에서 중요한것은 `useFormContext` 를 사용하여, `register` 및 `formState` 를 받아 처리했다는 것이다.

`useFormContext` 는 오직 `FormProvider` 로 감싸여 있는 `Input` 에서 `Hooks` 로 받아 처리 가능하도록 되어 있다.

이는 `Input` 을 통해 각 `Props` 를 받아 따로 처리가능하도록 만들 수 있는 장점이 있다.

위의 `회워가입 Form` 에서는 만들어진 `Input` 컴포넌트 하나만을 사용하여 모든 `Input` 을 구현했다.

이는 매우 효과적이고 편리한 방법이다.
심지어 `Error` 처리까지 알아서 처리해준다.

이번 처리를 하면서 조금 시간이 걸렸던 `Logic` 은 `Input` 을 통한 `Image` 연결이다.

이는 `React-hooks-form` 을 사용하여 구현해야 했기에, 기존의 `setState` 처리로는 하고 싶지 않았다.

즉 `button` 을 누르면 `onSumit` 함수의 `data` 를 통해 `image` 처리가 가능하도록 만들어야 한다는 것이다.

이에 대한 `imageInput` 은 다음과 같다.

```ts
const { ImgInputRef, ImgBoxRef, img, register, onClickImgInput, onChangeInput } = useImgInput()

const { ref, ...rest } = register(props.name, {
  onChange: withPromiseVoidFunc(onChangeInput(props.mutationFunc, props.shopId)),
})

return (
  <ImgInputWrapper>
    <Input
      type="file"
      {...rest}
      ref={(e) => {
        ref(e)
        ImgInputRef.current = e
      }}
    />
    <ImgDiv url={img} ref={ImgBoxRef} onClick={onClickImgInput} />
  </ImgInputWrapper>
)
```

위의 상황은, `input` 과 `imageBox` 를 서로 연결해주어야만 했다.
`imageBox` 를 누르면, `input` 이 작동되도록 만들어야 한다는것이다.

이러한 처리르 위해 `ref` 로 서로 연결하여, 처리한다.
`useImgInput` 에는 각 `inputRef` 와 `imgBoxRef` 를 만들었으며, 각 `ref` 를 가져와, `onClickImgInput` 함수가 작동되면 `input` 이 `click` 되게 된다.

`click` 된 `input` 은 `file type` 을 가지므로, `image` 를 받을 수 있다.

받은 `file` 을 `onChange` 함수를 사용하여 처리를 해야 하므로, `register` 에서 `onChange` 함수를 지정하고, `input` 에서 사용가능하도록, `rest` 객체를 `Input` 에 `Props` 로 넘긴다.

> `rest` 객체는 `register` 에서 `ref` 를 제외하고 반환된 객체이다..

이렇게 처리하면, `imgBox` 클릭시 `file upload` 가 가능하며, `file` 을 선택한다면, `input` 의 `onChange` 가 작동하여, 알아서 처리 로직이 실행된다.

## 공통컴포넌트를 만들때는 많은것들이 고려되어야 한다.

이번 `Project` 를 하며 느낀것은, 공통컴포넌트는 앞으로 어떻게 사용될지 를 미리 예상하고, 코드를 짜야 한다는 것이다.

그렇지 않다면, 지속적인 컴포넌트 수정이 들어갈 수 있으며, 오히려 좋은 효율을 보기가 어려울 수 있다.

이는 많은 경험에서 우러나올 수 있다는 생각이 드므로, 항상 같은 로직을 만들지 말고, 재사용을 위한 컴포넌트를 만드는 습관을 들이는 것이 좋을 것 같다.

공통컴포넌트를 만들며, 많은 수정작업도 같이 진행되었지만 한가지 느끼는것은 한번 만들어 놓으면 굉장히 편하다는 것이다.

또한, 만약 컴포넌트의 변경사항이 생길수 있다.
이때, 각 컴포넌트에 들어가 일일히 수정보는 것이 아닌 하나의 컴포넌트만 수정하면 나머지 컴포넌트들이 전부 수정된다.

그것 뿐만 아닌, 코드의 양도 획기적으로 줄어든다.
유지보수의 기적과도 같아보인다

공통컴포넌트를 분리하면서, 추가적인 사항이 지속생겨 수정에 수정을 거듭해왔지만, 이는 굉장히 좋은 경험이라는 생각이다.
