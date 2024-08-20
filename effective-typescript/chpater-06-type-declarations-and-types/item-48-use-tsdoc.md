# 📎 아이템 48 API 주석에 TSDoc 사용하기

```javascript
// Generate a greeting. Result is formatted for display.
function greet(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

함수의 앞부분에 주석이 있어 함수가 어떤 기능을 하는지 쉽게 알 수 있다. 사용자를 위한 문서라면 JSDoc 스타일의 주석을 다는 것이 좋다.

```javascript
/** Generate a greeting. Result is formatted for display. */
function greetJSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

대부분의 편집기는 함수가 호출되는 곳에서 함수에 붙어 있는 JSDoc 스타일의 주석을 툴팁으로 표시해주기 때문이다.

<figure><img src="../../.gitbook/assets/image (10).png" alt="" width="375"><figcaption></figcaption></figure>

인라인 주석은 편집기가 표시해 주지 않는다.

타입스크립트 언어 서비스가 JSDoc 스타일을 지원하기 때문에 적극적으로 활용하는 것이 좋다. 공개 API 주석을 붙인다면 JSDoc 형태로 작성해야 한다. (타입스크립트 관점에서는 TSDoc 라고 불린다.)

```typescript
/**
 * 인사말을 생성한다.
 * @param name 인사할 사람의 이름
 * @param title 그 사람의 칭호
 * @returns 사람이 보기 좋은 형태의 인사말
 */
function greetFullTSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

`@param` 과 `@returns` 를 추가하면 함수를 호출하는 부분에서 각 매개변수와 관련된 설명을 보여준다.

<figure><img src="../../.gitbook/assets/image (11).png" alt="" width="375"><figcaption></figcaption></figure>

타입 정의에 TSDoc을 사용할 수도 있다.

```typescript
/** A measurement performed at a time and place. */
interface Measurement {
  /** Where was the measurement made? */
  position: Vector3D;
  /** When was the measurement made? In seconds since epoch. */
  time: number;
  /** Observed momentum */
  momentum: Vector3D;
}
```

`Measurement` 객체의 각 필드에 마우스를 올려보면 필드별로 자세한 설명을 알 수 있다.

<figure><img src="../../.gitbook/assets/image (12).png" alt="" width="375"><figcaption></figcaption></figure>

TSDoc 주석은 마크다운 형식으로 꾸며지므로, 굵은 글씨, 기울임 글씨, 글머리기호 목록을 사용할 수도 있다.

```typescript
/**
  * 이 _interface_는 **세 가지** 속성을 가진다.
  * 1. x
  * 2. y
  * 3. z
  */
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
```

주석을 간단히 요점만 쓰도록 하자.

타입스크립트는 타입 정보가 코드에 있기 때문에 TSDoc에서는 타입 정보를 명시하면 안된다.

## 📍 요약

* 익스포트된 함수, 클래스, 타입에 주석을 달 때는 JSDoc/TSDoc 형태를 사용하자. JSDoc/TSDoc 형태의 주석을 달면 편집기가 주석 정보를 표시해 준다.
* @param, @returns 구문과 문서 서식을 위해 마크다운을 사용할 수 있다.
* 주석에 타입 정보를 포함하면 안된다.
