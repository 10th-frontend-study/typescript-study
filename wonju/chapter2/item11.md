# Item11 잉여 속성 체크의 한계 인지하기

### 📍잉여 속성 체크 (Excess Property Checking)

객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때,

정의된 타입 이외의 타입이 할당되는 것을 방어

- 객체 리터럴에서 타입 안정성을 확보하는 수단

- 구조적 타이핑 (aka. 덕 타이핑) 과 배치되는 개념인가 싶지만...

- 예시

```typescript
interface Room {
    numDoors: number;
    ceilingHeightFt: number;
}

// 타입이 Room 인 변수에 객체 리터럴 할당
// 1. 해당 타입의 속성이 있는지,
// 2. 그 외의 속성은 없는지 확인 : Excess Property Checking
const r: Room = {
    numDoors: 1,
    ceilingHeightFt: 10,
    elephant: "present" // 에러 : Room 타입에 없는 프로퍼티
}

const obj = {
    numDoors: 1,
    ceilingHeightFt: 10,
    elephant: "present"
};
// obj 타입
// const obj: {
//     numDoors: number;
//     ceilingHeightFt: number;
//     elephant: string;
// }

// 타입이 Room인 변수에 obj 타입인 obj 객체 리터럴을 할당
const s: Room = obj; // 정상
```

이유

- Room 타입이 obj 타입의 부분 집합이기 때문에

obj 타입을 Room 타입에 할당 가능 (구조적 타이핑)

```typescript
const obj: {
    numDoors: number;
    ceilingHeightFt: number;
    elephant: string;
}

interface Room {
    numDoors: number;
    ceilingHeightFt: number;
}

// obj 타입은 Room 타입을 충족시킴!
```

### 📍타입스크립트는 의도와 다르게 작성된 코드까지 찾으려고 함

- 예시

```typescript
interface Options {
    title: string;
    darkMode?: boolean; // 옵셔널
}

function createWindow(options: Options) {
    if (options.darkMode) setDarkMode();
    // ...
}

createWindow({
    title: "Spider Solitaire",
    darkmode: true // 런타임에서는 OK (옵셔널 프로퍼티), 하지만 타입에러 발생 -> darkMode의 오타인지 물어봄
});
```

Options 타입의 darkMode 프로퍼티는 옵셔널하기 때문에 원래는 에러가 아니지만, 오타인지 확인해주는 에러 발생

Options 타입은 범위가 매우 넓다

- string 타입인 title 프로퍼티만 가지면 됨

- 따라서 아래 값들도 할당 가능

```typescript
const o1: Options = document; // 정상
const o2: Options = new HTMLAnchorElement; // 정상
```

### 📍잉여 속성 체크를 피하는 방법1

- 잉여 속성 체크 예시2

_**임시 변수**_를 활용해 잉여 속성 체크를 우회

```typescript
interface Options {
    title: string;
    darkMode?: boolean; // 옵셔널
}

const o: Options = {
    darkmode: true, // 에러 : 객체리터럴에 대해 잉여 속성 체크 발동
    title: "Ski Free"
}

const intermediate = { darkmode: true, title: "Ski Free" };
const p: Options = intermediate; // 정상 -> 객체 리터럴이 아님
```

### 📍잉여 속성 체크를 피하는 방법2

인덱스 시그니처를 사용하여 타입스크립트가 추가적인 프로퍼티를 예상하도록 하면 된다

- 계산된 프로퍼티 네임 활용

```typescript
interface Options {
    darkMode?: boolean;
    [otherOption: string]: unknown; // 계산된 프로퍼티 네임 -> string 타입의 어떤 문자열이 프로퍼티 키가 됨
}

const o: Options = { darkmode: true }; // 정상
```

### 📍공통 속성 체크

weak type : 옵셔널 프로퍼티만 갖는 타입

- weak type에는 값 타입과 선언 타입에 공통 프로퍼티가 있는지 확인하는 별도의 체크 수행 (공통 속성 체크)

```typescript
// 옵셔널 프로퍼티만 갖는 약한 타입(weak type)
interface LineChartOptions {
    logscale?: boolean;
    invertedYAxis?: boolean;
    areaChart?: boolean;
}

const opts = { logScale: true };
// 객체 리터럴이 아님에도 공통 속성 체크 동작
const o: LineChartOptions = opts; // 에러: 공통 프로퍼티 없음
```

잉여 속성 체크와의 공통점

- 오타 체크

차이점

- 공통 속성 체크는 객체 리터럴이 아니어도 동작