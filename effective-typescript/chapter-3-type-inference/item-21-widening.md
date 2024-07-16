# 📎 아이템 21 타입 넓히기

## 📍 넓히기

타입스크립트는 런타임에 모든 변수는 유일한 값을 가진다. 그러나, 타입스크립트가 작성된 코드를 체크하는 정적 분석 시점에, 변수는 '가능한' 값들의 집합인 타입을 가진다. 상수를 사용해서 변수를 초기화할 때 타입을 명시하지 않으면 타입 체커는 타입을 결정해야 한다.

→ 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추해야 한다.

⭐️ 넓히기(widening) ⭐️

넓히기의 과정을 이해하면 오류의 원인을 파악하고 타입 구문을 더 효과적으로 사용할 수 있다.

## 📍 예제

### ⭐️ 예제 1

벡터를 다루는 라이브러리 작성, 3D 벡터에 대한 타입과 그 요소들의 값을 얻는 함수를 작성한다.

```typescript
interface Vector3 { x: number; y: number; z: number; }
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}
```

Vecotr3 함수를 사용한 다음 코드는 런타임에 오류 없이 실행되지만, 편집기에서는 오류가 표시된다.

```typescript
let x = 'x';
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x);
//                ~ Argument of type 'string' is not assignable
//                  to parameter of type '"x" | "y" | "z"'
```

`getComponent` 함수는 두 번째 매개변수에 `"x" | "y" | "z"` 타입을 기대했지만, `x`의 타입은 할당 시점에 넓히기가 동작해서 `string`으로 추론된다. string 타입은 `"x" | "y" | "z"`에 할당이 불가능하므로 오류가 된 것이다.

### ⭐️ 예제 2

타입 넓히기가 진행될 때, 주어진 값으로 추론 가능한 타입이 여러 개이기 때문에 과정이 모호하다.

```typescript
const mixed = ['x', 1];
```

mixed 타입이 어떻게 추론되는지 살펴보자.

다음은, mixed의 타입 후보이다.

```
('x' | 1)[]
['x', 1]
[string, number]
readonly [string, number]
(string|number)[]
readonly (string|number)[]
[any, any]
any[]
```

타입스크립트는 정보가 충분하지 않은 경우, mixed가 어떤 타입으로 추론되어야 하는지 알 수 없으므로, 작성자의 의도를 추측한다. 이 경우에는 `(string|number)[]`으로 추측한다. (추측한 답이 항상 옳을 수 없다)

### ⭐️ 예제 1 - 넓히기 과정

타입스킓트는 다음 예제와 같은 코드를 예상했기 때문에 `x`의 타입을 `string`으로 추론했다.

```typescript
let x = 'x';
x = 'a';
x = 'Four score and seven years ago...';
```

자바스크립트에서는 다음처럼 작성해도 유효하다.

```javascript
const x = 'x';
//    ^? const x: "x"
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x);  // OK
```

자바스크립트는 `x`의 타입을 `string`으로 추론할 때, 명확성과 유연성 사이의 균형을 유지하려고 한다. 일반적인 규칙은 변수가 선언된 후로는 타입이 바뀌지 않아야 하므로, `string|RegExp`나 `string|string[]`이나 `any` 보다 `string`을 사용하는게 낫다.

## 🔗 타입스크립트는 넓히기의 과정을 제어할 수 있도록 몇 가지 방법을 제공한다.

### ✓ `const`

let 대신 const로 변수를 선언하면 더 좁은 타입이 된다. 실제로 const를 사용하면 앞에서 발생한 오류가 해결된다.

```typescript
const x = 'x';
//    ^? const x: "x"
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x);  // OK
```

x는 재할당될 수 없으므로 타입스크립트는 더 좁은 타입("x")로 추론할 수 있다.&#x20;

→ `"x" | "y" | "z"`에 할당 가능하므로 코드가 타입체커를 통과한다.

### ⭐️ 예제 3

const가 만능은 아니다. 객체와 배열의 경우에는 여전히 문제가 있다.&#x20;

다음 코드는 자바스크립트에서 정상이다.

```javascript
const obj = {
  x: 1,
};
obj.x = 3;  
obj.x = '3';
obj.y = 4;
obj.z = 5;
obj.name = 'Pythagoras';
```

obj의 타입은 구체적인 정도에 따라 다양한 모습으로 추론될 수 있다.&#x20;

* 가장 구체적인 경우: `{readonly x: 1}`
* 추상적인 경우: `{x: number}`
* 가장 추상적인 경우: `{[key: string]: number}` or `object`

객체의 경우 타입스크립트의 넓히기 알고리즘은 각 요소를 let으로 할당된 것처럼 다룬다. → obj의 타입은 `{x: number}`

덕분에, `obj.x`를 다른 숫자로 재할당할 수 있게 되지만 `string`으로는 안된다. 그리고 다른 속성을 추가하지도 못한다.

따라서 다음 코드는 마지막 세 문장에서 오류가 발생한다.

```typescript
const obj = {
  x: 1,
};
obj.x = 3;  // OK
obj.x = '3';
//  ~ Type 'string' is not assignable to type 'number'
obj.y = 4;
//  ~ Property 'y' does not exist on type '{ x: number; }'
obj.z = 5;
//  ~ Property 'z' does not exist on type '{ x: number; }'
obj.name = 'Pythagoras';
//  ~~~~ Property 'name' does not exist on type '{ x: number; }'
```

타입스크립트는 명확성과 유연성 사이의 균형을 유지하려고 한다. 오류를 잡기 위해서는 충분히 구체적으로 타입을 추론해야 하지만, 잘못된 추론을 할 정도로 구체적으로 수행하지는 않는다.

타입 추론의 강도를 직접 제어하려면 타입스크립트의 기본 동작을 재정의해야 한다.

## 📍 타입스크립트의 기본 동작 재정의하는 방법

### ✓ 명시적 타입 구문 제공하기

```typescript
const obj: { x: string | number } = { x: 1 };
//    ^? const obj: { x: string | number; }
```

### ✓ 타입 체커에 추가적인 문맥 제공하기

함수의 매개변수로 값을 전달하기

### ✓ const 단언문 사용하기

const 단언문과 변수 선언의 let or const를 혼동해서는 안된다. const 단언문은 온전히 타입 공간의 기법이다.

```typescript
const obj1 = { x: 1, y: 2 };
//    ^? const obj1: { x: number; y: number; }

const obj2 = { x: 1 as const, y: 2 };
//    ^? const obj2: { x: 1; y: number; }

const obj3 = { x: 1, y: 2 } as const;
//    ^? const obj3: { readonly x: 1; readonly y: 2; }
```

값 뒤에 `as const`를 작성하면, 타입스크립트는 최대한 좁은 타입으로 추론한다. `obj3`에는 넓히기가 동작하지 않는다. `obj3`가 진짜 상수라면, 주석에 보이는 추론된 타입이 실제로 원하는 형태가 된다.&#x20;

배열과 튜플타입을 추론할 때도 `as const`를 사용할 수 있다.

```typescript
const arr1 = [1, 2, 3];
//    ^? const arr1: number[]
const arr2 = [1, 2, 3] as const;
//    ^? const arr2: readonly [1, 2, 3]
```

넓히기로 오류가 발생한다고 생각되면, 명시적 타입 구문 또는 const 단언문을 추가하는 것을 고려해야 한다. 단언문으로 인해 어떻게 변화하는지 편집기에서 주기적으로 타입을 살펴보자.

## 📍 요약

* 타입스크립트가 넓히기를 통해 상수의 탕비을 추론하는 법을 이해해야 한다.
* 동작에 영향을 줄 수 있는 방법인 const, 타입구문, 문맥, as const에 익숙해지자.
