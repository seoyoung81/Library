# 📎 아이템 26 타입 추론에 문맥이 어떻게 사용되는지 이해하기

타입스크립트는 타입을 추론할 때 값과 값이 존재하는 곳의 문맥까지 살핀다. 그런데 문맥을 고려해 타입을 추론하면 가끔 이상한 결과가 나온다. 타입 추론에 문맥이 어떻게 사용되는지 이해하고, 대치하자.

### 예시

자바스크립트는 코드의 동작과 실행 순서를 바꾸지 않으면서 표현식을 상수로 분리할 수 있다. 예를 들어, 다음 두 문장은 동일하다.

```typescript
// 인라인 형태
setLanguage('JavaScript');

// 참조 형태
let language = 'JavaScript';
setLanguage(language);
```

타입스크립트에서는 다음 리팩터링이 여전히 동작한다.

```typescript
function setLanguage(language: string) { /* ... */ }

setLanguage('JavaScript');  // OK

let language = 'JavaScript';
setLanguage(language);  // OK
```

문자열 타입을 더 특정해서 문자열 리터럴 타입의 유니온으로 바꾼다고 가정해보자.

```typescript
type Language = 'JavaScript' | 'TypeScript' | 'Python';
function setLanguage(language: Language) { /* ... */ }

setLanguage('JavaScript');  // OK

let language = 'JavaScript';
setLanguage(language);
//          ~~~~~~~~ Argument of type 'string' is not assignable
//                   to parameter of type 'Language'
```

인라인 형태에서 타입스크립트는 함수 선언을 통해 매개변수가 `Language` 타입이어야 한다는 것을 알고 있다.&#x20;

그러나, 이 값을 변수로 분리해내면 타입스크립트는 할당 시점에 타입을 추론한다. 이 경우 `string`으로 추론했고, `Language` 타입으로 할당이 불가능하므로 오류가 발생한다.

### 오류를 해결하는 두 가지 방법

#### ⭐️ 첫 번째 방법, 타입 선언에서 language의 가능한 값을 제한하는 것

```typescript
let language: Language = 'JavaScript';
setLanguage(language);  // OK
```

만약 language 값에 오타가 있어도 오류를 표시해주는 장점이 있다.

#### ⭐️ language 상수로 만들기

```typescript
const language = 'JavaScript';
//    ^? const language: "JavaScript"
setLanguage(language);  // OK
```

const를 사용하여 타입 체커에게 language는 변경할 수 없다고 알려준다. 따라서 타입스크립트는 language에 대해 더 정확한 타입인 문자열 리터럴 "JavaScript"로 추론할 수 있다. "JavaScript"는 Language에 할당할 수 있으므로 타입 체크를 통과해야 한다. language를 재할당해야 한다면 타입 선언이 필요하다.

이 과정에서 사용되는 문맥으로부터 값을 분리했다. 문맥과 값을 분리하면 추후에 근복적인 문제를 발생시킬 수 있다.

문맥의 소실로 인해 오류가 발생하는 경우와, 해결방법을 살펴보자.

### ✓ 튜플 사용 시 주의점

이동이 가능한 지도를 보여주는 프로그램 코드 예시

```typescript
// Parameter is a (latitude, longitude) pair.
function panTo(where: [number, number]) { /* ... */ }

panTo([10, 20]);  // OK

const loc = [10, 20];
//    ^? const loc: number[]
panTo(loc);
//    ~~~ Argument of type 'number[]' is not assignable to
//        parameter of type '[number, number]'
```

타입스크립트가 `loc`의 타입을 `number[]`로 추론하여(길이를 알 수 없는 숫자의 배열) 많은 배열이 이와 맞지 않는 수의 요소를 가지므로 튜플 타입에 할당할 수 없다.

#### 해결 방법

⭐️ 타입스크립트의 의도를 정확히 파악할 수 있도록 타입 선언을 제공하는 방법을 시도해보자. (any를 사용하지 않기 위한)

```typescript
const loc: [number, number] = [10, 20];
panTo(loc);  // OK
```

⭐️ 상수 문맥 제공하기

as const를 사용하여 내부까지 상수라는 사실을 타입스크립트에게 알려준다.

```typescript
const loc = [10, 20] as const;
//    ^? const loc: readonly [10, 20]
panTo(loc);
//    ~~~ The type 'readonly [10, 20]' is 'readonly'
//        and cannot be assigned to the mutable type '[number, number]'
```

이 추론은 너무 과하게 정확하다. panTo의 타입 시그니처는 where의 내용이 불변이라고 보장하지 않는다. 즉, loc 매개변수가 readonly 타입이므로 동작하지 않는다.

⭐️ panTo 함수에 readonly 구문 추가하기

```typescript
function panTo(where: readonly [number, number]) { /* ... */ }
const loc = [10, 20] as const;
panTo(loc);  // OK
```

as const는 문맥 손실과 관련한 문제를 해결할 수 있지만, 타입 정의에 실수가 있다면 오류는 타입 정의가 아니라 호출되는 곳에서 발생한다. 특히 여러 겹 중첩된 객체에서 오류가 발생한다면 근복적인 원인을 파악하기 어렵다.

```typescript
const loc = [10, 20, 30] as const;  // error is really here.
panTo(loc);
//    ~~~ Argument of type 'readonly [10, 20, 30]' is not assignable to
//        parameter of type 'readonly [number, number]'
//          Source has 3 element(s) but target allows only 2.
```

### ✓ 객체 사용 시 주의점

```typescript
type Language = 'JavaScript' | 'TypeScript' | 'Python';
interface GovernedLanguage {
  language: Language;
  organization: string;
}

function complain(language: GovernedLanguage) { /* ... */ }

complain({ language: 'TypeScript', organization: 'Microsoft' });  // OK

const ts = {
  language: 'TypeScript',
  organization: 'Microsoft',
};
complain(ts);
//       ~~ Argument of type '{ language: string; organization: string; }'
//            is not assignable to parameter of type 'GovernedLanguage'
//          Types of property 'language' are incompatible
//            Type 'string' is not assignable to type 'Language'
```

ts 객체에서 language의 타입은 string으로 추론된다. 이 문제는 타입 선언을 추가하거나(const ts: GovernedLanguage = ... ) 상수 단언(as const)을 사용해 해결한다.

### ✓ 콜백 사용 시 주의점

콜백을 다른 함수로 전달할 때, 타입스크립트는 콜백의 매개변수 타입을 추론하기 위해 문맥을 사용한다.

```typescript
function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
  fn(Math.random(), Math.random());
}

callWithRandomNumbers((a, b) => {
  //                   ^? (parameter) a: number
  console.log(a + b);
  //              ^? (parameter) b: number
});
```

`callWithRandom`의 타입 선언으로 인해 a와 b의 타입이 number로 추론된다. 콜백을 상수로 뽑아내면 문맥이 소실되고 noImplicitAny 오류가 발생하게 된다.

```typescript
const fn = (a, b) => {
  //        ~    Parameter 'a' implicitly has an 'any' type
  //           ~ Parameter 'b' implicitly has an 'any' type
  console.log(a + b);
}
callWithRandomNumbers(fn);
```

이런 경우는 매개변수에 타입 구문을 추가해서 해결할 수 있다.

```typescript
const fn = (a: number, b: number) => {
  console.log(a + b);
}
callWithRandomNumbers(fn);
```

또는 가능한 경우 전체 함수 표현식에 타입 선언을 적용하는 것이다.(아이템 12)

## 📍 요약

* 타입 추론에서 문맥이 어떻게 쓰이는지 주의해서 살펴봐야 한다.
* 변수를 뽑아서 별도로 선언했을 때 오류가 발생한다면 타입 선언을 추가해야 한다.
* 변수가 정말로 상수라면 상수 단언(as const)을 사용해야 한다. 그러나 상수 단언을 사용하면 정의한 곳이 아니라 사용한 곳에서 오류가 발생하므로 주의해야 한다.
