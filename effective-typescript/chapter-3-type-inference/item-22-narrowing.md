# 📎 아이템 22 타입 좁히기

타입 넓히기의 반대는 타입 좁히기이다. 타입 좁히기는 타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 말한다.

## 📍 예시

### ⭐️ null 체크

```typescript
const elem = document.getElementById('what-time-is-it');
//    ^? const elem: HTMLElement | null
if (elem) {
  elem.innerHTML = 'Party Time'.blink();
  // ^? const elem: HTMLElement
} else {
  elem
  // ^? const elem: null
  alert('No element #what-time-is-it');
}
```

만약 `elem`이 `null`이라면, 분기문의 첫번째 블록이 실행되지 않는다. 첫번째 블록에서 `HTMLElement | null` 타입의 `null`을 제외하므로, 더 좁은 타입이 되어 작업이 쉬워진다.

타입 체커는 일반적으로, 조건문에서 타입 좁히기를 잘 해내지만, 타입 별칭이 존재한다면 그러지 못할 수 있다. (아이템24)

### ⭐️ 변수의 타입 좁히기

분기문에서 예외를 던지거나 함수를 반환하여 블록의 나머지 부분에서 변수의 타입을 좁힐 수도 있다.

```typescript
const elem = document.getElementById('what-time-is-it');
//    ^? const elem: HTMLElement | null
if (!elem) throw new Error('Unable to find #what-time-is-it');
// ^? const elem: HTMLElement
elem.innerHTML = 'Party Time'.blink();
```

이 외에도 타입을 좁히는 방법은 많이 있다.

### ⭐️ instanceof 사용

```typescript
function contains(text: string, search: string | RegExp) {
  if (search instanceof RegExp) {
    return !!search.exec(text);
    //       ^? (parameter) search: RegExp
  }
  return text.includes(search);
  //                   ^? (parameter) search: string
}
```

속성 체크로도 타입을 좁힐 수 있다.

```typescript
interface Apple { isGoodForBaking: boolean; }
interface Orange { numSlices: number; }
function pickFruit(fruit: Apple | Orange) {
  if ('isGoodForBaking' in fruit) {
    fruit
    // ^? (parameter) fruit: Apple
  } else {
    fruit
    // ^? (parameter) fruit: Orange
  }
  fruit
  // ^? (parameter) fruit: Apple | Orange
}
```

### ⭐️ Array.isArray (내장 함수 사용)

```typescript
function contains(text: string, terms: string | string[]) {
  const termList = Array.isArray(terms) ? terms : [terms];
  //    ^? const termList: string[]
  // ...
}
```

타입스크립트는 일반적으로 조건문에서 타입을 좁히는 데 매우 능숙하다.

### ⭐️ 잘못된 예제 - 유니온 타입에서 null 제외하기

```typescript
const elem = document.getElementById('what-time-is-it');
//    ^? const elem: HTMLElement | null
if (typeof elem === 'object') {
  elem;
  // ^? const elem: HTMLElement | null
}
```

자바스크립트에서 typeof null이 "object"이기 때문에, if 구문에서 null이 제외되지 않는다.&#x20;

기본형 값이 잘못되어도 비슷한 사례가 발생한다.

### ⭐️ 잘못된 예제 - 기본형 값이 잘못된 경우

```typescript
function maybeLogX(x?: number | string | null) {
  if (!x) {
    console.log(x);
    //          ^? (parameter) x: string | number | null | undefined
  }
}
```

빈 문자열 ' '과 0 모두 false가 되기 때문에, 타입은 전혀 좁혀지지 않고 x는 여전히 string or number이다.

### ⭐️ 명시적 태그 붙이기

```typescript
interface UploadEvent { type: 'upload'; filename: string; contents: string }
interface DownloadEvent { type: 'download'; filename: string; }
type AppEvent = UploadEvent | DownloadEvent;

function handleEvent(e: AppEvent) {
  switch (e.type) {
    case 'download':
      console.log('Download', e.filename);
      //                      ^? (parameter) e: DownloadEvent
      break;
    case 'upload':
      console.log('Upload', e.filename, e.contents.length, 'bytes');
      //                    ^? (parameter) e: UploadEvent
      break;
  }
}
```

이 패턴은 **'태그된 유니온'** 또는 **'구별된 유니온**'이라고 불리며, 타입스크립트 어디서나 찾아볼 수 있다.

### ⭐️ 커스텀 함수 도입

타입스크립트가 타입을 식별하지 못한다면, 식별을 돕기 위해 커스텀 함수를 도입한다.

```typescript
function isInputElement(el: Element): el is HTMLInputElement {
  return 'value' in el;
}

function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
    return el.value;
    //     ^? (parameter) el: HTMLInputElement
  }
  return el.textContent;
  //     ^? (parameter) el: HTMLElement
}
```

**'사용자 정의 타입 가드'** 라고 한다. 반환 타입의 `el is HTMLInputElement`는 함수의 반환이 true인 경우, 타입 체커에게 매개변수의 타입을 좁힐 수 있다고 알려준다.

### ⭐️ 타입 가드 사용

타입 가드를 사용하여 배열과 객체의 타입 좁히기를 할 수 있다.

배열에서 어떤 탐색을 수행할 때 `undefined`가 될 수 있는 타입을 사용할 수 있다.

```typescript
const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
const members = ['Janet', 'Michael'].map(
  who => jackson5.find => n === who)
); // 타입이 (string|undefined)[]
```

filter 함수를 사용해 undefined를 걸러내려고 해도 잘 동작하지 않는다. 이럴 때 타입 가드를 사용하면 타입을 좁힐 수 있다.

```typescript
function isDefined<T>(x: T | undefined): x is T {
  return x !== undefined;
}
const members = ['Janet', 'Michael'].map(
  who => jackson5.find => n === who)
).filter(isDefined); // 타입이 string[]
```

**📍 정리**

편집기에서 타입을 조사하는 습관을 가지면 타입 좁히기가 어떻게 동작하는지 익힐 수 있다. 타입이 좁혀지는 과정을 이해한다면 타입 추론에 대한 개념을 잡을 수 있고, 오류 발생의 원인을 알 수 있으며, 타입 체커를 더 효율적으로 이용할 수 있다.

## 📍 요약

* 분기문 외에도 여러 종류의 제어 흐름을 살펴보며 타입스크립트가 타입을 좁히는 과정을 이해하자.
* 태그된/구별된 유니온과 사용자 정의 타입 가드를 사용하여 타입 좁히기 과정을 원활하게 만들 수 있다.
