# 📎 아이템 13 타입과 인터페이스의 차이점 알기

## 📍 타입스크립트에서 명명된 타입(named type)을 정의하는 방법은 두가지이다.

### ⭐️ 타입을 사용하는 방법

```typescript
type TState = {
  name: string;
  capital: string;
};
```

### ⭐️ 인터페이스를 사용하는 방법

```typescript
interface IState {
  name: string;
  capital: string;
}
```

타입과 인터페이스 사이에 존재하는 차이를 분명하게 알고, 같은 상황에서는 동일한 방법으로 명명된 타입을 정의해 일관성을 유지해야한다. 하나의 타입에 대해 두 가지 방법을 모두 사용해보자.

### 인터페이스 선언과 타입 선언의 비슷한 점

🔗 명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태에는 차이가 없다.

`IState`와 `TState`를 추가 속성과 함께 할당한다면 동일한 오류가 발생한다.

```typescript
const wyoming: TState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 578_000
  // ~~~~~~~ Object literal may only specify known properties,
  //         and 'population' does not exist in type 'TState'
  // ... 형식은 'Tstate' 형식에 할당할 수 없다.
  // 개체 리터럴은 알려진 속성만 지정할 수 있으며, 'TState' 형식에 'populatio'이 없다.
};
```

🔗 인덱스 시그니처는 인터페이스와 타입에서 모두 사용할 수 있다.

```typescript
type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}
```

🔗 함수 타입도 인터페이스나 타입으로 정의할 수 있다.

```typescript
type TFn = (x: number) => string;
interface IFn {
  (x: number): string;
}
type TFnAlt = {
  (x: number): string;
};

const toStrT: TFn = x => '' + x;  // OK
const toStrI: IFn = x => '' + x;  // OK
const toStrTAlt: TFnAlt = x => '' + x;  // OK
```

🔗 단순한 함수 타입에는 타입 별칭(alias)이 더 낫겠지만, 함수 타입에 추가적인 속성이 있다면 타입이나 인터페이스 어떤 것을 선택하든 차이가 없다.

```typescript
type TFnWithProperties = {
  (x: number): number;
  prop: string;
}
interface IFnWithProperties {
  (x: number): number;
  prop: string;
}
```

🔗 타입 별칭과 인터페이스는 모두 제너릭이 가능하다.

```typescript
type TBox<T> = {
  value: T;
};
interface IBox<T> {
  value: T;
}
```

🔗 인터페이스는 타입을 확장할 수 있으며, 타입은 인터페이스를 확장할 수 있다.

```typescript
interface IStateWithPop extends TState {
  population: number;
}
type TStateWithPop = IState & { population: number; };
```

`IStateWithPop`과 `IStateWithPop`은 동일하다. 인터페이스는 유니온 타입 같은 복잡한 타입을 확장하지는 못한다. (&을 사용하여 확장해야 한다.)

🔗 클래스로 구현할 때는, 타입(TState)과 인터페이스(IState) 둘 다 사용할 수 있다.

```typescript
class StateT implements TState {
  name: string = '';
  capital: string = '';
}
class StateI implements IState {
  name: string = '';
  capital: string = '';
}
```

### 인터페이스 선언과 타입 선언의 다른 점

🔗 유니온 타입은 있지만, 유니온 인터페이스는 없다.

```typescript
type AorB = 'a' | 'b';
```

🔗 인터페이스는 타입을 확장할 수 있지만, 유니온은 할 수 없다.&#x20;

유니온 타입을 확장하는게 필요한 경우가 있다.

```typescript
type Input = { /* ... */ };
type Output = { /* ... */ };
interface VariableMap {
  [name: string]: Input | Output;
}
```

`Input`과 `Output` 별도의 타입을 하나의 변수명으로 매핑하는 `VariableMap` 인터페이스를 만들 수 있다.

또는 유니온 타입에 name 속성을 붙인 타입을 만들 수 있다.

```typescript
type NamedVariable = (Input | Output) & { name: string };
```

이 타입은 인터페이스로 표현할 수 없다. `type` 키워드는 일반적으로 `interface`보다 쓰임새가 많다.

`type` 키워드는 유니온이 될 수도, 매핑된 타입 또는 조건부 타입 같은 고급기능에 활용되기도 한다.

🔗 튜플과 배열 타입도 `type` 키워드를 이용해 더 간결하게 표현할 수 있다.

```typescript
type Pair = [a: number, b: number];
type StringList = string[];
type NamedNums = [string, ...number[]];
```

인터페이스로도 튜플과 비슷하게 구현할 수 있기는 하다.

```typescript
interface Tuple {
  0: number;
  1: number;
  length: 2;
}
const t: Tuple = [10, 20] // OK
```

그러나 인터페이스로 튜플과 비슷하게 구현하면 튜플에서 사용할 수 있는 `concat` 같은 메서드를 사용할 수 없다.&#x20;

→ 튜플은 `type` 키워드로 구현하는 것이 낫다.&#x20;

🔗 인터페이스에는 타입에 없는 몇 가지 기능이 있다.

🔗 인터페이스는 **보강(augment)**이 가능하다.

```typescript
interface IState {
  name: string;
  capital: string;
}
interface IState {
  population: number;
}
const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 578_000
};  // OK
```

`State` 예제에 `population` 필드를 추가할 때 보강 기법을 사용할 수 있다. 속성을 확장하는 것을 **'선언 병합'**이라고 한다. 타입 선언 파일을 작성할 때는 선언 병합을 지원하기 위해 반드시 인터페이스를 사용해야 하며 표준을 따라야 한다. (선언 병합은 주로 타입 선언 파일에서 사용된다.)

🔗 타입스크립트는 여러 버전의 자바스크립트 표준 라이브러리에서 여러 타입을 모아 병합한다.

병합은 선언처럼 일반적인 코드라서 언제든지 가능하다. 그러므로 프로퍼티가 추가되는 것을 원하지 않는다면 인터페이스 대신 타입을 사용해야 한다.

## 📍타입 VS 인터페이스

1. 복잡한 타입이라면 타입 별칭
2. 두 가지 방법으로 모두 표현할 수 있는 간단한 객체 타입이라면 일관성과 보강의 관점에서 고려하기 → 일관되게 인터페이스를 사용하고 있다면 인터페이스를, 타입을 사용중이라면 타입을 사용하면 된다.
3. 아직 스타일이 확립되지 않은 프로젝트라면, 향후 보강의 가능성을 생각하자.
   1. API에 대한 타입 선언은 인터페이스를 사용하는 것이 좋다. API가 변경될 때 사용자가 인터페이스를 통해 새로운 필드를 병합할 수 있어 유용하기 때문이다.
   2. 프로젝트 내부적으로 사용되는 타입에 선언 병합이 발생하는 것은 잘못된 설계이다. 이럴 때는 타입을 사용해야 한다.

## 📍요약

* 타입과 인터페이스의 차이점과 비슷한 점을 이해하자.
* 한 타입을 `type`과 `Interface` 두 가지 문법을 사용해서 작성하는 방법을 터득하자.
* 프로젝트에서 어떤 문법을 사용할 지 결정할 때 한 가지 일관된 스타일을 확립하고, 보강 기법이 필요한지 고려하자.
