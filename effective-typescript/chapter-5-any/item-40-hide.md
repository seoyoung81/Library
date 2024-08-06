# 📎 아이템 40 함수 안으로 타입 단언문 감추기

함수를 작성하다 보면, 외부로 드러난 타입은 간단하지만 내부 로직이 복잡해서 안전한 타입으로 구현하기 어려운 경우가 있다. 함수의 모든 부분을 안전한 타입으로 구현하는 것이 이상적이지만, 불필요한 예외 상황까지 고려해 가며 타입 정보를 힘드렉 구성할 필요는 없다.

내부에는 타입 단언을 사용하고, 외부로 드러나는 타입 정의를 정확히 명시하는 정도로 끝내는게 낫다. 타입 단언문이 드러나는 것보다 타입이 정의된 함수 안으로 숨기는 것이 더 좋은 설계이다.

## 📍 예제

어떤 함수가 자신의 마지막 호출을 캐시하도록 만든다고 가정해보자. (실행 시간이 오래걸리는 함수 호출을 개선하는 방법) 어떤 함수든 캐싱할 수 있도록 래퍼 함수 cacheLast를 만들 수 있다.

```typescript
declare function cacheLast<T extends Function>(fn: T): T;

declare function shallowEqual(a: any, b: any): boolean;
function chcheLast<T extends Function>(fn: T): T {
  let lastArgs: any[]|null = null;
  let lastResult: any;
  retturn function(...args: any[]) {
  // 에러: (...args: any[]) => any 형식은 T 형식에 할당할 수 없다.
  if (!lastArgs || !shallowEqual(lastArgs args)) {
    lastResult = fn(...args);
    lastArgs = args;
  }
  return lastResult;
  }
}
```

원본 함수와 T 타입의 관련을 알지 못해 오류가 발생했다. 그러나 원본 함수 T 타입과 동일한 매개변수로 호출되고 반환값 역시 예상한 결과가 되기 때문에, 타입 단언문을 추가해서 오류를 제거하는 것이 큰 문제가 되지는 않는다.

```typescript
function chcheLast<T extends Function>(fn: T): T {
  let lastArgs: any[]|null = null;
  let lastResult: any;
  retturn function(...args: any[]) {
  if (!lastArgs || !shallowEqual(lastArgs args)) {
    lastResult = fn(...args);
    lastArgs = args;
  }
  return lastResult;
  } as unkown as T;
}
```

함수 내부에는 `any`가 많지만 타입 정의에는 `any`가 없기 때문에 `cacheLast`를 호출하는 쪽에서는 `any`가 사용됐는지 알지 못한다.

## 📍 요약

* 타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 한다. 불가피하게 사용해야 한다면, 정확한 정의를 가지는 함수 안으로 숨기자.
