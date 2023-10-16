### 이번 item에서 알아볼 것

타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법은 2가지가 있다.

1. 타입 선언
2. 타입 단언

이 두가지 중에서 타입 선언을 사용하는 것이 낫다. 이번 아이템에서는 그 이유에 대해서 알아 볼 것이다.

---

### 타입 선언과 타입 단언

```jsx
interface Person { name: string };

const alice: Person = { name: 'Alice' };  // Type is Person
const bob = { name: 'Bob' } as Person;  // Type is Person
```

- 첫 번째 줄은 alice뒤에 :Person이라는 타입 선언을 붙였다. 이를 통해 그 값이 선언된 타입임을 명시했다.
- 두 번째 줄은 as Person이라는 타입 단언을 붙였다. 이는 타입스크립트가 추론한 값이 있더라도 무시하고 무조건 Person 타입으로 간주한다.

### 타입 선언을 사용해야 하는 이유

- 타입 단언문은 강제로 Person 타입을 지정했으니 타입 체커에게 오류를 무시하라고 한다.

```jsx
interface Person { name: string };
const alice: Person = {};
   // ~~~~~ Property 'name' is missing in type '{}'
   //       but required in type 'Person'
const bob = {} as Person;  // No error
```

- 속성을 추가할 때도 기존 타입에 없는 속성이 있어도 오류를 발생하지 않는다.

```jsx
interface Person { name: string };
const alice: Person = {
  name: 'Alice',
  occupation: 'TypeScript developer'
// ~~~~~~~~~ Object literal may only specify known properties
//           and 'occupation' does not exist in type 'Person'
};
const bob = {
  name: 'Bob',
  occupation: 'JavaScript developer'
} as Person;  // No error
```

### 화살표 함수의 타입 선언

- 화살표 함수는 추론된 타입이 모호할 때가 많다.
- 다음 코드는 Person 타입을 원했지만, 그냥 객체 타입으로 반환된다.

```jsx
interface Person {
  name: string;
}
const people = ["alice", "bob", "jan"].map((name) => ({ name }));
// { name: string; }[]... but we want Person[]
```

- 이때 해결할 수 있는 방법은 타입과 함께 화살표 함수의 콜백함수에서 반환 시 타입을 지정하고 반환해주는 것이다.

```jsx
interface Person {
  name: string;
}
const people = ["alice", "bob", "jan"].map((name) => {
  const person: Person = { name };
  return person;
}); // Type is Person[]
```

- 좀더 간단히 하면 다음과 같다.
  - 콜백함수의 인자인 name을 Person 타입이라고 명시한다.

```jsx
interface Person {
  name: string;
}
const people = ["alice", "bob", "jan"].map((name): Person => ({ name })); // Type is Person[]
```

### 타입 단언이 꼭 필요할 때

- 타입 단언은 타입 체커가 추론한 타입보다 우리가 판단하는 타입이 더 정확할 때 의미가 있다.

---

### 요약

1. 타입 단언보다 타입 선언을 사용해야 한다.
2. 화살표 함수의 반환 타입을 명시하는 방법을 터득해야 한다.
3. 타입스크립트 보다 타입 정보를 더 잘 알고 있는 상황에서는 타입 단언문과 null 아님 단언문을 사용하면 된다.
