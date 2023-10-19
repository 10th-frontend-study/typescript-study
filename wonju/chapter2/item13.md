# Item13 타입과 인터페이스의 차이점 알기

> 📌 **요약**
> - **한 가지 일관된 스타일을 확립**하고, **보강 기법이 필요한가** 고려해야 한다.
> - interface로는 union 연산자 사용 불가
> - 보강이 필요하다면 `interface`, 그렇지 않은 일반적인 경우라면 `type`이 유용하다.
> - 기존에 작업 중이던 타입이 있다면 컨벤션을 맞춰 따라가는 것도 중요하다.

참고: 예제에서 식별자 이름 앞에 I, T를 붙여 구분하는 것은 현재 안티패턴이다. 이는 c#에서 유래된 것이다.  

## type, interface 정의 방법

```ts
// 타입
type TState = {
  name: string;
  capital: string;
}

// 인터페이스
interface IState {
  name: string;
  capital: string;
}

// IState, TState 모두 잉여 속성 체크를 통해 에러를 발생시킨다.
const wyoming: TState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000
// ~~~~~~~~~~~~~~~~~~ Type ... is not assignable to type 'TState'
//                    Object literal may only specify known properties, and
//                    'population' does not exist in type 'TState'
};
```

## type, interface 공통점

대부분의 상황에서는 타입, 인터페이스 두 가지 다 타입 정의를 할 수 있다.

#### 인덱스 시그니처  

```ts
// 타입
type TDict = { [key: string]: string };

// 인터페이스
interface IDict {
  [key: string]: string;
}
```

#### 함수 타입

```ts
  // 타입
  type TFn = (x: number) => string;
  
  // 인터페이스
  interface IFn {
    (x: number): string;
  }
  
  const toStrT: TFn = x => '' + x;  // OK
  const toStrI: IFn = x => '' + x;  // OK
```

#### 제너릭

```ts
// 타입
type TPair<T> = {
  first: T;
  second: T;
}

// 인터페이스
interface IPair<T> {
  first: T;
  second: T;
}
```

#### 확장 가능
    - extends 로 확장한 것 (test1) → 이미 확장되어 있기 때문에 한번에 모든 에러를 불러냄
    - intersection 연산자 (test2) → 순서대로 체크한다. 앞의 문제가 해결되어야 다음 에러를 불러냄

```ts
interface IStateWithPop extends TState {
  population: number;
}

type TStateWithPop = IState & { population: number; };

const test1:IStateWithPop = {name: 'd'};
// Type '{ name: string; }' is missing the following properties 
// from type 'IStateWithPop': population, capitalts(2739)

const test2:TStateWithPop = {name: 'd'};
// Type '{ name: string; }' is not assignable to type 'TStateWithPop'.
//  Property 'capital' is missing in type '{ name: string; }' but required in type 'IState' ts(2322)
```

#### 클래스 구현

```ts
class StateT implements TState {
  name: string = '';
  capital: string = '';
}
class StateI implements IState {
  name: string = '';
  capital: string = '';
}
```

✍️ **implements, extends 차이**
- extends
    - 상속
- implements
    - 클래스 형태를 제한하는 것


## type, interface 차이점

#### 1. union은 type만 가능하다 ⭕

```ts
type AorB = 'a' | 'b';
```

- 활용하는 예제
    
    ```tsx
    type Input = { /* ... */ };
    type Output = { /* ... */ };
    interface VariableMap {
      [name: string]: Input | Output;
    }
    
    // 유니온을 활용
    type NamedVariable = (Input | Output) & { name: string };
    
    // 튜플, 배열 등 타입 정의
    type Pair = [number, number];
    type StringList = string[];
    type NamedNums = [string, ...number[]];
    ```
    
    - 튜플은 인터페이스로도 정의할 수 있지만, concat 과 같은 내장 메서드를 사용할 수 없으므로 type으로 하는 것이 낫다.

#### 2. 보강은 인터페이스만 가능하다 ⭕

```ts
interface IState {
  name: string;
  capital: string;
}
interface IState {
  population: number;
}
const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000
};  // OK
```

- 선언 병합(declaration merging) : 같은 이름의 인터페이스로 다른 속성을 정의했을 때 둘이 합쳐짐
- 예시: 타입스크립트에서 설정에 따라 다른 버전의 에크마 스크립트를 지원할 때 보강기법을 사용해 선언 병합한다. (lib.es5.d.ts 와 같이 추가된 에크마 스크립트 타입들이 인터페이스로 정의되어 있음)


## 요약

> - 인터페이스는 보강 기법을 사용할 수 있지만 union 등 복잡한 연산자를 사용할 수 없다.
> - 기존에 작성되던 타입 정의 방법을 따르되 정의 목적에 따라 통일성이 있어야 한다.
> - 새로 정의할 수 있다면 보강 기법이 필요한 곳엔 인터페이스, 그 외엔 타입으로 통일하자.