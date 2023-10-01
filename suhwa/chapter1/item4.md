### 이번 item에서 알아볼 것

덕 타이핑이란

객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당하는 타입에 속하는 것으로 간주하는 방식이다. 

타입스크립트 또한 이러한 자바스크립트의 덕 타이핑 방식을 그대로 모델링한다. 

사람마다 타입 체커의 타입에 대한 이해도가 조금씩 다르기 때문에 예상치 못한 결과가 나오기도 한다. 

따라서 구조적 타이핑을 제대로 이해하면 오류와 오류가 아닌 경우에 대해 알 수 있고, 더욱 견고한 코드를 작성할 수 있다. 

---

### 구조적 타이핑

```jsx
interface Vector2D {
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}

interface NamedVector {
  name: string;
  x: number;
  y: number;
}

const v: NamedVector = { x: 3, y: 4, name: 'Zee' };
calculateLength(v);  // OK, result is 5
```

- 여기서 주목해야 할 것은 NamedVector 타입인 v를 Vector2D 타입 인자를 받는 calculateLength 함수에 넣었을 때 잘 실행된다는 것이다.
- 여기서 나오는 것이 구조적 타이핑이다.
    - NamedVector의 구조가 Vector2D와 호환되기 때문이다.
    - 핵심은 Vector2D에 있는 프로퍼티인 x, y를 NamedVector 또한 가지고 있기 때문에 calculatedLength 함수에 전달할 수 있는 것이다.

- 해당 ts 파일은 다음과 같이 컴파일된다.

```jsx
function calculateLength(v) {
    return Math.sqrt(v.x * v.x + v.y * v.y);
}
const v = { x: 3, y: 4, name: 'Zee' };
calculateLength(v); // OK, result is 5
```

---

### 요약

1. 자바스크립트가 덕 타이핑 기반이고 타입스크립트가 이를 모델링하기 위해 구조적 타이핑을 사용한다. 어떤 인터페이스에 할당 가능한 값이라면 타입 선언에 명시적으로 나열된 속성을 가지고 있을 것이다. 타입은 봉인되어있지 않다. (열려있다)
2. 클래스 역시 구조적 타이핑 규칙을 따른다.
3. 구조적 타이핑을 사용하면 유닛 테스트를 쉽게 할 수 있다. 

---

### 구조적 타이핑 vs 덕 타이핑

- 공통점
    - 모두 객체의 형태나 동작(`프로퍼티`나 `메서드`)에 따라 그것의 타입을 판단하는 방식

- 차이점
    - 사용되는 컨텍스트와 검사 시점에 있어서 몇 가지 중요한 차이점을 가진다.
    1. 검사 시점
        1. 구조적 타이핑 : 정적 언어에서 사용, 컴파일 시점에 수행 (typescript) 
        2.  덕 타이핑 : 동적 언어에서 사용, 런타임 시점에 수행 (python, javascript)
    2. 사용되는 컨텍스트
        1. 구조적 타이핑 : 정적 타입체크가 필요한 상황 
        2. 덕 타이핑 : 동적 언어에서 유연하게 객체들 사이의 호환성을 활용하려 할 때