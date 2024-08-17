# 📎 아이템 43 몽키 패치보다는 안전한 타입을 사용하기

자바스크립트는 객체와 클래스에 임의의 속성을 추가할 수 있을만큼 유연하다. 종종 웹페이지에서 `window`나 `document`에 값을 할당하여 전역 변수를 만드는 데 사용된다.

<pre class="language-typescript"><code class="lang-typescript"><strong>window.monkey = 'Howler';
</strong><strong>document.monkey = 'Tamarin';
</strong></code></pre>

또는 `DOM` 엘리먼트에 데이터를 추가하기 위해서도 사용된다.

```typescript
const el = document.getElementById('colobus');
el.home = 'tree';
```

객체에 속성을 추가하는 코드 스타일은 제이쿼리(jQuery)를 사용하는 코드에서 흔히 볼 수 있다.

내장 기능의 프로토타입에도 속성을 추가할 수 있다. 하지만, 이상한 결과를 보일 때도 있다.

```typescript
> RegExp.prototype.monkey = 'Capuchin'
"Capuchin"
> /123/.monkey
"Capuchin"
```

정규식 /123/에 monkey라는 속성을 추가한 적이 없는데 "Capuchin"이라는 값이 들어있다.

⭐️ 객체에 임의의 속성을 추가하는 것은 좋은 설계가 아니다.

## 📍 예제

#### 🔗 window 또는 DOM 노드에 데이터를 추가한다고 가정해보자.

그 데이터는 기본적으로 전역변수가 되고, 전역 변수를 사용하면 프로그램 내에서 멀리 떨어진 부분들 간에 의존성을 만들게 된다. 그러면 함수를 호출할 때마다 부작용(side effect)를 고려해야만 한다.

타입스크립트로 나아가면, 타입체커는 Document와 HTMLElement의 내장 속성에 대해서는 알고 있지만, 임의로 추가한 속성에 대해서는 알지 못한다.

```typescript
document.monkey = 'Tamarin';
//       ~~~~~~ Property 'monkey' does not exist on type 'Document'
```

이 오류를 해결하는 가장 간단한 방법은 any 단언문을 사용하는 것이다.

```typescript
(document as any).monkey = 'Tamarin';  // OK
```

타입 체커는 통과하지만 any를 사용함으로써 타입 안정성을 잃고, 언어 서비스를 사용할 수 없게 되었다.

```typescript
(document as any).monky = 'Tamarin';  // Also OK, misspelled
(document as any).monkey = /Tamarin/;  // Also OK, wrong type
```

⭐️ 최선의 해결책은 document 또는 DOM으로부터 데이터를 분리하는 것이다. 분리할 수 없는 경우(라이브러리 사용 or 자바스크립트 마이그레이션 과정), 두 가지 차선책이 존재한다.

**✓ `interface`의 특수 기능 중 하나인 보강(augmentation)을 사용한다.**

```typescript
interface Document {
  /** 몽키 패치의 속(genus) 또는 종(species) */
  monkey: string;
}

document.monkey = 'Tamarin';  // 정상
```

**보강을 사용한 방법이 `any` 보다 나은 점 4가지**

* 타입이 더 안전하다.
* 속성에 주석을 붙일 수 있다.
* 속성에 자동완성을 사용할 수 있다.
* 몽키 패치가 어떤 부분에 적용되었는지 정확한 기록이 남는다.

모듈의 관점에서(타입스크립트 파일이 import, export 를 사용하는 경우) 제대로 동작하게 하려면 global 선언을 추가해야 한다.

```typescript
export {};
declare global {
  interface Document {
    monkey: string;
  }
}
document.monkey = 'Tamarin';  // 정상
```

보강을 사용할 때 주의해야 할 점은 모듈 영역(scope)과 관련이 있다. 보강은 전역적으로 적용되기 때문에, 코드의 다른 부분이나 라이브러리로부터 분리할 수 없다. 애플리케이션이 실행되는 동안 속성을 할당하면 실행 시점에서 보강을 적용할 방법이 없다.

(웹 페이지 내의 HTML 엘리먼트를 조작할 때, 어떤 엘리먼트는 속성이 있고 어떤 엘리먼트는 속성이 없는 경우 문제가 된다. 이러한 이유로 속성을 string|undefined로 선언할 수도 있다. → 선언을 정확히 할 수 있지만, 다루기에는 더 불편해진다.)

**✓ 더 구체적인 타입 단언문을 사용한다.**

```typescript
interface MonkeyDocument extends Document {
  monkey: string;
}
(document as MonkeyDocument).monkey = 'Macaque';
```

MonkeyDocument는 Document를 확장하기 때문에 타입 단언문은 정상이며 할당문의 타입은 안전하다. 또한 Document 타입을 건드리지 않고 별도로 확장하는 새로운 타입을 도입했기 때문에 모듈 영역 문제도 해결할 수 있다. (import하는 곳의 영역에서만 해당됨)

⭐️ 몽키 패치 속성을 참조하는 경우에만 단언문을 사용하거나 새로운 변수를 도입하면 된다.

❎ 몽키 패치를 남용해서는 안되며 더 잘 설계된 구조로 리팩터링 하는 것이 좋다.

## 📍 요약

* 전역 변수나 DOM에 데이터를 저장하지 말고, 데이터를 분리하여 사용해야 한다.
* 내장 타입에 데이터를 저장해야 하는 경우, 안전한 타입 접근법 중 하나(보강이나 사용자 정의 인터페이스로 단언)를 사용해야 한다.
* 보강의 모듈 영역 문제를 이해해야 한다.
