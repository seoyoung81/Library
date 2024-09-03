# 📎 아이템 61 의존성 관계에 따라 모듈 단위로 전환하기

점진적 마이그레이션을 할 때, 의존성과 관련된 오류 없이 작업하기 위해서 다른 모듈에 의존하지 않는 최하단 모듈부터 시작해서 의존성의 최상단에 있는 모듈을 마지막으로 완성하면 된다.

모듈 단위로 마이그레이션을 시작하기 전에, 모듈 간의 의존성 관계를 시각화해보면 도움이 된다. (dependency graph) 서드파티 모듈과 외부 API 호출에 대한 타입을 먼저 작성하는 것이 좋다.

마이그레이션 과정에서 타입 정보 추가만 하고, 리팩터링을 해서는 안 된다. 오래된 프로젝트일수록 개선이 필요한 부분을 자주 마주치겠지만, 타입스크립트로 전환하는 것에 집중해야 한다.

## 📍 타입스크립트로 전환하며 발견하게 되는 일반적인 오류들

### 선언되지 않은 클래스 멤버

자바스크립트는 클래스 멤버 변수를 선언할 필요가 없지만, 타입스크립트에서는 명시적으로 선언해야 한다. 멤버 변수를 선언하지 않은 클래스가 있는 .js 파일을 .ts로 바꾸면, 참조하는 속성마다 오류가 발생한다.

```typescript
class Greeting {
  constructor(name) {
    this.greeting = 'Hello';
    //   ~~~~~~~~ Property 'greeting' does not exist on type 'Greeting'
    this.name = name;
    //   ~~~~ Property 'name' does not exist on type 'Greeting'
  }
  greet() {
    return `${this.greeting} ${this.name}`;
    //             ~~~~~~~~         ~~~~ Property ... does not exist
  }
}
```

편집기에서 빠른 수정 기능인 '누락된 모든 멤버 추가'를 사용하면 타입을 추론하여 선언문이 추가된다.

```typescript
class Greeting {
  greeting: string;
  name: any;
  constructor(name) {
    this.greeting = 'Hello';
    this.name = name;
  }
  greet() {
    return `${this.greeting} ${this.name}`;
  }
}
```

빠른 수정을 적용한 후에 속성을 훑어보고 any로 추론된 부분은 직접 수정하면 된다.

### 타입이 바뀌는 값

다음 코드는 자바스크립트일 때는 문제가 없지만, 타입스크립트가 되는 순간 오류가 발생한다.

```typescript
const state = {};
state.name = 'New York';
//    ~~~~ Property 'name' does not exist on type '{}'
state.capital = 'Albany';
//    ~~~~~~~ Property 'capital' does not exist on type '{}'
```

한꺼번에 객체를 생성하면 간단히 오류를 해결할 수 있다. 한꺼번에 생성하기 곤란한 경우에는 임시 방편으로 타입 단언문을 사용할 수도 있다.

```typescript
// 한번에 객체 생성
const state = {
  name: 'New York',
  capital: 'Albany',
};  // OK

// 타입 단언문 사용 -> 리팩터링 필요
interface State {
  name: string;
  capital: string;
}
const state = {} as State;
state.name = 'New York';  // OK
state.capital = 'Albany';  // OK
```

🔗 자바스크립트 상태에서 JSDoc과 @ts-check를 사용해 타입 정보를 추가한 상태라면, 타입스크립트로 전환하는 순간 타입 정보가 **'무효화'**된다는 것을 주의해야 한다.

```javascript
// @ts-check
/**
 * @param {number} num
 */
function double(num) {
  return 2 * num;
}

double('trouble');
//     ~~~~~~~~~
// Argument of type 'string' is not assignable to parameter of type 'number'
```

자바스크립트 상태에서 제대로 오류를 표시하지만, 타입스크립트로 전환하게 되면 @ts-check와 JSDoc는 작동하지 않으므로 오류가 사라진다. (double 함수의 매개변수 num의 타입을 any로 추론한다.)

⭐️ JSDoc 타입 정보를 타입스크립트 타입으로 전환해주는 빠른 수정 기능을 사용하자. 타입 정보가 생성된 후에는 JSDoc을 제거하면 된다. → 정상적으로 오류가 표시된다.

**마지막 단계로**, 테스트 코드를 타입스크립트로 전환하면 된다. 로직 코드가 테스트 코드에 의존하지 않기 때문에, 테스트 코드는 항상 의존성 관계도 최상단에 위치하여 마이그레이션의 마지막 단계가 된다. 최하단 모듈부터 타입스크립트로 전환하는 와중에도 테스트 코드를 변경하지 않아도 테스트를 수행할 수 있다는 이점이 있다.

## 📍 요약

* 마이그레이션의 첫 단계는, 서드파티 모듈과 외부 API 호출에 대한 @types를 추가하는 것이다.
* 의존서어 관계도의 아래에서부터 위로 올라가며 마이그레이션을 하면 된다. 첫 번째 모듈은 보통 유틸리티 모듈이다. 의존성 관계도를 시각화하여 진행 과정을 추적하는 것이 좋다.
* 이상한 설계를 발견하더라도 리팩터링을 하면 안 된다. 마이그레이션 작업은 타입스크립트 전환에 집중해야 하며, 나중의 리팩터링을 위해 목록을 만들어 두는 것이 좋다.
* 타입스크립트로 전환하며 발견하게 되는 일반적인 오류들을 놓치지 않아야 한다. 타입 정보를 유지하기 위해 필요에 따라 JSDoc 주석을 활용해야 할 수도 있다.
