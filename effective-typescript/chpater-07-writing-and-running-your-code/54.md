# 📎 아이템 54 객체를 순회하는 노하우

✏️ k의 타입을 구체적으로 명시해야 obj 객체를 순회할 수 있다.

```typescript
const obj = {
  one: string;
  two: string;
  three: string;
}

let k: keyof typeof obj;  // "one" | "two" | " three"
for (k in obj) {
  const v = obj[k];
}
```

k의 타입을 명시하지 않는 경우 → k는 string으로 추론된다.

**📍 string 타입으로 추론하는 이유**

```typescript
interface ABC {
  a: string;
  b: string;
  c: number;
}

function foo(abc: ABC) {
  for (const k in abc) { // const k: string
    const v = abc[k];
    // element: any
  }
}

const x = {a: 'a', b: 'b', c: 2, d: new Date()}
foo(x); // OK
```

foo 함수는 a, b, c 속성 외에 d를 가지는 x 객체로 호출이 가능하다. foo 함수가 ABC 타입에 '할당 가능한' 어떠한 값이든 매개변수로 허용했기 때문이다.&#x20;

⭐️ ABC 타입에 할당 가능한 객체에는 a, b, c외에 다른 속성이 존재할 수 있기 때문에, string 타입으로 선택해야 한다.

**📍 keyof 키워드의 문제점**

```typescript
function foo(abc: ABC) {
  let k: keyof ABC;
  for (k in abc) {
    const v = abc[k];  // string|number
}
```

k가 "a" | "b" | "c" 타입으로 한정되면, v는 string|number로 한정된다. d: new Date()가 있을 수 있기 때문에 v 가 string|number로 추론되는 것은 잘못된 것이며 런타임의 동작을 예상하기 어렵다.

## 📍객체 순회하기

`Object.entries` 사용하기

```typescript
function foo(abc: ABC) {
    for (const [k, v] of Object.entries(abc)) {
        k    // string
        v    // any
    }
}
```

또한, 객체를 다룰 때는 프로토타입이 오염의 가능성을 열어두자. for-in 구문을 사용하면, 객체의 정의에 없는 속성이 갑자기 등장할 수 있다.

```typescript
Object.prototype.z = 3;    // 이렇게 하지 말자.
const obj = {x: 1, y: 2};
for (const k in obj) { console.log(k) };
```

실제 작업에서는 Object.prototype에 순회 가능한 속성을 절대로 추가하면 안된다. for-in 루프에서 k가 string 키를 가지게 되면 프로토타입 오염의 가능성을 의심해야 한다.

객체를 순회하며 키와 값을 얻으려면, (let k: keyof T) 같은 keyof 선언이나 Object.entries를 사용하면 된다.

✓ keyof 선언

상수이거나 추가적인 키 없이 정확한 타입을 원하는 경우

✓ Object.entries

더욱 일반적인 경우, 키와 값의 타입을 다루기 까다롭다.

## 📍 요약

* 객체 순회시, 키가 어떤 타입인지 정확하게 파악하고 있다면 `keyof T`와 `for-in` 루프를 사용하자. 또한 함수의 매개변수로 쓰이는 객체에는 추가적인 키가 존재할 수 있다는 점을 명심하자.
* 객체를 순회하며 키와 값을 얻는 가장 일반적인 방법은 `Object.entries`를 사용하는 것이다.
