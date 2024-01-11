# 8.1장 리액트 컴포넌트의 타입

리액트 애플리케이션을 타입스크립트로 작성할 때 `@types/react` 패키지에 정의된 리액트 내장 타입을 가져와서 사용하곤 하는데, 가끔 역할이 비슷해서 모호한 타입들이 있다.

역할이 비슷한 타입들 중 어떤 것을 사용하면 좋을지 알아보자.

<br/>

# 함수 컴포넌트 타입

- FC는 Function Component의 약자
- `React.FC`와 `React.VFC`

```tsx
// 함수 선언을 사용한 방식
function Welcome(props: WelcomeProps): JSX.Element {}

// 함수 표현식을 사용한 방식 - React.FC 사용
const Welcome: React.FC<WelcomeProps> = ({ name }) => {};

// 함수 표현식을 사용한 방식 - React.VFC 사용
const Welcome: React.VFC<WelcomeProps> = ({ name }) => {};

// 함수 표현식을 사용한 방식 - JSX.Element를 반환 타입으로 지정
const Welcome = ({ name }: WelcomeProps): JSX.Element => {};

type FC<P = {}> = FunctionComponent<P>;

interface FunctionComponent<P = {}> {
  // props에 children을 추가
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}

type VFC<P = {}> = VoidFunctionComponent<P>;

interface VoidFunctionComponent<P = {}> {
  // children 없음
  (props: P, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}
```

<aside>

💡 `React.FC` vs `React.VFC`
이 둘의 차이는 children props의 타입을 포함하느냐 하지 않느냐인데,
React.FC는 암묵적으로 children을 포함하고 있기 때문에 children props가 필요하지 않은 컴포넌트에서는 React.VFC를 많이 사용한다.

근데 `리액트 v18부터는 React.VFC가 삭제되고` React.FC에서 children이 사라졌다.

그래서 앞으로는 React.VFC 대신 `React.FC` 또는 `props` 타입\반환 타입을 직접 지정해줘야 한다.

</aside>

<br/>

## Children props 타입 지정

가장 보편적인 children 타입은 `ReactNode | undefined` 이다.

```tsx
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };
```

<aside>

💡 **ReactNode 타입**
`ReactNode`는 `ReactElement` 외에도 `boolean`, `number` 등 여러 타입을 포함하고 있는 타입이다.

</aside>

- ReactNode는 여러 타입을 포함하고 있기 때문에 특정 문자열만 허용하고 싶을 때는 children에 대해 아래와 같이 추가로 타이핑해줘야 한다.
    
    ```tsx
    // example 1
    type WelcomeProps = {
      children: "천생연분" | "더 귀한 분" | "귀한 분" | "고마운 분";
    };
    
    // example 2
    type WelcomeProps = { children: string };
    
    // example 3
    type WelcomeProps = { children: ReactElement };
    ```
    

<br/>

# render 메서드와 함수 컴포넌트의 반환 타입

> **React.ReactElement vs JSX.Element vs React.ReactNode**
> 

### 함수 컴포넌트의 반환 타입 ReactElement

함수 컴포넌트의 반환 타입인 ReactElement는 아래와 같이 정의된다.

```tsx
interface ReactElement<
  P = any,
  T extends string | JSXElementConstructor<any> =
    | string
    | JSXElementConstructor<any>
> {
  type: T;
  props: P;
  key: Key | null;
}
```

```tsx
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {}
  }
}
```

`JSX.Element` 타입은 리액트의 ReactElement를 확장하고 있는 타입이다.

글로벌 네임스페이스에 정의되어 있어 외부 라이브러리에서 컴포넌트 타입을 재정의할 수 있는 유연성을 제공한다.

<aside>
💡 **글로벌 네임스페이스(Global Namespace)**
프로그래밍에서 식별자(변수, 함수, 타입 등)가 정의되는 전역적인 범위임
전역(글로벌) 스코프에서 선언된 변수나 함수 등은 글로벌 네임스페이스에 속함

</aside>

```tsx
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
type ReactFragment = {} | Iterable<ReactNode>;

type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/6893ebac-4a6a-48cf-b10a-764573b75430/af92edb0-d655-461d-b723-d5ccd0b1e78f/Untitled.png)

---

## ReactElement, ReactNode, JSX.Element 활용하기

리액트의 요소를 나타내는 세 타입을 모두 살펴보고 상황에 따라 사용해보자.

```tsx
declare namespace React {
  // ReactElement
  interface ReactElement<
    P = any,
    T extends string | JSXElementConstructor<any> =
      | string
      | JSXElementConstructor<any>
  > {
    type: T;
    props: P;
    key: Key | null;
  }

  // ReactNode
  type ReactText = string | number;
  type ReactChild = ReactElement | ReactText;
  type ReactFragment = {} | Iterable<ReactNode>;

  type ReactNode =
    | ReactChild
    | ReactFragment
    | ReactPortal
    | boolean
    | null
    | undefined;
  type ComponentType<P = {}> = ComponentClass<P> | FunctionComponent<P>;
}

// JSX.Element
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {
      // ...
    }
    // ...
  }
}
```

## :: ReactNode

ReactNode를 보기 전에 ReactChild 의 타입을 먼저 보자.

```tsx
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
```

- `ReactNode` 타입

```tsx
type ReactFragment = {} | Iterable<ReactNode>; // ReactNode의 배열 형태
type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

`ReactNode` 타입은 리액트 컴포넌트가 가질 수 있는 모든 타입을 의미한다.

children을 포함하는 props 타입을 선언하면 다음과 같다.

```tsx
interface MyComponentProps {
  children?: React.ReactNode;
  // ...
}
```

- 리액트 내장 타입인 PropsWithChildren 타입도 ReactNode 타입으로 children을 선언하고 있다.
    
    ```tsx
    type PropsWithChildren<P = unknown> = P & {
      children?: ReactNode | undefined;
    };
    
    interface MyProps {
      // ...
    }
    
    type MyComponentProps = PropsWithChildren<MyProps>;
    ```
    

<aside>
💡 `ReactNode`는 prop으로 리액트 컴포넌트가 다양한 형태를 가질 수 있게 하고 싶을때 유용하게 사용된다.

</aside>

## :: JSX.Element

`JSX.Element`는 ReactElement의 제네릭으로 props와 타입 필드에 대해 `any` 타입을 가지도록 확장하고 있다.

```tsx
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {
      // ...
    }
    // ...
  }
}
```

props를 `any` 타입으로 전달받기 때문에 render props 패턴으로 컴포넌트를 구현할 때 유용하다.

- render props pattern이란
    
    `Fahrenheit`컴포넌트와 `Kelvin` 컴포넌트가 사용자가 입력한 값을 전달받기 위한 방법 중 하나는 상태를 부모 컴포넌트로 올려보내는 방법이 있다.
    
    ```tsx
    function Input({ value, handleChange }) {
      return <input value={value} onChange={e => handleChange(e.target.value)} />
    }
    
    export default function App() {
      const [value, setValue] = useState('')
    
      return (
        <div className="App">
          <h1>☃️ Temperature Converter 🌞</h1>
          <Input value={value} handleChange={setValue} />
          <Kelvin value={value} />
          <Fahrenheit value={value} />
        </div>
      )
    }
    ```
    
    이 방법도 유효하긴 하지만 규모가 큰 앱에서 컴포넌트가 여러 자식 컴포넌트를 가지고 있는 경우 이 작업을 하기란 까다로운 일이다. 상태의 변경은 모든 자식 컴포넌트의 리렌더링을 유발할 수 있고 이런 상황이 쌓이면 앱의 전체적인 성능을 떨어트릴 수 있다.
    
    그 대신에 render props 패턴을 활용할 수 있다. `Input` 컴포넌트가 render prop을 받도록 리펙토링 해 보자.
    
    ```jsx
    function Input(props) {
      const [value, setValue] = useState('')
    
      return (
        <>
          <input
            type="text"value={value}onChange={e => setValue(e.target.value)}placeholder="Temp in °C"/>
          {props.render(value)}
        </>)
    }
    
    export default function App() {
      return (
        <div className="App">
          <h1>☃️ Temperature Converter 🌞</h1>
          <Inputrender={value => (
              <>
                <Kelvin value={value} />
                <Fahrenheit value={value} />
              </>)}/>
        </div>)
    }
    ```
    
    이로써 `Kelvin`과 `Fahrenheight` 컴포넌트는 사용자의 입력 값을 받을 수 있게 되었다.
    

JSX.Element 사용 예

```tsx
interface Props {
  icon: JSX.Element;
}

const Item = ({ icon }: Props) => {
  // prop으로 받은 컴포넌트의 props에 접근할 수 있다
  const iconSize = icon.props.size;

  return <li>{icon}</li>;
};

// icon prop에는 JSX.Element 타입을 가진 요소만 할당할 수 있다
const App = () => {
  return <Item icon={<Icon size={14} />} />;
};
```

## :: ReactElement

<aside>
💡 JSX.ReactElement에서 JSX는 createElement 메서드를 호출하기 위한 문법이다.

</aside>

ReactElement 타입은 JSX의 createElement 메서드 호출로 생성된 리액트 엘리먼트를 나타내는 타입이다.

앞서 살펴본 JSX.Element 예시를 추론 관점에서 더 유용하게 활용하는 법은 JSX.Element 대신에 ReactElement를 사용하여 제네릭에 직접 해당 컴포넌트의 props 타입을 명시하는 것이다.

```tsx
interface IconProps {
  size: number;
}

interface Props {
  // ReactElement의 props 타입으로 IconProps 타입 지정
  icon: React.ReactElement<IconProps>;
}

const Item = ({ icon }: Props) => {
  // icon prop으로 받은 컴포넌트의 props에 접근하면, props의 목록이 추론된다
  const iconSize = icon.props.size;

  return <li>{icon}</li>;
};
```

코드를 작성하면 icon.props에 접근할 때 어떤 props가 있는지가 추론된다.

<br/><br/>

# 리액트에서 기본 HTML 요소 타입 활용하기

```tsx
const SquareButton = () => <button>정사각형 버튼</button>;
```

기존 HTML 태그의 속성 타입을 활용하여 타입을 지정하는 방법에 대해 알아보자.

### DetailedHTMLProps와 ComponentWithoutRef

HTML 태그의 속성 타입을 활용하는 대표적인 2가지 방법

### React.DetailedHTMLProps

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

type ButtonProps = {
  onClick?: NativeButtonProps["onClick"];
};
```

위의 코드에서 ButtonProps의 onClick 타입은 실제 HTML button 태그의 onClick 이벤트 핸들러 타입과 동일하게 할당된다.

### React.ComponentPropsWithoutRef

```tsx
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
type ButtonProps = {
  onClick?: NativeButtonType["onClick"];
};
```

### 언제 ComponentPropsWithoutRef를 사용하면 좋을까

기존 button 태그의 HTML 속성을 props로 받을 수 있게 한 코드이다.

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

const Button = (props: NativeButtonProps) => {
  return <button {...props}>버튼</button>;
};
```

ref를 props로 받아서 사용할 경우

```tsx
function Button(props) {
  const buttonRef = useRef(null);

  return <button ref={buttonRef}>버튼</button>;
}
```

```tsx
function Button(ref: NativeButtonProps["ref"]) {
  const buttonRef = useRef(null);

  return <button ref={buttonRef}>버튼</button>;
}

// 함수 컴포넌트로 만들어진 Button 컴포넌트를 사용할 때
const WrappedButton = () => {
  const buttonRef = useRef();

  return (
    <div>
      <Button ref={buttonRef} />{" "}
    </div>
  );
};
```

위 코드는 기대와는 달리 전달받은 ref가 Button 컴포넌트의 button태그를 바라보지 않음

이러한 제약을 극복하기 위한 메서드가 `React.forwardRef` 이다.

```tsx
// forwardRef를 사용해 ref를 전달받을 수 있도록 구현
const Button = forwardRef((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});

// buttonRef가 Button 컴포넌트의 button 태그를 바라볼 수 있다
const WrappedButton = () => {
  const buttonRef = useRef();

  return (
    <div>
      <Button ref={buttonRef} />
    </div>
  );
};
```

```tsx
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;

// forwardRef<ref에 대한 타입 정보, props에 대한 타입 정보>
// forwardRef의 제네릭 인자를 통해 ref에 대한 타입으로 HTMLButtonElement를, props에 대한 타입으로 NativeButtonType을 정의했다
const Button = forwardRef<HTMLButtonElement, NativeButtonType>((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});
```