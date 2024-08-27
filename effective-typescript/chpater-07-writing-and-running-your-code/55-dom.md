# 📎 아이템 55 DOM 계층 구조 이해하기

브라우저에서 동작하는 프로젝트 다루기

DOM 계층은 웹브라우저에서 자바스크립트를 실행할 때 어디에서나 존재한다.&#x20;

타입스크립트에서는 DOM 엘리먼트의 계층 구조를 파악하기 쉽다. Element와 ElementTarget에 달려있는 Node의 구체적인 타입을 안다면 타입 오류를 디버깅할 수 있고, 언제 타입 단언을 사용해야 할 지 알 수 있다. 브라우저 API가 대부분 DOM을 기반으로 하기 때문에, 리액트나 d3 같은 프레임워크도 DOM이 관련되어 있다.

## 📍 DOM의 계층 구조

```markup
<p id="quote">and <i>yet</i> it moves</p>
```

p 엘리먼트의 참조 얻어보면, `HTMLParagraphElement` 타입인 것을 알 수 있다.

```typescript
const p = document.getElementsByTagName('p')[0];
//    ^? const p: HTMLParagraphElement
```

`HTMLParagraphElement`는 `HTMLElement`의 서브타입이고, `HTMLElement`는 `Element`의 서브타입이다. `Element`는 `Node`의 서브타입이고, `Node`는 `EventTarget`의 서브타입이다.

### EventTarget

DOM 타입 중 가장 추상화된 타입. 예시로 `window`, `XMLHttpRequest`가 있다. 이벤트 리스너를 추가하거나 제거하고, 이벤트를 보내는 것 밖에 할 수 없다.

```typescript
function handleDrag(eDown: Event) {
  const targetEl = eDown.currentTarget;
  targetEl.classList.add('dragging');
  // ~~~~~           'targetEl' is possibly 'null'
  //       ~~~~~~~~~ Property 'classList' does not exist on type 'EventTarget'
```

`Event`의 `currentTarget` 속성의 타입은 `EventTarget | null` 이다. 그래서 `null` 가능성이 오류로 표시되었고, `EventTarget` 타입에 `classList` 속성이 없기 때문에 오류가 표시되었다.

`eDown.currentTarget`은 실제로 `HTMLElement` 타입이지만, 타입 관점에서는 `window`나 `XMLHttpRequest`가 될 수도 있다.

### Node

Element가 아닌 Node인 경우 → 텍스트 조각과 주석 (document, Text, Comment)

```html
<p>
  and <i>yet</i> it moves
  <!-- quote from Galileo -->
</p>
```

가장 바깥 엘리먼트는 HTMLParagraphElement이고, childre과 childNodes 속성을 가지고 있다.

```typescript
> p.children    
HTMLCollection [i]
> p.childNodes
NodeList(5) [text, i, text, comment, text]
```

children은 자식 엘리먼트를 포함하는 배열과 유사한 구조인 HTMLCollection

childNodes는 배열과 유사한 Node의 컬렉션인 NodeList, 텍스트 조각과 주석도 포함

### Element와 HTMLElement

SVG 태그의 전체 계층 구조를 포함하면서 HTML이 아닌 엘리먼트가 Element의 또다른 종류인 SVGElement이다. 예를 들어, \<html>은 HTMLElement이고, \<svg>는 SVGElement이다.

### HTMLxxxElement

형태의 특정 엘리먼트들은 고유한 속성이 있다.

HTMLImageElement에는 src 속성, HTMLInputElement에는 value 속성

이런 속서에 접근하려면, 타입 정보도 실제 엘리먼트 타입이어야 하므로 구체적으로 타입을 지정해야한다.



⭐️ 일반적으로 타입 단언문은 지양해야 하지만, DOM 관련해서는 타입스크립트보다 우리가 더 정확히 알고 있는 경우이므로 단언문을 사용해도 좋다. 예를 들어, #my-div가 div 태그인 것을 알고 있으므로 문제가 안된다.

```typescript
document.getElementById('my-id) as HTMLDivElement;
```

`strictNullChecks`가 설정된 상태라면, `document.getElementById`가 `null`인 경우를 체크해야 한다. 실제로 null일 가능성이 있다면 if 분기문을 추가해야 한다.

#### Event 타입의 계층 구조

* UIEvent: 모든 종류의 사용자 인터페이스
* MouseEvent: 클릭처럼 마우스로부터 발생되는 이벤트
* TouchEvent: 모바일 기기의 터치 이벤트
* WheelEvent: 스크롤 휠 이벤트
* KeyboardEvent: 키보드 누름 이벤트

```typescript
function handleDrag(eDown: Event) {
  // ...
  const dragStart = [
     eDown.clientX, eDown.clientY
     //    ~~~~~~~        ~~~~~~~ Property '...' does not exist on 'Event'
  ];
  // ...
}
```

eUp이 `Event`로 선언되었지만, clientX와 clientY는 보다 구체적인 `MouseEvent` 타입이 있다. DOM에 대한 타입 추론은 문맥 정보를 폭넓게 활용해야 한다. 'mousedown' 이벤트 핸들러를 인라인 함수로 만들어 타입스크립트가 더 많은 문맥 정보를 사용하게 하고, 오류를 제거할 수 있다.

<pre class="language-typescript"><code class="lang-typescript">function addDragHandler(el: HTMLElement) {  // HTMLElement
<strong>  // mousedown  
</strong>  el.addEventListener('mousedown', eDown => {
    const dragStart = [eDown.clientX, eDown.clientY];
    const handleUp = (eUp: MouseEvent) => {  // MouseEvent
      el.classList.remove('dragging');
      el.removeEventListener('mouseup', handleUp);
      const dragEnd = [eUp.clientX, eUp.clientY];
      console.log('dx, dy = ', [0, 1].map(i => dragEnd[i] - dragStart[i]));
    }
    el.addEventListener('mouseup', handleUp);
  });
}

const surfaceEl = document.getElementById('surface');
if (surfaceEl) {
  addDragHandler(surfaceEl);
}
</code></pre>

surfaceEl 부분은 #surface 엘리먼트가 없는 경우를 체크한다. 만약 해당 엘리먼트가 반드시 존재한다는 것을 알고 있다면, if 구문 대신 단언문을 사용할 수도 있다. `addDragHanler(div!)`

## 📍 요약

* 자바스크립트를 사용할 때는 신경 쓰지 않았겠지만, DOM에는 타입 계층 구조가 있다. DOM 타입은 타입스크립트에서 중요한 정보이며, 브라우저 관련 프로젝트에서 타입스크립트를 사용할 때 유용하다.
* Node, Element, HTMLElement, EventTarget 간의 차이점, Event와 MouseEvent의 차이점을 알아야한다.
* DOM 엘리먼트와 이벤트에는 충분히 구체적인 타입 정보를 사용하거나, 타입스크립트가 추론할 수 있도록 문맥 정보를 활용해야 한다.
