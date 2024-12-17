# 📎 아이템 58 모던 자바스크립트로 작성하기

옛날 버전의 자바스크립트를 최신 버전의 자바스크립트로 바꾸는 작업은 타입스크립트로 전환하는 작업의 일부로 볼 수 있다. 모던 자바스크립트 기능 몇 가지에 대해 알아보자.

### 🔗 ECMAScript 모듈 사용하기

import, export를 사용하는 ECMAScript 모듈이 표준이 되었따. ES 모듈로 전환하는 것이 좋다. ES 모듈 시스템을 사용하기 위해 웹팩이나 ts-node 같은 도구가 필요할 수 있다. ES 모듈 시스템은 타입스크립트에서도 잘 동작하며, 모듈 단위로 전환할 수 있게 해주기 때문에 마이그레이션이 원활해진다.

### 🔗 프로토타입 대신 클래스 사용하기

ES2015에 class 키워드를 사용하는 클래스 기반 모델이 도입되었다. 마이그레이션 하려는 코드에서 단순한 객체를 다룰 때 프로토타입을 사용하고 있었다면 클래스로 바꾸는 것이 좋다.

```typescript
// 프로토타입 기반 객체
function Person(first, last) {
  this.first = first;
  this.last = last;
}

Person.prototype.getName = function() {
  return this.first + ' ' + this.last;
}

const marie = new Person('Marie', 'Curie');
console.log(marie.getName());
```

```typescript
// 클래스 기반 객체
class Person {
  constructor(first, last) {
    this.first = first;
    this.last = last;
  }

  getName() {
    return this.first + ' ' + this.last;
  }
}

const marie = new Person('Marie', 'Curie');
console.log(marie.getName());
```

### 🔗 var 대신 let/const 사용하기

자바스크립트의 var 키워드의 스코프 규칙에 문제가 있기 때문에 let과 const를 사용하여 스코프 문제를 피하는 것이 좋다. var 키워드를 let이나 const로 변경하면, 일부 코드에서 타입스크립트가 오류를 표시할 수도 있다. 오류가 발생한 부분은 잠재적으로 스코프 문제가 존재하는 코드이므로 수정하면 된다.

중첩된 함수 구문에도 var의 경우와 비슷한 스코프 문제가 존재한다.

### 🔗 for(;;) 대신 for-of 또는 배열 메서드 사용하기

자바스크립트에서 배열을 순회할 때 C 스타일의 for 루프를 사용했지만 모던 자바스크립트에는 for-of 루프가 존재한다.

```typescript
for (const el of array) {
  // ...
}
```

for-of 루프는 코드가 짧고, 인덱스 변수를 사용하지 않기 때문에 실수를 줄여준다. 인덱스 변수가 필요한 경우엔 forEach 메서드를 사용하자.

for-in 문법은 몇가지 문제점이 있기 때문에 사용하지 않는 것이 좋다. (아이템16)

### 🔗 함수 표현식보다 화살표 함수 사용하기

`this`가 클래스 인스턴스를 참조하는 것을 기대하지만 예상치 못한 결과가 나오는 경우도 있다. 대신 화살표 함수를 사용하면 상위 스코프의 `this`를 유지할 수 있다.

인라인(또는 콜백)에서는 일반 함수보다 화살표 함수가 더 직관적이며 코드도 간결해지므로 화살표 함수를 사용하는 것이 좋다.

컴파일러 옵션에 `noImplictThis`(또는 `strict`)를 설정하면, 타입스크립트가 this 바인딩 관련 오류를 표시해준다.

### 🔗 단축 객체 표현과 구조 분해 할당 사용하기

```typescript
const x = 1, y = 2, z = 3;
const pt = {
  x: x,
  y: y,
  z: z  
};
```

변수와 객체의 속성의 이름이 같다면

```typescript
const x = 1, y = 2, z = 3;
const pt = { x, y, z };
```

🧷 화살표 함수 내에서 객체를 반환할 때는 소괄호로 감싸야 한다. 함수의 구현부에는 블록이나 단일 표현식이 필요하기 때문에 소괄호로 감싸서 표현식으로 만들어 준 것이다.

**단축 객체 표현의 반대는 객체 구조 분해이다.**

```typescript
const props = obj.props;
const a = props.a;
const b = props.b;
```

다음처럼 줄여서 작성할 수 있다.

```typescript
const {props} = obj;
const {a, b} = props;
```

극단적으로는

```typescript
const {props: {a, b}} = obj
```

a와 b는 변수로 선언되었지만 props는 변수 선언이 아니라는 것을 주의하자.

🧷 구조 분해 문법에서는 기본값을 지정할 수도 있다.

```typescript
let {a} = obj.props;
if (a === undefined) a = 'default';
```

다음처럼 구조 분해 문법 내에서 기본 값을 지정할 수도 있다.

```typescript
const {a = 'default'} = obj.props;
```

🧷 배열에도 구조 분해 문법을 사용할 수 있다. 튜플처럼 사용하는 배열에서 특히 유용하다.

```typescript
const point = [1, 2, 3];
const [x, y, z] = point;
const [, a, b] = point;    // 첫 번째 요소 무시
```

🧷 함수 매개변수에도 구조 분해 문법을 사용할 수 있다.

```typescript
const points = [
  [1, 2, 3],
  [4, 5, 6]
];
points.forEach(([x, y, z]) => console.log(x+y+z));
// 6, 15
```

단축 객체 표현과 마찬가지로, 객체 구조 분해를 사용하면 문법이 간결해지고 변수를 사용할 때 실수를 줄일 수 있다.

### 🔗 함수 매개변수 기본값 사용하기

자바스크립트에서 함수의 모든 매개변수는 선택적이며, 매개변수를 지정하지 않으면 `undefined`로 간주된다.

모던 자바스크립트에서는 매개변수에 기본값을 직접 지정할 수 있다.

```javascript
// before
function ParseNum(str, base) {
  base = base || 10;
  return parseInt(str, base);
}

// after
function parseNum(str, base=10) {
  return parseInt(str, base);
}
```

매개변수에 기본값을 지정하면 코드가 간결해지고, base가 선택적 매개변수라는 것을 명확히 나타내는 효과도 있다. 기본값을 기반으로 타입 추론이 가능하기 때문에, 마이그레이션 할 때 매개변수에 타입 구문을 쓰지 않아도 된다.

### 🔗 저수준 프로미스나 콜백 대신 async/await 사용하기

async와 await를 사용하면 코드가 간결해지고 버그나 실수를 방지할 수 있고, 비동기 코드에 타입 정보가 전달되어 타입 추론을 가능하게 한다.

### 🔗 연관 배열에 객체 대신 Map과 Set 사용하기

```typescript
function countWords(text: string) {
  const counts: {[word: string]: number} = {};
  for (const word of text.split(/[\s,.]+/)) {
    counts[word] = 1 + (counts[word] || 0);
  }
  return counts;
}
```

counstructor라는 특정 문자열이 주어지면 문제가 발생한다.

```typescript
console.log(countWords('Object have a counstructor'));

// result
{
    Objects: 1,
    have: 1,
    a: 1,
    constructor: "function Object() { [native code] }"
}
```

counstructor의 초깃값은 undefined가 아니라 Object.prototype에 있는 생성자 함수이다. 원치 않는 값이고, 타입도 number가 아닌 string이다. 이런 문제를 방지하기 위해 Map을 사용하자.

```typescript
function countWords(text: string) {
  const counts = new Map<string, number>();
  for (const word of text.split(/[\s,.]+/)) {
    counts[word] = 1 + (counts[word] || 0);
  }
  return counts;
}
```

### 🔗 타입스크립트에 use strict 넣지 않기

엄격 모드가 활성화된다.

타입스크립트에서 수행되는 안전성 검사가 엄격모드보다 훨씬 더 체크를 하기 때문에, 'use strict'는 타입스크립트에서 무의미하다. 실제로는 타입스크립트 컴파일러가 생성하는 자바스크립트 코드에서 'use strict'가 추가된다. alwaysStrict 또는 strict 컴파일러 옵션을 설정하면, 타입스크립트는 엄격 모드로 코드를 파싱하고 생성되는 자바스크립트에 'use strict'를 추가한다.&#x20;

즉, 타입스크립트 코드에 'use strict'를 사용하지 않고, alwaysStrict 설정을 사용해야 한다.



## 📍 요약

* 타입스크립트 개발 환경은 모던 자바스크립트도 실행할 수 있으므로 모던 자바스크립트의 최신 기능들을 적극적으로 사용하자. 코드 품질을 향상시킬 수 있고, 타입스크립트의 타입 추론도 더 나아진다.
* 타입스크립트 개발 환경에서는 컴파일러와 언어 서비스를 통해 클래스, 구조 분해, async/await 같은 기능을 쉽게 배울 수 있다.
* 'use strict'는 타입스크립트 컴파일러 수준에서 사용되므로 코드에서 제거해야 한다.
* TC39의 깃헙 저장소와 타입스크립트의 릴리스 노트를 통해 최신 기능을 확인할 수 있다.
