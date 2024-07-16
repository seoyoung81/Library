# 📎 아이템 4 구조적 타이핑에 익숙해지기

자바스크립트는 본질적으로 덕 타이핑 기반이다.

덕 타이핑(duck typing)이란❓

> 객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주하는 방식이다. \
> "만약 어떤 새가 오리처럼 걷고, 헤엄치고, 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를 것이다."

만약 어떤 함수의 매개변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지 신경 쓰지 않고 사용한다. 타입스크립트는 이런 동작, 즉 매개변수 값이 요구사항을 만족한다면 타입이 무엇인지 신경 쓰지 않는 동작을 그대로 모델링 한다. 하지만, 타입 체커의 타입에 대한 이해도의 차이로 예상치 못한 결과가 나올 수 있다.

## 📍 구조적 타이핑

물리 라이브러리와 2D 벡터 타입을 다루는 경우를 가정해보자.

<pre class="language-typescript"><code class="lang-typescript">interface Vector2D {
  x: number;
  y: number;
}

<strong>interface NamedVector {
</strong>  name: string;
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x ** 2 + v.y ** 2);
}

// NameVector는 number 타입의 x와 y 속성이 있기 때문에 calculateLength 함수로 호출 가능하다.
const v: NamedVector = { x: 3, y: 4, name: 'Pythagoras' };
calculateLength(v);  // OK, result is 5
</code></pre>

`Vector2D`와 `NamedVector`는 관계를 전혀 선언하지 않았다. `NamedVector`를 위한 `calculateLength`를 구현할 필요도 없다. 타입스크립트 타입 시스템은 자바스크립트 런타임 동작을 모델링한다.&#x20;

→ `NamedVector` 구조가 `Vector2D`와 호환되기 때문에 `calculateLength` 호출이 가능하다. 여기서 <mark style="background-color:blue;">**구조적 타이핑**</mark>이라는 용어가 사용된다.

### 구조적 타이핑의 단점

구조적 타이핑 때문에 문제가 발생하기도 한다.

#### ⭐️ 예시1

3D 벡터를 만들어보자.

```typescript
interface Vector3D {
  x: number;
  y: number;
  z: number;
}

// 벡터의 길이를 1로 만드는 정규화 함수
function normalize(v: Vector3D) {
  const length = calculateLength(v);
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}
```

하지만 이 함수는 1.41을 출력한다.

```
> normalize({x: 3, y: 4, z: 5 })
{ x: 0.6, y:0.8 , z: 1 }
```

`calculateLength`는 2D 벡터를 기반으로 연산하는데, 버그로 인해 `normalize`가 3D 벡터로 연산되어 z가 정규화에서 무시된 것이다. 타입 체커가 이 문제를 잡아내지 못하였다. &#x20;

왜 일까❓

`Vector3D`와 호환되는 `{x, y, z}` 객체로 `calculateLength`를 호출하면, 구조적 타이핑 관점에서 x와 y가 있어 `Vector2D`와 호환된다. 따라서 오류가 발생하지 않고, 타입 체커가 문제로 인식하지 않는다.

함수를 작성할 때, 호출에 사용되는 매개변수의 속성들이 매개변수의 타입에 선언된 속성만을 가질거라 생각하기 쉽다. 이런 타입은 '봉인된', '정확한' 타입이라 불리며, 타입스크립트 타입 시스템에서는 표현할 수 없다.

```typescript
function calculateLengthL1(v: Vector3D) {
  let length = 0;
  for (const axis of Object.keys(v)) {
    const coord = v[axis];
    //            ~~~~~~~ Element implicitly has an 'any' type because ...
    //                    'string' can't be used to index type 'Vector3D'
    length += Math.abs(coord);
  }
  return length;
}
```

`coord`의 타입이 `number`라고 예상되지만 `Vector3D`는 봉인(다른 속성이 없다)되었다고 가정했기 때문에 타입스크립트의 오류가 맞다.

```typescript
const vec3D = {x: 3, y: 4, z: 1, address: '123 Broadway'};
calculateLengthL1(vec3D);  // OK, returns NaN
```

v는 어떤 속성이든 가질 수 있기 때문에 정확한 타입으로 객체를 순회하는 것은 까다로운 문제이다.

```typescript
function calculateLengthL1(v: Vector3D) {
  return Math.abs(v.x) + Math.abs(v.y) + Math.abs(v.z);
}
```

루프보다는 모든 속성을 각각 더하는 구현으로 해결하는 것이 더 낫다.

#### ⭐️ 예시 2

구조적 타이핑은 클래스와 관련된 할당문에서도 문제점이 있다.

<pre class="language-typescript"><code class="lang-typescript">class C {
  foo: string;
  constructor(foo: string) {
    this.foo = foo;
  }
}

<strong>const c = new C('instance of C');
</strong>const d: C = { foo: 'object literal'}; // 정상!
</code></pre>

`d`가 `C` 타입에 할당되는 이유를 알아보자.

`d`는 `string` 타입의 `foo` 속성을 가진다. 하나의 매개변수로 호출되는 생성자를 가져 구조적으로는 필요한 속성과 생성자가 존재하기 때문에 문제가 없다. 만약 `c`의 생성자에 단순 할당이 아닌 연산 로직이 존재하면, `d`의 경우는 생성자를 실행하지 않으므로 문제가 발생한다.&#x20;

### 구조적 타이핑의 장점

#### ⭐️ 예시3 - 구조적 타이핑이 유리한 경우

테스트를 작성할 때는 구조적 타이핑이 유리하다.

데이터베이스에 쿼리하고 결과를 처리하는 함수를 가정해보자.

```typescript
interface Author {
  first: string;
  last: string;
}
function getAuthors(database: PostgresDB): Author[] {
  const authorRows = database.runQuery(`SELECT first, last FROM authors`);
  return authorRows.map(row => ({first: row[0], last: row[1]}));
}
```

`getAuthors` 함수를 테스트하기 위해서는 모킹한 `PostgresDB`를 생성해야 한다. 그러나 구조적 타이핑을 활용하여 구체적인 인터페이스를 정의하는 것이 더 나은 방법이다.

```typescript
interface DB {
  runQuery: (sql: string) => any[];
}
function getAuthors(database: DB): Author[] {
  const authorRows = database.runQuery(`SELECT first, last FROM authors`);
  return authorRows.map(row => ({first: row[0], last: row[1]}));
}
```

`runQuery` 메서드로 실제 환경에서도 `getAuthors`에 `PostgresDB`를 사용할 수 있다.&#x20;

→ 구조적 타이핑으로 `PostgresDB`가 `DB` 인터페이스를 구현하는지 명확히 선언할 필요가 없다.

테스트를 작성할 때, 더 간단한 객체를 매개변수로 사용할 수도 있다.

```typescript
test('getAuthors', () => {
  const authors = getAuthors({
    runQuery(sql: string) {
      return [['Toni', 'Morrison'], ['Maya', 'Angelou']];
    }
  });
  expect(authors).toEqual([
    {first: 'Toni', last: 'Morrison'},
    {first: 'Maya', last: 'Angelou'}
  ]);
});
```

타입스크립트는 테스트 DB가 해당 인터페이스를 충족하는지 확인한다. 테스트 코드에는 실제 환경의 데이터베이스에 대한 정보가 불필요하고, 모킹 라이브러리도 필요 없다. 추상화를 함으로써, 로직과 테스트를 특정한 구현(PostgresDB)으로부터 분리한 것이다.

테스트 이외에 구조적 타이핑의 장점은 라이브러리 간의 의존성을 완벽히 분리할 수 있다는 것이다.

## 📍 요약

* 자바스크립트가 덕 타이핑 기반이고, 타입스크립트가 이를 모델링하기 위해 구조적 타이핑을 사용한다. 어떤 인터페이스에 할당 가능한 값이라면 타입 선언에 명시적으로 나열된 속성들을 가지고 있을 것이다. 타입은 봉인되어 있지 않다.
* 클래스 역시 구조적 타이핑 규칙을 따른다. 클래스의 인스턴스가 예상과 다를 수 있다.
* 구조적 타이핑을 사용하면 유닛 테스팅을 손쉽게 할 수 있다.
