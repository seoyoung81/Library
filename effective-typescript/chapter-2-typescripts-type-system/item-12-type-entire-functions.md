# 📎 아이템 12 함수 표현식에 타입 적용하기

자바스크립트와 타입스크립트에서는 함수 **'문장(statement)'**과 함수 **'표현식(expression)'**을 다르게 인식한다.

```typescript
function rollDice1(sides: number): number { /* ... */ }  // Statement
const rollDice2 = function(sides: number): number { /* ... */ };  // Expression
const rollDice3 = (sides: number): number => { /* ... */ };  // Also expression
```

## 📍 타입스크립트에서는

함수 표현식을 사용하는 것이 좋다. 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있기 때문이다.

### 예시

```typescript
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = sides => { /* ... */ };
```

타입스립트는 `sides`의 타입을 `number`로 인식한다. → **함수 타입 선언의 장점**

### **⭐️ 함수 타입 선언의 장점**

🔗 불필요한 코드의 반복을 줄인다.

```typescript
function add(a: number, b: number) { return a + b; }
function sub(a: number, b: number) { return a - b; }
function mul(a: number, b: number) { return a * b; }
function div(a: number, b: number) { return a / b; }
```

반복되는 함수 시그니처를 하나의 함수 타입으로 통합할 수 있다.

```typescript
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

함수 구현부도 분리되어 로직이 분명해진다. 모든 함수 표현식의 반환타입까지 `number`로 선언한 것이다.

🔗 라이브러리는 공통 함수 시그니처를 타입으로 제공하기도 한다.

리액트는 `MouseEvent` 타입 대신에 함수 전체에 적용할 수 있는 `MouseEventHandler` 타입을 제공한다. 라이브러리를 직접 만드는 경우, 공통 콜백 함수를 위한 타입 선언을 제공하는 것이 좋다.

🔗 시그니처가 일치하는 다른 함수가 있을 때도 함수 표현식에 타입을 적용해볼만 하다.

웹브라우저에서 `fetch` 함수는 특정 리소스에 `HTTP` 요청을 보낸다. `response.json()` 또는 `response.text()`를 사용해 응답의 데이터를 추출한다.

```typescript
const response = fetch('/quote?by=Mark+Twain');
//    ^? const response: Promise<Response>

async function getQuote() {
  const response = await fetch('/quote?by=Mark+Twain');
  const quote = await response.json();
  return quote;
}
// {
//   "quote": "If you tell the truth, you don't have to remember anything.",
//   "source": "notebook",
//   "date": "1894"
// }
```

여기에 버그가 있다. `/quote` 가 존재하지 않는 API라면 '404 Not Found'를 응답한다. 응답은 json 형식이 아닐 수 있기 때문에 response.json()은 JSON 형식이 아니라는 새로운 오류 메세지를 담아, 거절된 프로미스를 반환한다.

→ 새로운 오류 메시지가 전달되어 실제 오류인 404가 감춰진다.

fetch가 실패하면 거절된 프로미스를 응답하지 않는다는 걸 간과하기 쉽다.

→ 상태 체크를 수행해 줄 `checkedFetch` 작성해보자.

```typescript
declare function fetch(
  input: RequestInfo, init?: RequestInit,
): Promise<Response>;

async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init);
  if (!response.ok) {
    // 비동기 함수 내에서는 거절된 프로미스로 변환한다.
    throw new Error(`Request failed: ${response.status}`);
  }
  return response;
}
```

다음과 같이 더 간결하게 작성할 수 있다.

```typescript
const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
  }
  return response;
}
```

1️⃣ 함수 문장을 함수 표현식으로 바꾸고 함수 2️⃣ 전체에 타입(typeof fetch)을 적용한다.&#x20;

→ 타입스크립트가 `input`과 `init`의 타입을 추론할 수 있게 해준다.

3️⃣ `checkedFetch`의 반환 타입을 보장하며, `fetch`와 동일하다. (throw 대신 return을 사용했다면, 타입스크립트가 실수를 잡아낸다.)

```typescript
const checkedFetch: typeof fetch = async (input, init) => {
  //  ~~~~~~~~~~~~
  //  'Promise<Response | HTTPError>' is not assignable to 'Promise<Response>'
  //    Type 'Response | HTTPError' is not assignable to type 'Response'
  const response = await fetch(input, init);
  if (!response.ok) {
    return new Error('Request failed: ' + response.status);
  }
  return response;
}
```

### ⭐️ 정리

함수의 매개변수에 타입을 선언하는 것보다 함수 표현식 전체 타입을 정의하는 것이 코드도 간결하고 안전하다. 다른 함수의 시그니처와 동일한 타입을 가지는 새 함수를 작성하거나, 동일한 타입 시그니처를 가지는 여러 개의 함수를 작성할 때는 매개변수의 타입과 반환 타입을 반복해서 작성하지 말고, 함수 전체의 타입 선언을 적용해야 한다.

## 📍 요약

* 매개변수나 반환 값에 타입을 명시하기보다는 함수 표현식 전체에 타입 구문을 적용하는 것이 좋다.
* 만약 같은 타입 시그니처를 반복적으로 작성한 코드가 있다면 함수 타입을 분리해 내거나 이미 존재하는 타입을 찾자. 라이브버리를 직접 만든다면 공통 콜백에 타입을 제공해야 한다.
* 다른 함수의 시그니처를 참조하려면 `typeof fn`을 사용하면 된다.
