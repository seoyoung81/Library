# 📎 아이템 33 string 타입보다 더 구체적인 타입 사용하기

string 타입의 범위는 매우 넓다. string 타입으로 변수를 선언할 때, 그보다 더 좁은 타입이 있는지 생각해보자.

## 📍 예시

### 🔗 음악 컬렉션을 위한 앨범 타입 정의

```typescript
interface Album {
  artist: string;
  title: string;
  releaseDate: string;  // YYYY-MM-DD
  recordingType: string;  // E.g., "live" or "studio"
}
```

❓ `string` 타입이 남발되었다. 주석에 타입 정보를 적어둔 걸 보면 현재 인터페이스가 잘못되었다는 것을 알 수 있다.

→ Album 타입에 엉뚱한 값을 설정할 수 있다.

```typescript
const kindOfBlue: Album = {
  artist: 'Miles Davis',
  title: 'Kind of Blue',
  releaseDate: 'August 17th, 1959',  // Oops!  // 날짜 형식 다름
  recordingType: 'Studio',  // Oops!  // live or studio
};  // OK
```

잘못된 값이 설정되었지만 두 값 모두 문자열이므로, `Album` 타입에 할당 가능하며 타입 체커를 통과한다.&#x20;

❓ `string` 타입의 범위가 넓어 `Album` 객체를 사용하더라도 매개변수 순서가 잘못된 것이 오류로 드러나지 않는다.

```typescript
function recordRelease(title: string, date: string) { /* ... */ }
recordRelease(kindOfBlue.releaseDate, kindOfBlue.title);  // OK, should be error
```

recordRelease의 매개변수 순서가 바뀌었지만, 둘 다 string이므로 타입 체커가 정상으로 인식한다.

**⭐️ 타입의 범위를 좁히자.**&#x20;

✓ artist나 title같은 필드는 string이 적절하다.

✓ releaseDate 필드는 Date 객체를 사용해서 날짜 형식으로만 제한하자.

✓ recordingType 필드는 "live" or "studio", 두 개의 값으로 유니온 타입을 정의하자. (enum은 추천X)

```typescript
type RecordingType = 'studio' | 'live';

interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}
```

타입스크립트가 오류를 더 세밀하게 체크한다.

```typescript
const kindOfBlue: Album = {
  artist: 'Miles Davis',
  title: 'Kind of Blue',
  releaseDate: new Date('1959-08-17'),
  recordingType: 'Studio'
// ~~~~~~~~~~~~ Type '"Studio"' is not assignable to type 'RecordingType'
};
```

#### ⭐️ 이러한 방식의 세 가지 장점

✓ 타입을 명시적으로 정의하여 다른 곳으로 값이 전달되더라도 타입 정보가 유지된다.

```typescript
// 특정 레코드 타입의 앨범을 찾는 함수
function getAlbumsOfType(recordingType: string): Album[] {
  // ...
}
```

recordingType의 값이 string 타입인 것 외에 정보가 없다.

✓ 타입을 명시적으로 정의하고, 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있다.

```typescript
/** What type of environment was this recording made in? */
type RecordingType = 'live' | 'studio';
```

매개변수를 string 대신 Recording 타입으로 바꾸면, 함수를 사용하는 곳에서 RecordingType의 설명을 볼 수 있다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

✓ keyof 연산자로 더욱 세밀하게 객체의 속성 체크가 가능하다.

함수의 매개변수에 string을 잘못 사용하는 일은 흔하다. 어떤 배열에서 한 필드의 값만 추출하는 함수를 작성해보자.

```typescript
// 언더스코어 라이브러리 pluck 함수
function pluck(records, key) {
  return records.map(r => r[key]);
}

// pluck 함수의 시그니처
function pluck(records: any[], key: string): any[] {
  return records.map(r => r[key]);
}
```

❓ 타입 체크가 되지만 any 타입으로 인해 정밀함이 떨어진다. 특히, 반환값에 any를 사용하는 것은 좋지 않다.

⭐️ 타입 시그니처 개선을 위해 제너릭 타입 도입

```typescript
function pluck<T>(records: T[], key: string): any[] {
  return records.map(r => r[key]);
  //                      ~~~~~~ Element implicitly has an 'any' type
  //                             because type '{}' has no index signature
}
```

❓ key의 값이 string이므로 범위가 너무 넓다는 오류 발생

key는 단 네 개의 값("artist", "title", releaseData", "recordingType")만이 유효하다. 다음 예시는 keyof Album 타입으로 얻게 되는 결과이다.

```typescript
type K = keyof Album;
//   ^? type K = keyof Album
//      (equivalent to "artist" | "title" | "releaseDate" | "recordingType")
```

string을 keyof T로 바꾼다.

```typescript
function pluck<T>(records: T[], key: keyof T) {
  return records.map(r => r[key]);
}
```

이 코드는 타입 체커를 통과하며, 타입스크립트가 반환 타입을 추론할 수 있게 해준다.

```typescript
function pluck<T>(records: T[], key: keyof T): T[keyof T][]
```

`T[keyof T][]`는 T 객체 내의 가능한 모든 값의 타입이다.

key의 값으로 하나의 문자열을 넣게 되면 범위가 너무 넓어서 적절한 타입이 아니다.

```typescript
const releaseDates = pluck(albums, 'releaseDate');
//    ^? const releaseDates: (string | Date)[]
```

`(string | Date)[]` 가 아니라 `Date[]` 이어야 한다. `keyof T`는 `string`에 비해 훨씬 범위가 좁지만 여전히 넓다. 범위를 더 좁히기 위해 `keyof T`의 부분집합으로 두 번째 제너릭 매개변수를 도입해야 한다.

```typescript
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
  return records.map(r => r[key]);
}
```

⭐️ 시그니처가 완벽

```typescript
const dates = pluck(albums, 'releaseDate');
//    ^? const dates: Date[]
const artists = pluck(albums, 'artist');
//    ^? const artists: string[]
const types = pluck(albums, 'recordingType');
//    ^? const types: RecordingType[]
const mix = pluck(albums, Math.random() < 0.5 ? 'releaseDate' : 'artist');
//    ^? const mix: (string | Date)[]
const badDates = pluck(albums, 'recordingDate');
//                             ~~~~~~~~~~~~~~~
// Argument of type '"recordingDate"' is not assignable to parameter of type ...
```

pluck을 여러 방식으로 호출해도, 제대로 반환 타입을 추론하고, 무효한 매개변수를 방지하고 있다. 매개변수 타입이 정밀해져 언어서비스는 Album의 키에 자동 완성 기능을 제공할 수 있다.

## 📍 정리

string 타입은 any와 비슷한 문제를 가지고 있다. 무효한 값을 허용하고, 타입 간의 관계도 감춰버리는 문제가 있다. 타입 체커를 방해하고 실제 버그를 찾기 못하게 한다. string의 부분 집합을 정의할 수 있는 기능이 타입의 안정성을 높인다.

## 📍 요약

* 문자열을 남발하여 선언된 코드를 피하자. 모든 문자열을 할당할 수 있는 string 타입보다는 더 구체적인 타입을 사용하는 것이 좋다.
* 변수의 범위를 보다 정확하게 표현하고 싶다면 string 타입보다는 문자열 리터럴 타입의 유니온을 사용하면 된다. 타입 체크를 더 엄격히 하여 생산성을 향상시킬 수 있다.
* 객체의 속성 이름을 함수 매개변수로 받을 때는 string보다는 keyof T를 사용하는 것이 좋다.
