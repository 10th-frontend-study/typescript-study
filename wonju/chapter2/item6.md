# Item6 편집기를 사용하여 타입 시스템 탐색하기

### iTEM6 편집기를 사용하여 타입 시스템 탐색하기
* [편집기에서 타입스크립트 언어 서비스를 적극 활용해야 한다.](#타입스크립트의-언어-서비스)
* [편집기를 이용하여 타입 시스템이 동작하는 원리와 타입을 추론하는 방법을 알 수 있다.]()
* [타입 선언 파일 `lib.dom.d.ts` 파일을 찾아보는 방법을 알기](#lib-dom-d-ts)


## 타입스크립트의 언어 서비스
타입스크립트를 설치하면 1. `타입스크립트 컴파일러(tsc)` 와 2. `타입스크립트 서버(tsserver)`를 제공한다.
여기서 타입스크립트 서버는 언어 서비스와 편집기를 제공하는데 이 두 가지를 잘 사용하는 것이 중요하다.

### 언어 서비스에 포함되는 네 가지
* 코드 자동 완성
* 명세(specification) 검사
* 검색
* 리팩터링

## 편집기
편집기는 타입스크립트가 언제 어떻게 타입을 추론하는지 알 수 있게 해줌
예를 들어 아래와 같이 타입을 명시하지 않은 변수도 타입스크립트는 추론을 통해 타입을 밝혀낸다.
```js
let num = 10;
```
num에 마우스를 갖다 대면 편집기가 뜨고 num의 타입이 number라고 추론한 모습을 볼 수 있다. 


```ts
function getElement(elOrId: string|HTMLElement|null): HTMLElement {
  if (typeof elOrId === 'object') {
    return elOrId;
 // ~~~~~~~~~~~~~~ 'HTMLElement | null' is not assignable to 'HTMLElement'
  } else if (elOrId === null) {
    return document.body;
  } else {
    const el = document.getElementById(elOrId);
    return el;
 // ~~~~~~~~~~ 'HTMLElement | null' is not assignable to 'HTMLElement'
  }
}
```


## lib.dom.d.ts 
타입스크립트의 DOM 타입 선언이 모아져 있는 lib.dom.d.ts파일을 보고 타입스크립트가 어떻게 동작을 모델링하는지 알아볼 필요가 있다.