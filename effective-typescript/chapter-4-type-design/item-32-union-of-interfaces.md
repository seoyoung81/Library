# 📎 아이템 32 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

유니온 타입의 속성을 가지는 인터페이스를 작성 중일 때, 인터페이스의 유니온 타입을 사용하는 것이 알맞지 않을 지 고민해봐야 한다.

## 📍예시

### 🔗 벡터를 그리는 프로그램, 기하학적 타입을 가지는 계층의 인터페이스

```typescript
interface Layer {
  layout: FillLayout | LineLayout | PointLayout; // 모양이 그려지는 방법과 위치를 제어
  paint: FillPaint | LinePaint | PointPaint;     // 스타일 제어
}
```

`layout`이 `LineLayOut` 타입이면서 `paint` 속성이 `FillPaint` 타입인 것은 말이 되지 않는다. (선인데 채우기 속성이 있는 경우) 이런 조합을 허용한다면 라이브러리에서는 오류가 발생하며 인터페이스를 다루기도 어렵다.

#### ⭐️ 각각 타입의 계층을 분리된 인터페이스로 모델링

```typescript
interface FillLayer {
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

이런 형태로 Layer를 정의하면 layout과 paint 속성이 잘못된 조합으로 섞이는 경우를 방지한다. (유효한 상태만을 표현하도록 타입을 정의)

이러한 패턴의 일반적인 예시는 태그된 유니온이다. → Layer의 경우, 속성 중 하나는 문자열 리터럴 타입의 유니온이 된다.

```typescript
interface Layer {
  type: 'fill' | 'line' | 'point';
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

`type: 'fill'`과 함께 `LineLayout`과 `PointLayout` 타입이 쓰이는 것은 말이 되지 않는다.

#### ⭐️ Layer를 인터페이스의 유니온으로 변환하자.

```typescript
interface FillLayer {
  type: 'fill';
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  type: 'line';
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  type: 'paint';
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

`type` 속성은 '태그'이며 런타임에 어떤 타입의 `Layer`가 사용되는지 판단하는데 쓰인다. 타입스크립트는 태그를 참고하여 `Layer`의 타입의 범위를 좁힐 수도 있다.

```typescript
function drawLayer(layer: Layer) {
  if (layer.type === 'fill') {
    const {paint} = layer;
    //     ^? const paint: FillPaint
    const {layout} = layer;
    //     ^? const layout: FillLayout
  } else if (layer.type === 'line') {
    const {paint} = layer;
    //     ^? const paint: LinePaint
    const {layout} = layer;
    //     ^? const layout: LineLayout
  } else {
    const {paint} = layer;
    //     ^? const paint: PointPaint
    const {layout} = layer;
    //     ^? const layout: PointLayout
  }
}
```

✏️ 각 타입의 속성들 간의 관계를 제대로 모델링하면, 타입스크립트가 코드의 정확성을 체크하는데 도움이 된다.

❓ 다만, 이 코드에서는 타입 분기 후 layer가 포함된 동일한 코드가 반복되고 있다.

✏️ 태그된 유니온은 타입스크립트 타입 체커와 잘 맞아 타입스크립트에서 잘 보인다. 이 패턴을 기억해서 적용해보자. 어떤 데이터 타입을 태그된 유니온으로 표현할 수 있다면, 그렇게 하자. 또는 여러 개의 선택적 필드가 동시에 값이 있거나 동시에 `undefined`인 경우도 태그된 유니온 패턴이 잘 맞는다.

### 🔗 선택적 필드, 동시에 값이 있거나, 동시에 `undefined`

```typescript
interface Person {
  name: string;
  // These will either both be present or not be present
  placeOfBirth?: string;
  dateOfBirth?: Date;
}
```

타입 정보를 달고 있는 주석은 문제가 될 가능성이 높다. placeOfBirth와 dateOfBirth필드는 실제로 관련되어 있지만, 타입 정보에는 어떠한 관계도 표시되지 않았다.

→ 두 개의 속성을 하나의 객체로 모으자. (null 값을 경계로 두는 방법과 유사)

```typescript
interface Person {
  name: string;
  birth?: {
    place: string;
    date: Date;
  }
}
```

❓ place만 있고, date가 없는 경우에는 오류가 발생

```typescript
const alanT: Person = {
  name: 'Alan Turing',
  birth: {
// ~~~~ Property 'date' is missing in type
//      '{ place: string; }' but required in type
//      '{ place: string; date: Date; }'
    place: 'London'
  }
}
```

⭐️ Person 객체를 매개변수로 받는 함수는 birth 하나만 체크하면 된다.

```typescript
function eulogize(person: Person) {
  console.log(person.name);
  const {birth} = person;
  if (birth) {
    console.log(`was born on ${birth.date} in ${birth.place}.`);
  }
}
```

타입의 구조를 손댈 수 없을 때는 (API), 앞서 다룬 인터페이스의 유니온을 사용해서 속성 사이의 관계를 모델링하자.

```typescript
interface Name {
  name: string;
}

interface PersonWithBirth extends Name {
  placeOfBirth: string;
  dateOfBirth: Date;
}

type Person = Name | PersonWithBirth;
```

중첩된 객체에서도 동일한 효과를 본다.

```typescript
function eulogize(person: Person) {
  if ('placeOfBirth' in person) {
    person
    // ^? (parameter) person: PersonWithBirth
    const {dateOfBirth} = person;  // OK
    //     ^? const dateOfBirth: Date
  }
}
```

앞의 두 경우 모두 타입 정의를 통해 속성 간의 관계를 더 명확하게 만들었다.

## 📍 요약

* 유니온 타입의 속성을 여러 개 가지는 인터페이스에서는 속성 간의 관계가 분명하지 않기 때문에 실수가 자주 발생하므로 주의해야 한다.
* 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기도 좋다.
* 타입스크립트가 제어 흐름을 분석할 수 있도록 타입에 태그를 넣는 것을 고려해야 한다. 태그된 유니온은 타입스크립트와 매우 잘 맞기 때문에 자주 볼 수 있는 패턴이다.
