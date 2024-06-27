# 📎 아이템 7 타입이 값들의 집합이라고 생각하기

런타임에 모든 변수는 자바스크립트 세상의 값으로부터 정해지는 각자의 고유한 값을 가진다. 변수에는 다음처럼 다양한 종류의 값을 할당할 수 있다.

```
42
null
undefined
'Canada'
{animal: 'Whale', weight_lbs: 40_000}
/regex/
new HTMLButtonElement
(x, y) => x + y
```

코드가 실행되기 전, 타입스크립트가 오류를 체크하는 순간에는 '타입'을 가지고 있다. '할당 가능한 값들의 집합'이 타입이라고 생각하면 된다. 이 집합은 타입의 '범위'라고 부르기도 한다.

예를 들어, 모든 숫자값의 집합을 `number` 타입이라고 생각할 수 있다. 42와 37.25는 `number` 타입에 해당되고, `'Canada'`는 그렇지 않다. `null`과 `undefined`는 `strictNullChecks` 여부에 따라 `number`에 해당될 수도, 아닐 수도 있다.

### never 타입

가장 작은 집합은 아무 값도 포함하지 않은 공집합이며, 타입스크립트에서는 **never 타입**이다. `never 타입`으로 선언된 변수의 범위는 공집합이기 때문에 아무런 값도 할당할 수 없다.

```typescript
const x: never = 12;
//    ~ Type 'number' is not assignable to type 'never'.
```

### 유닛 타입 (리터럴 타입)

그 다음으로 작은 집합은 한 가지 값만 포함하는 타입이다. 타입스크립트에서 유닛(unit) 타입이라고도 불리는 리터럴(literal) 타입이다.

```typescript
type A = 'A';
type B = 'B';
type Twelve = 12;
```

두 개, 세 개로 묶으려면 유니온 타입을 사용한다.

```typescript
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;
```

다양한 타입스크립트 오류에서 '할당 가능한'이라는 문구를 볼 수 있다. 이 문구는 집합의 관점에서 '\~의 원소' 또는 '\~의 부분 집합'을 의미한다.

```typescript
const a: AB = 'A';  // OK, value 'A' is a member of the set {'A', 'B'}
const c: AB = 'C';
//    ~ Type '"C"' is not assignable to type 'AB'
```

"C"는 유닛 타입이다. 범위는 단일값 "C"로 구성되며 AB의 부분 집합이 아니므로 오류이다. 집합의 관점에서 타입 체커의 주요 역할은 하나의 집합이 다른 집합의 부분 집합인지 검사하는 것이다.

```typescript
// OK, {"A", "B"} is a subset of {"A", "B"}:
const ab: AB = Math.random() < 0.5 ? 'A' : 'B';
const ab12: AB12 = ab;  // OK, {"A", "B"} is a subset of {"A", "B", 12}

declare let twelve: AB12;
const back: AB = twelve;
//    ~~~~ Type 'AB12' is not assignable to type 'AB'
//           Type '12' is not assignable to type 'AB'
```

실제로 다루게 되는 타입의 대부분은 범위가 무한대이므로 이해하기 어렵다. 범위가 무한대인 타입은 원소들을 일일이 추가해서 만든걸로 생각할 수 있다.

```typescript
type Int = 1 | 2 | 3 | 4 | 5 // | ...
```

다음처럼 원소를 서술하는 방법도 있다.

```typescript
interface Identified {
  id: string;
}
```

앞의 인터페이스가 타입 범위 내의 값들에 대한 설명이다. 어떤 객체가 `string`으로 할당 가능한 `id` 속성을 가지고 있다면 그 객체는 `Identified`이다.

→ 구조적 타이핑 규칙들을 어떠한 값이 다른 속성도 가질 수 있음을 의미한다.&#x20;

### 인터섹션(교집합)

연산과 관련된 이해를 돕기 위해 값의 집합을 타입이라고 생각해보자.

```typescript
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}
type PersonSpan = Person & Lifespan;
```

& 연산자는 두 타입의 인터섹션(교집합)을 계산한다. `Person`과 `Lifespan` 인터페이스는 공통으로 가지는 속성이 없기 때문에, `PersonSpan`을 공집합(`never 타입`)으로 예상할 수 있다. 하지만, 타입 연산자는 인터페이스의 속성이 아닌, 값의 집합(타입의 범위)에 적용된다.&#x20;

→ `Person`과 `Lifespan`을 둘 다 가지는 값은 인터섹션 타입에 속하게 된다.

```typescript
const ps: PersonSpan = {
  name: 'Alan Turing',
  birth: new Date('1912/06/23'),
  death: new Date('1954/06/07'),
};  // OK
```

세 가지보다 더 많은 값을 가지는 값도 `PersonSpan` 타입에 속한다. 인터섹션 타입의 값은 각 타입 내의 속성을 모두 포함하는 것이 일반적인 규칙이다.

### 유니온(합집합)

```typescript
type K = keyof (Person | Lifespan);
//   ^? type K = never
```

유니온 타입에 속하는 값은 어떠한 키도 없기 때문에, 유니온에 대한 `keyof`는 공집합(`never`)이어야만 한다.

이 등식은 타입스크립트의 타입을 이해하는데 도움이 될 것이다.

```typescript
keyof (A&B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```

### 타입 선언

일반적으로, `PersonSpan` 타입을 선언하는 방법은 `extends` 키워드를 쓰는 것이다.

```typescript
interface Person {
  name: string;
}
interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```

타입이 집합이라는 관점에서 `extends`의 의미는 '\~에 할당 가능한'과 비슷하게, '\~의 부분집합'이라는 의미로 받아들일 수 있다. `PersonSpan` 타입의 모든 값은 문자열 `name` 속성을 가져야 한다. 그리고 `birth` 속성을 가져야 제대로 된 부분집합이 된다.

### 서브 타입

1차원, 2차원, 3차원 벡터의 관점에서 생각해보자.

```typescript
interface Vector1D { x: number; }
interface Vector2D extends Vector1D { y: number; }
interface Vector3D extends Vector2D { z: number; }
```

`Vector3D`는 `Vector2D`의 서브타입이고, `Vector2D`는 `Vector1D`의 서브타입이다. 집합의 관점에서는 벤 다이어그램으로 그리는면 적절하다.

`extends` 없이 인터페이스로 코드를 재삭성해보면 부분집합, 서브타입, 할당 가능성의 관계가 바뀌지 않는다는 걸 알 수 있다.

```typescript
interface Vector1D { x: number; }
interface Vector2D { x: number; y: number; }
interface Vector3D { x: number; y: number; z: number; }
```

`extends` 키워드는 제너릭 타입에서 한정자로도 쓰이며, 이 문맥에서는 '\~의 부분 집합'을 의미하기도 한다.

### string 상속의 의미

```typescript
function getKey<K extends string>(val: any, key: K) {
  // ...
}
```

`string`을 상속한다는 의미를 집합의 관점으로 생각해보자.&#x20;

→ `string`의 부분 집합 범위를 가지는 어떠한 타입이 된다. 이 타입은 `string` 리터럴 타입, `string` 리터럴 타입의 유니온, `string` 자신을 포함한다.

```typescript
getKey({}, 'x');  // OK, 'x' extends string
getKey({}, Math.random() < 0.5 ? 'a' : 'b');  // OK, 'a'|'b' extends string
getKey({}, document.title);  // OK, string extends string
getKey({}, 12);
//         ~~ Type 'number' is not assignable to parameter of type 'string'
```

마지막 오류에서 "할당될 수 없습니다"는 상속의 관점에서 "상속할 수 없습니다"로 바꿀 수 있다. 두 표현 모두 '\~의 부분 집합'의 의미로 받아들인다면 문제가 없다.

할당과 상속의 관점을 전환해보면, 객체의 키 타입을 반환하는 `keyof T`를 이해하기 수월하다.

`string|number`와 `string|Date` 사이의 인터섹션은 공집합이 아니며 `string`이고, 서로의 부분 집합도 아니다. 유니온 타입이 상속 관점에서는 어색하지만, 집합 관점에서는 자연스럽다.

### 배열과 튜플의 관계

```typescript
const list = [1, 2];
//    ^? const list: number[]
const tuple: [number, number] = list;
//    ~~~~~ Type 'number[]' is not assignable to type '[number, number]'
//          Target requires 2 element(s) but source may have fewer
// number[] 타입은 [number, number] 타입의 0, 1 속성에 없다.
```

숫자의 배열을 숫자들의 쌍이라고 할 수는 없다. 빈 리스트와 \[1] 이 그 반례이다. number\[]는 \[number, number]의 부분집합이 아니기 때문에 할당할 수 없다. (반대로 할당하면 동작한다.)

트리플은 구조적 타이핑의 관점으로 생각하면 쌍으로 할당 가능할 것으로 생각된다.

```typescript
const triple: [number, number, number] = [1, 2, 3];
const double: [number, number] = triple;
//    ~~~~~~ '[number, number, number]' is not assignable to '[number, number]'
//           Source has 3 element(s) but target allows only 2.
```

타입스크립트는 숫자의 쌍을 `{0: number, 1: number}`로 모델링하지 않고, `{0: number, 1: number, length: 2}`로 모델링했다. `length`의 값이 맞지 않기 때문에 할당문에 오류가 발생했다. 쌍에서 길이를 체크하는 것은 합리적이다.

### 타입스크립트 타입이 되지 못하는 값의 집합

타입이 값의 집합이라는 건, 동일한 값의 집합을 가지는 두 타입은 같다는 의미가 된다. 두 타입이 의미적으로 다르고, 우연히 같은 범위를 가진다고 하더라도, 같은 타입을 두 번 정의할 이유가 없다.

한편, 타입스크립트 타입이 되지 못하는 값의 집합이 있다. 정수에 대한 타입, x와 y 속성 외에 다른 속성이 없는 객체는 타입스크립트 타입에 존재하지 않다.&#x20;

가끔 `Exclude`를 사용해서 일부 타입을 제외할 수 있지만, 그 겨로가가 적절한 타입스크립트 타입일 때만 유효하다.

```typescript
type T = Exclude<string|Date, string|number>;
//   ^? type T = Date
type NonZeroNums = Exclude<number, 0>;
//   ^? type NonZeroNums = number
```

## 📍요약

* 타입을 값의 집합으로 생각하면 이해하기 편하다. 이 집합은 유한(boolean or literal)하거나 무한(number or string)하다.
* 타입스크립트 타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합으로 표현된다. 두 타입은 서로 서브타입이 아니면서도 겹쳐질 수 있다.
* 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도 그 타입에 속할 수 있다.
* 타입 연산은 집합의 범위에 적용된다. A와 B의 인터섹션은 A의 범위와 B의 범위의 인터섹션이다. 객체 타입에서는 A & B 인 값이 A 와 B의 속성을 모두 가짐을 의미한다.
* 'A는 B를 상속', A는 B에 할당 가능', A는 B의 서브타입'은 A는 B의 부분집합'과 같은 의미이다.
