---
title: 'react-hook-from 에 대한 고찰'
date: '2023-04-15'
tags: ['react-hook-form', 'next.js']
draft: false
summary: 'react-hook-from 을 왜 사용하며, 사용하면서 알아야할 개념들'
---

# react-hook-form 에 대한 고찰

react-hook-form 을 사용하면서 그저 `uncontrolled component` 이다. 정도로만 사용했다.  
사실, 그보다는 `hooks` 로 빼서 처리할 수 있다는 굉장함에 많이 사용했던것 같다.  

`react-hook-form` 이라는 라이브러리를 사용하면서 느낀 편리함보다,  
이러한 라이브러리가 왜 만들어 졌으며, 이로인해 얻을수 있는 이득은 무엇일까 라는  
의문이 들었다.

이는 내가 나름 찾아보고, `react-hook-form(input)` 을 어떻게 사용해야 더 잘 사용할지에 대해 정리한 내용이다.

## uncontrolled component

> `Less code, More performant`

`react-hook-form docs` 에 제일먼저 등장하는 문구이다.  
그러면서 다음과 같은 설명이 나온다.

> `react-hook-form`은 불필요한 `re-render`을 제거하면서, 코드의 양을 줄여준다.

이러한 방식을 가능하게 해주는 것이 바로 `uncontrolled component` 이다.

### controlled component 와 uncontrolled component

`controlled component` 는 일반적으로 `useState` 를 사용하여,  
`react` 자체에서 `state` 를 `controll` 할 수 있는 `component` 를 말한다.

> controlled component 의 예

```tsx

interface IValues {
  email: string
  password: string
}

const testForm = () => {
  const [values, setValues] = useState<IValues>({
    email: '',
    password: '',
  })

  const onSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    alert(values)
  }

  const onChangeValues = (e: ChangeEvent<HTMLInputElement>) => {
    setValues(prev => ({
      ...prev,
      [e.target.name]: e.target.value
    }))
  }

  return (
    <form onSubmit={onSubmit}>
      <label>
      email:
      <input name="email" value={values.email} onChange={onChangeValues}/>
      </label>
      <label>
      password:
      <input type="password" name="password" value={values.password} onChange={onChangeValues}/>
      </label>
      <button>완료</button> 
    </form>
  )
}

```

그렇다면, `uncontrolled component` 란 `state` 로 작동을 안한다는 말인가?  
맞다. `uncontrolled component` 는 말 그대로 `react` 에서 `controll` 하지 않는다.  

`uncontrolled component` 는 `DOM` 에 접근하여 값을 받아온다.

> uncontrolled component

```tsx

interface IValues {
  email: string
  password: string
}

const testForm = () => {
  const emailRef = useRef<HTMLInputElement>(null)
  const passwordRef = useRef<HTMLInputElement>(null)

  const onSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    const emailValue = emailRef.current.value
    const passwordValue = passwordRef.current.value
    alert(`emailRef: ${emailValue}, passwordRef: ${passwordValue}`)
  }

  return (
    <form onSubmit={onSubmit}>
      <label>
      email:
      <input name="email"  ref={emailRef}/>
      </label>
      <label>
      password:
      <input type="password" name="password" ref={passwordRef}/>
      </label>
      <button>완료</button> 
    </form>
  )
}
```

위의 `code` 를 보면 알겠지만, `DOM` 에 접근하여 처리된 값을 받아오기 위해 `ref` 를 사용해서,  
값을 받아오는것을 볼 수 있다.

근데, `DOM` 에서 받아오는것(`uncontrolled`)과, `state` 를 통해 처리(`controlled`)하는것과 무엇이 다른데?

이를 명확하게 이해하기 위해서는 `State Colocation` 에 대한 이해가 필요하다.

### State Colocation(상태 공존)

> 원글은 [state-colocation-will-make-your-react-app-faster]( https://kentcdodds.com/blog/state-colocation-will-make-your-react-app-faster) 에서 참고하였다.  

`State Colocation` 은 `Global State` 가 얼마나 좋지 않은지 알려주며,  
`Application` 이 더 빠르게 작동할 수 있도록 해주는 방법이다.

해당 글을 본다면, `State Colocation` 개념은 은 `RE-Render` 되는 `react` 에서 얼마나  
중요한지 설명해준다.

다음을 보도록 하자.  

```tsx

const SlowComponent = ({value, onChange}: {value: string, onChange: (e: ChangeEvent<HTMLInputElement>: void)}) => {
  ...오래걸리는 작업
  return (
    <label>
    오래걸리는 작업: 
    <input type="text" value={value} onChange={onChange}/>
    </label>
  )
}

const TestComponent = ({slow, value, onChange}: {slow: string, value: string, onChange: (e: ChangeEvent<HTMLInputElement>: void)}) => {
  return (
    <label>
    오래걸리는 SlowComponent 의 slow 값을 사용하는 Input: 
    <input type="text" value={value} onChange={onChange}/>
    <span>`slow값은 ${slow}에요.`</span>
    </label>
  )
}

const AppComponent = () => {
  const [value, setValue] = useState('')
  const [slow, setSlow] = useState('느리다')

  const onChangeValue = (e: ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value)
  }

  const onChangeSlow = (e: ChangeEvent<HTMLInputElement>) => {
    setSlow(e.target.value)
  }

  return (
    <TestComponent 
      slow={slow}
      value={value}
      onChange={onChangeValue} 
    />
    <SlowComponent 
      value={slow}
      onChange={onChangeSlow} 
    />
  )
}

```

위 내용은 해당 `blog` 의 내용을 참고해서 이해한 내용을 정리한 `code` 이다.

`SlowComponent` 는 어떠한 작업이 오래걸리는 `component` 를 말한다.

여기서 중요한것은 `SlowComponent` 에서 변경하는 `slow` 값을 `TextComponent` 에서  
`props` 로 전달받아 사용한다는 것이다.

만약, `SlowComponent` 가 `onChangeSlow` 를 실행하는데 `1sec` 이 걸린다고 가정해보자.
그리고 `TestComponent` 의 `input` 에 글을 작성한다고 가정해보자.

`TestComponent` 의 `input` 은 곧바로 잘 작성될까?  
아니면, `1sec` 이후에 `input` 이 작성될까?

결과는, `1sec` 이후에 `input value` 가 작성된다.

이는 굉장히 재미있는 상황이다.  
`TestComponent` 는 `SlowComponent` 의 `오래걸리는 작업` 의 영향을 받아서, `value` 작성이 `오래걸리는 작업(1sec)` 만큼 시간이 걸리는 상황이 발생한 것이다.

이는, `react` 의 `re-render` 와 연관있다.

`react` 는 `Component re-rendering` 하는 조건이 몇가지가 존재한다  

---

1. `State` 가 변경될때
2. 부모 컴포넌트가 다시 랜더링될때
3. 새로운 `Props` 가 들어올때
4. `Props` 가 변경될때

---

지금 상황은 각 `Component` 가 부모 `Component` 에 존재하는 `state` 값을 변경한다.

그렇다는건,`오래걸리는 작업(1sec)` 을 가진 `SlowComponent` 도 `input` 의 `state`가  
변경될때 마다 `re-rendering` 된다는 것이다.

`input` 하나 변경하겠다고, 모든 `Component` 가 다시 그려지는, 굉장히 비효율적인  
상황이 발생한 것이다.

그래서 위의 참고한 `Blog` 에서는 이에 대한 해결책으로 `State Colocation` 을 강조하며,  
컴포넌트단위로 `state` 를 생성하여, `re-rendering` 되는 현상을 없애고, 효율적으로 관리하라고 설명해준다.

이는 다음과 같다.

```tsx

const SlowComponent = ({value, onChange}: {value: string, onChange: (e: ChangeEvent<HTMLInputElement>: void)}) => {
  ...오래걸리는 작업
  return (
    <label>
    오래걸리는 작업: 
    <input type="text" value={value} onChange={onChange}/>
    </label>
  )
}

const TestComponent = ({slow}: {slow: string}) => {
  const [value, setValue] = useState('')

  const onChangeValue = (e: ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value)
  }

  return (
    <label>
    오래걸리는 SlowComponent 의 slow 값을 사용하는 Input: 
    <input type="text" value={value} onChange={onChange}/>
    <span>`slow값은 ${slow}에요.`</span>
    </label>
  )
}

const AppComponent = () => {
  const [slow, setSlow] = useState('느리다')

  const onChangeSlow = (e: ChangeEvent<HTMLInputElement>) => {
    setSlow(e.target.value)
  }

  return (
    <TestComponent 
      slow={slow}
      value={value}
      onChange={onChangeValue} 
    />
    <SlowComponent 
      value={slow}
      onChange={onChangeSlow} 
    />
  )
}

```

위의 코드는`TestComponent` 의 `state` 값이 변경될때, `re-rendering` 되지 않도록  
`TestComponent` 내부에 `state` 를 선언하여 처리하는 `logic` 이다.

이제, `TestComponent` 의 `Input` 값 변경시, 부모 컴포넌트에 위해 전체 `re-rendering` 되는
상황은 없어졌다.

이것이 바로, `State Colocation` 의 장점이다.

### 그래서, Uncontrolled Component 와 무슨 상관인데?

다시, `Controlled Component` 의 `Code` 로 되돌아 가보자.

> controlled component 의 예

```tsx

interface IValues {
  email: string
  password: string
}

const testForm = () => {
  const [values, setValues] = useState<IValues>({
    email: '',
    password: '',
  })

  const onSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    alert(values)
  }

  const onChangeValues = (e: ChangeEvent<HTMLInputElement>) => {
    setValues(prev => ({
      ...prev,
      [e.target.name]: e.target.value
    }))
  }

  return (
    <form onSubmit={onSubmit}>
      <label>
      email:
      <input name="email" value={values.email} onChange={onChangeValues}/>
      </label>
      <label>
      password:
      <input type="password" name="password" value={values.password} onChange={onChangeValues}/>
      </label>
      <button>완료</button> 
    </form>
  )
}

```

이 `code` 의 문제점은 `State Colocation` 을 설명하면서 발생한 문제와 같다.

`Input` 하나 변경하면, `State` 값이 변경되므로, 해당 컴포넌트가 전체 `Re-Rendering` 되는 상황이다.

물론, `Input` 이 몇개 되지 않는다면, 별 문제가 없겠지만, 대형 서비스 같은경우에는  
`input` 사용이 `1000개` 가까이 이루어진다는 글을 본적이 있다.

이러한 상황이면 `1000개`가 `re-rendering` 된다는 것이다.

만약, `State` 를 통한 `Input` 조작이 아니라, `DOM` 을 통해 접근(`Uncontrolled Component`)한다면,  
`State` 를 통하지 않았으므로 이러한 쓸데없는 `Re-rendering` 이 발생안하지 않을까?

바로 이러한 관점에서 `react-hook-form` 은 `uncontrolled Component` 를 통해,  
`input` 값을 수정하는 방식을 사용한다.

이를 말해주는 부분은 `Docs` 에서 잘 보여주고 있다.

![react-hook-form rerendering](/images/2023/04/re-render.gif)

`Controlled Component` 일때, 각 `Component` 의 움직임이 남다르다

이를 통해 `react-hook-form` 이 왜 `Uncontrolled Compoennt` 방식을 사용하여,  
`Input` 을 처리하는지에 대한 개념을 알게 되었다.

### 하지만, `Controlled Component` 를 사용해야만 할때가 있다면...?  

예를 들어, `React-Select`, `MUI`, `AntD` 같은, 미리 작성된 `UI Component` 를 가져올때 `controlled component` 로 작성되어 있다면, 어떻게 처리해야할까?

`react-hook-form` 은 이러한 미리 작성된 `Controlled Component` 역시 같이 관리가능하도록 해당 `API` 를 제공한다.

### Controller

`Controller` 는 `controlled component` 를 `wrapping` 해서, `react-hook-form` 에서 같이 관리할 수 있도록 만들어주는 `api` 이다.

`docs` 에서는 이 `api` 를 다음과 같이 사용하라고 말한다.

```tsx
<Controller
  control={control}
  name="test"
  render={({
    field: { onChange, onBlur, value, name, ref },
    fieldState: { invalid, isTouched, isDirty, error },
    formState,
  }) => (
    <Checkbox
      onBlur={onBlur} // notify when input is touched
      onChange={onChange} // send value to hook form
      checked={value}
      inputRef={ref}
    />
  )}
/>
```

각 `Props` 에 대해서는 다음의 [Docs](https://react-hook-form.com/api/usecontroller/controller/) 를 참고하자.

여기서 중요한 부분은, `control` 과 `render` 부분이다.

`control` 은 `docs` 에서 다음처럼 설명하고 있다.
> 이 `Object` 는 `Component` 를 `react-hook-form` 에 등록하기 위한 메서드를 포함하고 있다.

`control` 을 불러올때, `useForm` 을 사용하여 가져오는데, 해당 `type` 을 살펴보니, 정말 내부적으로 사용하는 `code` 가 가득했다.

```tsx
export type Control<TFieldValues extends FieldValues = FieldValues, TContext = any> = {
    _subjects: Subjects<TFieldValues>;
    _removeUnmounted: Noop;
    _names: Names;
    _state: {
        mount: boolean;
        action: boolean;
        watch: boolean;
    };
    _reset: UseFormReset<TFieldValues>;
    _options: UseFormProps<TFieldValues, TContext>;
    _getDirty: GetIsDirty;
    _resetDefaultValues: Noop;
    _formState: FormState<TFieldValues>;
    _updateValid: (shouldUpdateValid?: boolean) => void;
    _updateFormState: (formState: Partial<FormState<TFieldValues>>) => void;
    _fields: FieldRefs;
    _formValues: FieldValues;
    _proxyFormState: ReadFormState;
    _defaultValues: Partial<DefaultValues<TFieldValues>>;
    _getWatch: WatchInternal<TFieldValues>;
    _updateFieldArray: BatchFieldArrayUpdate;
    _getFieldArray: <TFieldArrayValues>(name: InternalFieldName) => Partial<TFieldArrayValues>[];
    _executeSchema: (names: InternalFieldName[]) => Promise<{
        errors: FieldErrors;
    }>;
    register: UseFormRegister<TFieldValues>;
    unregister: UseFormUnregister<TFieldValues>;
    getFieldState: UseFormGetFieldState<TFieldValues>;
};
```

`control` 객체는 `react-hook-from` 에서 입력 필드를 추적하고 관리하는데 사용된다.  

`render` 부분은 `React` 에서 제공하는 `render prop` 을 사용하여 처리한다.
이는 비표준 `props` 를 가진 외부 `controlled component` 와의 통합을 단순하게 만들어준다.

즉, `render` 에 사용되는 자식 컴포넌트에 `onChange`, `onBlur`, `name`, `ref`, `value` 를 제공한다.

`render` 시 제공되는 `props` 는 다음의 순서를 갖는다.

> `field`, `fieldState`, `formState`

이 값들을 가진 객체를 통해서 `controleld component` 관리가 이루어진다.

그렇다면, `Controller` 를 `hooks` 로 뺄수는 없을까?

### useController

`useController` 는 `react hook form` 에서 정고하는 `hooks` 중 하나이다.
이는 `controlled component` 를 관리하기 위해 사용된다.

간단하게 `controller` 의 `hooks` 버전으로 생각하면 된다.

`api` 에 대해서는 다음의 [useController Docs](https://legacy.react-hook-form.com/v6/api/#useController) 에서 살펴보자

`code` 는 이러하다.

```tsx
import React from "react";
import { TextField } from "@material-ui/core";
import { useController, control } from "react-hook-form";

function Input({ control, name }) {
  const {
    field: { ref, ...inputProps },
    fieldState: { invalid, isTouched, isDirty },
    formState: { touchedFileds, dirtyFileds }
  } = useController({
    name,
    control,
    rules: { required: true },
    defaultValue: "",
  });

  return <TextField {...inputProps} inputRef={ref} />;
}

function App() {
  const { control } = useForm();
  
  return <Input name="firstName" control={control} />;
}
```

이는 `hooks` 로 변경되었을 뿐 `Contorller` 와 동일한 `api` 를 가진다.

### 마무리

`react-hook-form` 을 왜 사용하는지 알기 위해
`uncontrolled component` 와 `controlled compoennt` 에 대해서 알아보았으며, 
그로인해 `State Colocation` 이 어떻게 발생하는지도 알아보았다.

이러한, 지식을 가지고 접근한다면, 추후 `react` 를 사용하여 `application` 을 만들때,  
많은 도움이 될것으로 생각이 든다.

그저, `많이 사용하니까` 가 아닌 `왜 사용하는지`  를 아는 중요한 계기가 된것 같다.

앞으로 `library` 에 대해서 더 많이 알아가면서, 이러한 개념적 토대를 기반으로 한다면,  
`library` 가 만들어지게된 이유에 맞추어 `code` 를 작성하기 좋을 것이다.

`react-hook-form` 은 정말 멋진 `libray` 이다.
