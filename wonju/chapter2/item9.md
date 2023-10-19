# Item9 타입 단언보다는 타입 선언을 사용하기

* 타입 단언(as Type) 보다는 타입 선언(: Type)을 사용하기
* 화살표 함수의 반환 타입을 명시하는 방법 알기
* 타입 단언문과 null 아님 단언문 잘 사용하기


## 타입 단언(as Type) 보다는 타입 선언(: Type)을 사용하기

```ts
interface Person {name: string};
```

위에 Person이라는 클래스가 있다고 가정할 때 타입 단언과 타입 선언을 각각 알아보자.

#### 타입 선언
```ts
const alice: Person = {name: 'Alice'};
```

#### 타입 단언
```ts
const alice = {name: 'Alice'} as Person;
```

위의 두 가지 방법 모두 alice의 타입은 Person이다.


### 타입 선언을 사용해야 하는 이유?
```ts
const alice: Person = {}
// ~~~~~ 'Person' 유형에 필요한 'name' 속성이 '{}'유형에 없습니다.
const bob = {} as Person // 오류 없음
```

타입 선언을 했을 경우에는 오류를 검출해내지만, 타입 단언을 이용한 경우 오류를 검출해내지 못한다.

> 타입 단언은 강제로 타입을 지정했으니 타입 체커에게 오류를 무시하라고 하는 것이기 때문이다.


이는 속성을 추가할 때에도 마찬가지로 오류를 검출해내지 못한다.
```ts
interface Person {
  name: string
}
const alice: Person = {
  name: 'Alice',
  occupation: 'TypeScript developer',
  // ~~~~~~~~~ Object literal may only specify known properties
  //           and 'occupation' does not exist in type 'Person'
}
const bob = {
  name: 'Bob',
  occupation: 'JavaScript developer',
} as Person // No error
```


> 간혹 아래와 같은 코드가 보이는데, 이는 타입 단언문의 원래 문법으로 as Person과 똑같다.

```ts
const bob = <Person>{}
```
현재에는 잘 쓰이지 않는다.


## 화살표 함수의 반환 타입

* people 배열 각 요소에 Person 타입을 지정해 주고 싶을 때
```ts
const people = ['alice', 'bob', 'jan'].map(name => ({ name }))
// { name: string; }[]... but we want Person[]
```

아래와 같이 화살표 함수의 반환 타입을 선언하여 표현할 수 있다.
```ts
const people = ['alice', 'bob', 'jan'].map(
	(name):Person => ({ name })
); // 타입은 Person[]
```

여기서 소괄호의 의미는 name의 타입은 선언하지 않고 name의 반환 타입으로 Person을 지정해주겠다는 의미이다.