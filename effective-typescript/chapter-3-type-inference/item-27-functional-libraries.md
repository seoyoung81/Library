# 📎 아이템 27 함수형 기법과 라이브러리로 타입 흐름 유지하기

표준 라이브러리들의 일부 기능(map, flatMap, filter, reduce)은 순수 자바스크립트로 구현되어 있다. 이러한 기법은 루프를 대체할 수 있고, 타입스크립트와 조합하여 사용하면 더욱 빛을 발한다.&#x20;

타입 정보가 그대로 유지되면서 타입 흐름이 계속 전달되도록 하기 때문이다. 반면에 직접 루프를 직접 구현하면 타입 체크에 대한 관리도 직접해야 한다.

### 📍 CSV 데이터 파싱 예제

🔗 순수 자바스크리븥에서는 절차형(imperative) 프로그래밍 형태로 구현할 수 있다.

```typescript
const csvData = "...";
const rawRows = csvData.split('\n');
const headers = rawRows[0].split(',');

const rows = rawRows.slice(1).map((rowStr) => {
  const row = {};
  rowStr.split(",").forEach((val, j) => {
    row[headers[j]] = val;
    // {} 형식에서 'string' 형식의 매개변수가 포함된 인덱스 시그니처를 찾을 수 없다.
  });
  return row;
});
```

🔗 함수형 마인드를 가진 자바스크립트 개발자라면, reduce를 사용해서 행 객체를 만드는 방법을 선호할 수도 있다.&#x20;

```typescript
const rows = rawRows.slice(1)
  .map(rowStr => rowStr.split(',').reduce(
    (row, val, i) => (row[headers[i]] = val, row),
    // {} 형식에서 'string' 형식의 매개변수가 포함된 인덱스 시그니처를 찾을 수 없다.
    {}));
```

코드 줄을 줄여주지만, 보는 사람에 따라 복잡해 보일 수도 있다.&#x20;

🔗 로대시의 `zipObject` 함수를 이용하면 코드를 더욱 짧게 쓸 수 있다.

```typescript
import _ from 'lodash';
const rows = rawRows.slice(1)
    .map(rowStr => _.zipObject(headers, rowStr.split(',')));
```

코드는 매우 짧아졌지만, 자바스크립트에서는 프로젝트에 서드파티라이브러리 종속성을 추가할 때 신중해야한다. 코드를 짧게 줄이는데 시간이 많이 든다면 사용하지 않는게 낫다.

하지만, 같은 코드를 타입스크립트로 작성하면 서드파티 라이브러리 사용하는 것이 무조건 유리하다. 타입 정보를 참고하여 작업할 수 있기 때문에 시간이 단축된다.

🔗 로대시 버전

별도의 수정 없이도 타입 체커를 통과한다.

```typescript
const rows = rawRows.slice(1)
    .map(rowStr => _.zipObject(headers, rowStr.split(',')));
    // 타입이 _.Dictionary<string>[]
```

Dictionary는 로대시의 타입 별칭이다. `Dictionary<string>[]`은 `{[key: string]: string}` 또는 `Record<string, string>`과 동일하다. 여기서 중요한 점은 타입 구문이 없어도 rows의 타입이 정확하다는 것이다.

### 📍 NBA 팀의 선수 명단

데이터 가공이 정교해질수록 이러한 장점은 더욱 분명해진다.

```typescript
interface BasketballPlayer {
  name: string;
  team: string;
  salary: number;
}
declare const rosters: {[team: string]: BasketballPlayer[]};
```

🔗 루프를 사용해 단순 목록을 만들려면 배열에 concat을 사용해야 한다.

다음 코드는 동작하지만 타입 체크는 되지 않는다.

```typescript
let allPlayers = [];
//  ~~~~~~~~~~ Variable 'allPlayers' implicitly has type 'any[]'
//             in some locations where its type cannot be determined
for (const players of Object.values(rosters)) {
  allPlayers = allPlayers.concat(players);
  //           ~~~~~~~~~~ Variable 'allPlayers' implicitly has an 'any[]' type
}
```

🔗 오류를 고치기 위해 `allPlayers`에 타입 구문을 추가하거나

```typescript
let allPlayers: BasketballPlayer[] = [];
for (const players of Object.values(rosters)) {
  allPlayers = allPlayers.concat(players);  // OK
}
```

🔗 더 나은 해법으로 `Array.prototype.flat`을 사용하는 것이다.

```typescript
const allPlayers = Object.values(rosters).flat(); // OK
//    ^? const allPlayers: BasketballPlayer[]
```

flat 메서드는 다차원 배열을 평탄화해준다. 타입 시그니처는 T\[]\[] => T\[] 같은 형태이다. 이 버전이 가장 간결하고 타입 구문도 필요 없다.



내장된 함수형 기법들과 로대시 같은 라이브러리에 타입 정보가 잘 유지되는 것은 우연이 아니다. 함수 호출 시 전달된 매개변수 값을 건드리지 않고, 매번 새로운 값으로 반환함으로써, 새로운 타입으로 안전하게 반환할 수 있다.

타입스크립트의 많은 부분이 자바스크립트 라이브러리의 동작을 정확히 모델링하기 위해서 개발되었다. 그러므로 라이브러리를 사용할 때 타입 정보가 잘 유지되는 점을 활용해야 타입스트립트의 원래 목적을 달성할 수 있다.

## 📍요약

* 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해 직접 구현하기보다는 내장된 함수형 기법과 로대시 같은 유틸리티 라이브러리를 사용하는 것이 좋다.
