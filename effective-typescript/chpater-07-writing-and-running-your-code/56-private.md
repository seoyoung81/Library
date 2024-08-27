# 📎 아이템 56 정보를 감추는 목적으로 private 사용하지 않기

## 📍 `private`

자바스크립트는 클래스에 비공개 속성을 만들 수 없다. 관례로 접두사에 언더스코어(\_)를 붙여 비공개 속성임을 나타냈다. 단순히 비공개라고 표시한 것 뿐이지 일반적인 속성과 동일하게 클래스 외부로 공개되었다는 점을 주의해야 한다.

타입스크립트에는 public, protected, private 접근 제어자를 사용해서 공개 규칙을 강제할 수 있는 것으로 오해할 수 있다. 하지만, 타입스크립트 키워드이기 때문에 컴파일 후에는 제거된다.&#x20;

🔗 타입스크립트 코드를 컴파일 후,  자바스크립트 코드로 변환되는 예제

```typescript
class Diary {
  private secret = '...';
}
const diary = new Diary();
diary.secret;  // private 접근 불가
```

<pre class="language-javascript"><code class="lang-javascript"><strong>class Diary {
</strong>  constructor() {
    this.secret = '...';
  }
}
const diary = new Diary();
diary.secret;
</code></pre>

`private` 키워드는 사라지고, secret은 일반 속성이므로 접근할 수 있다. 타입스크립트접근 제어자들은 컴파일 시점에만 오류를 표시하고, 언더스코어 관례와 마찬가지로 런타임에는 효력이 없다.

단언문을 사용하면 타입스크립트 상태에서도 `private` 속성에 접근할 수 있다.

```typescript
(diary as any).secret;
```

즉, 정보를 감추기 위해 `private`를 사용하면 안 된다.

## 📍 정보를 숨기기 위해서

### closure

자바스크립트에서 정보를 숨기기 위해 가장 효과적인 방법은 클로저(closure)를 사용하는 것이다. 생성자에서 클로저를 만들어낼 수 있다.

생성자 외부에서 변수에 접근할 수 없기 때문에 변수에 접근해야하는 메서드도 생성자 내부에 정의되어야 하고, 메서드 정의가 생성자 내부에 존재하게 되면, 인스턴스를 생성할 때 마다 각 메서드의 복사본이 생겨 메모리를 낭비하게 된다.

또한 동일한 클래스로부터 생성된 인스턴스라도 서로의 비공개 데이터에 접근하는 것이 불가능하기 때문에 철저하게 비공개이면서 동시에 불편하다.

```typescript
declare function hash(text: string): number;

class PasswordChecker {
  checkPassword: (password: string) => boolean;
  constructor(passwordHash: number) {
    this.checkPassword = (password: string) => {
      return hash(password) === passwordHash;
    }
  }
}

const checker = new PasswordChecker(hash('s3cret'));
checker.checkPassword('s3cret'); // true
```

### 비공개 필드 기능

\#를 붙여서 타입 체크와 런타임 모두에서 비공개로 만드는 역할을 한다.

```typescript
class PasswordChecker {
  #passwordHash: number;
  
  constructor(passwordHash: number) {
    this.#passwordHAsh = passwordHash;
  }
  
  checkPassword(password: string) {
    return hash(password) === this.#passwordHash;
  }
}

const checker = new PasswordChecker(hash('s3cret'));
checker.checkPassword('secret');  // false
checker.checkPassword('s3cret');  // true
```

\#passwordHash 속성은 클래스 외부에서 접근할 수 없다. 클로저 기법과 다르게 클래스 메서드나 동일한 클래스의 개별 인스턴스끼리는 접근이 가능하다. 비공개 필드를 지원하지 않는 자바스크립트 버전으로 컴파일하게 되면, WeapMap을 사용한 구현으로 대체된다.

만약 설계 관점에서 캡슐화가 아닌 '보안'에 대해 걱정하고 있다면, 내장된 프로토타입과 함수에 대한 변조 같은 문제를 알고 있어야 한다.

## 📍 요약

* public, protected, private 접근 제어자는 타입 시스템에서만 강제된다. 런타임에는 소용이 없으며 단언문을 통해 우회할 수 있다. 접근제어자로 데이터를 감추려고 해서는 안 된다.
* 확실히 데이터를 감추고 싶다면 클로저를 사용해야 한다.
