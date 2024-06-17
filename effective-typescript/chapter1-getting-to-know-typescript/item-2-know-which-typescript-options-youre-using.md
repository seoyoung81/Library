# 📎 아이템 2 타입스크립트 설정 이해하기

❓다음 코드가 오류 없이 타입 체커를 통과할 수 있을까❓

```typescript
function add(a, b) {
  return a + b;
}
add(10, null);
```

타입스크립트 컴파일러 설정을 모른다면 대답할 수 없는 질문이다. (현재 시점에서는 설정이 거의 100개이다.)

이 설정들은 커맨드 라인 또는 tsconfig.json 설정 파일을 통해서 가능하다.

`$ tsc --noImplicitAny program.ts`

```json
{
    "compilerOptions": {
        "noImplicitAny": true
    }
}
```

설정 파일을 이용하여 동료들이나 다른 도구들이 알 수 있게 하는 것이 좋다.

타입스크립트의 설정을 제대로 사용하려면 `noImplicitAny`와 `strictNullChecks`를 이해해야 한다.

## 📍타입스크립트 설정

### `noImplicitAny`

> 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어한다.

```typescript
// noImplicitAny가 해제되어 있을 때에는 유효하다.
function add(a, b) {
  return a + b;
}
```

편집기에서 add 부분에 마우스를 올려보면, 타입스크립트가 추론한 함수의 타입을 알 수 있다.

`function add(a: any, b: any): any`

`any` 타입을 매개변수에 사용하면 타입 체커는 무력해진다. `any`는 우용하지만 주의해서 사용해야 한다.

```typescript
function add(a, b) {
  //         ~    Parameter 'a' implicitly has an 'any' type
  //            ~ Parameter 'b' implicitly has an 'any' type
  return a + b;
}
```

any를 코드에 넣지 않았지만, any 타입으로 간주되기 때문에 이를 `암시적 any`라고 부른다. 하지만 같은 코드임에도 `noImplicitAny`가 설정되었다면 오류가 된다.&#x20;

→ 이 오류들은 명시적으로 `: any`라고 선언해주거나 더 분명한 타입을 사용하면 해결할 수 있다.

```typescript
function add(a: number, b: number) {
  return a + b;
}
```

타입스크립트는 타입 정보를 가질 때 효과적이므로 `noImplicitAny`를 설정하여 모든 변수에 타입을 명시하는 것이 좋다.

→ 문제를 발견하기 수월해지고, 코드의 가독성이 좋아지며, 개발자의 생산성이 향상된다.&#x20;

(`noImplicitAny` 설정해제는 자바스크립트를 타입스크립트로 전환하는 상황에만 필요하다.)



### `strictNullChecks`

> `null`과 `undefined`가 모든 타입에서 허용되는지 확인하는 설정이다.

```typescript
// strictNullChecks 가 해제되었을 때 유효한 코드이다.
const x: number = null;  // OK, null is a valid number
```

`strictNullChecks`를 설정하면 오류가 된다.

```typescript
const x: number = null;
//    ~ Type 'null' is not assignable to type 'number'
// null 대신 undefined를 써도 같은 오류가 난다.
```

만약 null을 허용하려고 한다면, 의도를 명시적으로 드러냄으로써 오류를 고칠 수 있다.

```typescript
const x: number | null = null;
```

null을 허용하지 않으려면, 이 값이 어디서부터 왔는지 찾아야 하고, null을 체크하는 코드나 단언문을 추가해야 한다.

```typescript
const statusEl = document.getElementById('status');
statusEl.textContent = 'Ready';
// ~~~~~ 'statusEl' is possibly 'null'.

if (statusEl) {
  statusEl.textContent = 'Ready';  // OK, null has been excluded
}
statusEl!.textContent = 'Ready';  // OK, we've asserted that el is non-null
```

`strictNullChecks`는 null과 undefined 관련된 오류를 잡는데 도움이 되지만, 코드 작성을 어렵게 한다. 따라서, 설정하는 것이 좋지만, 타입스크립트가 처음이거나 자바스크립트 코드를 마이그레이션하는 중이라면 설정하지 않아도 괜찮다. `strictNullChecks`를 설정하려면 `noImplicitAny`를 먼저 설정해야 한다. (설정 없이 개발하기로 선택했다면, "undefined는 객체가 아니다" 라는 런타임 오류를 주의하자.)



언어에 의미적으로 영향을 미치는 설정을 모두 체크하고 싶다면 strict 설정을 하면된다. strict 설정을 하면 대부분의 오류를 잡아낼 수 있다.



## 📍요약

* 타입스크립트 컴파일러는 언어의 핵심 요소에 영향을 미치는 몇 가지 설정을 포함하고 있다.
* 타입스크립트 설정은 커맨드 라인보다는 tsconfig.json 파일을 사용하는 것이 좋다.
* 자바스크립트 프로젝트를 타입스크립트로 전환하는게 아니라면 noImplicitAny를 설정하는 것이 좋다.
* undifined는 객체가 아니다 같은 런타임 오류를 방지하기 위해 strictNullChecks를 설정하는 것이 좋다.
* 타입스크립트에서 엄격한 체크를 하고 싶다면 strict 설정을 고려해야 한다.

