# 📎 아이템 9 타입 단언보다는 타입 선언을 사용하기

타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법은 두가지이다.

```typescript
interface Person { name: string };

const alice: Person = { name: 'Alice' };    // 타입은 Person
const bob = { name: 'Bob' } as Person;      // 타입은 Person
```

이 두 가지 방법은 결과가 같아 보이지만 그렇지 않다.

## 📍 타입 선언과 타입 단언

1. `alice: Person`은 변수에 **'타입 선언'**을 붙여서 그 값이 선언된 타입임을 명시한다.
2. `as Person`은 **'타입 단언'**을 수행한다. → 타입스크립트가 추론한 타입이 있더라고 Person타입으로 간주한다.

타입 단언보다 타입 선언을 사용하는 것이 낫다.

```typescript
const alice: Person = {};
//    ~~~~~ Property 'name' is missing in type '{}' but required in type 'Person'
const bob = {} as Person;  // No error
```

**타입 선언**은 할당되는 값이 해당 인터페이스를 만족하는지 검사한다. → 앞의 예제에서는 그러지 못하여 오류가 발생

**타입 단언**은 강제로 타입을 지정했으니 타입 체커에게 오류를 무시하라고 하는 것이다.

속성을 추가할 때도 마찬가지이다.

```typescript
const alice: Person = {
  name: 'Alice',
  occupation: 'TypeScript developer'
// ~~~~~~~~~ Object literal may only specify known properties,
//           and 'occupation' does not exist in type 'Person'
};
const bob = {
  name: 'Bob',
  occupation: 'JavaScript developer'
} as Person;  // No error
```

**타입 선언**에서는 잉여 속성 체크가 동작했지만, **단언문에서는** 적용되지 않는다.

타입 단언이 꼭 필요한 경우가 아니라면, 안정성 체크도 되는 타입 선언을 사용하는 것이 좋다.

### 화살표 함수

화살표 함수의 타입 선언은 추론된 타입이 모호할 때가 있다. 예를 들어, 다음 코드에서 Person 인터페이스를 사용하고 싶다고 가정해보자.

```typescript
const people = ['alice', 'bob', 'jan'].map(name => ({name}));
// { name: string; }[]... but we want Person[]
```

`{name}`에 타입 단언을 쓰면 문제가 해결되는 것처럼 보인다.

```typescript
const people = ['alice', 'bob', 'jan'].map(
  name => ({name} as Person)
); // Type is Person[]
```

그러나 타입 단언을 사용하면 앞선 예제들처럼 런타임에 문제가 발생한다.

```typescript
const people = ['alice', 'bob', 'jan'].map(name => ({} as Person));
// No error
```

단언문을 쓰지 않고, 다음과 같이 화살표 함수 안에서 타입과 함께 변수를 선언하는 것이 가장 직관적이다.

```typescript
const people = ['alice', 'bob', 'jan'].map(name => {
  const person: Person = {name};
  return person
}); // Type is Person[]
```

그러나 원래 코드에 비해 번잡하게 보인다. 코드를 좀 더 간결하게 보이기 위해 변수 대신 화살표 함수의 반환 타입을 선언해보자.

```typescript
const people = ['alice', 'bob', 'jan'].map(
  (name): Person => ({name})
); // Type is Person[]
```

여기서 소괄호는 `(name): Person name`의 타입이 없고, 반환 타입이 `Person`이라고 명시한다.&#x20;

`(name: Person)`은 `name`의 타입이 `Person`임을 명시하고 반환 타입은 없기 때문에 오류가 발생한다.

최종적으로, 원하는 타입을 직접 명시하고 타입스크립트가 할당문의 유효성을 검사하게 한다.

```typescript
const people: Person[] = ['alice', 'bob', 'jan'].map(name => ({name})); // OK
```

함수 호출 체이닝이 연속되는 곳에서는 체이닝 시작에서부터 명명된 타입을 가져야한다. 그래야 정확한 곳에 오류가 표시된다.

### 타입 단언이 꼭 필요한 경우

타입 단언은 타입체커가 추론한 타입보다 우리가 판단하는 타입이 더 정확할 때 의미가 있다.&#x20;

#### DOM 엘리먼트에 접근하는 경우

```typescript
document.querySelector('#myButton')?.addEventListener('click', e => {
  e.currentTarget 
  // ^? (property) Event.currentTarget: EventTarget | null
  // currentTarget is #myButton is a button element
  const button = e.currentTarget as HTMLButtonElement; 
  //    ^? const button: HTMLButtonElement
});
```

타입스크립트는 DOM에 접근할 수 없기 때문에 `#myButton`이 버튼 엘리먼트인지 알지 못한다. 그리고 이벤트의 `currentTarget`이 같은 버튼이어야 하는 것도 알지 못한다.&#x20;

#### 자주 쓰이는 특별한 문법(!)을 사용해서 null이 아님을 단언하는 경우

```typescript
const elNull = document.getElementById('foo');
//    ^? const elNull: HTMLElement | null
const el = document.getElementById('foo')!;
//    ^? const el: HTMLElement
```

변수의 접두사 `!` 은 `boolean`의 부정문이지만 접미사 `!`은 그 값이 `null`이 아니라는 단언문으로 해석된다. 단언문은 컴파일 과정 중에 제거되므로, 타입 체커는 알지 못하지만 그 값이 `null`이 아니라고 확신할 수 있을 때 사용해야 한다. 그렇지 않다면 `null`인 경우를 체크하는 조건문을 사용해야 한다.

### 타입 단언문으로 타입 간 변환

타입 단언문으로 임의의 타입 간에 변환 할 수 없다. A가 B의 부분 집합인 경우에 타입 단언문을 사용해 변환할 수 있다.&#x20;

* HTMLElement는 HTMLElement | null의 서브타입이기 때문에 이러한 타입 단언은 동작한다.&#x20;
* HTMLButtonElement는 EventTarget의 서브타입이기 때문에 동작한다.&#x20;
* Person은 {}의 서브타입이므로 동작한다.
* Person과 HTMLElement는 서로의 서브타입이 아니기 때문에 변환이 불가능하다.

```typescript
interface Person { name: string; }
const body = document.body;
const el = body as Person;
//         ~~~~~~~~~~~~~~
// Conversion of type 'HTMLElement' to type 'Person' may be a mistake because
// neither type sufficiently overlaps with the other. If this was intentional,
// convert the expression to 'unknown' first.
```

이 오류를 해결하려면 `unknown`타입을 사용해야한다. 모든 타입은 `unknown`의 서브타입이므로 `unknown`이 포함된 단언문은 항상 동작한다. `unknown`단언은 임의의 타입 간에 변환을 가능케 하지만, 위험한 동작임을 인지하자.

```typescript
const el = document.body as unknown as Person;  // OK
```

## 📍 요약

* 타입 단언(as Type)보다 타입 선언(: Type)을 사용하자.
* 화살표 함수의 반환 타입을 명시하는 방법을 터득하자.
* 타입스크립트보다 타입 정보를 더 잘 알고 있는 상황에서는 타입 단언문과 null 아님 단언문을 사용하면 된다.
