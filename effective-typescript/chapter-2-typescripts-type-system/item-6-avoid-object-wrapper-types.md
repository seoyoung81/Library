# 📎 아이템 10 객체 래퍼 타입 피하기

자바스크립트에는 객체 이외에도 기본형 값들에 대한 일곱가지 타입(string, number, boolean, null, undefined, symbol, bigint)이 있다.

기본형들은 불변(immutable)이며 메서드를 가지지 않는다는 점에서 객체와 구분된다. 하지만, string의 경우 메서드를 가지고 있는 것처럼 보인다.

## 📍 string

```
> 'primitive'.charAt(3)
"m"
```

charAt은 string의 메서드가 아니며 string을 사용할 때 자바스크립트 내부적으로 많은 동작이 일어난다. string '기본형'에는 메서드가 없지만, 자바스크립트에는 메서드를 가지는 String '객체' 타입이 정의되어 있다. 자바스크립트는 기본형과 객체 타입을 서로 자유롭게 변환한다.

→ string 기본형에 charAt 같은 메서드를 사용할 때, 자바스크립트는 기본형을 String 객체로 래핑(wrap)하고, 메서드를 호출하고, 마지막에 래핑한 객체를 버린다.

```typescript
// Don't do this!
const originalCharAt = String.prototype.charAt;
String.prototype.charAt = function(pos) {
  console.log(this, typeof this, pos);
  return originalCharAt.call(this, pos);
};
console.log('primitive'.charAt(3));
```

이 코드는 다음을 출력한다.

```typescript
[String: 'primitive'] 'object' 3
m
```

메서드 내의 `this`는 `string` 기본형이 아닌 `String` 객체 래퍼이다. `String` 객체를 직접 생성할 수도 있으며, `string` 기본형처럼 동작한다. 그러나 `string` 기본형과 `String` 객체 래퍼가 항상 동일하게 동작하는 것은 아니다.

`String` 객체는 오직 자기 자신하고만 동일하다.

```typescript
> "hello" === new String("hello")
false
> new Stirng("hello") === new String("hello")
false
```

객체 래퍼 타입의 자동 변환은 당황스러운 동작을 보일 때가 있다. 예를 들어, 어떤 속성을 기본형에 할당한다면 그 속성이 사라진다.

```typescript
> x = "hello"
> x.language = 'English'
'English'
> x.language
undefined
```

실제로는 x가 `String` 객체로 변환된 후 `language` 속성이 추가되었고, `language` 속성이 추가된 객체는 버려진 것이다.

## 📍 다른 기본형들

다른 기본형에도 동일하게 객체 래퍼 타입이 존재한다. number에는 Number, boolean에는 Boolean, symbol에는 Symbol, bigint에는 Bigint가 존재한다.&#x20;

null과 undefined에는 객체 래퍼가 없다.

이 래퍼 타입들 덕분에 기본형 값에 메서드를 사용할 수 있고, 정적 메서드(String.fromCharCode 같은)도 사용할 수 있다. 그러나 보통은 래퍼 객체를 직접 생성할 필요가 없다.

## 📍 타입스크립트

타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링한다.&#x20;

* string과 String
* number과 Number
* boolean과 Boolean
* symbol과 Symbol
* bigint과 Bigint

특히, `string`을 사용할 때는 유의해야한다. `string`을 `String`이라고 잘못 타이핑하기 쉽고, 실수를 하더라도 처음에는 잘 동작하는 것처럼 보이기 때문이다.

```typescript
function getStringLen(foo: String) {
  return foo.length;
}

getStringLen("hello");  // OK
getStringLen(new String("hello"));  // OK
```

그러나 `string`을 매개변수로 받는 메서드에 `String` 객체를 전달하면 문제가 생긴다.

```typescript
function isGreeting(phrase: String) {
  return ['hello', 'good day'].includes(phrase);
  //                                    ~~~~~~
  // Argument of type 'String' is not assignable to parameter of type 'string'.
  // 'string' is a primitive, but 'String' is a wrapper object.
  // Prefer using 'string' when possible.
}
```

`string`은 `String`에 할당할 수 있지만 `String`은 `string`에 할당할 수 없다. 오류메세지대로 `string` 타입을 사용해야한다.



타입스크립트가 제공하는 타입 선언은 전부 기본형 타입으로 되어있다.

래퍼 객체는 타입 구문의 첫 글자를 대문자로 표기하는 방법으로도 사용할 수 있다.

```typescript
const s: String = "primitive";
const n: Number = 12;
const b: Boolean = true;
```

런타임의 값은 객체가 아니고 기본형이다. 그러나 기본형 타입은 객체 래퍼에 할당할 수 있기 때문에 타입스크립트는 기본형 타입을 객체 래퍼에 할당하는 선언을 허용한다. 그러나 기본형 타입을 객체 래퍼에 할당하는 구문은 오해하기 쉽고, 굳이 그렇게 할 필요도 없다.

그냥 기본형 타입을 사용하는 것이 낫다.

new 없이 BigInt와 Symbol를 호출하는 경우는 기본형을 생성하기 때문에 사용해도 좋다.

```typescript
> typeof BigInt(1234)
"bigint"
> typeof Symbol('sym')
"symbol"
```

이들은 BigInt와 Symbol '값'이지만, 타입스크립트 타입은 아니다. 앞 예제의 결과는 bigint와 symbol 타입의 값이 된다.

## 📍 요약

* 기본형 값에 메서드를 제공하기 위해 객체 래퍼 타입이 어떻게 쓰이는지 이해해야 한다. 직접 사용하거나 인스턴스를 생성하는 것은 피해야 한다.
* 타입스크립트 객체 래퍼 타입은 지양하고, 대신 기본형 타입을 사용해야 한다. String 대신 string, Number 대신 number, Boolean 대신 boolean, Symbol 대신 symbol, BigInt 대신 bigint를 사용해야한다.
