# 📎 아이템 47 공개 API에 등장하는 모든 타입을 익스포트하기

타입스크립트를 사용하다보면, 서드파티의 모듈에서 익스포트되지 않은 타입 정보가 필요한 경우가 생긴다. 라이브러리 제작자는 프로젝트 초기에 타입익스포트부터 작성해야 한다. 만약 함수의 선언에 이미 타입 정보가 있다면 제대로 익스포트 되고 있는 것이며, 타입 정보가 없다면 타입을 명시적으로 작성해야 한다.

**❓만약, 어떤 타입을 숨기고 싶어 익스포지 않았다면?**

```typescript
interface SecretName {
  first: string;
  last: string;
}

interface SecretSanta {
  name: SecretName;
  gift: string;
}

export function getGift(name: SecretName, gift: string): SecretSanta {
  // ...
}
```

해당 라이브러리 사용자는 SecreName 또는 SecretSanta를 직접 임포트할 수 없고, getGift만 임포트 가능하다. 그러나 타입들은 익스포트된 함수 시그니처에 등장하기 때문에 추출할 수 있다.

⭐️ 추출하는 방법은 Parameters와 ReturnType 제너릭 타입을 사용하는 것

```typescript
type MySanta = ReturnType<typeof getGift>;
//   ^? type MySanta = SecretSanta
type MyName = Parameters<typeof getGift>[0];
//   ^? type MyName = SecretName
```

굳이 숨기려 하지말고 라이브러리 사용자를 위해 명시적으로 임포트 하는 것이 좋다. 만약 융통성을 위해 타입들을 일부러 익스포트 하지 않아도, 공개 API 매개변수에 놓이는 순간 타입은 노출된다.

## 📍 요약

* 공개 메서드에 등장한 어떤 형태의 타입이든 익스포트 하자. 어차피 라이브러리 사용자가 춫룰할 수 있으므로 익스포트 하기 쉽게 만드는 것이 좋다.
