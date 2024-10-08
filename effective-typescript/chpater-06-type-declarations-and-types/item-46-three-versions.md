# 📎 아이템 46 타입 선언과 관련된 세 가지 버전 이해하기

의존성 관리는 개발자에게 매우 힘든 일이다. 타입스크립트는 의존성 관리를 오히려 더 복잡하게 한다. 타입스크립트를 사용하면 다음 세 가지 사항을 추가로 고려해야 하기 때문이다.

✓ 라이브러리의 버전

✓ 타입 선언(@types)의 버전

✓ 타입스크립트의 버전

세 가지 버전 중 하나라도 맞지 않으면, 의존성과 상관없어 보이는 곳에서 엉뚱한 오류가 발생할 수 있다. 라이브러리 관리의 메커니즘을 이해하면 프로젝트 내에서 작성한 타입 선언을 외부에 공개해야 할 때, 버전과 관련해서 제대로 된 결정을 내릴 수 있다.

## 📍 타입스크립트의 의존성 사용 방식

특정 라이브러리를 `dependencies`로 설치하고, 타입 정보는 `devDependencies`로 설치한다.

```typescript
$ npm install react
+ react@16.8.6

$ npm install --save-dev @types/react
+ @types/react@16.8.19
```

**메이저 버전과 마이너 버전(16.8)이 일치하지만 패치버전(.6 과 .19)은 일치하지 않는다.**

@types/react의 16.8.19는 타입 선언들이 리액트 16.8 버전의 API를 나타낸다는 것을 의미한다. 리액트 모듈이 시맥틱 버전 규칙을 제대로 지킨다고 가정하면, 패치 버전들은 공개 API의 사양을 변경하지 않는다. (타입 선언을 업데이트 할 필요가 없다.)

타입 선언 자체에도 버그나 누락이 존재할 수 있으며 `@types` 모듈의 패치 버전은 버그나 누락으로 인한 수정과 추가에 따른 것이다. 앞의 예제의 경우 라이브러리 자체보다 타입 선언에 더 많은 업데이트가 있었다.

실제 라이브러리와 타입 정보의 버전이 별도로 관리되는 방식은 4가지 문제점이 있다.

## 📍라이브러리와 타입 정보 버전 관리 방식의 문제점

**1️⃣ 라이브러리를 업데이트 했지만 실수로 타입 선언은 업데이트하지 않는 경우**

라이브러리 업데이트와 관련된 새로운 기능을 사용하려 할 때마다 타입 오류 발생. 특히 하위 호환성이 깨지는 변경이 있었다면, 코드가 타입 체커를 통과하더라도 런타임에 오류가 발생한다.

→ 타입 선언도 업데이트하여 라이브러리와 버전 맞추기

❓ 업데이트 해야 할 타입 선언의 버전이 준비되지 않은 경우

⭐️ 보강 기법을 활용하여, 사용하려는 새 함수와 메서드의 타입 정보를 프로젝트 자체에 추가한다.&#x20;

⭐️ 타입 선언의 업데이트를 직접 작성하고 공개하여 커뮤니티에 기여한다.

**2️⃣ 라이브러리보다 타입 선언의 버전이 최신인 경우**

타입 정보 없이 라이브러리를 사용해오다가 타입 선언을 설치하려고 할 때 오류가 발생한다. 그 사이 라이브러리와 타입 선언의 새 버전이 릴리즈 된다면 라이브러리와 타입 선언의 버전 정보는 어긋난다.

타입 체커는 최신 API를 기준으로 코드를 검사하게 되지만 런타임에 실제로 쓰이는 것은 과거 버전이다.

⭐️ 라이브러리와 타입 선언의 버전이 맞도록 라이브러리 버전을 올리거나, 타입선언의 버전 내리기

**3️⃣ 프로젝트에서 사용하는 타입스크립트 버전보다 라이브러리에서 필요로 하는 타입스크립트 버전이 최신인 경우**

일반적으로 유명한 라이브러리 로대시, 리액트, 람다는 타입 정보를 정확하게 표현하기 위해 타입스크립트에서 타입 시스템이 개선되고 버전이 올라가게 된다.

⭐️ 라이브러리들의 최신 타입 정보를 얻기 위해서라면 타입스크립트의 최신 버전을 사용하자.

**4️⃣ @types 의존성의 중복**

@types/foo와 @types/bar에 의존하는 경우, @types/bar가 현재 프로젝트와 호환되지 않는 버전의 @types/foo에 의존한다면 npm은 중첩된 폴더에 별도로 해당 버전을 설치하여 문제를 해결하려고 한다.

```typescript
node/modules/
  @types/
    foo/
      index.d.ts @1.2.3
    bar/
      index.d.ts
      node_modules/
        @types/
          foo/
            index.d.ts @2.3.4
```

런타임에 사용되는 모듈이면 괜찮지만, 전역 네임스페이스(name-space)에 있는 타입 선언 모듈이라면 대부분 문제가 발생한다.

전역 네임 스페이스에 타입 선언이 존재하면, 중복된 선언 또는 선언이 병합될 수 없다는 오류로 나타나게 된다.

```typescript
npm ls @types/foo
```

를 실행하여 타입 선언 중복의 발생지를 추적할 수 있다.

⭐️ @types/foo를 업데이트하거나 @types/bar를 업데이트해서 서로 버전이 호환되게 하는 것이다.

@types이 전이 의존성을 가지도록 만드는 것은 문제가 될 수 있다. 타입 선언을 작성하고 공개하려고 한다면, 아이템 51을 참고하여 문제점을 해결하자.

## 📍 번들링

일부 라이브러리, 특히 <mark style="background-color:blue;">타입스크립트로 작성된 라이브러리들은 자체적으로 타입 선언을 포함(번들링, bundling</mark>)하게 된다. 자체적인 타입 선언은 `package.json`의 `"types"` 필드에서, `.d.ts` 파일을 가르키도록 되어 있다.

```json
{
  "name": "left-pad",
  "version": "1.3.0",
  "description": "String left pad",
  "main": "index.js",
  "types": "index.d.ts",
  // ...
}
```

❓ "types": "index.d.ts"을 추가하면 모든 문제가 해결될까?

번들링하여 타입 선언을 포함하는 경우, 특히 라이브러리가 타입스크립트로 작성되고 컴파일러를 통해 타입 선언이 생성된 경우라면 버전 불일치를 해결하지만, 번들링 방식은 4가지 문제점이 있다.

### 번들링의 문제점

1️⃣ 번들된 타입 선언에 보강 기법으로 해결할 수 없는 오류가 있는 경우,  공개 시점에는 잘 동작했지만 타입스크립트가 버전이 올라가면서 오류가 발생하는 경우

@types를 별도 사용하면 라이브러리 자체의 버전에 맞추어 선택할 수 있지만, 번들된 타입에서는 @types의 버전을 선택할 수 없다.&#x20;

2️⃣ 프로젝트 내의 타입 선언이 다른 라이브러리의 타입 선언에 의존하는 경우

보통 의존성은 `devDependencies`에 들어가게 되는데, 프로젝트를 공개하여 다른 사용자가 설치하게 되면 `devDependencies`는 설치되지 않아 오류가 발생한다.

3️⃣ 프로젝트의 과거 버전에 있는 타입 선언에 문제가 있는 경우

과거 버전으로 돌아가서 패치 업데이트를 해야한다. 번들링된 타입 선언에서는 어려운 일이지만, DefinitelyTyped는 동일 라이브러리의 여러 버전의 타입 선언을 동시에 유지보수 할 수 있는 메커니즘을 갖고 있다.

4️⃣ 타입 선언의 패치 업데이트를 자주하기 어려움

앞의 react 예제에서, 라이브러리 자체보다 타입 선언에 대한 패치 업데이트가 3배나 높았다.

## 📍 타입스크립트의 의존성 관리를 잘하면

타입스크립트 의존성 관리는 쉽지 않지만, 잘 관리하면 그에 따른 보상이 존재한다.

잘 작성된 타입 선언은 라이브러리를 올바르게 사용하는 방법을 배우며 생산성도 향상된다. 만약 의존성 관리에 문제가 생긴다면 이번 아이템의 처음에 언급했던 세 가지 버전을 기억하자.

### 라이브러리를 공개하려는 경우

타입 선언을 자체적으로 포함하는 것과 타입 정보만 분리하여 `DefinitelyTyped`에 공개하는 것의 장단점을 비교해야 한다. 공식적인 권장사항은 라이브러리가 타입스크립트로 작성된 경우만 타입 선언을 라이브러리에 포함하는 것이다.

자바스크립트로 작성된 라이브러리는 타입 선언을 DefinitelyTyped에 공개하여 커뮤니티에서 관리하고 유지보수하도록 맡기는 것이 좋다.

## 📍 요약

* @types 의존성과 관련된 세 가지 버전이 있다. 라이브러리 버전, @types 버전, 타입스크립트 버전
* 라이브러리를 업데이트 하는 경우, 해당 @types도 업데이트해야 한다.
* 타입 선언을 라이브러리에 포함하는 것과 DefinitelyTyped에 공개하는 것 사이의 장단점을 이해하자. 타입스크립트로 작성된 라이브러리라면 타입 선언을 자체적으로 포함하고, 자바스크립트로 작성된 라이브러리라면 타입 선언은 DefinitelyTyped에 공개하는 것이 좋다.
