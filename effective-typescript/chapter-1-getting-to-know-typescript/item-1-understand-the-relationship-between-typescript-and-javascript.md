# 📎 아이템 1 타임스크립트와 자바스크립트의 관계 이해하기

❓ 자바스크립트 ⊂ 타입스크립트 or 타입이 정의된 자바스크립트 ⊂ 타입스크립트 ❓

타입스크립트와 자바스크립트의 관계, 서로 어떻게 연관되어 있는지 알아보자.

## 📍 자바스크립트 프로그램이 타입스크립트 명제는 참이다.

타입스크립트는 문법적으로 자바스크립트의 상위집합이다.&#x20;

→ 자바스크립트는 프로그램에 문법 오류가 없다면, 유효한 타입스크립트 프로그램이지만, 프로그램에 어떤 이슈가 존재한다면 문법 오류가 아니더라도 타입 체커에게 지적당할 가능성이 높다.

문법의 유효성과 동작의 이슈는 독립적인 문제이다.

타입스크립트는 작성된 코드를 파싱하고 자바스크립트로 변환할 수 있다

자바스크립트는 .js(또는 .jsx) 확장자를 사용하는 반면, 타입스크립트는 .ts(또는 .tsx) 확장자를 사용하지만, 완전히 다른 언어는 아니다. 타입스크립트가 상위집합이기 때문에 .js 파일에 있는 코드는 타입스크립트라고 할 수 있다. (main.js → main.ts) 로 바꿔도 달라지는 점이 없다)

이러한 특성(기존 코드를 그대로 유지하면서 일부분에만 타입스크립트 적용이 가능)은 자바스크립트 코드를 타입스크립트로 마이그레이션하는데 엄청난 이점이 된다.&#x20;

## 📍 타입스크립트 프로그램이지만 자바스크립트가 아닌 프로그램이 존재한다.

타입을 명시하는 추가적인 문법을 가지기 때문이다.

```typescript
// 유효한 타입스크립트 프로그램
function greet(who: string) {
    console.log('Hello', who);
}
```

자바스크립트를 구동하는 노드(node)같은 프로그램으로 실행하면 오류를 출력한다.

`SyntaxError: Unexpeted token :`

: string은 타입스크립트에서 쓰이는 타입 구문이다. 타입 구문을 사용하는 순간부터 자바스크립트는 타입스크립트 영역으로 들어가게된다.

&#x20;![](<../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png>)

타입스크립트 컴파일러는 타입스크립트뿐만 아니라 일반 자바스크립트 프로그램에도 유용하다.

## 📍자바스크립트 프로그램 예시

### 🧷 타입스크립트 컴파일러가 일반 자바스크립트 프로그램에 쓰이는 경우

```javascript
let city = 'new york city';
console.log(city.toUppercase());
```

```javascript
// TypeError: city.toUppercase is not a function

let city = 'new york city';
console.log(city.toUppercase());
//               ~~~~~~~~~~~ Property 'toUppercase' does not exist on type
//                           'string'. Did you mean 'toUpperCase'?
```

→ `city` 변수가 문자열이라는 것을 알려주지 않아도 타입스크립트는 초깃값으로부터 타입을 추론한다.&#x20;

타입 시스템의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것이다. 타입스크립트가 '정적' 타입 시스템이라는 것은 이런 특징을 말한다. 그러나 타입 체커가 모든 오류를 찾아내지는 않는다.

### 🧷 오류가 발생하지는 않지만 의도와 다르게 동작하는 코드를 찾아내는 경우

```javascript
const states = [
  {name: 'Alabama', capital: 'Montgomery'},
  {name: 'Alaska',  capital: 'Juneau'},
  {name: 'Arizona', capital: 'Phoenix'},
  // ...
];
for (const state of states) {
  console.log(state.capitol);
}
```

```
실행결과
undefined
undefined
undefined
```

앞의 코드는 유효한 자바스크립트(또한 타입스크립트)이며 어떠한 오류도 없이 실행되나 `state.capitol`은 의도한 코드가 아닌게 분명하다. 이러한 경우 타입스크립트 타입 체커는 추가적인 타입 구문 없이도 오류를 찾아낸다.

```javascript
for (const state of states) {
  console.log(state.capitol);
  //                ~~~~~~~ Property 'capitol' does not exist on type
  //                        '{ name: string; capital: string; }'.
  //                        Did you mean 'capital'?
}
```

타입스크립트는 타입 구문 없이도 오류를 잡을 수 있지만, 타입 구문을 추가한다면 훨씬 더 많은 오류를 찾아낼 수 있다. **코드의 의도가 무엇인지** 타입 구문을 통해 타입스크립트에게 알려줄 수 있기 때문에 코드의 동작과 의도가 다른 부분을 찾을 수 있다.

`capital`과 `captol`을 맞바꾸어보자.

```javascript
const states = [
  {name: 'Alabama', capitol: 'Montgomery'},
  {name: 'Alaska',  capitol: 'Juneau'},
  {name: 'Arizona', capitol: 'Phoenix'},
  // ...
];
for (const state of states) {
  console.log(state.capital);
  //                ~~~~~~~ Property 'capital' does not exist on type
  //                        '{ name: string; capitol: string; }'.
  //                        Did you mean 'capitol'?
}
```

이 때, 타입스립트는 어느쪽이 오타인지 판단하지 못한다. 오류의 원인을 추측할 수는 있겠지만 항상 정확하지는 않다. 따라서 명시적으로 `states`를 선언하여 의도를 분명하게 하는 것이 좋다.

```javascript
interface State {
  name: string;
  capital: string;
}
const states: State[] = [
  {name: 'Alabama', capitol: 'Montgomery'},
  //                ~~~~~~~
  {name: 'Alaska',  capitol: 'Juneau'},
  //                ~~~~~~~
  {name: 'Arizona', capitol: 'Phoenix'},
  //                ~~~~~~~ Object literal may only specify known properties,
  //                        but 'capitol' does not exist in type 'State'.
  //                        Did you mean to write 'capital'?
  // ...
];
for (const state of states) {
  console.log(state.capital);
}
```

오류가 어디에서 발생했는지 찾을 수 있고, 제시된 해결책도 올바르다. 의도를 명확히 해서 타입스크립트가 잠재적 문제점을 찾을 수 있게 했다.

예를 들어, 타입 구문 없이 배열 안에 딱 한 번 오타를 내면 오류가 되지 않을 것이다. 타입 구문을 추가하면 오류를 찾을 수 있다.

```javascript
const states: State[] = [
  {name: 'Alabama', capital: 'Montgomery'},
  {name: 'Alaska',  capitol: 'Juneau'},
  //                ~~~~~~~ Did you mean to write 'capital'?
  {name: 'Arizona', capital: 'Phoenix'},
  // ...
];
```

이 내용을 정리하면, 벤 다이어그램에서 새로운 영역을 추가할 수 있다. 평소 작성하는 타입스크립트 코드가 이 영역에 해당한다. 타입 체크에서 오류가 발생하지 않도록 타입스크립트 코드를 작성하기 때문이다.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt="" width="329"><figcaption><p>모든 자바스크립트는 타입스크립트이지만, 일부 자바스크립트(그리고 타입스크립트)만이 타입체크를 통과합니다.</p></figcaption></figure>

### 🧷  타입스크립트 타입 시스템은 자바스크립트의 런타임 동작을 '모델링'한다.&#x20;

```javascript
const x = 2 + '3';  // OK
//    ^? const x: string
const y = '2' + 3;  // OK
//    ^? const y: string
```

이 예제는 다른 언어였다면 런타임 오류가 될 만한 코드이지만 타입스크립트의 타입 체커는 정상으로 인직한다. 두 줄 모두 문자열  "23"이 되는 자바스크립트 런타임 동작으로 모델링된다.

반대로 정상 동작하는 코드에 오류를 표시하기도 한다. 다음은 런타임 오류가 발생하지 않는 코드인데, 타입 체커는 문제점을 표시한다.

```javascript
const a = null + 7;  // Evaluates to 7 in JS
//        ~~~~ The value 'null' cannot be used here.
const b = [] + 12;  // Evaluates to '12' in JS
//        ~~~~~~~ Operator '+' cannot be applied to types ...
alert('Hello', 'TypeScript');  // alerts "Hello"
//             ~~~~~~~~~~~~ Expected 0-1 arguments, but got 2
```

자바스크립트의 런타임 동작을 모델링하는 것은 타입스크립트 타입 시스템의 기본 원칙이다. 단순히 런타임 동작을 모델링 하는 것뿐만 아니라 의도치 않은 이상한 코드가 오류로 이어질 수도 있다는 점까지 고려한다. (프로그램 오류가 아니더라고 타입 체커가 오류를 표시한다.)

❓언제 자바스크립트 런타임 동작을 그대로 모델링할지, 또는 추가적인 타입 체크를 할지 분명하지 않다면 과연 타입스크립트를 사용해도 될까❓

타입스크립트 사용 여부는 선택이다&#x20;

→ 타입 체커를 통해 오류가 적은 코드를 작성할 수 있지만, 앞의 이상한 코드들처럼 사용한다면 안쓰는 것이 낫다

### 🧷  타입 체크를 통과하고 여전히 런타임 오류가 발생하는 경우

```typescript
const names = ['Alice', 'Bob'];
console.log(names[2].toUpperCase());
// TypeError: Cannot read property 'toUpperCase' of undefined
```

타입스크립트는 앞의 배열이 범위 내에서 사용될 것이라 가정했지만 실제로는 그렇지 않아 오류가 발생했다.

앞서 등장한 오류들이 발생하는 근본 원인은 타입스크립트가 이해하는 값의 타입과 실제 값에 차이가 있기 때문이다. 타입 시스템이 정적 타입의 정확성을 보장해줄 것 같지만 그렇지 않다. 애초에 타입 시스템은 그런 목적으로 만들어지지 않았다.&#x20;

## 📍요약

* 타입스크립트는 자바스크립트의 상위집합이다. 모든 자바스크립트는 타입스크립트이지만 반대로, 타입스크립트는 별도의 문법을 가지고 있기 때문에 일반적으로 유효한 자바스크립트 프로그램이 아니다.
* 타입스크립트는 자바스크립트 런타임 동작을 모델링하는 타입 시스템을 가지고 있기 때문에 런타임 오류를 발생시키는 코드를 찾아내려고 한다. 그러나 모든 오류를 찾아내려고 기대하면 안된다. 타입 체커를 통과하면서도 런타임 오류를 발생시키는 코드는 충분히 존재할 수 있다.
* 타입스크립트 타입 시스템은 전반적으로 자바스크립트 동작을 모델링한다. 그러나 잘못된 매개변수 개수로 함수를 호출하는 경우처럼, 자바스크립트에서는 허용되지만 타입스크립트에서는 문제가 되는 경우가 있다. 이러한 문법의 엄격함은 취향의 차이이다.
