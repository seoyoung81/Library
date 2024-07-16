# 📎 아이템 20 다른 타입에는 다른 변수 사용하기

자바스크립트에서는 한 변수를 다른 목적을 가지는 다른 타입으로 재사용해도 된다.

```javascript
let productId = "12-34-56";
fetchProduct(productId); // string으로 사용

productId = 123456;
fetchProductBySerialNumber(productId); // number로 사용
```

반면 타입스크립트에서는 두 가지 오류가 발생한다.

```typescript
let productId = "12-34-56";
fetchProduct(productId);

productId = 123456;
// '123456' 형식은 'string'에 할당할 수 없다.
fetchProductBySerialNumber(productId); 
// 'string' 형식의 인수는 'number' 형식의 매개변수에 할당될 수 없다.
```

타입스크립트는 `"12-34-56`"보고, `id`의 타입을 `string`으로 추론했다. `string` 타입에는 `number` 타입을 할당할 수 없기 때문에 오류가 발생한다.&#x20;

**⭐️ 변수의 값은 바뀔 수 있지만, 그 타입은 보통 바뀌지 않는다.**

타입을 바꿀 수 있는 방법은 범위를 좁히는 것인데, 새로운 변수값을 포함하도록 확장하는 것이 아니라 타입을 더 작게 제한하는 것이다. 이 관점에 반하는 타입 지정 방법이 있고, 이 방법은 예외이지 규칙이 아니다. (아이템22, 41)

## 📍 유니온 타입

🔗 오류가 발생한 앞의 예제를 고쳐보자.&#x20;

id 타입을 바꾸지 않으려면, string과 number를 모두 포함할 수 있도록 타입을 확장하면 된다. string|number로 표현하며, 유니온 타입이라고 한다.

```typescript
let productId: string | number = "12-34-56";
fetchProduct(productId);

productId = 123456;  // OK
fetchProductBySerialNumber(productId);  // OK
```

할당문에서 유니온 타입으로 범위가 좁혀졌기 때문에 첫 번째 함수에서는 `id`가 `string`으로, 두 번째 함수에서는 `id`가 `number`라고 판단한다.&#x20;

유니온 타입으로 코드가 동작하겠지만, 더 많은 문제가 생길 수 있다. id를 사용할 때마다 값이 어떤 타입인지 확인해야 하기 때문에 유니온 타입은 `string`이나 `number` 같은 간단한 타입에 비해 다루기 더 어렵다.

🔗 차라리 별도의 변수를 도입하는 것이 낫다.

```typescript
const productId = "12-34-56";
fetchProduct(productId);

const serial = 123456;  // OK
fetchProductBySerialNumber(serial);  // OK
```

## 📍 다른 타입에는 별도의 변수를 사용하게 바람직한 이유

앞의 id와 재사용한 id는 서로 관련이 없다. 그냥 변수를 재사용했을 뿐이다. 변수를 무분별하게 재사용하면 타입 체커와 사람 모두에게 혼란을 준다.

* 서로 관련이 없는 두 개의 값을 분리한다.(id, serial)
* 변수명을 더 구체적으로 지을 수 있다.
* 타입 추론을 향상시키며, 타입 구문이 불필요해진다.
* 타입이 좀 더 간결해진다. (string|number 대신 string, number 사용)
* let 대신 const로 변수를 선언하게 된다. const로 변수를 선언하면 코드가 간결해지고, 타입 체커가 타입을 추론하기에도 좋다.

타입이 바뀌는 변수는 되도록 피해야 하며, 목적이 ㄷ라느 곳에는 별도의 변수명을 사용해야한다.

## 📍 재사용되는 변수, 가려지는 변수

다음은 가려지는 변수(shadowed) 예제이다.

```typescript
const productId = "12-34-56";
fetchProduct(productId);

{
  const productId = 123456;  // OK
  fetchProductBySerialNumber(productId);  // OK
}
```

여기서 두 id는 이름이 같지만 실제로는 아무런 관계가 없다.&#x20;

→ 각기 다른 타입으로 사용되어도 문제 없다.

그런데 동일한 변수명에 타입이 다르다면, 타입스크립트 코드는 잘 동작할지 몰라도 사람에게 혼란을 준다. 목적이 다른 곳에는 별도의 변수명을 사용하자.

\*린터 규칙을 통해 '가려지는' 변수를 사용하지 못하도록 한다.

이번 아이템에서는 변수에 기본형(primitive) 타입을 할당하는 방법에 대해 다뤘다. 아이템 23에서 변수에 객체를 할당하는 방법이 나온다.

## 📍 요약

* 변수의 값은 바뀔 수 있지만 타입은 일반적으로 바뀌지 않는다.
* 혼란을 막기 위해 타입이 다른 값을 다룰 때에는 변수를 재사용하지 않도록 한다.
