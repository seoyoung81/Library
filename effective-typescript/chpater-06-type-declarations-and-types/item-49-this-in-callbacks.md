# 📎 아이템 49 콜백에서 this에 대한 타입 제공하기

`this`는 다이나믹 스코프이다. 다이나믹 스코프의 값은 **'정의된'** 방식이 아니라 **'호출된'** 방식에 따라 달라진다. this는 전형적으로 객체의 현재 인스턴스를 참조하는 클래스에서 가장 많이 사용된다.

```typescript
class C {
  vals = [1, 2, 3];
  logSquares() {
    for (const val of this.vals) {
      console.log(val ** 2);
    }
  }
}

const c = new C();
c.logSquares();

// 코드를 실행하면
1
4
9
```

`logSquares`를 외부 변수에 넣고 호출해보자.

```typescript
const c = new C();
const method = c.logSquares;
method();

// 런타임 오류 발생
Uncought TypeError: undefined의 'vals' 속성을 읽을 수 없다.
```

`c.logSquares()`가 실제로는 두 가지 작업을 수행하기 때문에 문제가 발생한다. `C.prototype.logSquares`를 호출하고, 또한 this의 값을 c로 바인딩한다. 앞듸 코드에서는 logSquares의 참조변수를 사용함으로써 두 가지 작업을 분리 했고, this의 값은 undefined로 설정된다.

## 🔗 this 바인딩 제어

**1️⃣ call을 사용하여 명시적으로 this 바인딩하기**

```typescript
const c = new C();
const method = c.logSquares;
method.call(c);  // Logs the squares again
```

this가 반드시 c의 인스턴스에 바인딩되어야 하는 것은 아니며, 어떤 것이든 바인딩할 수 있다. 라이브러리들은 API의 일부에서 this의 값을 사용할 수 있게 하고, DOM에서도 this를 바인딩할 수 있다.

**2️⃣ 화살표 함수 사용하기**

this 바인딩은 자바스크립트 동작이기 때문에, 타입스크립트도 this 바인딩을 그대로 모델링한다. 작성 중인 라이브러리에 this를 사용하는 콜백 함수가 있다면, this 바인딩 문제를 고려해야 한다.

콜백 함수의 매개변수에 this를 추가하고, 콜백 함수를 call로 호출해서 해결할 수 있다. 콜백 함수의 매개변수에 this를 추가하면 this 바인딩이 체크되기 때문에 실수를 방지할 수 있다.

만약, 라이브러리 사용자가 콜백을 화살표 함수로 작성하고 this를 참조하려고하면 타입스크립트가 문제를 잡아낸다.&#x20;

## 📍 정리

this 사용법을 기억하고, 콜백 함수에서 this 값을 사용해야한다면 this는 API의 일부가 되는 것이기 때문에 타입 선언에 포함해야 된다.

## 📍 요약

* this 바인딩이 동작하는 원리를 이해해야 한다.
* 콜백 함수에서 this를 사용해야 한다면, 타입 정보를 명시해야 한다.
