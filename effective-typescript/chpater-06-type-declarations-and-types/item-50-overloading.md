# 📎 아이템 50 오버로딩 타입보다는 조건부 타입을 사용하기

```typescript
function double(x) {
  return x + x;
}
```

double 함수에 타입 정보를 추가해보자. (타입스크립트의 오버로딩 개념을 사용했다.)

```typescript
function double(x: number|string): number|string;
function double(x: any) { return x + x; }
```

선언이 틀린 것은 아니지만, 모호한 부분이 있다.

```typescript
const num = double(12);    // string | number
const str = double('x');    // string | number
```

double에 number 타입을 매개변수로 넣으면 number 타입을 반환한다. string 타입을 매개변수로 넣으면 string 타입을 반환한다. 그러나 선언문에는 number 타입을 매개변수로 넣고 string 타입을 반환하는 경우도 포함되어 있다.

## 🔗 오버로딩 개선하기

### **⭐️ 제너릭을 사용하기**

```typescript
function double<T extends number | string>(x: T): T; 

function double(x: any) {
  return x + x;
}

const num = double(12); // 타입이 12
const str = double('x'); // 타입이 "x"
```

타입을 구체적으로 만들었지만 너무 과하게 구체적이다.

### **⭐️ 여러 가지 타입 선언으로 분리하기**

타입스크립트에서 함수의 구현체는 하나지만, 타입 선언은 몇 개든지 만들 수 있다. 이를 활용하여 double의 타입을 개선하자.

```typescript
function double(x: number): number; 
function double(x: string): string; 

function double(x: any) {
  return x + x;
}

const num = double(4); // 타입이 number
const str = double('3'); // 타입이 string
```

함수 타입이 조금 명확해졌지만 여전히 버그는 남아있다. string이나 number 타입의 값으로는 잘 동작하지만, 유니온 타입 관련해서 문제가 발생한다.

```typescript
function f(x: number|string) {
  return double(x);
  // 'string|number' 형식의 인수는 'string' 형식의 매개변수에 할당될 수 없다.
```

double 함수의 호출은 정상적이며 string|number 타입이 반환되기를 기대한다. <mark style="background-color:blue;">하지만, 타입스크립트는 오버로딩 타입 중에서 일치하는 타입을 찾을 때 까지 순차적으로 검색한다.</mark> 그래서, 오버로딩 타입의 마지막 선언(string 버전)까지 검색했을 때, string|number 타입은 string에 할당할 수 없기 때문에 오류를 발생한다.

### **⭐️ 조건부 타입(conditional type)을 사용하기**

조건부 타입은 타입 공간의 if 구문과 같다.

```typescript
function double<T extends number | string>(
  x: T
): T extends string ? string: number'
function double(x: any) { return x + x; }
```

제너릭 예제보다 반환타입이 더 정교하다. 조건부 타입은 자바스크립트의 삼항 연산자(?:)처럼 사용하면 된다.

* T가 string의 부분집합이면(string, 문자열 리터럴, 문자열 리터럴의 유니온), 반환 타입이 string이다.
* 그 외의 경우는 반환 타입이 number

조건부 타입이라면 앞선 모든 예제가 동작한다.

```typescript
const num = double(12);    // number
const str = double('x');    // string

// function f(x: string | number): string | number
function f(x: number|string) {
  return double(x);
}
```

유니온에 조건부 타입을 적용하면, 조건부 타입의 유니온으로 분리되기 때문에 number|string의 경우에도 동작한다. 예를 들어, T가 number|string이라면, 타입스크립트는 조건부 타입을 다음 단계로 해석한다.

## 📍 정리

오버로딩 타입이 작성하기는 쉽지만, 조건부 타입은 개별 타입의 유니온으로 일반화하기 때문에 타입이 더 명확해진다. 오버로딩 타입은 독립적으로 처리되며, 조건부 타입은 타입 체커가 단일 표현식으로 받아들이기 때문에 유니온 문제를 해결할 수 있다. 오버로딩 타입을 작성중이면 조건부 타입으로 개선할 수 있을지 검토해보자.

## 📍 요약

* 오버로딩 타입보다 조건부 타입을 사용하는 것이 좋다. 조건부 타입은 추가적인 오버로딩 없이 유니온 타입을 지원할 수 있다.
