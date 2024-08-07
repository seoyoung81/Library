# 📎 아이템 42 모르는 타입의 값에는 any 대신 unkown을 사용하기

`unknown`에는 함수의 반환값과 관련된 형태, 변수 선언과 관련된 형태, 단언문과 관련된 형태가 있다.

## `unknown`

### 📍 함수의 반환값과 관련된 `unknown`

```typescript
// 반환 타입을 any로
function parseYAML(yaml: string): any {
  // ...
}
```

반환 타입을 any로 사용하는 것은 좋지 않은 설계이다.

```typescript
// 반환값을 원하는 타입으로 
interface Book {
  name: string;
  author: string;
}
const book: Book = parseYAML(`
  name: Wuthering Heights
  author: Emily Brontë
`);
```

그러나, 함수의 반환값에 타입 선언을 강제할 수 없어서 호출한 곳에서 타입 선언을 생략하면 book 변수는 암시적 any 타입이 되고, 사용되는 곳마다 타입 오류가 발생하게 된다.

```typescript
const book = parseYAML(`
  name: Jane Eyre
  author: Charlotte Brontë
`);
console.log(book.title);  // No error, logs "undefined" at runtime
book('read');  // No error, throws "book is not a function" at runtime
```

대신 `parseYAML`이 `unkown` 타입을 반환하게 만드는 것이 더 안전하다.

```typescript
function safeParseYAML(yaml: string): unknown {
  return parseYAML(yaml);
}
const book = safeParseYAML(`
  name: The Tenant of Wildfell Hall
  author: Anne Brontë
`);
console.log(book.title);
//          ~~~~ 'book' is of type 'unknown'
book("read");
// Error: 'book' is of type 'unknown'
```

`unkown` 타입을 이해하기 위해서는 할당 가능성의 관점에서 `any`를 생각해 볼 필요가 있다. `any`가 강력하면서도 위험한 두 가지 특징으로부터 비롯된다.

1. 어떠한 타입이든 `any` 타입에 할당 가능하다.
2. `any` 타입은 어떠한 타입으로도 할당 가능하다. (`never` 예외)

'타입을 값의 집합으로 생각하기' 관점에서, `any`는 타입 시스템과 상충된다. → `any` 타입의 강력하면서도 문제를 일으키는 원인

`unkown`은 `any` 대신 쓸 수 있는 타입 시스템에 부합하는 타입이다. `unkown` 타입은 `any`의 첫번째 속성을 만족하지만, 두 번째 속성은 만족하지 않는다. `unkown` 타입인 채로 값을 사용하면 오류가 발생한다. `unkown`인 값에 함수 호출하거나 연산을 하려고 해도 마찬가지이다. `unkown` 상태로 사용하려고 하면 오류가 발생하기 때문에, 적절한 타입으로 변환하도록 강제할 수 있다.

```typescript
const book = safeParseYAML(`
  name: Villette
  author: Charlotte Brontë
`) as Book;
console.log(book.title);
//               ~~~~~ Property 'title' does not exist on type 'Book'
book('read');
// Error: This expression is not callable
```

함수의 반환 타입인 `unkown` 그대로 값을 사용할 수 없기 때문에 `Book`으로 타입 단언을 해야 한다. (애초에 반환값이 `Book`이라고 기대하며 함수를 호출하기 때문에 단언문은 문제가 안된다.)

`Book` 타입 기준으로 타입 체크가 되기 때문에, `unkown` 타입 기준으로 오류를 표시했던 예제보다 오류의 정보가 더 정확하다.

### 📍 변수 선언과 관련된 `unknown`

어떠한 값이 있지만 그 타입을 모르는 경우에 `unkown`을 사용한다.

```typescript
// JSON 속성은 직렬화가 가능한 모든 것을 담으므로 타입을 예상할 수 없어서 unkown을 사용한다.
interface Feature {
  id?: string | number;
  geometry: Geometry;
  properties: unknown;
}
```

⭐️ 타입 단언문이 `unkown`에서 원하는 타입으로 변환하는 유일한 방법은 아니다. `instanceof`를 체크한 후 `unkown`에서 원하는 타입으로 변환할 수 있다.

```typescript
function processValue(value: unknown) {
  if (value instanceof Date) {
    value
    // ^? (parameter) value: Date
  }
}
```

⭐️ 사용자 정의 카드도 `unkown`에서 원하는 타입으로 변환할 수 있다.

```typescript
function isBook(value: unknown): value is Book {
  return (
      typeof(value) === 'object' && value !== null &&
      'name' in value && 'author' in value
  );
}
function processValue(value: unknown) {
  if (isBook(value)) {
    value;
    // ^? (parameter) value: Book
  }
}
```

`unkown` 타입의 범위를 좁히기 위해서는 많은 노력이 필요하다.&#x20;

* `in` 연산자에서 오류를 피하기 위해 `val`이 객체임을 확인
* `typeof null === 'object'`이므로 별도로 `val`이 `null`이 아님을 확인

📍 가끔 `unknown` 대신 제너릭 매개변수가 사용되는 경우도 있다.

```typescript
// 제너릭을 사용하기 위해 safeParseYAML 선언
function safeParseYAML<T>(yaml: string): T {
  return parseYAML(yaml);
}
```

타입스크립트를 위한 코드는 아니다. 제너릭을 사용한 스타일은 타입 단언문과 달라 보이지만 기능적으로는 동일하다. 제너릭보다는 `unknown`을 반환하고 사용자가 직접 단언문을 사용하거나 원하는대로 타입을 좁히도록 강제하는 것이 좋다.

### 📍 단언문과 관련된 `unknown`

이중 단언문에서 any 대신 unknown을 사용할 수도 있다.

```typescript
declare const foo: Foo;
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

`barAny`와 `barUnk`는 기능적으로 동일하지만, 나중에 두 개의 단언문을 분리하면 `unknown` 형태가 더 안전하다.&#x20;

`any` → 분리되는 순간 영향력이 전염병처럼 퍼진다.

`unknown` → 분리되는 즉시 오류를 발생하게 되므로 더 안전하다.

### 📍 `unknown`과 유사하지만 다른 타입들

`object` 또는 `{}`를 사용하는 코드들이 존재한다. `object` 또는 `{}`를 사용하는 방법 역시 `unknown` 만큼 범위가 넓은 타입이지만, `unknown` 보다는 범위가 약간 좁다.

* {} 타입은 null과 undefined를 제외한 모든 값을 포함한다.
* object 타입은 모든 비기본형 타입으로 이루어진다. 여기에는 true 또는 12 또는 'foo'가 포함되지 않지만 객체와 배열은 포함된다.

unknown 타입이 도입되기 전에는 {}가 더 일반적으로 사용되었지만, 최근에는 {}를 사용하는 경우가 드물다. 정말로 null과 undefined가 불가능하다고 판단되는 경우만 unknown 대신 {}를 사용하면 된다.

## 📍 요약

* unknown은 any 대신 사용할 수 있는 안전한 타입이다. 어떠한 값이 있지만 그 타입을 알지 못하는 경우에 unknown을 사용하면 된다.
* 사용자가 타입 단언문이나 타입 체크를 사용하도록 강제하려면 unknown을 사용하면 된다.
* {}, object, unknown의 차이점을 이해하자.
