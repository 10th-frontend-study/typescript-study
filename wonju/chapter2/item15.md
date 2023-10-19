# Item15 동적 데이터에 인덱스 시그니처 사용하기

### 요약

✅ 런타임 때까지 객체의 속성을 알 수 없을 때에만 인덱스 시그니처를 사용 (안전한 접근을 위해 undefined를 유니온 타입으로 추가 고려

✅ 가능한 인터페이스, Record 타입, 매핑된 타입 처럼 정확한 타입을 사용하는 것이 좋다


## 📍인덱스 시그니처는 부정확하다

```typescript
// 인덱스 시그니처
type Rocket = {[property: string]: string};

const rocket: Rocket = {
  namme: 'Falcon 9',
  variant: 'v1.0',
  thrust: '4,940 kN',
};  // OK

const temp: Rocket = {};  // OK
```

1️⃣ 오타를 바로잡아주지 못함

2️⃣ 빈 객체도 허용

3️⃣ 키 마다 다른 타입을 사용할 수 없다. number 혹은 string 처럼 한 가지로 통일해야 한다

4️⃣ 자동완성 기능이 동작하지 않음

(모든 경우의 수가 가능하기 때문)
따라서, 일반적인 경우에는 인터페이스를 사용하는 편이 좋다

```typescript
interface Rocket {
  name: string;
  variant: string;
  thrust_kN: number;
}
const falconHeavy: Rocket = {
  name: 'Falcon Heavy',
  variant: 'v1',
  thrust_kN: 15_200 // 숫자 Numeric separator
};
```


> [!tip]+ Numeric separator
> ES2021에서 추가된 스펙으로, 숫자를 자릿수에 따라 언더바로 끊어서 가독성을 살려준다
> 	- 1_000_000 : 1,000,000 이렇게 표현 가능 (실제로는 붙여 쓴 것으로 처리됨)
> 	- 소수점 이하에서도 사용 가능
> 	- 2자리로 끊으면 센트 단위 표시



## 📍string 타입이 너무 광범위할 때 인덱스 시그니처를 대체할 수 있는 방법

#### Record 제네릭(유틸리티) 타입

```typescript
type Vec3D = Record<'x' | 'y' | 'z', number>;

// Type Vec3D = {
//   x: number;
//   y: number;
//   z: number;
// }
```

#### 매핑된 타입

키마다 별도의 타입을 사용하게 해줌

```typescript
type Vec3D = {[k in 'x' | 'y' | 'z']: number};
// Same as above
type ABC = {[k in 'a' | 'b' | 'c']: k extends 'b' ? string : number};

// Type ABC = {
//   a: number;
//   b: string;
//   c: number;
// }
```