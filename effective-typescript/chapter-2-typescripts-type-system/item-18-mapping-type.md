# 📎 아이템 18 매핑된 타입을 사용하여 값을 동기화하기

## 📍 예제

산점도(scatter plot)를 그리기 위한 UI 컴포넌트를 작성한다고 가정하자. 디스플레이와 동작을 제어하기 위한 몇 가지 타입의 속성이 포함된다.

```typescript
interface ScatterProps {
  // The data
  xs: number[];
  ys: number[];
  
  // Display
  xRange: [number, number];
  yRange: [number, number];
  color: string;
  
  // Events
  onClick: (x: number, y: number, index: number) => void;
}
```

불필요한 작업을 피하기 위해, 필요할 때에만 차트를 다시 그릴 수 있다. 데이터나 디스플레이 속성이 변경되면 다시 그려야 하지만, 이벤트 핸들러가 변경되면 다시 그릴 필요가 없다.

→ 이런 종류의 최적화는 리액트 컴포넌트에서 일반적인데, 렌더링할 때마다 이벤트 핸들러 Prop이 새 화살표 함수로 설정된다.

최적화를 두 가지 방법으로 구현해보자.

### 1️⃣ 첫 번째 방법

```typescript
function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: kefof ScatterProps,
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== 'OnClick') return true;
    }
  }
  return false;
}
```

새로운 속성이 추가되면 `shouldUpdate` 함수는 값이 변경될 때마다 차트를 다시 그릴 것이다.

→ **보수적(conservative) 접근법 or 실패에 닫힌(fail close) 접근법**

차트가 정확하지만 너무 자주 그려질 가능성이 있다.

### 2️⃣ 두 번째 방법

**실패에 열린 접근법**

```typescript
function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
    // no check for onClick
  );
}
```

차트를 불필요하게 다시 그리는 단점을 해결했지만, 실제로 차트를 다시 그러야 할 경우에 누락되는 일이 생길 수 있다.

'우선 망치지 말 것(first, do no harm)'을 어기기 때문에 일반적인 경우에 쓰이는 방법이 아니다.

**☑️ 두 가지 모두 이상적이지 않다. 새로운 속성이 추가될 때 직접 shouldUpdate를 고치도록 하는게 낫다.**

### 3️⃣ 세 번째 방법

```typescript
interface ScatterProps {
  xs: number[];
  ys: number[];
  // ...
  onClick: (x: number, y: number, index: number) => void;
  
  // 참고: 여기에 속성을 추가하려면, shouldUpdate를 고쳐라.
```

이 방법도 최선이 아니며, 타입 체커가 대신할 수 있게 하는 것이 좋다.

### 4️⃣ 네 번째 방법

타입 체커가 동작하도록 개선한 코드 → <mark style="background-color:yellow;">**매핑된 타입과 객체를 사용하는 것**</mark>

```typescript
const REQUIRES_UPDATE: {[k in keyof ScatterProps]: boolean} = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};
function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: kefof ScatterProps,
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}

```

`[k in keyof ScatterProps]`은 타입 체커에게 `REQUIRES_UPDATE`가 `ScatterProps`과 동일한 속성을 가져야 한다는 정보를 제공한다. 나중에 `ScatterProps`에 새로운 속성을 추가하는 경우 다음 코드와 같은 형태가 된다.

```typescript
interface ScatterProps {
  // ...
  onDubleClick: () => void;
}
```

그리고 `REQUIRES_UPDATE` 의 정의에 오류가 발생한다.

```typescript
const REQUIRES_UPDATE: {[k in keyof ScatterProps]: boolean} = {
  // 'onDubleClick' 속성이 타입에 없습니다.
};
```

이런 방식은 오류를 정확히 잡아낸다. 속성을 삭제하거나 이름을 바꾸어도 비슷한 오류가 발생한다. 여기서 `boolean` 값을 사용했다는 점이 중요하다.&#x20;

배열을 사용했다면

```typescript
const REQUIRES_UPDATE: (keyof ScatterProps)[] = [
  'xs',
  'ys',
  // ...
];
```

최적화할 때 실패에 열린 방법 or 단힌 방법을 선택할 지 정해야한다.&#x20;

매핑된 타입은 한 객체가 또 다른 객체와 정확히 같은 속성을 가지게 할 때 이상적이다. 이 예제는 매핑된 타입을 사용해 타입스크립트가 코드에 제약을 강제하도록 할 수 있었다.

## 📍 요약

* 매핑된 타입을 사용해서 관련된 값과 타입을 동기화하도록 한다.
* 인터페이스에 새로운 속성을 추가할 때, 선택을 강제하도록 매핑된 타입을 고려해야 한다.
