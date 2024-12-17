# 📎 아이템 59 타입스크립트 도입 전에 @ts-check와 JSDoc으로 시험해보기

타입스크립트로 전환하기 전, `@ts-check` 지시자를 사용하면 타입스크립트 전환시에 어떤 문제가 발생하는지 미리 시험할 수 있다. 타입 체커가 파일을 분석하고, 발견된 오류를 보고하도록 지시한다. 하지만 `@ts-check` 지시자는 매우 느슨한 수준으로 타입 체커를 수행하여, 심지어는 `noImplictAny` 설정을 해제한 것보다 헐거운 체크를 수행한다는 점을 주의해야 한다.

## 📍 @ts-check 사용법

```javascript
// @ts-check
const person = {first: 'Grace', last: 'Hopper'};
2 * person.first
//  ~~~~~~~~~~~~ The right-hand side of an arithmetic operation must be of type
//               'any', 'number', 'bigint' or an enum type
```

`person.first`의 타입은 string으로 추론되었고, `2 * person.first`는 타입 불일치 오류가 되었다. 타입 체커가 동작된 것.

## 📍 @ts-check로 찾을 수 있는 오류들

### 선언되지 않은 전역 변수

어딘가에 숨어있는 변수(HTML 파일내의 \<script>태그)라면, 변수를 제대로 인식할 수 있게 별도로 타입 선언 파일을 만들어야 한다.

```typescript
// @ts-check
console.log(user.firstName);
//          ~~~~ Cannot find name 'user'
```

user 선언을 위해 types.d.ts 파일 생성하기

```typescript
interface UserData {
  firstName: string;
  lastName: string;
}
declare let user: UserData;
```

타입 선언 파일을 만들면 오류가 해결된다. 선언 파일을 찾지 못하는 경우는 '트리플 슬래시' 참조를 사용하여 명시적으로 임포트할 수 있다.

```typescript
// @ts-check
/// <reference path="./types.d.ts" />
console.log(user.firstName);
```

### 알 수 없는 라이브러리

서드파티 라이브러리를 사용하는 경우, 서드파티 라이브러리의 타입 정보를 알아야 한다.

예를 들어, 제이쿼리를 사용하여 HTML 엘리먼트의 크기를 설정하는 코드에서 @ts-check 지시자를 사용하면 제이쿼리를 사용한 부분에서 오류가 발생한다.

```typescript
// @ts-check
$('#graph').style({'width': '100px', 'height': '100px'});
// Error: Cannot find name '$'
```

제이쿼리 타입 정보를 설치하고, .style을 .css로 바꾸면 오류가 사라진다.

@ts-check를 사용하면, 서드파티 라이브러리들의 타입 선언을 활용하여 타입 체크를 시행할 수 있다.

### DOM 문제

웹 브라우저에서 동작하는 코드라면, 타입스크립트는 DOM 엘리먼트 관련된 부분에 수많은 오류를 표시하게 된다.

```javascript
// @ts-check
const ageEl = document.getElementById('age');
ageEl.value = '12';
//    ~~~~~ Property 'value' does not exist on type 'HTMLElement'
```

HTMLInputElement 타입에는 value 속성이 있지만, document.getElementById는 더 상위 개념인 HTMLElement 타입을 반환하는 것이 오류의 원인이다. #age가 확실히 input 엘리먼트라는 것을 알고 있다면 타입 단언문을 사용해야 한다.&#x20;

이 경우에는 자바스크립트 파일이므로 as HTMLInputElement를 사용할 수 없다. 대신 JSDoc을 사용하여 타입 단언을 대체할 수 있다.

```javascript
// @ts-check
const ageEl = /** @type {HTMLInputElement} */(document.getElementById('age'));
ageEl.value = '12';  // OK
```

타입을 감싸는 중괄호가 필요하다는 것에 주의하며 사용하자.

한편, @ts-check를 활성화하면 이미 존재하던 JSDoc에서 부작용이 발생하기도 한다.

### 부정확한 JSDoc

JSDoc 스타일의 주석을 사용중이었다면, @ts-check 지시자를 설정하면서 기존 주석에 타입 체크가 동작하게 되어 수많은 오류가 발생한다. 타입 정보를 차근차근 추가해 나가면 된다.

```typescript
// @ts-check
/**
 * Gets the size (in pixels) of an element.
 * @param {Node} el The element
 * @return {{w: number, h: number}} The size
 */
function getSize(el) {
  const bounds = el.getBoundingClientRect();
  //                ~~~~~~~~~~~~~~~~~~~~~
  //     Property 'getBoundingClientRect' does not exist on type 'Node'
  return {width: bounds.width, height: bounds.height};
  //      ~~~~~ Type '{ width: any; height: any; }' is not
  //            assignable to type '{ w: number; h: number; }'
}
```

첫 번째 오류는, DOM 타입 불일치이므로 @param 태그를 Node에서 Element로 수정하면 된다.

두 번째 오류는, @return 태그에 명시된 타입과 실제 반환 타입이 일치하지 않아서 발생했다. @return 태그를 수정하자.

오류를 모두 수정했다면, JSDoc를 사용하여 타입 정보를 점진적으로 추가할 수 있다. 자바스크립트 환경에서도 @ts-check 지시자와 JSDoc 주석이라면 타입 스크립트와 비슷한 경험으로 작업이 가능하다. 마이그레이션 과정 중에 유용하지만, 너무 장기간 사용하지 말자. 주석이 코드 분량을 늘려서 로직 해석에 방해가 될 수 있다.

궁극적인 목표는 모든 코드가 타입스크립트 기반으로 전환되는 것이다. 이미 JSDoc 주석으로 타입 정보가 많이 담겨 있는 프로젝트라면 @ts-check 지시자만 간단히 추가하여 기존 코드에 타입 체커를 실험해볼 수 있고, 초기 오류를 빠르게 잡아낼 수 있다.

## 📍 요약

* 파일 상단에 //@ts-check를 추가하면 자바스크립트에서도 타입 체크를 수행할 수 있다.
* 전역 선언과 서드파티 라이브러리의 타입 선언을 추가하는 방법을 익히자.
* JSDoc 주석을 잘 활용하면 자바스크립트 상태에서도 타입 단언과 타입 추론을 할 수 있다.
