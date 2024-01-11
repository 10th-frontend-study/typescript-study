# 8.2장 타입스크립트로 리액트 컴포넌트 만들기

이번에는 타입스크립트로 Select 컴포넌트를 구현해보면서 타입스크립트의 장점을 알아보자.

## JSX로 구현된 Select 컴포넌트

아래는 JSX로 구현된 Select 컴포넌트이다.

```tsx
const Select = ({ onChange, options, selectedOption }) => {
  const handleChange = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return (
    <select
      onChange={handleChange}
      value={selectedOption && options[selectedOption]}
    >
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </select>
  );
};
```

- Object.entries()
    
    **`Object.entries()`** 메서드는 `[for...in](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/for...in)`와 같은 순서로 주어진 객체 자체의 enumerable 속성 `[key, value]` 쌍의 배열을 반환합니다.
    

## JSDocs로 일부 타입 지정하기

컴포넌트의 속성 타입을 명시하기 위해 `JSDocs`를 사용할 수 있다. `JSDocs`를 활용하면 컴포넌트에 대한 설명과 각 속성이 어떤 역할을 하는지 간단하게 알려줄 수 있다.

```tsx
/**
* Select 컴포넌트
* @param {Object} props - Select 컴포넌트로 넘겨주는 속성
* @param {Object} props.options - { [key: string]: string } 형식으로 이루어진 option 객체
* @param {string | undefined} props.selectedOption - 현재 선택된 option의 key값 (optional)
* @param {function} props.onChange - select 값이 변경되었을 때 불리는 callBack 함수 (optional)
* @returns {JSX.Element} 
*/
const Select = //...
```

<br/>

## props 인터페이스 적용하기

Select 컴포넌트의 props에 대한 인터페이스를 작성해보자.

```tsx
type Option = Record<string, string>; // {[key: string]: string}

interface SelectProps {
  options: Option;
  selectedOption?: string;
  onChange?: (selected?: string) => void;
}

const Select = ({
  options,
  selectedOption,
  onChange,
}: SelectProps): JSX.Element => {
  //...
};
```

예시에서는 먼저 Option이라는 타입을 정의하고, SelectProps에서 이 타입을 재사용하고

있다.

- **Record**는 키(key)와 값(value)의 타입이 모두 string인 객체 타입을 생성하는 유틸리티 타입으로 사용된다.'
    - `{[key: string]: string}`으로도 표현할 수 있다.
- **onChange**는 선택된 String 값(또는 undefined)을 매개변수로 받고 어떤 값도 반환하지 않는(void) 함수임을 표현하고 있다.
- 또한 **onChange**는 옵셔널 프로퍼티이기 때문에 부모 컴포넌트에서 넘겨주지 않아도 해당 컴포넌트를 사용할 수 있다.

<aside>
💡 `[key: string]`은 사실상 모든 키값을 가질 수 있음을 나타낸다. 하지만 넓은 범위의 타입은 해당 타입을 사용하는 함수에 잘못된 타입이 전달될 수 있기 때문에, 가능한 한 타입을 좁게 제한하여 사용하길 바란다.

</aside>

<br/>

## 리액트 이벤트

리액트 컴포넌트에 등록되는 이벤트 리스너(예: onClick, onChange)는 이벤트 버블링 단계에서 호출된다.

<aside>
💡 이벤트 캡처 단계에서 이벤트 핸들러를 등록하기 위해서는 `onClickCapture`, `onChangeCapture`와 같이 일반 이벤트 리스너 이름 뒤에 **Capture**를 붙여야 한다.

</aside>

```tsx
type EventHandler<Event extends React.SyntheticEvent> = (
  e: Event
) => void | null;
type ChangeEventHandler = EventHandler<ChangeEvent<HTMLSelectElement>>;

const eventHandler1: GlobalEventHandlers["onchange"] = (e) => {
  e.target; // 일반 Event는 target이 없음
};

const eventHandler2: ChangeEventHandler = (e) => {
  e.target; // 리액트 이벤트(합성 이벤트)는 target이 있음
};
```

리액트에서 제공하는 기본 컴포넌트도 각각 props에 대한 타입을 명시해 두고 있다.
앞의 예시에서 `React.ChangeEventHandler<HTMLSelectElement>` 타입은 `React.EventHandler<ChangeEvent<HTMLSelectElement>>`와 동일한 타입이다. onChange는 HTML의 select 엘리먼트에서 발생하는 change 이벤트에 대한 핸들러로 선언되었다.

이제 우리는 `ChangeEvent<HTMLSeLectElement>` 타입의 이벤트를 매개변수로 받아 해당 이벤트를 처리 하는 핸들러를 작성할 수 있게 되었다.

```tsx
const Select = ({ onChange, options, selectedOption }: SelectProps) => {
  const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return <select onChange={handleChange}>{/* ... */}</select>;
};
```

---

## 훅에 타입 추가하기

아래 예시는 Select 컴포넌트를 사용하여 과일을 선택할 수 있는 컴포넌트를 나타낸 것이다.

### useState

state의 제네릭 타입을 명시하지 않으면 타입스크립트 컴파일러는 초깃값(default value)의 타입을 기반으로 state 타입을 추론한다.

```tsx
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

const FruitSelect: VFC = () => {
  const [fruit, changeFruit] = useState<string | undefined>();

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption={fruit} />
  );
};
```

만약 타입 매개변수가 없다면 fruit의 타입이 undefined로만 추론되면서 onChange의 타입과 일치하지 않아 오류가 발생한다.

```tsx
// fruit: undefined;
// changeFruit: (v: React.SetStateAction<undefined>) => void;
const [fruit, changeFruit] = useState();

return (
  <Select
    // Error - SetStateAction<undefined>와 맞지 않음
    // (changeFruit에는 undefined만 매개변수로 넘길 수 있음)
    onChange={changeFruit}
    options={fruits}
    selectedOption={fruit}
  />
);
```

### 예시

> 작성자가 apple, banana, blueberry 중 하나를 fruit의 타입으로 기대할 경우
> 

```tsx
const [fruit, changeFruit] = useState("apple");

// 상황: 작성자가 fruit로 orange를 입력함
const func = () => {
  changeFruit("orange");
};
// 이는 컴파일러가 에러로 잡지 않아 예상치 못한 side effect가 발생할 수 있음
```

<aside>
💡 **사이드 이펙트(side effect)**
프로그램의 실행 결과가 예상치 못한 상태로 변경되거나 예상치 못한 동작을 하게 되는 상황을 가리킨다. 즉, 코드의 실행이 예상과 다르게 동작하여 예상치 못한 결과를 초래하는 것을 의미한다.

</aside>

이럴 때는 타입 매개변수로 좀 더 명확한 타입을 지정함으로써, 다른 개발자가 해당 **state**나, **changeState**를 한정된 타입으로만 다룰 수 있게 강제할 수 있다.

```tsx
type Fruit = keyof typeof fruits; // 'apple' | 'banana' | 'blueberry';
const [fruit, changeFruit] = useState<Fruit | undefined>("apple");

// 에러 발생
const func = () => {
  changeFruit("orange");
};
```

### keyof typeof obj

**keyof typeof obj**는 해당 객체의 키값을 유니온 타입으로 추출하는 패턴으로 자주 사용된다. 앞의 예시에서는 `keyof typeof fruits`를 사용하여 fruits 키값만 추출해서 `Fruit`라 는 타입을 생성함

string, number, boolean 같은 원시 타입은 자동으로 추론되므로 생략할 수 있다.

```tsx
const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>();

// changeFruit의 매개변수 Fruit는 prop으로 전달돼야 하는 onChange의 매개변수 타입인 string보다 좁기 때문에 타입 에러가 발생
  return (
    <Select onChange={changeFruit} options={fruits} selectedOption="orange" />
  );
};
```

---

## 제네릭 컴포넌트 만들기

아래는 객체 형식의 타입을 받아 해당 객체의 키값을 selectedOption, onChange의 매개변수에만 적용하도록 만든 예시이다.

```tsx
interface SelectProps<OptionType extends Record<string, string>> {
  options: OptionType;
  selectedOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = <OptionType extends Record<string, string>>({
  options,
  selectedOption,
  onChange,
}: SelectProps<OptionType>) => {
  // Select component implementation
};
```

# HTMLAttributes, ReactProps 적용하기

className, id와 같은 리액트 컴포넌트의 기본 props을 추가하려면 SelectProps에 직접 `className: string; id?: string;`을 넣어도 되지만 아래처럼 리액트에서 제공하는 타입을 사용하면 더 정확한 타입을 설정할 수 있다.

```tsx
type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;

interface SelectProps<OptionType extends Record<string, string>> {
  id?: ReactSelectProps["id"];
  className?: ReactSelectProps["className"];
  // ...
}
```

`ComponentPropsWithoutRef`는 리액트 컴포넌트의 prop 타입을 반환해주는 타입이다.

`Type['key']`를 활용하면 객체 형식의 타입 내부 속성값을 가져올 수 있다.

ReactProps에서 여러 개의 타입을 가져와야 한다면 Pick 키워드를 활용하여 아래처럼 사용할 수도 있다.
PicksType, 'Key1' 'Key?' .>는 객체 형식의 타입에서 Keyl, Key2•••의 속성만 추 출하여 새로운 객체 형식의 타입을 반환한다.

```tsx
interface SelectProps<OptionType extends Record<string, string>>
  extends Pick<ReactSelectProps, "id" | "key" | /* ... */> {
  // ...
}
```

---

# styled-components를 활용한 스타일 정의

Select 컴포넌트에 글꼴 크기(fontsize)와 현재 선택된 option의 글꼴 색상(font color)을 설 정할 수 있게 만들어보자. 일단 theme 객체를 생성하고 프로젝트에서 사용될 fontsize와 color, 해당 타입을 간단하게 구성한다.

```tsx
const theme = {
  fontSize: {
    default: "16px",
    small: "14px",
    large: "18px",
  },
  color: {
    white: "#FFFFFF",
    black: "#000000",
  },
};

type Theme = typeof theme;
type FontSize = keyof Theme["fontSize"];
type Color = keyof Theme["color"];

```

이제 스타일과 관련된 props를 작성하고, color와 font-size의 스타일 정의를 담은 Styledselect를 작성한다.

```tsx
interface SelectStyleProps {
  color: Color;
  fontSize: FontSize;
}

const StyledSelect = styled.select<SelectStyleProps>`
  color: ${({ color }) => theme.color[color]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
`;
```

Select를 사용하는 부모 컴포넌트에서 원하는 스타일을 적용하기 위해 Select 컴포넌트의 props에 SelectStyleProps 타입을 상속한다.
Partial<Type>을 사용하면 객체 형식의 타입 내 모든 속성이 선택적으로 설정된다.

```tsx
interface SelectProps extends Partial<SelectStyleProps> {
  // ...
}

const Select = <OptionType extends Record<string, string>>({
  fontSize = "default",
  color = "black",
}: // ...
SelectProps<OptionType>) => {
  // ...

  return (
    <StyledSelect
      // ...
      fontSize={fontSize}
      color={color}
      // ...
    />
  );
};
```

# 공변성과 반공변성

앞 예시의 onChange 처럼 객체의 메서드 타입을 정의하는 상황은 종종 발생한다.

객체의 메서드 타입을 정의하는 방법은 2가지가 있다.

```tsx
interface Props<T extends string> {
  onChangeA?: (selected: T) => void;
  onChangeB?(selected: T): void;
}

const Component = () => {
  const changeToPineApple = (selectedApple: "apple") => {
    console.log("this is pine" + selectedApple);
  };

  return (
    <Select
      // Error
      // onChangeA={changeToPineApple}
      // OK
      onChangeB={changeToPineApple}
    />
  );
};
```

부모 컴포넌트에서 매개변수가 apple일 때 실행되는 메서드를 생성했다고 해보자. `onChangeA`일 때는 타입 에러가 발생하지만, `onChangeB`일 때는 타입 에러가 발생하지 않는다. (strict 모드에서)

### 공변성

```tsx
// 모든 유저(회원, 비회원)은 id를 갖고 있음
interface User {
  id: string;
}

interface Member extends User {
  nickName: string;
}

let users: Array<User> = [];
let members: Array<Member> = [];

users = members; // OK
members = users; // Error
```

Member는 User를 상속하고 있는데 User 보다 더 좁은 타입이자 User의 서브타입이다.

타입 A가 B의 서브타입일 때, T\<A>가 T\<B>의 서브타입이 된다면 `공변성을 띠고 있다`고 말한다.


### 반공변성

```tsx
type PrintUserInfo<U extends User> = (user: U) => void;

let printUser: PrintUserInfo<User> = (user) => console.log(user.id);

let printMember: PrintUserInfo<Member> = (user) => {
  console.log(user.id, user.nickName);
};

printMember = printUser; // OK.
printUser = printMember; // Error - Property 'nickName' is missing in type 'User' but required in type 'Member'.
```

일반적인 타입들은 공변성을 가지고 있어서 좁은 타입에서 넓은 타입으로 할당이 가능하다.

하지만 `제네릭 타입을 지닌 함수는 반공변성을 가진다.` 즉, T\<B>가 T\<A>의 서브타입이 되어, 좁은 타입 T\<A>의 함수를 넓은 타입 T\<B>의 함수에 적용할 수 없다는 것을 의미한다.

```tsx
interface Props<T extends string> {
  onChangeA?: (selected: T) => void;
  onChangeB?(selected: T): void;
}
```

첫 번째 예시로 돌아가 --strict 모드에서 onChangeA같이 함수 타입을 화살표 표기법으로 작성한다면 반공변성을 띠게 된다. 또한 onChangeB와 같이 함수 타입을 지정하면 공변성과 반공변성을 모두 가지는 이변성을 띠게 된다.

<aside>

💡 안전한 타입 가드를 위해서는 특수한 경우를 제 외하고는 일반적으로 `반공변적인 함수 타입을 설정하는 것이 권장된다.`

</aside>

<br/>

## 최종 Select 컴포넌트

```tsx
import React, { useState } from "react";
import styled from "styled-components";

const theme = {
  fontSize: {
    default: "16px",
    small: "14px",
    large: "18px",
  },
  color: {
    white: "#FFFFFF",
    black: "#000000",
  },
};

type Theme = typeof theme;

type FontSize = keyof Theme["fontSize"];
type Color = keyof Theme["color"];

interface SelectStyleProps {
  color: Color;
  fontSize: FontSize;
}

const StyledSelect = styled.select<SelectStyleProps>`
  color: ${({ color }) => theme.color[color]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
`;

type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;

interface SelectProps<OptionType extends Record<string, string>>
  extends Partial<SelectStyleProps> {
  id?: ReactSelectProps["id"];
  className?: ReactSelectProps["className"];
  options: OptionType;
  selectedOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = <OptionType extends Record<string, string>>({
  className,
  id,
  options,
  onChange,
  selectedOption,
  fontSize = "default",
  color = "black",
}: SelectProps<OptionType>) => {
  const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return (
    <StyledSelect
      id={id}
      className={className}
      fontSize={fontSize}
      color={color}
      onChange={handleChange}
      value={selectedOption && options[selectedOption]}
    >
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </StyledSelect>
  );
};

const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

type Fruit = keyof typeof fruits;

const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>();

  return (
    <Select
      className="fruitSelectBox"
      options={fruits}
      onChange={changeFruit}
      selectedOption={fruit}
      fontSize="large"
    />
  );
};

export default FruitSelect;
```