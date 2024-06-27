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

앞의 인터페이스가 타입 범위 내의 값들에 대한 설명이다. 어떤 객체가 string으로 할당 가능한 id 속성을 가지고 있다면 그 객체는 Identified이다.



## 📍요약

* 타입을 값의 집합으로 생각하면 이해하기 편하다. 이 집합은 유한(boolean or literal)하거나 무한(number or string)하다.
* 타입스크립트 타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합으로 표현된다. 두 타입은 서로 서브타입이 아니면서도 겹쳐질 수 있다.
* 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도 그 타입에 속할 수 있다.
* 타입 연산은 집합의 범위에 적용된다. A와 B의 인터섹션은 A의 범위와 B의 범위의 인터섹션이다. 객체 타입에서는 A & B 인 값이 A 와 B의 속성을 모두 가짐을 의미한다.
* 'A는 B를 상속', A는 B에 할당 가능', A는 B의 서브타입'은 A는 B의 부분집합'과 같은 의미이다.
