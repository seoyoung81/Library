# 📎 아이템 38 any 타입은 가능한 한 좁은 범위에서만 사용하기

## 📍 함수와 관련된 any의 사용법

```typescript
declare function processBar(b: Bar) {}

const f1 () {
  const x = expressionReturnFoo();
  processBar(x); // 에러: 'Foo' 형식의 인수는 'Bar' 형식의 매개 변수에 할당될 수 없습니다.
}
```

문맥상 x라는 변수가 동시에 Foo 타입과 Bar 타입에 할당가능하다면, 오류를 제거하는 방법은 두가지이다.

```typescript
function f1() {
  const x: any = expressionReturnFoo();  // 이렇게 하지 말자.
  processBar(x);
}

function f2() {
 const x = expressionReturningFoo();
 processBar(x as any);  // 이게 낫다.
}
```

`x: any`보다 `x as any` 형태가 권장되는 이유는 `any` 타입이 `processBar` 함수의 매개변수에서만 사용된 표현식이므로 다른 코드에는 영향을 미치지 않기 때문이다.

(f1에선ㄴ 마지막까지 x의 타입이 any이고, f2에서는 processBar 호출 이후에 x가 그대로 Foo 타입이다.)

### 🔗 함수가 any를 반환하는 경우

만약, f1 함수가 x를 반환한다면 문제가 커진다.

```typescript
function f1() {
  const x: any = expressionReturnFoo();
  processBar(x);
  return x;
}

function g() {
 const foo = f1();  // 타입이 any
 foo.fooMethod();  // 이 함수 호출은 체크되지 않는다.
}
```

g 함수 내에서 f1이 사용되므로 f1의 반환 타입인 any 타입이 foo의 타입에 영향을 미친다. 이렇게 함수에서 any를 반환하면 프로젝트 전반에 영향을 미치게 된다. (any의 사용 범위를 좁게 제한하는 f2 함수는 영향을 미치지 않는다.)

**📍 함수의 반환 타입을 추론할 수 있는 경우, 함수 반환 타입을 명시하기**

함수의 반환 타입을 명시하면 any 타입이 함수 바깥으로 영향을 미치는 것을 방지할 수 있다.&#x20;

### 🔗 @ts-ignore사용하여 오류 제거하기

```typescript
function f1() {
  const x = expressionReturningFoo();
  // @ts-ignore
  processBar(x);
  return x;
}
```

`@ts-ignore`를 사용한 다음 줄의 오류가 무시된다. 그러나 근본적인 원인 해결이 아니기 때문에 좋지 않은 방법이다.

## 📍 객체와 관련한 any의 사용법

```typescript
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value
    // 에러: 'foo' 속성이 'Foo' 타입에 필요하지만 'Bar' 타입에는 없다.
  }
}
```

단순히 생각하면 `config` 객체 전체를 `as any`로 선언할 수 있다.

```typescript
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value
  }
} as any;  // 이렇게 하지 말자.
```

객체 전체를 `any`로 단언하면 다른 속성들(a와 b) 역시 타입 체크가 되지 않으므로 코드의 최소한의 범위에만 `any`를 사용하는 것이 좋다.

```typescript
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value as any
  }
};
```

## 📍 요약

* 의도치 않은 타입 안전성의 손실을 피하기 위해서 any의 사용범위를 최소한으로 좁혀야 한다.
* 함수의 반환 타입이 any인 경우 타입 안정성이 나빠지므로 any 타입을 절대 반환하지 말자.
* 강제로 타입 오류를 제거하려면 any 대신 @ts-ignore 를 사용하는 것이 좋다.
