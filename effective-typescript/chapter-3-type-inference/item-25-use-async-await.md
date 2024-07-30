# 📎 아이템 25 비동기 코드에는 콜백 대신 async 함수 사용하기

## 📍 콜백

자바스크립트에서 비동기 동작을 모델링하기 위해 콜백을 사용했고, 콜백 지옥(callback hell)을 필연적으로 마주칠 수 밖에 없다.

```typescript
declare function fetchURL(
  url: string, callback: (response: string) => void
): void;

fetchURL(url1, function(response1) {
  fetchURL(url2, function(response2) {
    fetchURL(url3, function(response3) {
      // ...
      console.log(1);
    });
    console.log(2);
  });
  console.log(3);
});
console.log(4);

// Logs:
// 4
// 3
// 2
// 1
```

실행 순서는 코드의 순서와 반대이다. 이러한 콜백 중첩 코드는 직관적으로 이해하기 어렵다. 요청들을 병렬로 실행하거나 오류 상황을 빠져나오고 싶다면 더욱 혼란스럽다.

## 📍 프로미스(promise)

ES2015 콜백 지옥을 극복하기 위한 프로미스(promise)를 사용해 앞의 코드를 수정해보자.

```typescript
const page1Promise = fetch(url1);
page1Promise.then(response1 => {
  return fetch(url2);
}).then(response2 => {
  return fetch(url3);
}).then(response3 => {
  // ...
}).catch(error => {
  // ...
});
```

코드의 중첩도 적어지고, 코드 순서와 실행순서가 같아졌다. → 오류를 처리하고, Promise.all 기법을 사용하기도 더 쉬워졌다.

## 📍 async, await

ES2017 async와 await 키워드를 도입하여 코드를 수정해보자.

```typescript
async function fetchPages() {
  const response1 = await fetch(url1);
  const response2 = await fetch(url2);
  const response3 = await fetch(url3);
  // ...
}
```

await 키워드는 각각의 프로미스가 처리(resolve) 될 때까지 fetchPages 함수의 실행을 멈춘다. async 함수 내에서 await 중인 프로미스가 거절(reject)되면 예외를 던진다. 이를 통해 일반적인 try/catch 구문을 사용할 수 있다.

```typescript
async function fetchPages() {
  try {
    const response1 = await fetch(url1);
    const response2 = await fetch(url2);
    const response3 = await fetch(url3);
    // ...
  } catch (e) {
    // ...
  }
}
```

타입스크립트는 런타임에 관계없이 async/await를 사용할 수 있다.

### ✓ 콜백보다 async/await를 사용해야 하는 이유

1. 콜백보다는 프로미스가 코드를 작성하기 쉽다.
2. 콜백보다는 프로미스가 타입을 추론하기 쉽다.

#### 🔗 병렬로 페이지를 로드하고 싶은 경우

→ Promise.all을 사용해서 프로미스를 조합하면 된다.

```typescript
async function fetchPages() {
  const [response1, response2, response3] = await Promise.all([
    fetch(url1), fetch(url2), fetch(url3)
  ]);
  // ...
}
```

이런 경우 await와 구조 분해 할당이 찰떡궁합이다.

타입스크립트는 세 가지 response 변수 각각의 타입을 Response로 추론한다. 그러나 콜백 스타일로 동일한 코드를 작성하려면 더 많은 코드와 타입 구문이 필요하다.

```typescript
function fetchPagesWithCallbacks() {
  let numDone = 0;
  const responses: string[] = [];
  const done = () => {
    const [response1, response2, response3] = responses;
    // ...
  };
  const urls = [url1, url2, url3];
  urls.forEach((url, i) => {
    fetchURL(url, r => {
      responses[i] = url;
      numDone++;
      if (numDone === urls.length) done();
    });
  });
}
```

→ 코드에 오류 처리를 포함하거나, Promise.all 같은 일반적인 코드로 확장하는 것은 쉽지 않다.

#### 🔗 입력된 프로미스들 중 첫 번째가 처리될 때 완료되는 Promise.race

Promise.race를 사용해서 프로미스에 타임아웃을 추가하는 방법은 흔하게 사용되는 패턴이다.

```typescript
function timeout(timeoutMs: number): Promise<never> {
  return new Promise((resolve, reject) => {
     setTimeout(() => reject('timeout'), timeoutMs);
  });
}

async function fetchWithTimeout(url: string, timeoutMs: number) {
  return Promise.race([fetch(url), timeout(timeoutMs)]);
}
```

타입 구문 없어도 fetchWithTimeout의 반환 타입은 Promise\<Response>로 추론된다. 추론이 동작하는 이유를 살펴보자.

Promise.race의 반환 타입은 입력 타입들의 유니온이고, 이번 경우는 Promise\<Response | never>가 된다. 그러나 never와의 유니온은 아무런 효과가 없으므로, 결과가 Promise\<Response>로 간단해진다. 프로미스를 사용하면 타입 추론이 제대로 동작한다.

#### 🔗 프로미스를 직접 생성해야 할 때

setTimeout과 같은 콜백 API를 래핑할 경우 → 일반적으로 프로미스를 생성하는 것보다 async/await를 사용하는 것이 더 좋다.

1. 일반적으로 더 간결하고 직관적인 코드가 된다.
2. async 함수는 항상 프로미스를 반환하도록 강제된다.

```typescript
// function getNumber(): Promise<number>
async function getNumber() { return 42; }
```

async 화살표 함수를 만들 수도 있다.

```typescript
const getNumber = async () => 42;
// type: () => Promise<number>
```

프로미스를 직접 생성하면

```typescript
const getNumber = () => Promise.resolve(42);
// type () => Promise<number>
```

즉시 사용 가능한 값에도 프로미스를 반환하는 것이 이상해보일 수 있지만, 실제로는 비동기 함수로 통일하도록 강제하는데 도움이 된다. 함수는 항상 동기 또는 비동기로 실행되어야 하며 절대 혼용해서는 안 된다.

🔗 예를 들어, fetchURL 함수에 캐시를 추가하기 위해 다음처럼 시도한다고 가정해보자.

```typescript
// Don't do this!
const _cache: {[url: string]: string} = {};
function fetchWithCache(url: string, callback: (text: string) => void) {
  if (url in _cache) {
    callback(_cache[url]);
  } else {
    fetchURL(url, text => {
      _cache[url] = text;
      callback(text);
    });
  }
}
```

코드가 최적화된 것 처럼 보이지만, 캐시된 경우 콜백 함수가 동기로 호출되기 때문에 `fetchWithCache` 함수는 이제 사용하기 무척 어려워졌다.

async를 두 함수에 모두 사용하면 일관적인 동작을 강제하게 된다.

```typescript
const _cache: {[url: string]: string} = {};
async function fetchWithCache(url: string) {
  if (url in _cache) {
    return _cache[url];
  }
  const response = await fetch(url);
  const text = await response.text();
  _cache[url] = text;
  return text;
}

let requestStatus: 'loading' | 'success' | 'error';
async function getUser(userId: string) {
  requestStatus = 'loading';
  const profile = await fetchWithCache(`/user/${userId}`);
  requestStatus = 'success';
}
// requestStatus가 'success'로 끝나는 것이 명백하다.
```

콜백 함수나 프로미스를 사용하면 실수로 반(half)동기 코드를 작성할 수 있지만, async를 사용하면 항상 비동기 코드를 작성하는 셈이다.

async 함수에서 프로미스를 반환하면 또 다른 프로미스로 래핑되지 않는다. 반환 타입은 Promise\<Promise\<T>>가 아닌 Promise\<T>가 된다. 타입스크립트를 사용하면 타입 정보가 명확히 드러나기 때문에 비동기 코드의 개념을 잡는데 도움이 된다.

```typescript
async function getJSON(url: string) {
  const response = await fetch(url);
  const jsonPromise = response.json();
  return jsonPromise;
  //     ^? const jsonPromise: Promise<any>
}
getJSON
// ^? function getJSON(url: string): Promise<any>
```

## 📍 요약

* 콜백보다는 프로미스를 사용하는게 코드 작성과 타입 추론 면에서 유리하다.
* 가능하면 프로미스를 생성하기보다는 async와 await를 사용하는 것이 좋다. 간결하고 직관적인 코드를 작성할 수 있고, 모든 종류의 오류를 제거할 수 있다.
* 어떤 함수가 프로미스를 반환한다면 async로 선언하는 것이 좋다.
