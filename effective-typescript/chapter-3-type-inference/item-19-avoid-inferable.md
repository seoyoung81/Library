# 📎 아이템 19 추론 가능한 타입을 사용해 장황한 코드 방지하기

타입스크립트가 타입을 위한 언어이기 때문에, 변수를 선언할 때마다 타입을 명시해야 한다고 생각하지만, 타입스크립트의 많은 타입 구문은 사실 불필요하다.

## 📍 예제 1 - 변수

다음과 같이 코드의 모든 변수에 타입을 선언하는 것은 비생산적이며 형편없는 스타일이다.

```typescript
let x: number = 12;
```

다음처럼만 해도 충분하다.

```typescript
let x = 12;
```

편집기에서 x에 마우스를 올려 보면, 타입이 number로 이미 추론되어 있음을 확인할 수 있다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt="" width="151"><figcaption><p>x의 추론된 타입은 number이다.</p></figcaption></figure>

## 📍 예제 2 - 객체

타입 추론이 된다면 명시적 타입 구문은 필요하지 않다. 오히려 방해가 된다. 만약 타입을 확신하지 못한다면 편집기를 통해 체크하면 된다. 타입스크립트는 더 복잡한 객체도 추론 가능하다.

```typescript
const person: {
  name: string;
  born: {
    where: string;
    when: string;
  };
  died: {
    where: string;
    when: string;
  }
} = {
  name: 'Sojourner Truth',
  born: {
    where: 'Swartekill, NY',
    when: 'c.1797',
  },
  died: {
    where: 'Battle Creek, MI',
    when: 'Nov. 26, 1883'
  }
};
```

타입을 생략하고, 다음처럼 작성해도 충분하다.

```typescript
const person = {
  name: 'Sojourner Truth',
  born: {
    where: 'Swartekill, NY',
    when: 'c.1797',
  },
  died: {
    where: 'Battle Creek, MI',
    when: 'Nov. 26, 1883'
  }
};
```

두 예제에서 person의 타입은 동일하다. 값에 추가로 타입을 작성하는 것은 좋지 않다.

## 📍 예제 3 - 배열

다음 예제처럼 배열의 경우도 마찬가지이다. 타입스크립트는 입력받아 연산을 하는 함수가 어떤 타입을 반환하는지 정확히 알고 있다.

```typescript
function square(nums: number[]) {
  return nums.map(x => x * x);
}
const squares = square([1, 2, 3, 4]);
//    ^? const squares: number[]
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt="" width="281"><figcaption></figcaption></figure>

타입스크립트는 우리가 예상한 것보다 더 정확하게 추론한다.

```typescript
const axis1: string = 'x';
//    ^? const axis1: string
const axis2 = 'y';
//    ^? const axis2: "y"
```

`axis2` 변수를 `string`으로 예상하기 쉽지만, 타입스크립트가 추론한 `"y"`가 더 정확한 타입이다.&#x20;

## 📍 예제 4 - 타입, 비구조화 할당문

타입이 추론되면 리팩터링이 용이해진다. Product 타입과 기록을 위한 함수를 가정해보자.

```typescript
interface Product {
  id: number;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const id: number = product.id;
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}
```

만약, `id`에 `string`도 들어 있을 수 있음을 나중에 알게된다고 가정해보자.

→ `Product` 내의 `id` 타입을 변경한다.

→ `logProduct` 내의 `id` 변수 선언에 있는 타입과 맞지 않기 때문에 오류가 발생한다.

```typescript
interface Product {
  id: string;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const id: number = product.id;
     // ~~ Type 'string' is not assignable to type 'number' 
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}
```

`logProduct` 함수 내의 명시적 타입 구문이 없었다면, 코드는 아무런 수정 없이도 타입 체커를 통과했을 것이다.&#x20;

`logProduct`는 비구조화 할당문을 사용해 구현하는게 낫다. (아이템 58)

```typescript
function logProduct(product: Product) {
  const {id, name, price} = product;
  console.log(id, name, price);
}
```

비구조화 할당문은 모든 지역 변수의 타입이 추론되도록 한다. 여기에 추가로 명시적 타입 구문을 넣는다면 불필요한 타입 선언으로 인해 코드가 번잡해진다.

```typescript
function logProduct(product: Product) {
  const {id, name, price}: {id: string; name: string; price: number } = product;
  console.log(id, name, price);
}
```

정보가 부족해 타입스크립트가 스스로 타입을 판단하기 어려운 상황도 일부 있다. 그럴 때는 명시적 타입 구문이 필요하다. `logProduct` 함수에서 매개변수 타입을 `product`로 명시한 경우가 그 예이다.



## 📍 타입스크립트

어떤 언어들은 매개변수의 최종 사용처까지 참고하여 타입을 추론하지만, 타입스크립트는 최종 사용처까지 고려하지 않는다. 타입스크립트에서 변수의 타입은 일반적으로 처음 등장할 때 결정된다.

이상적인 타입스크립트 코드는 함수/메서드 시그니처에 타입 구문을 포함하지만, 함수 내에서 생성된 지역 변수에는 타입 구문을 넣지 않는다. 타입 구문을 생략하여 구현 로직에 집중하는 것이 좋다.



## 📍 예시 5 - 매개변수 타입 구문 생략

함수 매개변수에 타입 구문을 생략하는 경우 - 기본값이 있는 경우

```typescript
function parseNumber(str: string, base=10) {
  //                              ^? (parameter) base: number
  // ...
}
```

기본값이 10이기 때문에 `base`의 타입은 `number`로 추론된다.

## 📍 예시 6 - 라이브러리

타입 정보가 있는 라이브러리에서, 콜백 함수의 매개변수 타입은 자동으로 추론된다.

`express HTTP` 라이브러리를 사용하는 `request`와 `response`의 타입 선언은 필요하지 않다.

```typescript
// Don't do this:
app.get('/health', (request: express.Request, response: express.Response) => {
  response.send('OK');
});

// Do this:
app.get('/health', (request, response) => {
  //                ^? (parameter) request: Request<...>
  response.send('OK');
  // ^? (parameter) response: Response<...>
});
```



**타입이 추론될 수 있음에도 여전히 타입을 명시하고 싶은 상황을 보자.**

## 📍 예시 7 - 객체 리터럴

```typescript
const elmo: Product = {
  name: 'Tickle Me Elmo',
  id: '048188 627152',
  price: 28.99,
};
```

이런 정의에 타입을 명시하면, 잉여 속성 체크가 동작한다. 잉여 속성 체크는 특히 선택적 속성이 있는 타입의 오타 같은 오류를 잡는데 효과적이다. 변수가 사용되는 순간이 아닌 할당하는 시점에 오류가 표시되도록 해준다.

만약 타입 구문을 제거한다면, 잉여 속성 체크가 동작하지 않는다

→ 객체를 선언한 곳이 아니라 객체가 사용되는 곳에서 타입 오류가 발생한다.

```typescript
const furby = {
  name: 'Furby',
  id: 630509430963,
  price: 35,
};
logProduct(furby);
//         ~~~~~ Argument ... is not assignable to parameter of type 'Product'
//               Types of property 'id' are incompatible
//               Type 'number' is not assignable to type 'string'
```

타입 구문을 제대로 명시한다면, 실제로 실수가 발생한 부분에 오류를 표시해준다.

```typescript
const furby: Product = {
   name: 'Furby',
   id: 630509430963,
// ~~ Type 'number' is not assignable to type 'string'
   price: 35,
};
logProduct(furby);
```

## 📍 예시 8 - 함수 반환

함수 반환에도 타입을 명시하여 오류를 방지할 수 있다. 타입 추론이 가능할지라도 구현상의 오류가 함수를 호출한 곳까지 영향을 미치지 않도록 하기 위해 타입 구문을 명시하는 것이 좋다.

주식 시세를 조회하는 함수

```typescript
function getQuote(ticker: string) {
  return fetch(`https://quotes.example.com/?q=${ticker}`)
      .then(response => response.json());
}
```

이미 조회한 종목을 다시 요청하지 않도록 캐시 추가

```typescript
const cache: {[ticker: string]: number} = {};
function getQuote(ticker: string) {
  if (ticker in cache) {
    return cache[ticker];
  }
  return fetch(`https://quotes.example.com/?q=${ticker}`)
      .then(response => response.json())
      .then(quote => {
        cache[ticker] = quote;
        return quote as number;
      });
}
```

이 코드에는 오류가 있다.

```typescript
getQuote;
// ^? function getQuote(ticker: string): number | Promise<number>
```

`getQuote`는 항상 `Promise`를 반환하므로 `if` 구문에는 `cache[ticker]`가 아니라 `Promise.resolve(cache[ticker])`가 반환되도록 해야한다.

실행해보면 `getQuote` 내부가 아닌 `getQuote`를 호출한 코드에서 발생한다.

```typescript
getQuote('MSFT').then(considerBuying);
//               ~~~~ Property 'then' does not exist on type
//                    'number | Promise<number>'
```

이 때 의도된 반환 타입(Promise\<number>)을 명시한다면, 정확한 위치에 오류가 표시된다.

```typescript
const cache: {[ticker: string]: number} = {};
function getQuote(ticker: string): Promise<number> {
  if (ticker in cache) {
    return cache[ticker];
    // ~~~ Type 'number' is not assignable to type 'Promise<number>'
  }
  // ...
}
```

반환 타입을 명시하면, 구현상의 오류가 사용자 코드의 오류로 표시되지 않는다.

### ⭐️ 반환 타입을 명시해야 하는 이유

#### 🔗 오류의 위치를 제대로 표시해준다.

#### 🔗 반환 타입을 명시하면 함수에 대해 명확히 알 수 있다.

반환 타입을 명시하려면 구현하기 전에 입력 타입과 출력 타입이 무엇인지 알아야 한다. 추후에 코드가 조금 변경되어도 그 함수의 시그니처는 쉽게 바뀌지 않는다.

미리 타입을 명시하는 방법

→ 함수를 구현하기 전에 테스트를 먼저 작성하는 테스트 주도 개발(test driven development, TDD)과 비슷하다.

전체 타입 시그니처를 먼저 작성하면 구현에 맞추어 주먹구구식으로 시그니처가 작성되는 것을 방지하고 제대로 원하는 모양을 얻게 된다.

#### 🔗 명명된 타입을 사용하기 위해서이다.

예를 들어, 함수에서 반환 타입을 명시하지 않는다고 해보자.

```typescript
interface Vector2D { x: number; y: number; }
function add(a: Vector2D, b: Vector2D) {
  return { x: a.x + b.x, y: a.y + b.y };
}
```

타입스크립트는 반환 타입을 `{ x: number;  y: number; }`로 추론한다. 이런 경우 `Vector2D`와 호환되지만, 입력이 `Vector2D`인데 출력은 `Vector2D`가 아니기 때문에 사용자 입장에서 당황스럽다.

반환 타입을 명시하면 더욱 직관적인 표현이 된다. 그리고 반환 값을 별도의 타입으로 정의하면 타입에 대한 주석을 작성할 수 있어, 더욱 자세한 설명이 가능하다. (아이템 48)

추론된 반환 타입이 복잡해질 수록 명명된 타입을 제공하는 이점은 커진다.

⭐️ 린터(linter)를 사용하고 있다면 `eslint` 규칙 중 **`no-inferrable-types`**&#xC744; 사용해 작성된 모든 타입 구문이 정말로 필요한지 확인할 수 있다.

## 📍 요약

* 타입스크립트가 타입을 추론할 수 있다면 타입 구문을 작성하지 않는게 좋다.
* 이상적인 경우 함수/메서드의 시그니처에는 타입구문이 있지만, 함수 내의 지역 변수에는 타입 구문이 없다.
* 추론될 수 있는 경우라도 객체 리터럴과 함수 반환에는 타입 명시를 고려해야 한다. 이는 내부 구현의 오류가 사용자 코드 위치에 나타나는 것을 방지한다.
*
