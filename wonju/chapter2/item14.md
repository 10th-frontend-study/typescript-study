# Item14 타입 연산과 제너릭 사용으로 반복 줄이기

타입 간에 매핑을 통해 타입 정의에서도 DRY(Don't Repeat Yourself)의 장점을 적용할 수 있다.
아래와 같은 작업을 통해 Javascript에서 헬퍼 함수 중복제거와 같은 활동을 타입스크립트에서도 수행할 수 있다.

## 1. 타입에 이름 붙이기

타입스크립트의 named type에는 Type과 Interface가 있다.
Type은 intersection 연산자, interface는 extend를 사용해 중복되는 필드의 반복을 피한다.

## 2. Type의 일부분 사용

기존 타입(State)를 인덱싱하여 사용할 수도 있다.  
Mapped Type을 사용하거나 미리 구현된 유틸리티 클래스 Pick을 사용.

```javascript
type ParOfState = {
	userId: State['userId'];
  	pageTitle: State['pageTitle'];
	recentFile: State['recentFile'];
	...
}

// Mapped Type을 사용해 직접 생성
type PartOfState = {
	[k in 'userId' | 'pageTtitle' | 'recentFile']: State[k]
};

// 유틸리티 클래스 Pick 사용
type Pick<T, K> = { [k in K]: T[k] };
```

## 3. 유니온에서 중복 회피

유니온 타입이 주어졌을때, 하위 속성의 타입을 직접 정의하면 업데이트가 반영되지 않는다.

```typescript
type Action = SaveAction | LoadAction;

// 유니온을 인덱싱하여 타입 반복 없이 ActionType 정의
type ActionType = Action['type'];


// Pick을 사용하면 interface를 반환
type ActionRec = Pick<Action, 'type'>; // {type: "save" | "load"}
```

## 4. 특정 타입의 부분 집합을 만족하는 타입을 정의

```typescript
type User = {
    name: string;
    age: number;
    email: string;
};

// 매핑된 타입으로 직접 순회하여 각 속성을 선택적으로.
type UpdatedUser = {[k in keyof Options]?: Options[k]};

// 유틸리티 클래스 Partial 사용
type PartialUser = Partial<User>;
```

## 5. 값의 형태에 해당하는 타입을 정의

```typescript
let user = {
    name: "John",
    age: 25,
    email: "john.doe@example.com"
};

let newUser: typeof user;
```

위 코드에서의 typeof는 자바스크립트 런타임 연산자 typeof가 아니라 타입스크립트 단게에서 연산되는 typeof임.

이렇게 값으로부터 타입을 만들어 낼 때는, 타입 정의를 먼저하고 값이 그 타입에 할당 가능하다고 선언하는 것이 바람직하다. 그렇지 않으면 타입이 불명확해지고, 예상하기 어려운 타입 변동이 있을 수 있다.

그리고 함수나 메서드의 반환 값에 명명된 타입을 만들고 싶은 경우

```typescript
function getUserInfo(userId: string) {
	// ...
  	return {
    	userId,
      	name,
      	age,
      	height,
      	weight,
      	favoriteColor
    }
}

type UserInfo = ReturnType<typeof getUserInfo>;
```

다음과 같이 빌트인 유틸리티 클래스 ReturnType 제네릭을 사용하면 된다.

그런데 함수에서 매개변수로 매핑할수 있는 값을 제한하기 위해 타입 시스템이 필요한 것 처럼, 제네릭 타입에서도 매개변수를 제한할 수 있는 방법이 필요하다.

제네릭 타입에서 매개변수를 제한할 수 있는 방법은 extends를 사용하는 것이다. extned를 사용하면 제네릭 매개변수가 특정 타입을 확장한다고 선언할 수 있다.

```typescript
interface Name {
  first: string;
  last: string;
}

type DancingDuo<T extends Name> = [T,T];

const couple1: DancingDuo<Name> = [
  {first: 'Fred', last: 'Astaire'},
  {first: 'Ginger', last: 'Rogers'},
  ]

const couple2: DancingDuo<{first: string}> = [
  {first: 'Sonny'},
  {first: 'Cher'}
]
```

Pick을 직접 구현할 때 이러한 extend를 사용해 완벽히 구현할 수 있다.

```typescript
//K는 T 타입과 무관하고 범위가 너무 넓다. K의 조건은 T의 키의 부분 집합 , 즉 keyof T가 되어야 한다.
type Pick<T,K> = {
  [k in K]: T[k]
};



type Pick<T,K extends keyof T> = {
  [k in K]: T[k]
```


# 요약

- DRY를 타입스크립트에서도 최대한 적용해야 한다.
- 타입에 이름을 붙이고 extends를 활용해 반복을 줄이자.
- 타입들 간의 매핑을 위해 TS가 제공하는 도구들을 알아두어야 함.(keyof, tepeof, 인덱싱, 매핑된 타입 등)
- 제네릭 타입은 타입을 위한 함수다.
- 제네릭 타입과 extend를 함께 활용하여 매개변수를 제한하자.
- 표준 라이브러리의 Pick, partial, ReturnType 등의 제네릭 타입에 익숙해져야 한다.
