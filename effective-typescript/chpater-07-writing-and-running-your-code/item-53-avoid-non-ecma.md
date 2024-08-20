# 📎 아이템 53 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

초기 `JavaScript`에 없던 기능들을 `TypeScript`에서 독립적으로 만들었고, 이후에 `JavaScript`에서 다른 방식으로 표준이 돼서 호환성에 문제가 생기는 부분에 대해 알아보자.\
`TypeScript`는 `JavaScript`의 호환성을 포기했기 때문에 발생하는 혼란스러운 기능들이 있다.

## 열거형(enum)

```typescript
enum Flavor {
  Vanilla = 0,
  Chocolate = 1,
  Strawberry = 2,
}

let flavor = Flavor.Chocolate;
//  ^? let flavor: Flavor

Flavor  // Autocomplete shows: Vanilla, Chocolate, Strawberry
Flavor[0]  // Value is "Vanilla"
```

단순히 값을 나열하는 것보다 실수가 적고 명확하기 때문에 일반적으로 열거형을 사용하는 것이 좋다. 하지만 타입스크립트의 열거형은 상황에 따라 다르게 동작할 수 있다.

* 숫자 열거형에 0, 1, 2 외의 다른 숫자가 할당되면 위험하다.
* 상수 열거형은 보통의 열거형과 달리 런타임에 완전히 제거된다. const enum Flavor로 바꾸면 컴파일러는 Flavor.Chocolate을 0으로 바꿔버린다. (문자열 열거형과 숫자 열거형과는 다른 동작이다.)
* preserveConstEnums 플래그를 설정한 상태의 상수 열거형은 보통의 열거형처럼 런타임 코드에 상수 열거형 정보를 유지한다.
* 문자열 열거형은 런타임의 타입 안전성과 투명성을 제공한다. 그러나 타입스크립트의 다른 타입과 달리 구조적 타이핑이 아닌 명목적 타이핑을 사용한다.

타입스크립트의 일반적인 타입들이 할당 가능성을 체크하기 위해 구조적 타이핑을 사용하지만, 문자열 열거형은 명목적 타이핑을 사용한다.

```typescript
enum Flavor {
  Vanilla = 'vanilla',
  Chocolate = 'chocolate',
  Strawberry = 'strawberry',
}

let favoriteFlavor = Flavor.Chocolate;  // Type is Flavor
favoriteFlavor = 'strawberry';
// ~~~~~~~~~~~ Type '"strawberry"' is not assignable to type 'Flavor'
```

명목적 타이핑은 라이브러리를 공개할 때 필요하다.&#x20;

⭐️ 자바스크립트와 타입스크립트의 문자열 열거형은 동작이 다르기 때문에 사용하지 않는 것이 좋다.

→ 열거형 대신 리터럴 타입의 유니온을 사용하자.

```typescript
type Flavor = 'vanilla' | 'chocolate' | 'strawberry';

let favoriteFlavor: Flavor = 'chocolate';  // OK
favoriteFlavor = 'americone dream';
// ~~~~~~~~~~~ Type '"americone dream"' is not assignable to type 'Flavor'
```

리터럴 타입의 유니온은 열거형만큼 안전하며 자바스크립트와 호환되는 장점이 있고, 편집기에서 열거형처럼 자동완성 기능을 사용할 수 있다.

## 매개변수 속성

일반적으로 클래스를 초기화할 때 속성을 할당하기 위해 생성자의 매개변수를 사용한다.

```typescript
class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

// 더 간결한 문법
class Person {
  constructor(public name: string) {}
}
```

`public`의 `name`은 '**매개변수 속성'**이라고 불리며, 멤버 변수로 `name`을 선언한 이전 예제와 동일하게 동작한다. 하지만 매개변수 속성과 관련된 문제점이 있다.

* 컴파일 시 코드가 늘어난다. (타입스크립트 컴파일은 타입 제거가 이루어지므로 일반적으로는 코드가 줄어든다.)
* 매개변수 속성이 런타임에는 실제로 사용되지만, 타입스크립트 관점에서는 사용되지 않는 것처럼 보인다.
* 일반 속성과 섞어서 사용하면 혼란이 발생한다.

저자는 매개변수 속성을 선호하지 않는다. 매개변수의 속성은 타입스크립트의 다른 패턴들과 이질적이고, 초급자에게 생소한 문법이다. 또한, 매개변수 속성과 일반 속성을 같이 사용하면 설계가 혼란스러워지기 때문에 한 가지만 사용하자.

## 네임스페이스와 트리플 슬래시 임포트

타입스크립트 자체적 모듈시스템으로 module 키워드와 '트리플 슬래시' 임포트를 사용한다. 타입스크립트는 ECMAScript 2015가 공식적으로 모듈 시스템을 도입한 이후, 충돌을 피하기 위해 module과 같은 기능을 하는 namespace 키워드를 추가했다.

```typescript
namespace foo {
  function bar() {}
}

/// <reference path="other.ts"/>
foo.bar()
```

트리플 슬래시 임포트와 module 키워드는 호환성을 위해 남아있을 뿐이며, 이제는 ECMAScript 2015 스타일의 모듈(import export)을 사용해야 한다.

## 데코레이터

클래스, 메서드, 속성에 애너테이션을 붙이거나 기능을 추가하는 데 사용할 수 있다. 예를 들어, 클래스의 메서드가 호출될 때마다 로그를 남기려면 logged 애너테이션을 정의할 수 있다.

데코레이터는 처음에 앵귤러 프레임워크를 지원하기 위해 추가되었다. 현재까지도 표준화가 완료되지 않았기 때문에, 사용 중인 데코레이터는 비표준으로 바뀌거나 호환성이 깨질 가능성이 있다.

타입스크립트에서는 데코레이터를 사용하지 않는 게 좋다.

## 📍 요약

* 일반적인 `TypeScript` 코드에서 모든 타입 정보를 제거하면 `JavaScript`가 되지만, 열거형, 매개변수 속성, 트리플 슬래시 임포트, 데코레이터는 타입 정보를 제거한다고 `JavaScript`가 되지 않는다.
* 타입스크립트의 역할을 명확하게 하려면, 열거형, 매개변수 속성, 트리플 슬래시 임포트, 데코레이터를 사용하지 않는 것이 좋다.
