# 📎 아이템 45 devDependencies에 typescript와 @types 추가하기

## 📍 npm

`npm(node package manager)`은 자바스크립트에서 필수적이다. 자바스크립트 라이브러리 저장소(`npm` 레퍼지토리)와, 프로젝트가 의존하고 있는 라이브러리들의 버전을 지정하는 방법(`package.json`)을 제공한다.

### 🔗 npm의 의존성 관리

각각의 의존성은 `package.json` 파일 내의 별도 영역에 들어 있다.

**1️⃣ `dependencies`**

현재 프로젝트를 실행하는 데 필수적인 라이브러리들이 포함된다. ex. 프로젝트의 런타임에 lodash가 사용된다면 `dependencies`에 포함되어야 한다. 프로젝트를 `npm`에 공개하여 다른 사용자가 해당 프로젝트를 설치하면, `dependencies`에 라이브러리들도 함께 설치된다.

**전의(transitive) 의존성**이라고 한다.

**2️⃣ `devDependencies`**

현재 프로젝트를 개발하고 테스트하는 데 사용되지만, 런타임에는 필요 없는 라이브러리들이 포함된다. ex. 프로젝트에서 사용중인 테스트 프레임워크가 devDependencies에 포함된다. 프로젝트를 `npm`에 공개하여 다른 사용자가 해당 프로젝트를 설치하면, 제외된다.

**3️⃣ `peerDependencies`**

런타임에 필요하긴 하지만, 의존성을 직접 관리하지 않는 라이브러리들이 포함된다. ex. 플러그인(제이쿼리의 플러그인은 다양한 버전의 제이쿼리와 호환되므로 제이쿼리의 버전을 플러그인에서 직접 선택하지 않고, 실제 프로젝트에서 선택하도록 만들 때 사용한다.)

일반적으로 `dependencies`와 `devDependencies`가 사용된다. 라이브러리를 추가할 때 어떤 종류의 의존성을 사용해야 하는지 알고 있어야 한다. 일반적으로, 타입스크립트와 관련된 라이브러리는 `devDependencies`에 속한다. (타입스크립트는 개발도구이며 타입 정보는 런타임에 존재하지 않기 때문)

### 🔗 프로젝트에서 고려해야 할 의존성

**1️⃣ 타입스크립트 자체 의존성**

**⭐️ 타입스크립트를 시스템 레벨로 설치하는 것을 추천하지 않는 이유**

* 팀원들 모두가 동일한 버전을 설치한다는 보장이 없다.
* 프로젝트를 셋업할 때 별도의 단계가 추가된다.

→ 타입스크립트를 시스템 레벨로 설치하는 것보다 `devDependencies`에 넣는 것이 좋다. `devDependencies`에 포함되어 있다면, npm install을 실행할 때 팀원들 모두 정확한 타입스크립트 버전을 설치할 수 있다.

대부분의 타입스크립트 IDE와 빌드 도구는 `devDependencies를` 통해 설치된 타입스크립트의 버전을 인식할 수 있다.  커맨드 라인에서 npx를 사용해 타입스크립트 컴파일러를 실행할 수 있다.

```typescript
npx tsc
```

**2️⃣ 타입 의존성(@types)을 고려해야 한다.**

사용하려는 라이브러리에서 타입 선언이 포함되어 있지 않더라도, `DefinitelyTyped`(커뮤니티)에서 타입 정보를 얻을 수 있다. `@types` 라이브러리는 타입 정보만 포함하고 있으며 구현체는 포함하지 않는다.

원본 라이브러리가 `dependencies`에 있더라도 `@types` 의존성은 `devDependencies`에 있어야 한다. 예를 들어, 리액트의 타입 선언과 리액트를 의존성에 추가하려면 다음을 실행하자.

```typescript
$ npm install react

$ npm install --save-dev @types/react
```

다음과 같은 `package.json` 파일이 생성된다

```json
{
  "devDependencies": {
    "@types/react": "^16.8.19",
    "typescript": "^3.5.3"
  },
  "dependencies": {
    "react": "^16.8.6"
  }
}
```

런타임에 `@types/reac`t 와 `typescript`에 의존하지 않겠다는 것이다.  의존성을 `devDependencies`에 넣는 것은 항상 유효한 것은 아니다. `@types` 의존성과 관련된 문제점이 있다. (아이템 46)

## 📍 요약

* 타입스크립트를 시스템 레벨로 설치하면 안 된다. 타입스크립트를 프로젝트의 devDependencies에 포함시키고 팀원 모두가 동일한 버전을 사용하도록 하자.
* @types 의존성은 dependencies가 아니라 devDependencies에 포함시켜야 한다. 런타임에 @types가 필요한 경우라면 별도의 작업이 필요할 수 있다.
