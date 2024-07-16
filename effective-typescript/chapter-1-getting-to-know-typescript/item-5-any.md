# 📎 아이템 5 any 타입 지양하기

타입스크립트의 타임시스템은 코드에 타입을 조금씩 추가할 수 있기 때문에 **점진적**이고, 언제든지 타입 체커를 해제할 수 있기 때문에 **선택적**이다. 이 기능들의 핵심은 `any` 타입이다.

```typescript
let ageInYears: number;
ageInYears = '12';
// ~~~~~~~ Type 'string' is not assignable to type 'number'.
ageInYears = '12' as any;  // OK
```

오류는 `as any`를 추가해 해결할 수 있다. 타입 선언을 추가하는 데에 시간을 쏟고 싶지 않아 `any` 타입이나 타입 단언문(`as any`)을 사용하고 싶기도 할 테지만, 일부 특별한 경우를 제외하고는 `any`를 사용하면 타입스크립트의 장점을 누릴 수 없게 된다. `any`의 위험성을 알고 있자.

## 📍any 타입에는 타입 안정성이 없다

앞의 예제에서 `ageInYears`는 `number` 타입으로 선언되었다. 그러나 `as any`를 사용하면 `string` 타입을 할당할 수 있게 된다. 타입 체커는 선언에 따라 `number` 타입으로 판단할 것이고 혼돈이 생긴다.

```typescript
ageInYears += 1;  // OK; at runtime, ageInYears is now "121"
```

## 📍any는 함수 시그니처를 무시해 버린다

함수를 작성할 때는 시그니처를 명시해야 한다. 호출하는 쪽은 약속된 타입의 입력을 제공하고, 함수는 약속된 타입의 출력을 반환한다. `any` 타입은 이런 약속을 어긴다.

```typescript
function calculateAge(birthDate: Date): number {
  // ...
}

let birthDate: any = '1990-01-19';
calculateAge(birthDate);  // OK
```

`birthDate` 매개변수는 `string`이 아닌 `Date` 타입이어 한다. `any` 타입을 사용하면 `calculateAge`의 시그니처를 무시하게 된다. (자바스크립트에서 암시적으로 타입이 변환되기 때문에 특히 문제가 된다.)

→ `string` 타입은 `number` 타입이 필요한 곳에서 오류 없이 실행될 때가 있고, 그럴 경우 다른 곳에서 문제를 일으키게 될 것이다.

## 📍any 타입에는 언어 서비스가 적용되지 않는다

타입스크립트 언어 서비스는 자동완성 기능과 적절한 도움말을 제공한다. `any` 타입인 심벌을 사용하면 도움을 받지 못한다.

이름 변경 기능은 또 다른 언어 서비스이다. 다음 코드는 Person 타입과 이름을 포매팅해주는 함수이다.

```typescript
interface Person {
  first: string;
  last: string;
}

const formatName = (p: Person) => `${p.first} ${p.last}`;
const formatNameAny = (p: any) => `${p.first} ${p.last}`;
```

편집기에서 앞 코드의 `first`를 선택하고 'Rename Symbol'을 클릭해, `firstName`으로 바꿀 수 있다. 예를 들어, `formatName` 함수 내의 `first`가 `firstName`으로 바뀌지만 `any` 타입의 심벌은 바뀌지 않는다.

```typescript
interface Person {
  firstName: string;
  last: string;
}
const formatName = (p: Person) => `${p.firstName} ${p.last}`;
const formatNameAny = (p: any) => `${p.first} ${p.last}`;
```

타입스크립트의 모토는 '확장 가능한 자바스크립트'이다. '확장'의 중요한 부분은 타입스크립트 경험의 핵심 요소인 언어 서비스이다. 언어 서비스를 제대로 누려야 생산성이 향상된다.

## 📍any 타입은 코드 리팩터링 때 버그를 감춘다

웹 어플리케이션을 만들 때 `onSelectItem` 콜백 컴포넌트의 선택하려는 아이템의 타입이 무엇인지 알기 어려워 `any`를 사용해보자.

```typescript
interface ComponentProps {
  onSelectItem: (item: any) => void;
}

// onSelectItem 콜백이 있는 컴포넌트를 사용하는 코드
function renderSelector(props: ComponentProps) { /* ... */ }

let selectedId: number = 0;
function handleSelectItem(item: any) {
  selectedId = item.id;
}

renderSelector({onSelectItem: handleSelectItem});
```

`onSelectItem`에 아이템 객체를 필요한 부분만 전달하도록 컴포넌트를 개선해보자. 여기서는 `id`만 필요하다. `ComponentProps`의 시그니처를 변경한다.

```typescript
interface ComponentProps {
  onSelectItem: (id: number) => void;
}
```

컴포넌트를 수정하고, 타입 체크를 모두 통과했다.

타입 체크를 통과했지만 `handleSelectItem`은 `any` 매개변수를 받는다. 따라서 `id`를 전달받아도 문제가 없다고 나온다. 타입 체커를 통과해도 런타임에는 오류가 발생한다. `any`가 아니라 구체적인 타입을 사용했다면, 타입 체커가 오류를 발견했을 것이다.

## 📍any는 타입 설계를 감춰버린다

애플리케이션 상태 객체 안에 수많은 속성의 타입을 일일이 작성할 때 `any` 타입을 사용하면 간단히 끝내버릴 수 있다.

하지만, 이 때도 `any`를 사용하면 안 된다. 특히, 객체를 정의 할 때 상태 객체의 설계를 감춰버리기 때문에 문제가 된다. `any` 타입을 사용하면 타입 설계가 불분명해져 설계가 잘 되었는지, 설계가 어떻게 되어 있는지 전혀 알 수 없다.

설계가 명확해 보이도록 타입을 일일이 작성하는 것이 좋다.

## 📍any는 타입시스템의 신뢰도를 떨어뜨린다

보통은 타입 체커가 실수를 잡아주고 코드의 신뢰도가 높아진다. 그러나 런타임에 타입 오류를 발견하게 된다면 타입 체커를 신뢰할 수 없게 된다. 타입 체커를 신뢰할 수 없게 된다면 큰 문제가 된다.

→ `any` 타입을 쓰지 않으면 런타임에 발견될 오류를 미리 잡아 신뢰도를 높일 수 있다.

타입스크립트는 개발자의 삶을 편하게 하는데 목적이 있지만, any 타입으로 인해 자바스크립트보다 일을 더 어렵게 만들기도 한다. 타입 오류를 고쳐야 하고, 실제 타입을 기억해야 하기 때문이다. (타입이 실제 값과 일치한다면 타입 정보를 기억해 둘 필요가 없다.)

어쩔 수 없이 `any` 타입을 써야만 하는 상황도 있다. `any`의 단점을 어떻게 보완해야 하는지는 추후에 5장에서 알아보자.

## 📍요약

* any 타입을 사용하면 타입 체커와 타입스크립트 언어 서비스를 무력화시킨다. any 타입은 진짜 문제점을 감추며, 개발 경험을 나쁘게 하고, 타입 시스템의 신뢰도를 떨어뜨린다.
* 최대한 사용을 피하도록 하자.
