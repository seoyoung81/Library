# 📎 아이템 31 타입 주변에 null 값 배치하기

## 📍 null 값 배치의 필요성

strictNullChecks 설정을 처음 키면, null이나 undefined 값 관련된 오류들이 갑자기 나타나기 때문에, 오류를 걸러내는 if 구문을 코드 전체에 추가해야 한다고 생각할 수 있다. (어떤 변수가 null이 될 수 있는지 없는지 타입만으로는 명확히 알기 어렵다.)

예를 들어, B 변수가 A 변수의 값으로부터 비롯되는 값이라면, A가 null이 될 수 없을 때 B 역시 null이 될 수 없고, 그 반대도 마찬가지이다.

→ 이러한 관계들은 겉으로 드러나지 않기 때문에 사람과 타입체커 모두에게 혼란스럽다.

⭐️ 값이 전부 null이거나 전부 null이 아닌 경우로 분명히 구분된다면, 값이 섞여 있을 때보다 다루기 쉽다. 타입에 null을 추가하는 방식으로 이러한 경우를 모델링 할 수 있다.

## 📍예제

### 🔗 숫자들의 최솟값과 최댓값을 계산하는 extent 함수

```typescript
// @strictNullChecks: false
function extent(nums: Iterable<number>) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
    }
  }
  return [min, max];
}
```

이 코드는  없이 타입 체커를 통과하고, 반환 타입은 number\[]로 추론된다. 그러나 여기에는 버그와 함께 설계적 결함이 있다.

1. 최솟값이나 최댓값이 0인 경우, 값이 덧씌워진다.  extent(\[0, 1, 2])의 결과는 \[0, 2] 가 아니라 \[1, 2]가 된다.
2. nums 배열이 비어 있다면 함수는 \[undefined, undefined]를 반환한다. (undefined를 포함하는 객체는 다루기 어려워 절대 권장하지 않는다.)

☑️ `strictNullChecks`을 키면 두 가지 문제점이 드러난다.

```typescript
function extent(nums: Iterable<number>) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
      //             ~~~ Argument of type 'number | undefined' is not
      //                 assignable to parameter of type 'number'
    }
  }
  return [min, max];
}
```

extent의 타입이 (number | undefined)\[]로 추론되어서 설계적 결함이 분명해졌다. 이제는 extent를 호출하는 곳마다 타입 오류의 형태로 나타난다.

```typescript
const [min, max] = extent([0, 1, 2]);
const span = max - min;
//           ~~~   ~~~ Object is possibly 'undefined'
```

⭐️ min과 max를 한 객체 안에 넣고 null이거나 null이 아니게 하면 된다.

```typescript
function extent(nums: Iterable<number>) {
  let minMax: [number, number] | null = null;
  for (const num of nums) {
    if (!minMax) {
      minMax = [num, num];
    } else {
      const [oldMin, oldMax] = minMax;
      minMax = [Math.min(num, oldMin), Math.max(num, oldMax)];
    }
  }
  return minMax;
}
```

이제는 반환 타입이 \[number, number] | null이 되어 사용하기 더 수월해졌다. null이 아님 단언(!)을 사용하면 min과 max를 얻을 수 있다.

```typescript
const [min, max] = extent([0, 1, 2])!;
const span = max - min;  // OK
```

null 아님 단언 대신 단순 if 구문으로 체크할 수도 있다.

```typescript
const range = extent([0, 1, 2]);
if (range) {
  const [min, max] = range;
  const span = max - min;  // OK
}
```

✏️ extent의 결괏값으로 단일 객체를 사용함으로써 설계를 개선하고, 타입스크립트가 null 값 사이의 관계를 이해할 수 있도록 하여 버그를 제거했다. if (!result) 체크도 제대로 동작한다.

null과 null이 아닌 값을 섞어서 사용하면 클래스에서도 문제가 생긴다.

## 📍요약

* 한 값의 null 여부가 다른 값의 null 여부에 암시적으로 관련되도록 설계하면 안된다.
* API 작성 시에는 반환 타입을 큰 객체로 만들고 반환 타입 객체가 null이거나 null이 아니게 만들어야 한다. 사람과 타입 체커 모두에게 명료한 코드가 될 것이다.
* 클래스를 만들 때는 필요한 모든 값이 준비되었을 때 생성하여 null이 존재하지 않도록 하는 것이 좋다.
* strictNullChecks를 설정하면 코드에 많은 오류가 표시되겠지만, null 값과 관련된 문제점을 찾아낼 수 있기 때문에 반드시 필요하다.
