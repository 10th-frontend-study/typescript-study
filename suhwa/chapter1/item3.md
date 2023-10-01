### 이번 item에서 알아볼 것

타입스크립트의 컴파일러 역할은 총 2가지가 있다.

1. 최신 타입스크립트 / 자바스크립트를 브라우저에서 동작할 수 있도록 구 버전의 자바스크립트 파일로 트랜스파일하기
2. 코드의 타입 오류를 체크하기

이 두 가지의 행위는 완벽히 독립적이라는 것을 이해해야 한다.

---

### 타입 오류가 있는 코드도 컴파일이 가능하다.

- 해당 코드에서 타입 오류가 발생했다.

```jsx
$ cat test.ts
let x = 'hello';
x = 1234;

$ tsc test.ts
test.ts:2:1 - error TS2322: '1234' 형식은 'string' 형식에 할당할 수 없습니다.
```

- 하지만 컴파일 된 javascript 코드는 생성된 것을 볼 수 있다.

```jsx
$ cat test.js
var x = 'hello';
x = 1234;
```

- 타입스크립트의 타입 오류는 다른 언어들의 경고(`warning`)와 같은 것이다. 즉, 문제가 될 것 같은 부분을 알려주지만 빌드를 멈추지는 않는다.

### 런타임에는 타입 체크가 불가능하다.

```tsx
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if **(shape instanceof Rectangle)** {
    // ~~~~~~~~~ 'Rectangle' only refers to a type,
    //           but is being used as a value here
    return shape.width * shape.height;
    //         ~~~~~~ Property 'height' does not exist
    //                on type 'Shape'
  } else {
    return shape.width * shape.width;
  }
}
```

- instanceof 체크는 런타임에 발생하지만, 런타임 시점에 Rectangle은 제거되기 때문에 아무런 역할을 할 수 없다.
  - 실제 컴파일된 코드
  ```jsx
  function calculateArea(shape) {
    if (shape instanceof Rectangle) {
      return shape.width * shape.height;
    } else {
      return shape.width * shape.width;
    }
  }
  ```
- shape의 타입을 명확하게 하려면 런타임에 타입 정보를 유지하는 방법이 필요하다.

방법 1. 객체의 속성값(height)의 존재 여부를 확인한다.

```tsx
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;
function calculateArea(**shape**: Shape) {
  if (**'height' in shape**) {
    shape;  // Type is Rectangle
    return shape.width * shape.height;
  } else {
    shape;  // Type is Square
    return shape.width * shape.width;
  }
}
```

방법 2. 태그된 유니온을 활용한다.

```tsx
interface Square {
  kind: 'square';
  width: number;
}
interface Rectangle {
  kind: 'rectangle';
  height: number;
  width: number;
}
type Shape = Square | Rectangle; // 여기서는 타입으로 참조

function calculateArea(**shape**: Shape) {
  if (**shape**.kind === 'rectangle') {
    shape;  // Type is Rectangle
    return shape.width * shape.height;
  } else {
    shape;  // Type is Square
    return shape.width * shape.width;
  }
}
```

방법 3. 타입을 클래스로 선언한다.

- 클래스로 선언하면 타입, 값 모두 사용 가능하다.

```tsx
**class Square** {
  constructor(public width: number) {}
}
**class Rectangle extends Square** {
  constructor(public width: number, public height: number) {
    super(width);
  }
}
type Shape = Square | Rectangle;

function calculateArea(**shape**: Shape) {
  if (**shape** instanceof **Rectangle**) { // 값으로 참조
    shape;  // Type is Rectangle
    return shape.width * shape.height;
  } else {
    shape;  // Type is Square
    return shape.width * shape.width;  // OK
  }
}
```

### 타입 연산은 런타임에 영향을 주지 않는다.

- 다음 코드에서 타입 연산은 어떠한 영향도 가지지 못한다.

```tsx
function asNumber(val: number | string): number {
  return val **as number;**
}
```

- javascript로 변환 시 다음과 같다.

```tsx
function asNumber(val) {
  return val;
}
```

- 이는 런타임에 직접 값의 타입을 체크하고, 자바스크립트 연산을 통해 변환해야 한다.
  - 참고로 as Number은 타입 단언문이다.

```tsx
function asNumber(val: number | string): number {
  return typeof val === "string" ? Number(val) : val;
}
```

### 런타임 타입은 선언된 타입과 다를 수 있다.

```tsx
function turnLightOn() {}
function turnLightOff() {}
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      turnLightOn();
      break;
    case false:
      turnLightOff();
      break;
    default:
      console.log(`I'm afraid I can't do that.`);
  }
}

interface LightApiResponse {
  lightSwitchValue: boolean;
}

async function setLight() {
  const response = await fetch("/light");
  const result: LightApiResponse = await response.json();
  setLightSwitch(result.lightSwitchValue);
}
```

- 위의 코드는 value의 type이 boolean이라고 명시되어 있지만, api 통신과 같이 값이 유동적인 경우 항상 값이 boolean이 된다는 보장이 없고, 달라질 수 있음을 명심해야 한다.

### 타입스크립트 타입으로는 함수를 오버로드 할 수 없다.

- 동일한 이름에 매개변수만 다른 버전의 함수를 생성하는 것을 오버로딩이라고 한다.
- 하지만, 타입스크립트에서는 타입과 런타임의 동작이 무관하기 때문에 오버로딩이 불가능하다.

```tsx
function add(a: number, b: number) {
  return a + b;
}
// ~~~ Duplicate function implementation
function add(a: string, b: string) {
  return a + b;
}
// ~~~ Duplicate function implementation
```

- 타입스크립트가 오버로딩 기능을 지원하기는 하지만, 제한적이다.
  - add라는 선언문이 2개가 있는데, 이는 자바스크립트로 변환되면서 사라진다. 따라서 구현체만 남게된다.

```tsx
// tsConfig: {"noImplicitAny":false}

function add(a: number, b: number): number;
function add(a: string, b: string): string;

function add(a, b) {
  return a + b;
}

const three = add(1, 2); // Type is number
const twelve = add("1", "2"); // Type is string
```

### 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.

- 타입, 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에 런타임에서는 아무런 영향을 미치지 않는다.

---

### 요약

1. 코드 생성은 타입 시스템과 무관하다. 타입스크립트 타입은 런타임 동작이나 성능에 영향을 주지 않는다.
2. 타입 오류가 존재하더라도 코드 생성(컴파일)은 가능하다.
3. 타입스크립트 타입은 런타임에 사용할 수 없다. 타입을 사용하고 싶다면 별도의 방법이 필요한데, 보통은 태그된 유니온과 속성 체크 방법을 사용한다. 클래스도 사용한다.
