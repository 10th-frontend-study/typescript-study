### 이번 item에서 알아볼 것

typescript에서는 설정이라는 것이 존재한다.

이번 item에서는 typescript 컴파일러의 설정에 대해서 알아 볼 것이다.

---

### 설정에 들어가는 내용

- 어디서 소스파일을 찾을지, 어떤 종류의 출력을 생성할 지 등의 내용이 들어간다.
- 알아둬야 할 옵션
  - `**noImplicitAny**` : 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어
    - 이 옵션이 활성화 되면 ts 컴파일러는 타입을 명시적으로 지정하지 않은 값들에 대해 any를 자동 부여하는 것을 막는다.
    - 예시
      - 다음 코드는 noImplicitAny가 해제되어 있을 때는 유효하다. ( 마우스를 올려보면 해당 변수가 any타입으로 추론됨을 알 수 있다. )
      ```jsx
      function add(a, b) {
        return a + b;
      }
      ```
      - 하지만 noImplicitAny가 설정되어있을 때는 any라고 선언해주어야 한다.
      ```jsx
      function add(a: number, b: number) {
        return a + b;
      }
      ```
  - `**strictNullCheck**` : null과 undefined가 모든 타입에서 허용되는지 확인하는 설정
    - 이 옵션이 활성화 되면 null과 undefined를 가질 수 있는 변수에 대해 명시적인 타입 체크를 수행한다.
    - 예시
      - 다음은 strictNullChecks가 해제되었을 때 유효한 코드이다.
      ```jsx
      const x: number = null; // 정상, null은 유효한 값입니다.
      ```
      - 그러나, strictNullChecks가 설정되면 오류가 발생합니다.
      ```jsx
      const x: number = null; // 'null'형식은 'number' 형식에 할당할 수 없습니다.
      ```
      - 이는 다음과 같이 타입 선언에 null을 명시해줌으로서 해결할 수 있습니다.
      ```jsx
      const x: number | null = null;
      ```

### 설정을 생성하는 방법

```jsx
$ tsc --init
```

### 설정을 사용하는 방법

1. 커맨드 라인에서 입력할 수 있다.

```jsx
$ tsc --noImplicity program.ts
```

2. tsconfig.json 파일을 통해서 설정 가능하다.

```jsx
{
	"comfilerOptions": {
		"noImplicitAny": true
	}
}
```

---

### 요약

1. 타입스크립트 컴파일러는 언어의 핵심 요소에 영향을 미치는 몇 가지 설정을 포함하고 있다.
2. 타입스크립트 설정은 커맨드라인 보다는 tsconfig.json을 활용하는 것이 좋다.
3. 자바스크립트 프로젝트를 타입스크립트로 전환하는 것이 아니라면, noImplicitAny를 설정하는 것이 좋다.
4. ‘undefined는 객체가 아닙니다.’ 같은 런타임 오류를 방지하기 위해 strictNullChecks를 설정하는 것이 좋다.
5. 타입스크립트에서 엄격한 체크를 하고 싶다면 strict 설정을 고려해야한다.
