# 📎 아이템 3 코드 생성과 타입이 관계없음을 이해하기

## 📍타입스크립트 컴파일러의 두 가지 역할

1. 최신 타입/자바 스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일(번역+컴파일)한다.
2. 코드의 타입 오류를 체크한다.

이 두가지는 서로 완벽히 독립적이다. 타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입에는 영향을 주지 않는다. 자바스크립트의 실행 시점에도 타입은 영향을 미치지 않는다.

## 📍타입 오류가 있는 코드도 컴파일이 가능하다

> 컴파일은 타입 체크와 독립적으로 동작하기 때문에, 타입 오류가 있는 코드도 컴파일이 가능하다.

타입스크립트 오류는 C나 자바 같은 언어들의 경고와 비슷하다. 문제가 될 만한 부분을 알려주지만, 빌드를 멈추지는 않는다. 코드에 오류가 있더라도 컴파일된 산출물이 나오는 것이 실제로 도움이 된다.

→ 웹 어플리케이션을 만들면서 타입스크립트는 여전히 컴파일된 산출물을 생성하기 때문에, 문제가 된 오류를 수정하지 않더라도 웹 어플리케이션의 다른 부분을 테스트할 수 있다.

### 컴파일과 타입체크

코드에 오류가 있을 때 → 컴파일에 문제가 있다. 는 기술적으로 틀린말이다.

오직 코드 생성만이 '컴파일'이라고 할 수 있기때문이다. 작성한 타입스크립트가 유효한 자바스크립트라면 타입스크립트 컴파일러는 컴파일을 해낸다. 그러므로 코드에 오류가 있을 때 "타입 체크에 문제가 있다"고 말하는 것이 더 정확한 표현이다.

## 📍런타임에는 타입체크가 불가능하다

```typescript
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    //                 ~~~~~~~~~ 'Rectangle' only refers to a type,
    //                           but is being used as a value here
    return shape.height * shape.width;
    //           ~~~~~~ Property 'height' does not exist on type 'Shape'
  } else {
    return shape.width * shape.width;
  }
}
```

`instanceof` 체크는 런타임에 일어나지만, `Rectangle`은 타입이기 때문에 런타임시점에 아무런 역할을 할 수 없다. 타입스크립트의 타입은 '제거 가능'하다. 실제로 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 그냥 제거된다.&#x20;

#### ⭐️ `shape` 타입을 명확하게 하려면, 런타임에 타입 정보를 유지하는 방법이 필요하다. `height` 속성이 존재하는지 체크해보자.

```typescript
function calculateArea(shape: Shape) {
  if ('height' in shape) {
    shape; // 타입이 Rectangle
    return shape.width * shape.height;
    //     ^? (parameter) shape: Rectangle
  } else {
    shape; // 타입이 Square
    return shape.width * shape.width;
  }
}
```

속성 체크는 런타임에 접근 가능한 값에만 관련되지만, 타입 체커 역시 `shape`의 타입을 `Rectangle`로 보정해주기 때문에 오류가 사라진다.

#### ⭐️ 타입 정보를 유지하는 또 다른 방법으로는 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 '태그' 기법

```typescript
interface Square {
  kind: 'square';
  width: number;
}
interface Rectangle {
  kind: 'rectangle';
  height: number;
  width: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape.kind === 'rectangle') {
    return shape.width * shape.height;
    //     ^? (parameter) shape: Rectangle
  } else {
    return shape.width * shape.width;
    //     ^? (parameter) shape: Square
  }
}
```

여기서 Shape 타입은 '태그된 유니온'의 한 예이다. 이 기법은 런타임에 타입 정보를 손쉽게 유지할 수 있기 때문에, 타입스크립트에서 흔하게 볼 수 있다.

#### ⭐️ 타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법

타입을 클래스로 만들면 된다. Square와 Rectangle을 클래스로 만들면 오류를 해결할 수 있다.

```typescript
class Square {
  width: number;
  constructor(width: number) {
    this.width = width;
  }
}
class Rectangle extends Square {
  height: number;
  constructor(width: number, height: number) {
    super(width);
    this.height = height;
  }
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
    //     ^? (parameter) shape: Rectangle
  } else {
    return shape.width * shape.width;
    //     ^? (parameter) shape: Square
  }
}
```

인터페이스는 타입으로만 사용 가능하지만, Rectangle을 클래스로 선언하면 타입과 값으로 모두 사용할 수 있으므로 오류가 없다.

`type Shape = Square | Rectangle;` 부분에서 `Rectangle`은 타입으로 참조되지만, `shape instanceof Rectangle`에서는 값으로 참조된다.

## 📍타입 연산은 런타임에 영향을 주지 않는다

`string` 또는 `number` 값을 항상 `number`로 정제하는 경우를 가정해보자.

```typescript
// 타입 체커를 통과하지만 잘못된 방법이다
function asNumber(val: number | string): number {
  return val as number;
}
```

변환된 자바스크립트를 통해 이 함수가 어떻게 동작하는지 알아보자.

```javascript
function asNumber(val) {
  return Number(val);
}
```

코드에는 아무런 정제 과정이 없다. `as number`는 타입 연산이고, 런타임 동작에는 아무런 영향을 미치지 않는다. 값을 정제하기 위해서는 런타임의 타입을 체크해야 하고 자바스크립트의 연산을 통해 변환을 수행해야 한다.

```typescript
function asNumber(val: number | string): number {
  return typeof(val) === 'string' ? Number(val) : val;
}
```

`as number`는 타입 단언문이다.

## 📍런타임 타입은 선언된 타입과 다를 수 있다

다음 함수의 마지막 `console.log`까지 실행될 수 있을 지 생각해보자.

```typescript
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      turnLightOn();
      break;
    case false:
      turnLightOff();
      break;
    default:
      console.log(`I'm afraid I can't do that.`);
  }
}
```

타입스크립트는 일반적으로 실행되지 못하는 죽은(dead) 코드를 찾아내지만, 여기서는 `strict`를 설정하더라도 찾아내지 못한다. 마지막 부분을 실행할 수 있는 경우는 무엇일까?

`: boolean`이 타입 선언문이라는 것에 주목하자. 타입스크립트의 타입이기 때문에 `: boolean`은 런타임에 제거된다. 자바스크립트였다면 `setLightSwitch`를 "ON"으로 호출할 수도 있었을 것이다.

순수 타입스크립트에서도 마지막 코드를 실행하는 방법은 존재한다.

#### ⭐️ 네트워크 호출로부터 받아온 값으로 함수를 실행하는 경우가 있다.

```typescript
interface LightApiResponse {
  lightSwitchValue: boolean;
}
async function setLight() {
  const response = await fetch('/light');
  const result: LightApiResponse = await response.json();
  setLightSwitch(result.lightSwitchValue);
}
```

`/light` 요청하면 그 결과로 `LightApiResponse`를 반환하라고 선언했지만, 실제로 그렇게 되리라는 보장은 없다. API를 잘못 파악해서 `lightSwitchValue`가 문자열이었다면, 런타임에는 `setLightSwitch` 함수까지 전달 될 것이다. 또는 배포된 후에 API가 변경되어 `lightSwitchValue`가 문자열이 되는 경우도 있을 것이다.

타입스크립트에서는 런타임 타입과 선언된 타입이 맞지 않을 수 있다. 타입이 달라지는 상황을 피하고, 언제든 달라질 수 있다는 것을 명심하자.

## 📍타입스크립트 타입으로는 함수를 오버로드할 수 없다

함수 오버로딩: 동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용하는 것

타입스크립트에서는 타입과 런타임의 동작이 무관하기 때문에 함수 오버로딩은 불가능하다.

```typescript
function add(a: number, b: number) { return a + b; }
//       ~~~ Duplicate function implementation
function add(a: string, b: string) { return a + b; }
//       ~~~ Duplicate function implementation
```

타입스크립트의 함수 오버로딩 기능은 온전히 타입 수준에서만 동작한다. 하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만, 구현체는 오직 하나뿐이다.

```typescript
function add(a: number, b: number): number;
function add(a: string, b: string): string;

function add(a: any, b: any) {
  return a + b;
}

const three = add(1, 2);
//    ^? const three: number
const twelve = add('1', '2');
//    ^? const twelve: string
```

`add`에 대한 처음 두 개의 선언문의 타입 정보를 제공한다. 이 두 선언문은 타입스크립트가 자바스크립트로 변환되면서 제거되며, 구현체만 남게된다.

## 📍타입스크립트 타입은 런타임 성능에 영향을 주지 않는다

타입과 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에, 런타임의 성능에 아무런 영향을 주지 않는다. 타입스크립트의 정적 타입은 실제로 비용이 전혀 들지 않는다. 타입스크립트를 쓰는 대신 런타임 오버헤드를 감수하며 타입 체크를 해 본다면, 타입스크립트 팀이 다음  주의 사항들을 얼마나 잘 테스트해 왔는지 알 수 있다.

* '런타임' 오버헤드가 없는 대신, 타입스크립트 컴파일러는 '빌드타임' 오버헤드가 있다. 타입스크립트 팀은 성능을 중요하게 생각하므로 컴파일은 상당히 빠른 편이며 증분(incremental) 빌드 시에 더욱 체감 된다. 오버헤드가 커지면, 빌드 도구에서 '트랜스파일만(transpile only)'을 설정하여 타입 체크를 건너뛸 수 있다.
*   타입스크립트가 컴파일하는 코드는 오래된 런타임 환경을 지원하기 위해 호환성을 높이고 성능 오버헤드를 감안할지, 호환성을 포기하고 성능 중심의 네이티브 구현체를 선택할지의 문제에 맞닥뜨릴 수 있다.

    * 제너레이터 함수가 ES5 타깃으로 컴파일되려면, 타입스크립트 컴파일러는 호환성을 위한 특정 헬퍼 코드를 추가할 것이다.

    → 어떤 경우든지 호환성과 성능 사이의 선택은 컴파일 타깃과 언어 레벨의 문제이며 타입과는 무관하다.

## 📍요약

* 코드 생성은 타입 시스템과 무관한다. 타입스크립트 타입은 런타임 동작이나 성능에 영향을 주지 않는다.
* 타입 오류가 존재하더라도 코드 생성(컴파일)은 가능하다.
* 타입스크립트 타입은 런타임에 사용할 수 없다. 런타임에 타입을 지정하려면, 타입 정보 유지를 위한 별도의 방법이 필요하다.
  * 태그된 유니온과 속성 체크된 방법을 사용한다.
  * 클래스 같이 타입스크립트 타입과 런타임 값, 둘 다 제공하는 방법이 있다.
