# 📎 아이템 24 일관성 있는 별칭 사용하기

**🔗 어떤 값에 새 이름을 할당하기**

```typescript
const place = {name: 'New York', latLng: [41.6868, -74.2692]};
const loc = place.latLng;
```

`place.latLng` 배열에 `loc` 라는 별칭(`alias`)을 만들었다. 별칭의 값을 변경하면 원래 속성값에서도 변경된다.

```typescript
> loc[0] = 0
> place.latLng
[0, -74.2692]
```

하지만, 별칭을 남발하면 제어흐름을 분석하기 어렵다. 별칭을 신중하게 사용해야 코드를 잘 이해하고, 오류도 쉽게 찾을 수 있다.

**🔗 다각형을 표현하는 자료구조**

```typescript
interface Coordinate {
  x: number;
  y: number;
}

interface BoundingBox {
  x: [number, number];
  y: [number, number];
}

interface Polygon {
  exterior: Coordinate[];  // 다각형은 exterior과 holes 속성으로 정의됨
  holes: Coordinate[][];
  bbox?: BoundingBox;
}
```

`bbox` 속성(최적화 속성)을 사용하면 어떤 점이 다각형에 포함되는지 빠르게 체크할 수 있다.

```typescript
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  if (polygon.bbox) {
    if (pt.x < polygon.bbox.x[0] || pt.x > polygon.bbox.x[1] ||
        pt.y < polygon.bbox.y[0] || pt.y > polygon.bbox.y[1]) {
      return false;
    }
  }

  // ... more complex check
}
```

이 코드는 잘 동작하고, 타입체크도 통과하지만 반복되는 부분이 존재한다. `polygon.bbox`는 5번이나 등작한다. 중복을 줄이기 위해 임시변수를 뽑아보자.

```typescript
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if (polygon.bbox) {
    if (pt.x < box.x[0] || pt.x > box.x[1] ||
        //     ~~~                ~~~  'box' is possibly 'undefined'
        pt.y < box.y[0] || pt.y > box.y[1]) {
        //     ~~~                ~~~  'box' is possibly 'undefined'
      return false;
    }
  }
  // ...
}
```

편집기에서 오류로 표시되는데, `box` 별칭이 첫번째 예제에서는 잘 동작했던 제어 흐름 분석을 방해한다.

`box`와 `polygon.bbox`의 타입을 조사해보자.

```typescript
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  polygon.bbox
  //      ^? (property) Polygon.bbox?: BoundingBox | undefined
  const box = polygon.bbox;
  //    ^? const box: BoundingBox | undefined
  if (polygon.bbox) {
    console.log(polygon.bbox);
    //                  ^? (property) Polygon.bbox?: BoundingBox
    console.log(box);
    //          ^? const box: BoundingBox | undefined
  }
}
```

속성체크는 polygon.bbox의 타입을 정제했지만 box는 그렇지 않았기 때문에 오류가 발생한다. 이러한 오류는 '별칭은 일관성 있게 사용한다'는 기본 원칙(golden rule)을 지키면 방지할 수 있다.

→ 속성 체크에 box를 사용하도록 코드를 바꿔보자.

```typescript
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if (box) {
    if (pt.x < box.x[0] || pt.x > box.x[1] ||
        pt.y < box.y[0] || pt.y > box.y[1]) {  // OK
      return false;
    }
  }
  // ...
}
```

타입 체커의 문제는 해결되었지만, 코드를 읽는 사람에게는 문제가 남아있다. box와 bbox는 같은 값인데 다른 이름을 사용한 것이다. (아이템 36)

객체 비구조화를 이용하면 보다 간결한 문법으로 일관된 이름으로 사용할 수 있다. 배열과 중첩된 구조에서도 사용할 수 있다.&#x20;

```typescript
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const {bbox} = polygon;
  if (bbox) {
    const {x, y} = bbox;
    if (pt.x < x[0] || pt.x > x[1] || pt.y < y[0] || pt.y > y[1]) {
      return false;
    }
  }
  // ...
}
```

객체 비구조화를 이용할 때는 두 가지를 주의해야한다.

1. 전체 bbox 속성이 아니라 x와 y가 선택적 속성일 경우에 속성 체크가 더 필요하다. 따라서 타입의 경계에 null 값을 추가하는 것이 좋다.
2. bbox에는 선택적 속성이 적합했지만 holes는 그렇지 않다. holes가 선택적이라면, 값이 없거나 빈 배열이었을 것이다. 차이가 없는데 이름을 구별한 것이다. 빈 배열은 'holes 없음'을 나타내는 좋은 방법이다.

**🔗 별칭은 타입 체커뿐만 아니라 런타임에도 혼동을 야기할 수 있다.**

```typescript
const {bbox} = polygon;
if (!bbox) {
  calculatePolygonBbox(polygon);  // Fills in polygon.bbox
  // Now polygon.bbox and bbox refer to different values!
}
```

타입스크립트의 제어 흐름 분석은 지역 변수에는 꽤 잘 동작한다. 그러나 객체 속성에서는 주의해야 한다.

```typescript
function expandABit(p: Polygon) { /* ... */ }

polygon.bbox
//      ^? (property) Polygon.bbox?: BoundingBox | undefined
if (polygon.bbox) {
  polygon.bbox
  //      ^? (property) Polygon.bbox?: BoundingBox
  expandABit(polygon);
  polygon.bbox
  //      ^? (property) Polygon.bbox?: BoundingBox
}
```

`fn(polygon)` 호출은 `polygon.bbox`를 제거할 가능성이 있으므로 타입을 `BoundingBox | undefined`로 되돌리는 것이 안전할 것이다. 그러나 함수를 호출할 때마다 속성 체크를 반복해야 하기 때문에 좋지 않다. 그래서 타입스크립트는 함수가 타입 정제를 무효화하지 않는다고 가정한다. 그러나, 실제로는 무효화될 가능성이 있다. `polygon.bbox로 사용하는 대신 bbox` 지역변수로 뽑아내서 사용하면 `bbox`의 타입은 정확히 유지되지만, `polygon.bbox`의 값과 같게 유지되지 않을 수 있다.

## 📍 요약

* 별칭은 타입스크립트가 타입을 좁히는 것을 방해한다. 따라서 변수에 별칭을 사용할 때는 일관되게 사용해야 한다.
* 비구조화 문법을 사용해서 일관된 이름을 사용하는 것이 좋다.
* 함수 호출이 객체 속성의 타입 정제를 무효화할 수 있다는 점을 주의해야 한다. 속성보다 지역 변수를 사용하면 타입 정제를 믿을 수 있다.
