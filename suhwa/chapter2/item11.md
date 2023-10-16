### 이번 item에서 알아볼 것

타입이 명시된 변수에 객체 리터럴을 할당할 때 타입스크립트는 해당 타입의 속성이 있는지, 그 외의 속성은 없는지 확인한다.

- 잉여 속성 체크
  - 타입스크립트에서 객체 리터럴을 다른 타입에 할당할 때, 그 타입에 정의되지 않은 추가 프로퍼티가 있는지 검사하는 기능이다.

하지만 이러한 잉여 속성 체크는 한계를 가지는데, 이번 아이템에서는 이에 대해서 알아볼 것이다.

---

### 잉여 속성 체크의 한계 인지하기

- 첫 번째 예제는 잉여 속성 체크 과정이 수행되었다.

```jsx
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
  // ~~~~~~~~~~~~~~~~~~ Object literal may only specify known properties,
  //                    and 'elephant' does not exist in type 'Room'
};
```

- 하지만 두 번째 예제는 잉여 속성 체크 과정이 수행되지 않았다.

```jsx
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};
const r: Room = obj; // OK
```

- 잉여 속성 체크 기능은 **객체 리터럴**이나 **단언문**에서는 적용되지 않는다는 한계를 가지고 있다.

---

### 요약

1. 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때 잉여 속성 체크가 수행된다.
2. 잉여 속성 체크는 오류를 찾는 효과적인 방법이지만, 타입스크립트 타입 체커가 수행하는 일반적인 구조적 할당 가능성 체크와 역할이 다르다.
3. 할당의 개념을 정확히 알아야 잉여 속성 체크와 일반적인 구조적 할당 가능성 체크를 구분할 수 있다.
4. 잉여 속성 체크에는 한계가 있다. 임시 변수를 도입하면 잉여 속성 체크를 건너뛸 수 있다.
