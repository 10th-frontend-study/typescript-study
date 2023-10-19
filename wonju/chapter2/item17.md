# Item17 변경 관련된 오류 방지를 위해 readonly 사용하기

오류의 범위를 좁히기 위해서는 `readonly` 접근 제어자를 사용하면 된다.

```typescript
function arraySum (arr: readonly number[]) {
	let sum =0, num;
  	while ((num = arr.pop()) !== undefined) {
    	// readonly number[] 형식에 pop 속성이 없습니다.
      	sum += num;
    }
  	return sum;
}
```

`readonly` 는 몇가지 특성이 있는데,

1. 배열의 요소를 읽을 수 있지만, 쓸 수는 없다.
2. length를 읽을 수 있지만, 바꿀 수는 없다. (배열을 변경함)  
3. 배열을 변경하는 pop을 비롯한 다른 메서드를 호출할 수 없다.

number[]는 readonly number[]보다 기능이 많기 때문에, readonly number[]의 서브타입이 된다.  
➡ 변경 가능한 배열을 readonly 배열에 할당할 수 있지만, 그 반대는 **불가능**합니다.

```typescript
const a: number[] = [1,2,3];
const b: readonly number[] = a;
const c: number[] = b; // error 할당 불가능
```

위의 arraySum 함수를 수정을 해보자.

```typescript
function arraySum (arr: readonly number[]) {
	let sum =0, num;
  	for (const num of arr) {
    	sum += num;
    }
  	return sum;
}
```

> 만약 함수가 매개변수를 변경하지 않는다면, `readonly`로 선언하자.
> 어떤 함수를 `readonly`로 만들면, 그 함수를 호출하는 다른 함수도 모두 `readonly`로 만들어야 한다.

### readonly는 얕게(shallow) 동작한다.

```typescript
const dates: readonly Date[] = [new Date()];
dates.push(new Date());
// Error 'readonly Date[] 형식에 'push' 속성이 없습니다.

dates[0].setFullYear(2037);
```

현재 시점에는 깊은(deep) readonly 타입이 기본으로 지원되지 않지만, 제네릭을 만들면 깊은 readonly 타입을 사용할 수 있다.  
➡ 제네릭은 만들기 까다롭기 때문에 [ts-essentials DeepReadonly 제네릭](https://github.com/ts-essentials/ts-essentials) 사용한다.

### 인덱스 시그니처에 readonly 사용하기

인덱스 시그니처에도 readonly를 쓸 수 있다.  
읽기는 허용하되 쓰기를 방지하는 효과가 있다.

```typescript
let obj: {readonly [k:string]: number} = {};
// Readonly<{[k: string]: number}>

obj.hi =45; // Error ~ ...형식의 인덱스 시그니처는 읽기만 허용됩니다.
obj = {...obj, hi:12}; // 정상
obj = {...obj, bye: 34}; // 정상
```