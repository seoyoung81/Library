# 📎 아이템 30 문서에 타입 정보를 쓰지 않기

```typescript
/**
 * Returns a string with the foreground color.
 * Takes zero or one arguments. With no arguments, returns the
 * standard foreground color. With one argument, returns the foreground color
 * for a particular page.
 */
function getForegroundColor(page?: string) {
  return page === 'login' ? {r: 127, g: 127, b: 127} : {r: 0, g: 0, b: 0};
}
```

이 코드는 코드와 주석의 정보가 맞지 않다. 둘 중 어느 것이 옳은 것인지 판단하기에 정보가 부족하다. 앞의 예제에서 의도된 동작이 코드에 제대로 반영되고 있다고 가정하면, 주석에는 3가지 문제점이 있다. ❎

1. 함수가 string 형태의 색깔을 반환한다고 적혀있지만 실제로는 {r, g, b} 객체를 반환한다. (코드 작성자가 string에서 추후 객체를 변환하게 코드를 바꾸고 주석 수정을 하지 않았다.)
2. 주석에는 함수가 0개 또는 1개의 매개변수를 받는다고 설명하지만, 타입 시그니처만 보아도 명확하게 알 수 있는 정보이다.
3. 불필요하게 장황하다. 함수 선언과 구현체보다 주석이 더 길다.

타입스크립트의 타입 구문 시스템은 간결하고, 구체적이며, 쉽게 읽을 수 있도록 설계되었다. 함수의 입력과 출력의 타입을 코드로 표현하는 것이 주석보다 더 낫다.

코드와 주석은 동기화 되지 않는다. ↔️ 타입 체커는 타입 정보를 동기화하도록 강제한다.

**🎈 주석은 다음과 같이 개선할 수 있다.**

```typescript
/** Get the foreground color for the application or a specific page. */
function getForegroundColor(page?: string): Color {
  // ...
}
```

⭐️ 특정 매개변수를 설명하고 싶다면 JSDoc의 @param 구문을 사용하면 된다. (아이템 48)

❎ 값을 변경하지 않는다고 설명하는 주석도 좋지 않다. 또한 매개변수를 변경하지 않는다는 주석도 사용하지 않는 것이 좋다.

```typescript
// ** nums를 변경하지 않는다. */
function sort(nums: number[])
```

⭐️ 그 대신 readonly로 선언하여 타입스크립트가 규칙을 강제할 수 있게 하면 된다.

```typescript
function sort(nums: readonly number[])
```

⚒️ 주석에 적용한 규칙은 변수명에도 그대로 적용할 수 있다. 변수명에 타입 정보를 넣지 않도록 한다. 변수명을 ageNum으로 하는 것보다 age로 하고, 그 타입이 number임을 명시하는게 좋다.

그러나 단위가 있는 숫자들은 단위가 무엇인지 변수명 or 속성명에 포함시키는 것이 좋다. timeMs는 time보다 명확하고 temperatureC는 temperature보다 명확하다.

## 📍 요약

* 주석과 변수명에 타입 정보를 적는 것은 피하자. 타입 선언이 중복되는 것으로 끝난다면 다행이지만 최악의 경우는 타입 정보에 모순이 발생하게 된다.
* 타입이 명확하지 않은 경우는 변수명에 단위 정보를 포함하는 것을 고려하는 것이 좋다.
