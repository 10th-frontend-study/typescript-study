### 이번 item에서 알아볼 것

타입스크립트의 심벌은 타입 공간이나 값 공간 중의 한 곳에 존재한다.

심벌은 이름이 같아도 속하는 공간에 따라 다른 것을 나타낼 수 있기 때문에 혼란스러울 수 있다.

이번 아이템에서는 이러한 **타입 공간과 값 공간의 심벌을 구분하는 것**을 학습할 것이다.

---

### 타입 공간과 값 공간은 서로 관련이 없다.

```jsx
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });
```

위의 Cylinder은 타입으로서 쓰였고, 아래 Cylinder은 값으로서 사용되었다.

이 둘은 이름은 같지만 상황에 따라 값으로 쓰이고, 타입으로 쓰인다.

### 타입과 값 구분하기

보통 type이나 interface 다음에 나오면 타입이고, const나 let 뒤에 나오면 값이다.

컴파일 과정에서 사라지는 정보들은 타입 정보이므로 이때 사라지는 것들이 타입이다.

### Class와 enum은 상황에 따라 타입과 값 모두 사용 가능하다.

```jsx
class Cylinder {
  radius = 1;
  height = 1;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });
function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape.radius;
    // ~~~~~~ Property 'radius' does not exist on type '{}'
  }
}
```

- 여기서는 타입으로 사용되었다.
- 클래스가 타입으로 사용될 때는 속성과 메서드가 사용되는 반면, 값으로 쓰일 때는 생성자가 사용된다.

### typeof 연산자는 값과 타입에 따라 다른 역할을 한다.

- 타입의 관점 : 값을 읽어서 타입스크립트 타입을 반환한다.
- 값의 관점 : 자바스크립트 런타임의 typeof 연산자가 된다.

### 타입스크립트에서 타입과 값 공간을 잘못 구분한 예시

- email 함수는 단일 객체 매개변수를 받는다.

```jsx
interface Person {
  first: string;
  last: string;
}
function email(options: { person: Person, subject: string, body: string }) {
  // ...
}
```

- 그러나 타입스크립트에서 구조분해 할당을 하면 오류가 발생한다.

```jsx
function email({
  person: Person,
  // ~~~~~~ Binding element 'Person' implicitly has an 'any' type
  subject: string,
  // ~~~~~~ Duplicate identifier 'string'
  //        Binding element 'string' implicitly has an 'any' type
  body: string,
}) // ~~~~~~ Duplicate identifier 'string'
//        Binding element 'string' implicitly has an 'any' type
{
  /* ... */
}
```

- 값의 관점에서 Person과 string을 해석했기 때문에 발생하는 것이다.
- Person이라는 변수명과 string이라는 이름을 가지는 두 개의 변수를 생성하려한 것이다.
- 이를 해결하려면 타입과 값을 명확하게 구분해야한다.

```jsx
interface Person {
  first: string;
  last: string;
}
function email(
	// 값 : 타입
  **{person, subject, body}: {person: Person, subject: string, body: string}**
) {
  // ...
}
```

---

### 요약

1. 타입스크립트 코드를 읽을 때 타입인지 값인지 구분하는 방법을 터득해야 한다.
2. 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다. type과 interface같은 키워드는 타입 공간에만 존재한다.
3. class나 enum 같은 키워드는 타입과 값 두 가지로 사용될 수 있다.
4. ‘foo’는 문자열 리터럴이거나 문자열 리터럴 타입일 수 있다. 차이점을 알고 구별하는 방법을 터득해야한다.
5. typeof, this 그리고 많은 다른 연산자들과 키워드들은 타입 공간과 값 공간에서 다른 목적으로 사용될 수 있다.
