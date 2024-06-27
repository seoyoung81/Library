# 📎 아이템 8 타입 공간과 값 공간의 심벌 구분하기

타입스크립트의 심벌(symbol)은 타입 공간이나 값 공간 중의 한 곳에 존재한다.

심벌은 이름이 같더라도 속하는 공간에 따라 다른 것을 나타낼 수 있다.

```typescript
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({radius, height});
```

`interface Cylinder`에서 `Cylinder`는 타입으로 쓰인다. `const Cylinder`에서 `Cylinder`와 이름은 같지만 값으로 쓰이며, 서로 아무런 관련이 없다.

`Cylinder`는 타입으로 쓰일 수도 있고, 값으로도 쓰일 수도 있으며 이런 점이 가끔 오류를 야기한다.

```typescript
function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape.radius
    //    ~~~~~~ Property 'radius' does not exist on type '{}'
  }
}
```

`instanceof`를 이용해 `shape`가 `Cylinder` 타입인지 체크하려고 했을 것이다. 그러나 `instanceof`는 자바스크립트의 런타임 연산자이고, 값에 대한 연산을 한다. `instanceof Cylinder`는 타입이 아니라 함수를 참조한다.

한 심벌이 타입인지 값인지는 어떤 형태로 쓰이는지 문맥을 살펴 알아내야 한다.

### 타입과 값 구분하기

```typescript
type T1 = 'string literal';
const v1 = 'string literal';
type T2 = 123;
const v2 = 123;
```

일반적으로

* `type`이나 `instanceof` 다음에 오는 심벌은 타입
* `const`나 `let` 선언에 쓰이는 것은 값

두 공간에 대한 개념을 잡으려면 타입스크립트 플레이그라운드를 활용하면 된다. → 타입스크립트 소스로부터 변환된 자바스크립트 결과물을 보여준다. 컴파일 과정에서 타입 정보가 제거되기 때문에 심벌이 사라진다면, 타입에 해당된다.

* 타입 선언(:) 또는 단언문(as) 다음에 나오는 심벌은 타입
* \= 다음에 나오는 모든 것은 값

```typescript
interface Person {
  first: string;
  last: string;
}
const jane: Person = { first: 'Jane', last: 'Jacobs' };
//    ――――           ――――――――――――――――――――――――――――――――― Values
//          ―――――― Type
```

일부 함수에서 타입과 값이 반복적으로 번갈아 가며 나올 수도 있다.

```typescript
function email(to: Person, subject: string, body: string): Response {
  //     ――――― ――          ―――――――          ――――                    Values
  //               ――――――           ――――――        ――――――   ―――――――― Types
  // ...
}
```

#### 🧷 class 와 enum

`class`와 `enum`은 상황에 따라 타입과 값 두 가지 모두 가능한 예약어이다.

`Cylinder` 클래스는 타입으로 쓰였다.

```typescript
class Cylinder {
  radius: number;
  height: number;
  constructor(radius: number, height: number) {
    this.radius = radius;
    this.height = height;
  }
}

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape
    // ^? (parameter) shape: Cylinder
    shape.radius
    //    ^? (property) Cylinder.radius: number
  }
}
```

* 클래스가 타입으로 쓰일 때: 형태(속성과 메서드)가 사용된다.
* 값으로 쓰일 때: 생성자가 사용된다.

#### 🧷 typeof

타입에서 쓰일 때와 값에서 쓰일 때 다른 기능을 한다.

```typescript
type T1 = typeof jane;
//   ^? type T1 = Person
type T2 = typeof email;
//   ^? type T2 = (to: Person, subject: string, body: string) => Response

const v1 = typeof jane;  // Value is "object"
const v2 = typeof email;  // Value is "function"
```

* 타입의 관점에서, `typeof`는 값을 읽어서 타입스크립트 타입을 반환한다. 타입 공간의 `typeof`는 보다 큰 타입의 일부분으로 사용할 수 있고, `type` 구문으로 이름을 붙이는 용도로도 사용할 수 있다.
* 값의 관점에서, `typeof`는 자바스크립트 런타임의 `typeof` 연산자가 된다. 값 공간의 `typeof`는 대상 심벌의 런타임 타입을 가리키는 문자열을 반환하며, 타입스크립트 타입과는 다르다.&#x20;

자바스크립트의 런타임 타입 시스템은 타입스크립트의 정적 타입 시스템보다 훨씬 간단하다. 타입스크립트 타입의 종류가 무수히 많은 반면, 자바스크립트에는 단 6개(`string`, `number`, `boolean`, `undefined`, `object`, `function`)의 런타임 타입만이 존재한다.

클래스에 대한 `typeof`는 상황에 따라 다르게 동작한다.

```typescript
const v = typeof Cylinder; // 값이 "function"
type T = typeof Cylinder; // 타입이 typeof Cylinder
```

클래스가 자바스크립트에서는 실제 함수로 구현되기 때문에 첫 번째 줄의 값은 "function"이 된다.

두번째 줄의 타입은 Cylinder가 인스턴스 타입이 아니라는 것이다. 실제로는 new 키워드를 사용할 때 볼 수 있는 생성자 함수이다.

```typescript
declare let fn: T;
const c = new fn();    // 타입이 Cylinder
```

다음 코드처럼 InstanceType 제너릭을 사용해 생성자 타입과 인스턴스 타입을 전환할 숭 ㅣㅆ다.

```typescript
type C = InstanceType<typeof Cylinder>;     // 타입이 Cylinder
```

속성 접근자인 \[] 는 타입으로 쓰일 때에도 동일하게 동작한다. 그러나 `obj['field']`와 `obj.field`는 값이 동일하더라도 타입은 다를 수 있다.

→ 타입의 속성을 얻을 때에는 반드시 첫 번째 방법인 `obj['field']`을 사용해야 한다.

```typescript
const first: Person['first'] = jane['first'];  // Or jane.first
//    ―――――                    ――――――――――――― Values
//           ―――――― ―――――――                  Types
```

`Person['first']`는 여기서 타입 맥락(: 뒤에) 쓰였기 때문에 타입이다. 인덱스 위치에는 유니온 타입과 기본형 타입을 포함한 어떠한 타입이든 사용할 수 있다.

```typescript
type PersonEl = Person['first' | 'last'];
//   ^? type PersonEl = string
type Tuple = [string, number, Date];
type TupleEl = Tuple[number];
//   ^? type TupleEl = string | number | Date
```

두 공간 사이에서 다른 의미를 가지는 코드 패턴들이 있다.

* 값으로 쓰이는 this는 자바스크립트의 this 키워드이다. 타입으로 쓰이는 this는 일명 '다형성 this'라고 불리는 this의 타입스크립트 타입이다. 서브클래스의 메서드 체인을 구현할 때 유용하다.
* 값에서 &와 |는 AND와 OR 비트연산이다. 타입에서는 인터섹션과 유니온이다.
* const는 새 변수를 선언하지만, as const는 리터럴 또는 리터럴 표현식의 추론된 타입을 바꾼다.
* extends는 서브클래스(class A extends B) 또는 서브타입(interface A extends B) 또는 제너릭 타입의 한정자(Generic\<T extends number>)를 정의할 수 있다.
* in은 루프(for (key in object)) 또는 매핑된(mapped) 타입에 등장한다.

타입스크립트 코드가 잘 동작하지 않는다면 타입 공간과 값 공간을 혼동해서 작성했을 가능성이 크다.

단일 객체 매개변수를 받도록 email 함수를 변경했다고 생각해보자.

```typescript
function email(options: {to: Person, subject: string, body: string}) {
  // ...
}
```

자바스크립트에서는 객체 내의 각 속성을 로컬 변수로 만들어 주는 구조 분해(destructuring) 할당을 사용할 수 있다.

```typescript
function email({person, subject, body}) {
  // ...
}
```

타입스크립트에서 구조 분해 할당을 하면, 이상한 오류가 발생한다.

```typescript
function email({
  to: Person,
  //  ~~~~~~ Binding element 'Person' implicitly has an 'any' type
  subject: string,
  //       ~~~~~~ Binding element 'string' implicitly has an 'any' type
  body: string
  //    ~~~~~~ Binding element 'string' implicitly has an 'any' type
}) { /* ... */ }
```

값의 관점에서 Person과 string이 해석되었기 때문에 오류가 발생했다. Person이라는 변수명과 string이라는 이름을 가지는 두 개의 변수를 생성하려 한 것이다.&#x20;

문제를 해결하려면 타입과 값을 구분해야 한다.

```typescript
function email(
  {to, subject, body}: {to: Person, subject: string, body: string}
) {
  // ...
}
```

매개변수에 명명된 타입을 사용하거나 문맥에서 추론되도록 잘 동작한다. 타입과 값을 비슷한 방식으로 쓰는 점을 잘 터득하여 마치 연상 기호처럼 무의식적으로 쓸 수 있도록 하자.

## 📍요약

* 타입스크립트 코드를 읽을 때 타입인지 값인지 구분하는 방법을 터득해야 한다. 타입스크립트 플레이그라운드를 활용해 개념을 잡는 것이 좋다.
* 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다. `type`과 `interface` 같은 키워드는 타입 공간에만 존재한다.
* `class`나 `enum` 같은 키워드는 타입과 값 두 가지로 사용될 수 있다.
* `"foo"`는 문자열 리터럴이거나, 문자열 리터럴 타입일 수 있다. 차이점을 알고 구별하는 방법을 터득하자.
* `typeof`, `this` 그리고 많은 다른 연산자들과 키워드들은 타입 공간과 값 공간에서 다른 목적으로 사용될 수 있다.
