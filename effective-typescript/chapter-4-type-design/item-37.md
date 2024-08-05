# 📎 아이템 37 공식 명칭에는 상표를 붙이기

구조적 타이핑의 특성 때문에 가끔 코드가 이상한 결과를 낼 수 있다.

## 📍 예제

### 🔗 벡터

이 코드는 구조적 타이핑 관점에서는 문제가 없지만, 수학적으로 따지면 2차원 벡터를 사용해야 이치에 맞다.

```typescript
// 상표를 사용하지 않은 경우
interface Vector2D{
    x: number;
    y: number;
}

function calculateNorm(p: Vector2D){
    return Math.sqrt(p.x * p.x + p.y * p.y);
}
calculateNorm({x: 3, y: 4}); // 결과는 5
const vec3D = {x: 3, y: 4, z: 1}; 
calculateNorm(vec3D); // 잘못된 입력이 들어갔지만, 결과는 5 

```

calculateNorm 함수가 3차원 벡터를 허용하지 않게 하려면 공식 명칭을 사용하면 된다. 공식 명칭을 사용하는 것: 타입이 아니라 값의 관점에서 Vector2D라고 말하는 것이다. → 상표(brand)를 붙이면 된다. (스타벅스가 아니라 커피)

<pre class="language-typescript"><code class="lang-typescript">//상표를 사용한 경우
<strong>interface Vector2D{
</strong>    _brand: '2d';
    x: number;
    y: number;
}

function vec2D(x: number, y: number): Vector2D{
    return {x, y, _brand:'2d'}
}

function calculateNorm(p: Vector2D){
    return Math.sqrt(p.x * p.x + p.y * p.y);    // 기존과 동일
}

calculateNorm(vec2D(3, 4); // 결과는 5
const vec3D = {x: 3, y: 4, z: 1}; 
calculateNorm(vec3D); // 에러: '_brand' 속성이 없음
</code></pre>

상표(\_brand)를 사용해서 calculateNorm 함수가 Vector2D 타입만 받는 것을 보장한다.

상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다. 타입 시스템이기 때문에 런타임 오버헤드를 없앨 수 있고 추가 속성을 붙일 수 없는 string이나 nuber 같은 내장 타입도 상표화할 수 있다.

### 🔗 number 타입에 상표 붙이기

단위 붙이기

```typescript
type Meters = number & {_brand: 'meters'};
type Seconds = number & {_brand: 'seconds'};

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000);
const oneMin = seconds(60);
```

number 타입에 상표를 붙여도 산술 연산 후에는 상표가 없어지기 때문에 실제로 사용하기에는 무리가 있다.

```typescript
const tenKm = oneKm * 10    // 타입이 number
const v = oneKm / oneMin; // 타입이 number
```

그러나 코드에 여러 단위가 혼합된 많은 수의 숫자가 들어 있는 경우, 숫자의 단위를 문서화하는 방법이 될 수 있다.

## 📍 요약

* 타입스크립트는 구조적 타이핑(덕 타이핑)을 사용하기 때문에 값을 세밀하게 구분하지 못하는 경우가 있다. 값을 구분하기 위해 공식 명칭이 필요하다면 상표를 붙이는 것을 고려해야한다.
* 상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다.
