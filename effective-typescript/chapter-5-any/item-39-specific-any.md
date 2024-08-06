# 📎 아이템 39 any를 구체적으로 변형해서 사용하기

any는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입이다.

모든 숫자, 문자열, 배열, 객체, 정규식, 함수, 클래스, DOM dpfflajsxm, null, undefined까지도 포함된다.

↔️ 일반적인 상황에서는 any보다 더 구체적으로 표현할 수 있는 타입이 존재한다.&#x20;

→ 구체적인 타입을 찾아 안정성을 높이도록 하자.

## 📍예제

### 🔗 any 타입의 값을 그대로 정규식이나 함수에 넣기 (권장X)

```typescript
function getLengthBad(array: any) {  // Don't do this!
  return array.length;
}

function getLength(array: any[]) {  // This is better
  return array.length;
}
```

**⭐️ `any`보다 `any[]`를 사용하는 것이 더 좋은 이유**

1. 함수 내의 array.length 타입이 체크된다.
2. 함수의 반환 타입이 any 대신 number로 추론된다.
3. 함수 호출될 때 매개변수가 배열인지 체크된다.

배열이 아닌 값을 넣어 실행해보면, getLength는 오류를 표시하지만 getLengthBad는 오류를 잡아내지 못한다.

```typescript
getLengthBad(/123/);  // No error, returns undefined
getLength(/123/);
//        ~~~~~
// Argument of type 'RegExp' is not assignable to parameter of type 'any[]'.

getLengthBad(null);  // No error, throws at runtime
getLength(null);
//        ~~~~
// Argument of type 'null' is not assignable to parameter of type 'any[]'.
```

함수의 매개변수를 구체화할 때, 배열의 배열 형태라면 `any[][]`처럼 선언하면 된다. 함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 `{[key: string]: any}`처럼 선언하면 된다.&#x20;

```typescript
function hasAKeyThatEndsWithZ(o: Record<string, any>) {
  for (const key in o) {
    if (key.endsWith('z')) {
      console.log(key, o[key]);
      return true;
    }
  }
  return false;
}
```

앞의 예제처럼 함수의 매개변수가 객체지만 값을 알 수 없다면 `{[key: string]: any}` 대신 모든 기본형(non-primitive) 타입을 포함하는 `object` 타입을 사용할 수도 있다.

⭐️ `object` 타입은 객체의 키를 열거 할 수 있지만 속성에 접근하지 못한다는 점에서 `{[key: string]: any}`와 다르다.

```typescript
function hasAKeyThatEndsWithZ(o: object) {
  for (const key in o) {
    if (key.endsWith('z')) {
      console.log(key, o[key]);
      //               ~~~~~~ Element implicitly has an 'any' type
      //                      because type '{}' has no index signature
      // {} 형식에 인덱스 시그니처가 없으므로 요소에 암시적으로 'any' 형식이 있다.
      return true;
    }
  }
  return false;
}
```

⭐️ 객체지만 속성에 접근할 수 없어야 한다면 `unknown` 타입이 필요한 상황 (아이템 42)

### **🔗 함수의 타입에도 단순히 any를 사용해서는 안된다. 구체화 방법 3가지**

```typescript
type Fn0 = () => any;  // any function callable with no params
type Fn1 = (arg: any) => any;  // With one param
type FnN = (...args: any[]) => any;  // With any number of params
                                     // same as "Function" type
                                     // any는 any를 반환, any[]는 number를 반환
```

모두 any 보다 구체적이다.&#x20;

## 📍요약

* `any`를 사용할 때는 정말로 모든 값이 허용되어야만 하는지 검토하자.
* `any`보다 더 정확하게 모델링할 수 있도록 `any[]` 또는 `{[id: string]: any}` 또는 `() => any` 처럼 구체적인 형태를 사용해야 한다.
