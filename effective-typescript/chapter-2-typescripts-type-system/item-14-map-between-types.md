# 📎 아이템 14 타입 연산과 제너릭 사용으로 반복 줄이기

## 📍 **DRY(don't repeat yourself) 원칙**

원기둥의 반지름과 높이, 표면적, 부피를 출력하는 코드이다.

```typescript
console.log(
  'Cylinder r=1 × h=1',
  'Surface area:', 6.283185 * 1 * 1 + 6.283185 * 1 * 1,
  'Volume:', 3.14159 * 1 * 1 * 1
);
console.log(
  'Cylinder r=1 × h=2',
  'Surface area:', 6.283185 * 1 * 1 + 6.283185 * 2 * 1,
  'Volume:', 3.14159 * 1 * 2 * 1
);
console.log(
  'Cylinder r=2 × h=1',
  'Surface area:', 6.283185 * 2 * 1 + 6.283185 * 2 * 1,
  'Volume:', 3.14159 * 2 * 2 * 1
);
```

비슷한 코드가 반복되어있다. 값과 상수가 반복되며 드러나지 않은 오류도 포함되어 있다. 이 코드에서 함수, 상수, 루프의 반복을 제거해 코드를 개선해보자.

```typescript
type CylinderFn = (r: number, h: number) => number;
const surfaceArea: CylinderFn = (r, h) => 2 * Math.PI * r * (r + h);
const volume: CylinderFn = (r, h) => Math.PI * r * r * h;

for (const [r, h] of [[1, 1], [1, 2], [2, 1]]) {
  console.log(
    `Cylinder r=${r} × h=${h}`,
    `Surface area: ${surfaceArea(r, h)}`,
    `Volume: ${volume(r, h)}`);
}
```

## 📍 **타입에서의 DRY 원칙**

```typescript
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate {
  firstName: string;
  lastName: string;
  birth: Date;
}
```

타입 중복도 코드 중복만큼 많은 문제를 발생시킨다.&#x20;

예를 들어, `middleName`을 `Person`에 추가한다고 하면 `Person`과 `PersonWithBirthDate`는 다른 타입이 된다.

### 🔗 타입에서의 중복

타입에서 중복이 흔한 이유는 공유된 패턴을 제거하는 매커니즘이 기존 코드에서 하던 것과 비교해 덜 익숙하기 때문이다. 타입 간의 매핑하는 방법을 익혀 타입 정의에서도 **DRY**의 장점을 적용해보자.

### 🔗 반복 줄이기

#### ⭐️ 타입에 이름 붙이기

✓ 거리 계산 함수에서 타입이 반복적으로 등장한다.

```typescript
function distance(a: {x: number, y: number}, b: {x: number, y: number}) {
  return Math.sqrt((a.x - b.x) ** 2 + (a.y - b.y) ** 2);
}
```

타입에 이름을 붙여보자.

```typescript
interface Point2D {
  x: number;
  y: number;
}
function distance(a: Point2D, b: Point2D) { /* ... */ }
```

이 코드는 상수를 사용해서 반복을 줄이는 기법을 동일하게 타입 시스템에 적용한 것이다.&#x20;

✓ 몇몇 함수가 같은 타입 시그니처를 공유하고 있는 경우

```typescript
function get(url: string, opts: Options): Promise<Response> { /* ... */ }
function post(url: string, opts: Options): Promise<Response> { /* ... */ }
```

해당 시그니처를 명명된 타입으로 분리하자.

```typescript
type HTTPFunction = (url: string, opts: Options) => Promise<Response>;
const get: HTTPFunction = (url, opts) => { /* ... */ };
const post: HTTPFunction = (url, opts) => { /* ... */ };
```

#### ⭐️ 확장

✓ 위 `Person`과 `PersonWithBirthDate` 예제를 다시 보자

→ 한 인터페이스가 다른 인터페이스를 확장하게 해서 반복을 제거할 수 있다.

```typescript
interface Person {
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate extends Person {
  birth: Date;
}
```

추가적인 필드만 작성하면 된다. 두 인터페이스가 필드의 부분 집합을 공유한다면, 공통 필드만 골라서 기반 클래스로 분리해 낼 수 있다.

✓ 이미 존재하는 타입을 확장하는 경우

→ `&` 연산자를 사용한다. (일반적❎) 유니온 타입(확장할 수 없는)에 속성을 추가할 때 특히 유용하다.

```typescript
type PersonWithBrithDate = Person & { birth: Date };
```

✓ 전체 애플리케이션의 상태를 표현하는 타입(State)과 부분만 표현(TopNavState)하는 타입

```typescript
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}
interface TopNavState {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  // omits pageContents
}
```

`TopNavState`를 확장하여 `State`를 구성하기보다, `State`의 부분 집합으로 `TopNavState`을 구성하는 것이 좋다. 이 방법이 전체 앱의 상태를 하나의 인터페이스로 유지할 수 있게 해준다.

`State`를 인덱싱하여 속성의 타입에서 중복을 제거할 수 있다.

```typescript
interface TopNavState {
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
};
```

State 내의 pageTitle 타입이 바뀌면 TopNavState에도 반영이 된다. 여전히 반복되는 코드가 존재한다.

**'매핑된 타입'**을 사용하자.

```typescript
type TopNavState = {
  [K in 'userId' | 'pageTitle' | 'recentFiles']: State[K]
};
```

매핑된 타입은 배열의 필드를 루프 도는 것과 같은 방식이다. 이 패턴은 표준 라이브러리 `Pick`에서도 찾을 수 있다.

```typescript
type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

#### ⭐️ 태그된 유니온에서의 중복

```typescript
interface SaveAction {
  type: 'save';
  // ...
}
interface LoadAction {
  type: 'load';
  // ...
}
type Action = SaveAction | LoadAction;
type ActionType = 'save' | 'load';  // Repeated types!
```

`Action` 유니온을 인덱싱하면 타입 반복 없이 `ActionType`을 정의할 수 있다.

```typescript
type ActionType = Action['type'];
//   ^? type ActionType = "save" | "load"
```

`Action` 유니온에 타입을 더 추가하면 `ActionType`은 자동적으로 그 타입을 포함한다.



## 📍요약

* DRY 원칙을 타입에도 최대한 적용하자.
* 타입에 이름을 붙여서 반복을 피해야 한다. extends를 사용해서 인터페이스 필드의 반복을 피해야한다.
* 타입들 간의 매핑을 위해 타입스크립트가 제공한 도구들을 공부하면 좋다. keyof, typeof, 인덱싱, 매핑된 타입들이 포함된다.
* 제너릭 타입은 타입을 위한 함수와 같다. 타입을 반복하는 대신 제너릭 타입을 사용하여 타입들 간에 매핑을 하는 것이 좋다. 제너릭 타입을 제한하려면 extends를 사용하면 된다.
* 표준 라이브러리에 정의된 Pick, Partial, ReturnType 같은 제너릭 타입에 익숙해지자.



