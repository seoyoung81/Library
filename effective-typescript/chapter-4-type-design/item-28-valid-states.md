# 📎 아이템 28 유효한 상태만 표현하는 타입을 지향하기

타입을 잘 설계하면 코드는 직관적으로 작성할 수 있다. 효과적으로 타입을 설계하려면, 유효한 상태만 표현할 수 있는 타입을 만들어 내는 것이 가장 중요하다.

## 📍예제

### 🔗 웹 애플리케이션

애플리케이션에서 페이지를 선택하면, 페이지의 내용을 로드하고 화면에 표시한다. 페이지의 상태는 다음과 같이 설계한다.&#x20;

```typescript
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}

// 상태 객체의 필드를 전부 고려해서 상태 표시를 분기
function renderPage(state: State) {
  if (state.error) {
    return `Error! Unable to load ${currentPage}: ${state.error}`;
  } else if (state.isLoading) {
    return `Loading ${currentPage}...`;
  }
  return `<h1>${currentPage}</h1>\n${state.pageText}`;
}
```

코드를 살펴보면 조건이 명확히 분리되어 있지 않다. isLoading이 true이고 동시에 error 값이 존재하면 로딩 중인 상태인지 오류가 발생한 상태인지 명확히 구분할 수 없다. 필요한 정보가 부족하기 때문이다.

```typescript
// 페이지를 전환하는 changePage
async function changePage(state: State, newPage: string) {
  state.isLoading = true;
  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
    }
    const text = await response.text();
    state.isLoading = false;
    state.pageText = text;
  } catch (e) {
    state.error = '' + e;
  }
}
```

❓ changePage에는 많은 문제점이 있다.

☑️ 오류가 발생했을 때 state.isLoading을 false로 설정하는 로직이 빠졌다.

☑️ state.error를 초기화하지 않았기 때문에, 페이지 전환 중에 로딩 메시지 대신 과거의 오류 메시지를 보여주게 된다.

☑️ 페이지 로딩 중에 사용자가 페이지를 바꿔 버리면 어떤 일이 벌어질지 예상하기 어렵다. 새 페이지에 오류가 뜨거나, 응답이 오는 순서에 따라 두 번째 페이지가 아닌 첫 번재 페이지로 전환 될 수도 있다.

→ 바로 상태 값의 두 가지 속성이 동시에 정보가 부족하거나, 두 가지 속성이 충돌할 수 있다.

State 타입은 isLoading이 true이면서 동시에 error 값이 설정되는 무효한 상태를 허용한다. 무효한 상태가 존재하면 render()와 changePage() 둘 다 제대로 구현할 수 없다.

⭐️ 애플리케이션의 상태를 좀 더 제대로 표현하기

```typescript
interface RequestPending {
  state: 'pending';
}
interface RequestError {
  state: 'error';
  error: string;
}
interface RequestSuccess {
  state: 'ok';
  pageText: string;
}
type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: {[page: string]: RequestState};
}
```

네트워크 요청 과정 각각의 상태를 명시적으로 모델링하는 태그된 유니온이 사용되었다. 코드가 길어지긴 했지만, 무효한 상태를 허용하지 않도록 개선되었다.&#x20;

```typescript
function renderPage(state: State) {
  const {currentPage} = state;
  const requestState = state.requests[currentPage];
  switch (requestState.state) {
    case 'pending':
      return `Loading ${currentPage}...`;
    case 'error':
      return `Error! Unable to load ${currentPage}: ${requestState.error}`;
    case 'ok':
      return `<h1>${currentPage}</h1>\n${requestState.pageText}`;
  }
}

async function changePage(state: State, newPage: string) {
  state.requests[newPage] = {state: 'pending'};
  state.currentPage = newPage;
  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
    }
    const pageText = await response.text();
    state.requests[newPage] = {state: 'ok', pageText};
  } catch (e) {
    state.requests[newPage] = {state: 'error', error: '' + e};
  }
}
```

renderPage와 changePage의 모호함은 완전히 사라졌다. 현재 페이지가 무엇인지 정확히 하나의 상태로 맞아 떨어진다. 무효가 된 요청이 실행되긴 하겠지만 UI에는 영향을 미치치 않는다.

### 🔗 비행기 추락 사례

타입스크립트로 이중 입력 모드 상태를 모델링해보자.

```typescript
interface CockpitControls {
  /** Angle of the left side stick in degrees, 0 = neutral, + = forward */
  leftSideStick: number;
  /** Angle of the right side stick in degrees, 0 = neutral, + = forward */
  rightSideStick: number;
}
```

이 데이터 구조가 주어진 상태에서 현재 스틱의 설정을 계산하는 getStickSeting 함수를 작성해보자.

```typescript
// 기장(왼쪽 스틱)이 제어하고 있다고 가정
function getStickSetting(controls: CockpitControls) {
  return controls.leftSideStick;
}
```

```typescript
// 부기장이 제어하고 있는 상태 -> 중립
function getStickSetting(controls: CockpitControls) {
  const {leftSideStick, rightSideStick} = controls;
  if (leftSideStick === 0) {
    return rightSideStick;
  }
  return leftSideStick;
}
```

→ 기장이든 부기장이든 둘 중 하나의 스틱 값 중에서 중립이 아닌 값을 사용해야 한다.

❓ 왼쪽 스틱의 로직과 동일하게 오른쪽 스틱이 중립일 때 왼쪽 스틱 값을 사용해야 한다. 오른쪽 스틱에 대한 체크를 코드에 추가해야 한다.

```typescript
function getStickSetting(controls: CockpitControls) {
  const {leftSideStick, rightSideStick} = controls;
  if (leftSideStick === 0) {
    return rightSideStick;
  } else if (rightSideStick === 0) {
    return leftSideStick;
  }
  // ???
}
```

두 스틱이 모두 중립이 아닌 경우, 두 스틱이 비슷한 값이라면 스틱의 각도를 평균해서 계산할 수 있다.

```typescript
function getStickSetting(controls: CockpitControls) {
  const {leftSideStick, rightSideStick} = controls;
  if (leftSideStick === 0) {
    return rightSideStick;
  } else if (rightSideStick === 0) {
    return leftSideStick;
  }
  if (Math.abs(leftSideStick - rightSideStick) < 5) {
    return (leftSideStick + rightSideStick) / 2;
  }
  // ???
}
```

두 스틱의 각도가 매우 다른 경우는 해결하기 어렵다. 조종사에게 오류를 띄울 수는 없다.

→ 대부분의 비행기는 두 개의 스틱이 기계적으로 연결되어 있다. 부기장이 뒤로 당긴다면, 기장의 스틱도 뒤로 당겨진다.

⭐️ 기계적으로 연결된 스틱의 상태 표현

```typescript
interface CockpitControls {
  /** Angle of the stick in degrees, 0 = neutral, + = forward */
  stickAngle: number;
}
```

순서도(코드의 로직)이 분명해졌다. getStickSetting 함수는 전혀 필요 없다.



타입을 설계할 때는 어떤 값들을 포함하고 어떤 값들을 제외할 지 신중하게 생각해야 한다. 유효한 상태를 표현하는 값만 허용한다면 코드를 작성하기 쉬워지고, 타입 체크가 용이하다. 유효한 상태만 허용하는 것은 매우 일반적인 원칙이다.

## 📍 요약

* 유효한 상태와 무효한 상태를 둘 다 표현하는 타입은 혼란을 초래하기 쉽고 오류를 유발하게 된다.
* 유효한 상태만 표현하는 타입을 지향해야 한다. 코드가 길어지거나 표현하기 어렵지만 결국은 시간을 절약할 수 있다.
