# 📎 아이템 23 한꺼번에 객체 생성하기

변수의 값은 변경될 수 있지만, 타입스크립트의 타입은 일반적으로 변경되지 않는다.

→ 일부 자바스크립트 패턴을 타입스크립트로 모델링하는게 쉬워진다.

즉, 객체를 생성할 때는 속성을 하나씩 추가하기보다는 여러 속성을 포함해서 한꺼번에 생성해야 타입 추론에 유리하다.

## 📍 예시

### ⭐️ 자바스크립트에서 객체를 생성하는 방법

#### 🔗 2차원 점을 표현하는 객체를 생성하는 예제를 알아보자.

타입스크립트에서는 각 할당문에 오류가 발생한다.

```typescript
const pt = {};
//    ^? const pt: {}
pt.x = 3;
// ~ Property 'x' does not exist on type '{}'
pt.y = 4;
// ~ Property 'y' does not exist on type '{}'
```

`pt` 타입은 `{}` 값을 기준으로 추론된다. 존재하지 않는 속성은 추가할 수 없다.

#### 🔗 `Point` 인터페이스를 정의한다면 오류가 다음처럼 바뀐다.

```typescript
interface Point { x: number; y: number; }
const pt: Point = {};
   // ~~ Type '{}' is missing the following properties from type 'Point': x, y
pt.x = 3;
pt.y = 4;
```

이런 문제들을 객체를 한번에 정의하면 해결할 수 있다.

```typescript
const pt = {
  x: 3,
  y: 4,
}; // OK
```

#### 🔗 객체를 반드시 제각각 나눠서 만들어야 한다면, 타입 단언문을 사용해 타입 체커를 통과하게 할 수 있다.

```typescript
const pt = {} as Point;
//    ^? const pt: Point
pt.x = 3;
pt.y = 4;  // OK
```

이 경우도, 선언할 때 객체를 한꺼번에 만드는게 더 낫다.

```typescript
const pt: Point = {
  x: 3,
  y: 4,
};
```

#### 🔗 작은 객체들을 조합해서 큰 객체를 만들어야하는 경우에도 한꺼번에 만드는 것이 좋다.

```typescript
// 여러 단계를 거치는 경우
const pt = {x: 3, y: 4};
const id = {name: 'Pythagoras'};
const namedPoint = {};
Object.assign(namedPoint, pt, id);
namedPoint.name;
        // ~~~~ Property 'name' does not exist on type '{}'
```

다음과 같이, **'객체 전개 연산자'** `...` 을 사용하여 큰 객체를 한꺼번에 만들어보자.

```typescript
const namedPoint = {...pt, ...id};
//    ^? const namedPoint: { name: string; x: number; y: number; }
namedPoint.name;  // OK
//         ^? (property) name: string
```

#### 🔗 객체 전개 연산자를 사용하면 타입 걱정 없이 필드 단위로 객체를 생성할 수 있다. 이 때 모든 업데이트마다 새 변수를 사용하여 각각 새로운 타입을 얻도록 하는게 중요하다.

```typescript
const pt0 = {};
const pt1 = {...pt0, x: 3};
const pt: Point = {...pt1, y: 4};  // OK
```

이 방법은 간단한 객체를 만들기 위해 우회했지만, 객체에 속성을 추가하고 타입스크립트가 새로운 타입을 추론할 수 있게 해 유용하다.

#### 🔗 타입에 안전한 방식으로 조건부 속성을 추가하려면, 속성을 추가하지 않는 null 또는 {}으로 객체 전개를 사용하면 된다.

```typescript
declare let hasMiddle: boolean;
const firstLast = {first: 'Harry', last: 'Truman'};
const president = {...firstLast, ...(hasMiddle ? {middle: 'S'} : {})};
//    ^? const president: {
//         middle?: string;
//         first: string;
//         last: string;
//       }
// or: const president = {...firstLast, ...(hasMiddle && {middle: 'S'})};
```

편집기에 president 속성을 보면, 타입이 선택적 속성을 가진 것으로 추론된다는 것을 알 수 있다.

#### 🔗 전개 연산자로 한꺼번에 여러 속성을 추가할 수도 있다.

```typescript
declare let hasDates: boolean;
const nameTitle = {name: 'Khufu', title: 'Pharaoh'};
const pharaoh = { ...nameTitle, ...(hasDates && {start: -2589, end: -2566})};
//    ^? const pharaoh: {
//         start?: number;
//         end?: number;
//         name: string;
//         title: string;
//       }
```

pharaoh의 타입이 유니온으로 추론된다.

```typescript
const pharaoh: {
    start?: number;
    end?: number;
    name: string;
    title: string;
} | {
    name: string;
    title: string;
}
```

`start`와 `end`가 선택적 필드이기를 원했다면, 이런 결과가 당황스러울 수 있다. 이 타입에서는 `start`를 읽을 수 없다.

```typescript
pharaoh.start
// {name: string; title: string;} 형식에 'start' 속성이 없다.
```

이 경우에는 start와 end가 항상 함께 정의된다. 이 점을 고려하면 유니온을 사용하는게 가능한 값의 집합을 더 정확히 표현할 수 있다.

유니온 보다는 선택적 필드가 다루기에는 쉽다.



📍 정리

객체나 배열을 변환해서 새로운 객체나 배열을 생성하고 싶을 수 있다. 이런 경우 루프 대신 내장형 함수형 또는 로대시(Lodash) 같은 유틸리티 라이브러리를 사용하는 것이 '한꺼번에 객체 생성하기'관점에서 보면 옳다.

## 📍 요약

* 속성을 제각각 추가하지 말고 한꺼번에 객체로 만들어야 한다. 안전한 타입으로 속성을 추가하려면 객체 전개`({...a, ...b})`를 사용하면 된다.
* 객체에 조건부로 속성을 추가하는 방법을 익히도록 하자.
