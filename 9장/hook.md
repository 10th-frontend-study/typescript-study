# Hook

## useState 사용하기

useState의 타입 정의는 다음과 같다.

```tsx
function useState<S>(
  initialState: S | (() => S)
): [S, Dispatch<SetStateAction<S>>];

type Dispatch<A> = (value: A) => void;
type SetStateAction<S> = S | ((prevState: S) => S);
```

- useState의 반환 튜플
  - 첫 번째 요소 : 제네릭으로 지정한 S타입
  - 두 번째 요소 : 상태를 업데이트할 수 있는 Dispatch 타입의 함수
    - (Dispatch 함수의 제네릭으로 지정한) SetStateAction에는 useState로 관리할 상태 타입인 S 또는 이전 상태 값을 받아 새로운 상태를 반환하는 함수인 (prevState: S) ⇒ S가 들어갈 수 있음

### useState에 타입스크립트를 사용하면

아래와 같은 문제 상황에서 컴파일타임에 미리 타입에러를 발견할 수 있다.

```tsx
import { useState } from "react";

const MemberList = () => {
  const [memberList, setMemberList] = useState([
    {
      name: "KingBaedal",
      age: 10,
    },
    {
      name: "MayBaedal",
      age: 9,
    },
  ]);

  // 🚨 addMember 함수를 호출하면 sumAge는 NaN이 된다
  const sumAge = memberList.reduce((sum, member) => sum + member.age, 0);

  const addMember = () => {
    setMemberList([
      ...memberList,
      {
        name: "DokgoBaedal",
        agee: 11, // 🚨 기존 memberList배열 요소에는 없는 잘못된 속성이 포함된 객체가 추가됨
      },
    ]);
  };
};
```

agee라는 memberList배열 요소에는 없는 잘못된 속성이 포함된 객체가 추가되어 NaN이 되는 예상치 못한 사이드 이펙트가 발생한다.

```tsx
import { useState } from "react";

interface Member {
  name: string;
  age: number;
}

const MemberList = () => {
  const [memberList, setMemberList] = useState<Member[]>([]);

  // member의 타입이 Member 타입으로 보장된다
  const sumAge = memberList.reduce((sum, member) => sum + member.age, 0);

  const addMember = () => {
  // 🚨 Error: Type ‘Member | { name: string; agee: number; }’
  // is not assignable to type ‘Member’
    setMemberList([
      ...memberList,
      {
        name: "DokgoBaedal",
        agee: 11,
      },
    ]);
  };

  return (
    // ...
  );
};
```

setHemberList의 호출 부분에서 추가하려는 새 객체의 타입을 확인하여 컴파일타임에 타입 에러를 발견할 수 있다.

<br/>

# 의존성 배열을 사용하는 훅

> **useEffect**와 **useLayoutEffect**, **useMemo**와 **useCallback**

## useEffect

```tsx
function useEffect(effect: EffectCallback, deps?: DependencyList): void;

type DependencyList = ReadonlyArray<any>;
type EffectCallback = () => void | Destructor;
```

- **effect: EffectCallback** - `Destructor`를 반환하거나 아무것도 반환하지 않는 함수
  - Promise 타입은 반환하지 않으므로 useEffect의 콜백 함수에는 비동기 함수가 들어갈 수 없다.
  - useEffect에서 비동기 함수를 호출할 수 있다면 경쟁 상태(Race Condition)를 불러일으킴
- **deps?: DependencyList** : effect가 수행되기 위한 조건 나열
  - deps 배열의 원소가 변경되면 실행된다.
  - deps의 원소로 숫자나 문자열 등 기본 자료형이 아닌 객체나 배열을 넣을 때는 주의 해야함
- 경쟁 상태란?
  멀티스레딩 환경에서 동시에 여러 프로세스나 스레드가 공유된 자원에 접근하려고 할 때 발생할 수 있는 문제다. 이러한 상황에서 실행 순서나 타이밍을 예측할 수 없게 되어 프로그램 동작이 원 하지 않는 방향으로 흐를 수 있다.
- `Destructor` 함수란?
  - 컴포넌트가 마운트 해제될 때 실행되는 함수, **클린업 함수**라고도 함
  - deps가 빈 배열이라면 useEffect의 콜백 함수는 컴포넌트가 처음 렌더링 될 때만 실행되고,
  - 이때의 Destructor는 컴포넌트가 마운트 해제될 때에만 실행됨
    - 그러나, deps 배열이 존재한다면 배열의 값이 변경될 때마다 Destructor가 실행됨
- 클린업(Cleanup) 함수
  **useEffect**나 **useLayoutEffect**와 같은 리액트 훅에서 사용되며, 컴포넌트가 해제되기 전에 정리 (clean up) 작업을 수행하기 위한 함수를 말한다.

**:: deps의 원소로 객체를 넣을 경우**

```tsx
type SomeObject = {
  name: string;
  id: string;
};

interface LabelProps {
  value: SomeObject;
}

const Label: React.FC<LabelProps> = ({ value }) => {
  useEffect(() => {
    // value.name과 value.id를 사용해서 작업한다
  }, [value]);

  // ...
};
```

- 부모에게서 받은 value인자를 직접 deps로 작성함
- 이는 원치 않는 렌더링을 반복시킬 수 있음
- 이를 방지하려면 실제로 사용하는 값을 deps에 작성해야 함 (아래 코드 참고)

```jsx
const { id, name } = value;

useEffect(() => {
  // value.name과 value.id 대신 name, id를 직접 사용한다
}, [id, name]);
```

<br/>

## useLayoutEffect

useLayoutEffect는 useEffect와 비슷한 역할을 하는 훅으로 타입 정의는 useEffect와 동일하다.

```tsx
type DependencyList = ReadonlyArray<any>;

function useLayoutEffect(effect: EffectCallback, deps?: DependencyList): void;
```

그렇다면 useLayoutEffect는 어떤 경우에 사용될까?

아래는 유저의 이름을 name이라는 state에서 받아와서 `‘안녕하세요 ${name}님’`을 실행시키는 코드이다.

```jsx
const [name, setName] = useState("");

useEffect(() => {
  // 매우 긴 시간이 흐른 뒤 아래의 setName()을 실행한다고 생각하자
  setName("배달이");
}, []);

return <div>{`안녕하세요, ${name}님!`}</div>;
```

위와 같은 코드를 실행하면 처음에는 `"안녕하세요, 님!"`으로 nane이 빈칸으로 렌더링된 후, 다시 "안녕하세요, 배달이님!"으로 변경되어 렌더링될 것이다.

만약 name을 지정하는 setName이 오랜 시간이 걸린 후에 실행된다면 사용자는 빈 이름을 오랫동안 보고 있어야 할 것이다.

이럴 때 **useLayoutEffect**를 사용할 수 있다.

<aside>

💡 **useLayoutEffect**
화면에 해당 컴포넌트가 그려지기 전에 콜백 함수를 실행하기 때문에 첫 번째 렌더링 때 빈 이름이 뜨는 경우를 방지할 수 있다.

</aside>

<br/>

## useMemo와 useCallback

- useMemo와 useCallback 모두 이전에 생성된 값 또는 함수를 기억하는 hook이다.
- 동일한 값과 함수를 반복해서 생성하지 않도록 해 주는 훅
- 계산이 오래 걸리는 경우 혹은 렌더링이 자주 발생하는 form에서 유용하게 사용할 수 있다.

```tsx
type DependencyList = ReadonlyArray<any>;

function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T;
function useCallback<T extends (...args: any[]) => any>(
  callback: T,
  deps: DependencyList
): T;
```

<aside>
💡 주의
**useMemo**와 **useCallback**은 이전에 사용한 값과 함수를 메모이제이션하기 때문에 불필요한 곳에 사용하면 오히려 성능이 떨어질 수 있다.

</aside>

- 메모이제이션이란?
  이전에 계산한 값을 저장함으로써 같은 입력에 대한 인산을 다시 수행하지 않도록 최직화하는 기술이다.

<br/>
<br/>

# useRef

useRef는 다음과 같은 경우 사용한다.

- <input /> 요소에 포커스를 설정하거나
- 특정 컴포넌트의 위치로 스크롤을 하거나
- DOM을 직접 선택해야 하는 경우

다음 예시는 버튼을 누르면 ref에 저장된 **<input />** DOM에 포커스를 설정하려고 하는 코드이다.

```tsx
import { useRef } from "react";

const MyComponent = () => {
  const ref = useRef<HTMLInputElement>(null);

  const onClick = () => {
    ref.current?.focus();
  };

  return (
    <>
      <button onClick={onClick}>ref에 포커스!</button>
      <input ref={ref} />
    </>
  );
};

export default MyComponent;
```

위 코드에서는 useRef의 제네릭에 `HTMLInputElement | null` 이 아닌 `HTMLInputElement`만 넣어주었다. 그럼 이제 useRef의 타입 종류를 알아보자.

### useRef의 세 가지 타입

useRef는 세 종류의 타입 정의를 가지고 있다. useRef에 넣어주는 인자 타입에 따라 반환되는 타입이 달라진다.

**첫 번째 타입 정의**

```tsx
function useRef<T>(initialValue: T): MutableRefObject<T>;
```

두 **번째 타입 정의**

```tsx
function useRef<T>(initialValue: T | null): RefObject<T>;
```

세 **번째 타입 정의**

```tsx
function useRef<T = undefined>(): MutableRefObject<T | undefined>;
```

위와 같이 useRef는 **MutableRefObject** 또는 **RefObject**를 반환한다. 각각의 타입은 아래와 같다.

```tsx
interface MutableRefObject<T> {
  current: T;
}

interface RefObject<T> {
  readonly current: T | null;
}
```

### MutableRefObejct

> MutableRefObject는 current값을 `update`할 수 있다.

만약 처음 예시에서 null을 허용하기 위해 HTMLInputElement | null 타입을 넣어 주었다면, 해당 useRef는 **첫 번째 타입 정의** `function useRef<T>(initialValue: T): MutableRefObject<T>` 를 따랐을 것이다.

이때 MutableRefObject의 current는 변경할 수 있는 값이 되어 ref.current의 값이 바뀌는 사이드 이펙트가 발생할 수 있다.

### RefObject

> RefObject의 current는 `readonly`로 값을 변경할 수 없다.

처음 예시에서 null을 허용하기 위해 **두 번째 타입 정의** `function useRef<T>(initialValue: T | null): RefObject<T>` 를 따른다면, ref.current 값을 임의로 변경할 수 없게 된다.

<br/>
<br/>

# 자식 컴포넌트에 ref 전달하기

일반적으로 자식 컴포넌트에 props를넘겨주는 방식으로 ref를 전달하게 되면 브라우저에서 경고 메시지를 띄운다. 이를 해결할 방법을 알아보자.

<aside>
⚠️ ref는 리액트에서 `DOM 요소 접근` 이라는 특수한 목적으로 사용되기 때문에 props를 넘겨주는 방식으로 전달할 수 없다.

</aside>

아래 코드는 경고 메시지를 띄우는 경우이다.

```tsx
import { useRef } from "react";

const Component = () => {
  const ref = useRef<HTMLInputElement>(null);
  return <MyInput ref={ref} />;
};

interface Props {
  ref: RefObject<HTMLInputElement>;
}

/**
  * 🚨 Warning: MyInput: `ref` is not a prop. Trying to access it will result in
  `undefined` being returned
  * If you need to access the same value within the child component, you should pass
  it as a different prop
  */
const MyInput = ({ ref }: Props) => {
  return <input ref={ref} />;
};
```

- **리액트 컴포넌트에서 ref를 prop으로 전달하기 위해서는 `forwardRef`를 사용해야 한다.**
  ref가 아닌 inputRef 등의 다른 이름을 사용한다면 `forwardRef`를 사용하지 않아도 된다.

```tsx
interface Props {
  name: string;
}

// props를 받을 때 forwardRef 키워드 사용
const MyInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
  return (
    <div>
      <label>{props.name}</label>
      <input ref={ref} />
    </div>
  );
});
```

위의 예시처럼 두 번째 인자에 ref를 넣어 전달할 수 있다.

## **forwardRef 타입 정의**

```tsx
function forwardRef<T, P = {}>(
  render: ForwardRefRenderFunction<T, P>
): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>;
```

```tsx
interface ForwardRefRenderFunction<T, P = {}> {
  (props: P, ref: ForwardedRef<T>): ReactElement | null;
  displayName?: string | undefined;
  defaultProps?: never | undefined;
  propTypes?: never | undefined;
}
```

```tsx
type ForwardedRef<T> =
  | ((instance: T | null) => void)
  | MutableRefObject<T | null>
  | null;
```

- ForwardedRef에는 오직 MutableRefObejct만 들어올 수 있다.

## useImperativeHandle

`useImperativeHandle`은 `ForwardRefRenderFunction`과 함께 쓸 수 있는 훅이다.

- 부모 컴포넌트에서 ref를 통해 자식 컴포넌트에서 정의한 커스터마이징된 메서드를 호출 가능해짐
- 자식 컴포넌트는 내부 상태나 로직을 관리하면서 부모 컴포넌트와의 결합도를 낮출 수 있음

> 부모 컴포넌트: `CreatePage`
> 자식 컴포넌트: `JobCreateForm`

```tsx
// <form> 태그의 submit 함수만 따로 뽑아와서 정의한다
type CreateFormHandle = Pick<HTMLFormElement, "submit">;

type CreateFormProps = {
  defaultValues?: CreateFormValue;
};

const JobCreateForm: React.ForwardRefRenderFunction<
  CreateFormHandle,
  CreateFormProps
> = (props, ref) => {
  // useImperativeHandle Hook을 사용해서 submit 함수를 커스터마이징한다
  useImperativeHandle(ref, () => ({
    submit: () => {
      /* submit 작업을 진행 */
    },
  }));

  // ...
};
```

부모 컴포넌트에서는 다음처럼 current.submit()을 사용하여 자식 컴포넌트의 특정 메서드를 실행할 수 있음

```tsx
const CreatePage: React.FC = () => {
  // `CreateFormHandle` 형태를 가진 자식의 ref를 불러온다
  const refForm = useRef<CreateFormHandle>(null);

  const handleSubmitButtonClick = () => {
    // 불러온 ref의 타입에 따라 자식 컴포넌트에서 정의한 함수에 접근할 수 있다
    refForm.current?.submit();
  };

  // ...
};
```

### useRef의 여러 가지 특성

- useRef는 값이 바뀌어도 컴포넌트의 리렌더링이 발생하지 않음
- 리액트 컴포넌트의 상태는 상태 변경 함수를 호출하고 렌더링된 이후에 업데이트된 상태를 조회할 수 있는 반면에, useRef로 관리되는 변수는 값을 설정한 후 즉시 조회할 수 있다.

아래 코드에서 **isAutoPlayPause**는 현재 자동 재생이 일시 정지되었는지 확인하는 ref이다.

```tsx
type BannerProps = {
  autoplay: boolean;
};

const Banner: React.FC<BannerProps> = ({ autoplay }) => {
  const isAutoPlayPause = useRef(false);

  if (autoplay) {
    // keepAutoPlay 같이 isAutoPlay가 변하자마자 사용해야 할 때 쓸 수 있다
    const keepAutoPlay = !touchPoints[0] && !isAutoPlayPause.current;

    // ...
  }
  return (
    <>
      {autoplay && (
        <button
          aria-label="자동 재생 일시 정지"
          // isAutoPlayPause는 사실 렌더링에는 영향을 미치지 않고 로직에만 영향을 주므로 상태로 사용해서 불필요한 렌더링을 유발할 필요가 없다
          onClick={() => {
            isAutoPlayPause.current = true;
          }}
        />
      )}
    </>
  );
};

const Label: React.FC<LabelProps> = ({ value }) => {
  useEffect(() => {
    // value.name과 value.id를 사용해서 작업한다
  }, [value]);

  // ...
};
```

이 변수는 렌더링에 영향을 미치지 않으며, 값이 변경되더라도 리렌더링을 기다릴 필요 없이 사용할 수 있어야 한다.

---

# 커스텀 훅 만들기

useInput 이라는 커스텀 훅을 만들어 보자

```jsx
import { useState } from "react";

const useInput = (initialValue) => {
  const [value, setValue] = useState(initialValue);

  const onChange = (e) => {
    setValue(e.target.value);
  };

  return { value, onChange };
};
```

커스텀한 useInput 훅은 아래처럼 컴포넌트 내에서 사용할 수 있다.

```jsx
const MyComponent = () => {
  const { value, onChange } = useInput("");

  return (
    <div>
      <h1>{value}</h1>
      <input onChange={onChange} value={text} />
    </div>
  );
};

export default App;
```

위에서 작성한 useInput에 타입스크립트를 적용해 보자

파일명을 .js 에서 .ts 로 바꾸면 아래처럼 에러가 발생할 것이다.

```tsx
import { useState, useCallback } from "react";

// 🚨 Parameter ‘initialValue’ implicitly has an ‘any’ type.ts(7006)
const useInput = (initialValue) => {
  const [value, setValue] = useState(initialValue);

  // 🚨 Parameter ‘e’ implicitly has an ‘any’ type.ts(7006)
  const onChange = useCallback((e) => {
    setValue(e.target.value);
  }, []);

  return { value, onChange };
};

export default useInput;
```

useInput 함수의 인자로 넣어준 initialValue와, onChange 함수의 인자로 넣어준 e의 타입이 지정되지 않았기 때문에 발생하는 에러로, 두 군데 모두 타입을 명시적으로 정의해주면 해결된다.

```tsx
import { useState, useCallback, ChangeEvent } from "react";

// ✅ initialValue에 string 타입을 정의
const useInput = (initialValue: string) => {
  const [value, setValue] = useState(initialValue);

  // ✅ 이벤트 객체인 e에 ChangeEvent<HTMLInputElement> 타입을 정의
  const onChange = useCallback((e: ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  }, []);

  return { value, onChange };
};

export default useInput;
```
