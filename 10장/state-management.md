# 상태 관리

## 상태(State)

상태란 렌더링에 영향을 줄 수 있는 동적인 데이터 값을 말한다.

**상태의 세 종류**

- 지역 상태 : 컴포넌트 내부에서 사용되는 상태
- 전역 상태 : 앱 전체에서 공유하는 상태
- 서버 상태 : 외부 서버에 저장해야 하는 상태들

<br/>

## 상태를 잘 관리하기 위한 가이드

유지보수 및 성능 관점에서 상태의 개수를 최소화하는 것이 바람직함

<aside>
💡 State에 관한 오해

- 시간이 지나도 변하지 않는다면 `상태가 아니다.`
- 파생된 값, 부모에게서 props로 전달받으면 `상태가 아니다.`

</aside>

### useState 대신 useReducer를 권장하는 경우

- 다수의 하위 필드를 포함하고 있는 복잡한 상태 로직을 다룰 때
- 다음 상태가 이전 상태에 의존적일 때

아래는 배달의민족 리뷰 리스트를 필터링하여 보여주기 위한 쿼리를 상태로 지정해야 할 때의 코드이다.

각각 `useState`, `useReducer`로 구현한 예시를 보면서 이해해 보자.

`ver.useState`

```tsx
// 날짜 범위 기준 - 오늘, 1주일, 1개월
type DateRangePreset = "TODAY" | "LAST_WEEK" | "LAST_MONTH";

type ReviewRatingString = "1" | "2" | "3" | "4" | "5";

interface ReviewFilter {
  // 리뷰 날짜 필터링
  startDate: Date;
  endDate: Date;
  dateRangePreset: Nullable<DateRangePreset>;
  // 키워드 필터링
  keywords: string[];
  
	// 리뷰 점수 필터링
  ratings: ReviewRatingString[];
  
	// ... 이외 기타 필터링 옵션
}

// Review List Query State
interface State {
  filter: ReviewFilter;
  page: string;
  size: number;
}
```

`ver.useReducer`

```tsx
import React, { useReducer } from 'react';

// Action 정의
type Action =
| { payload: ReviewFilter; type: 'filter'; }
| { payload: number; type: 'navigate'; }
| { payload: number; type: 'resize'; };

// Reducer 정의
const reducer: React.Reducer<State, Action> = (state, action) => {
  switch (action.type) {
    case 'filter':
      return {
        filter: action.payload,
        page: 0,
        size: state.size,
      };
    case 'navigate':
      return {
        filter: state.filter,
        page: action.payload,
        size: state.size,
      };
    case 'resize':
      return {
        filter: state.filter,
        page: 0,
        size: action.payload,
      };
    default:
      return state;
    }
};

// useReducer 사용
const [state, dispatch] = useReducer(reducer, getDefaultState());

// dispatch 예시
dispatch({ payload: filter, type: 'filter' });
dispatch({ payload: page, type: 'navigate' });
dispatch({ payload: size, type: 'resize' });
```

```tsx
import { useReducer } from 'react';

//Before
const [fold, setFold] = useState(true);

const toggleFold = () => {
  setFold((prev) => !prev);
};

// After
const [fold, toggleFold] = useReducer((v) => !v, true);
```

<br/>

## 전역 상태 관리와 상태 관리 라이브러리

<aside>
💡 상태는 사용하는 곳과 최대한 가까워야 하며 사용 범위를 제한해야만 한다.

</aside>

전역 상태는 크게 두 가지 `React Context API`, `외부 상태 라이브러리` 를 통해 사용할 수 있다.

### Context API

```tsx
// 현재 구현된 것 - TabGroup 컴포넌트뿐 아니라 모든 Tab 컴포넌트에도 type prop을 전달
<TabGroup type='sub'>
  <Tab name='텝 레이블 1' type='sub'>
    <div>123</div>
  </Tab>
  <Tab name='텝 레이블 2' type='sub'>
    <div>123</div>
  </Tab>
</TabGroup>

// 원하는 것 - TabGroup 컴포넌트에만 전달
<TabGroup type='sub'>
  <Tab name='텝 레이블 1'>
   <div>123</div>
  </Tab>
  <Tab name='텝 레이블 2'>
    <div>123</div>
  </Tab>
</TabGroup>
```

```tsx
import {FC} from 'react';

const TabGroup: FC<TabGroupProps> = (props) => {
  const { type = 'tab', ...otherProps } = useTabGroupState(props);
  /* ... 로직 생략 */
  return (
    <TabGroupContext.Provider value={{ ...otherProps, type }}>
      {/* ... */}
    </TabGroupContext.Provider>
  );
};

const Tab: FC<TabProps> = ({ children, name }) => {
  const { type, ...otherProps } = useTabGroupContext();
  return <>{/* ... */}</>;
};
```

```tsx
import React from 'react';

type Consumer<C> = () => C;

export interface ContextInterface<S> {
  state: S;
}

export function createContext<S, C = ContextInterface<S>>(): readonly [React.FC<C>, Consumer<C>] {
  const context = React.createContext<Nullable<C>>(null);

  const Provider: React.FC<C> = ({ children, ...otherProps }) => {
    return (
      <context.Provider value= {otherProps as C}>{children}</context.Provider>
    );
  };

  const useContext: Consumer<C> = () => {
    const _context = React.useContext(context);
    if (!_context) {
      throw new Error(ErrorMessage.NOT_FOUND_CONTEXT);
    }
    return _context;
  };

  return [Provider, useContext];
}

// Example
interface StateInterface {}
const [context, useContext] = createContext<StateInterface>();
```

```tsx
import { useReducer } from 'react';

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <StateProvider.Provider value={{ state, dispatch }}>
      <ComponentA />
      <ComponentB />
    </StateProvider.Provider>
  );
}
```

<br/>
<br/>

# 상태 관리 라이브러리

우아한형제들에서 사용하는 전역 상태 라이브러리로 `MobX`, `Redux`, `Recoil`, `Zustand` 가 있다.

### MobX

- `객체지향 프로그래밍`과 `반응형 프로그래밍` 패러다임의 영향을 받은 라이브러리
- 상태 변경 로직을 단순하게 작성 가능
- 복잡한 업데이트 로직을 라이브러리에 위임 가능

단점

- 데이터가 언제, 어떻게 변하는지 추적하기 어렵기 때문에 트러블슈팅에 어려움을 겪을 수 있음

```tsx
import { observer } from 'mobx-react-lite';
import { makeAutoObservable } from 'mobx';

class Cart {
  itemAmount = 0;

  constructor() {
    makeAutoObservable(this);
  }

  increase() {
    this.itemAmount += 1;
  }

  reset() {
    this.itemAmount = 0;
  }
}

const myCart = new Cart();
const CartView = observer(({ cart }) => (
  <button onClick={() => cart.reset()}>
    amount of cart items: {cart.itemAmount}
  </button>
));

ReactDOM.render(<CartView cart= {myCart} />, document.body);
```

### Redux

- 함수형 프로그래밍의 영향을 받은 라이브러리
- 특정 UI 프레임워크에 종속되어있지 않아 독립적으로 상태 관리 라이브러리 사용 가능
- 상태 변경 추적에 용이해 트러블 슈팅에 유리

단점

- 단순한 상태 설정에도 많은 보일러플레이트가 필요
- 러닝커브가 높음

```tsx
import { createStore } from 'redux';

function counter(state = 0, action) {
  switch (action.type) {
    case 'PLUS':
      return state + 1;
    case 'MINUS':
      return state - 1;
    default:
      return state;
  }
}

let store = createStore(counter);

store.subscribe(() => console.log(store.getState()));

store.dispatch({ type: 'PLUS' });
// 1
store.dispatch({ type: 'PLUS' });
// 2
store.dispatch({ type: 'MINUS' });
// 1
```

### Recoil

- 상태를 저장하는 `Atom`과 상태 변형이 가능한 순수 함수 `selector`로 이루어짐
- 보일러플레이트가 적고 러닝커브가 낮다.

단점

- redux에 비해 아직 실험적인(experimental) 상태임
- 다양한 요구사항에 대한 충분한 검증이 이루어지지 않음

```tsx
import React from 'react';
import { RecoilRoot } from 'recoil';
import { TextInput } from './';

function App() {
  return (
    <RecoilRoot>
      <TextInput />
    </RecoilRoot>
  );
}
```

Atom은 상태의 일부를 나타내며 어떤 컴포넌트에서든 읽고 쓸 수 있도록 제공한다.

```tsx
import { atom } from 'recoil';

export const textState = atom({
  key: 'textState', // unique ID (with respect to other atoms/selectors)
  default: '', // default value (aka initial value)
});

import { useRecoilState } from 'recoil';
import { textState } from './';

export function TextInput() {
  const [text, setText] = useRecoilState(textState);
  const onChange = (event) => {
    setText(event.target.value);
  };
  return (
    <div>
      <input type='text' value= {text} onChange= {onChange} />
      <br />
      Echo: {text}
    </div>
  );
}

setInterval(() => {
  myCart.increase();
}, 1000);
```

### Zustand

- Flux 패턴을 사용하며 많은 보일러플레이트를 가지지 않음
- 훅 기반의 편리한 API 모듈 제공
- 클로저를 활용하여 스토어 내부 상태를 관리
- 상태와 상태를 변경하는 액션을 정의하고 반환된 훅을 어느 컴포넌트에서나 임포트하여 원하는 대로 사용 가능

```tsx
import { create } from 'zustand';

const useBearStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
}));

function BearCounter() {
  const bears = useBearStore((state) => state.bears);

  return <h1>{bears} around here ...</h1>;
}

function Controls() {
  const increasePopulation = useBearStore((state) => state.increasePopulation);
  return <button onClick={increasePopulation}>Plus</button>;
}
```