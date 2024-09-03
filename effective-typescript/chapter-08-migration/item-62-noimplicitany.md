# 📎 아이템 62 마이그레이션의 완성을 위해 noImplicitAny 설정하기

프로젝트 전체를 .ts로 전환한 뒤, 마지막으로 `noImplicitAny`를 설정한다. 숨겨진 실제 오류를 찾을 수 있다.

**🔗 `noImplicitAny` 설정 전**

```typescript
class Chart {
  indices: number[];

  // ...
}

getRanges() {
  for (const r of this.indices) {
    const low = r[0];
    //    ^? const low: any
    const high = r[1];
    //    ^? const high: any
    // ...
  }
}
```

`number[]`가 잘못된 타입이지만 오류를 나타내지 않는다. r은 `number` 타입으로 추론되는데 배열 인덱스 접근에 오류가 없다는 것을 주목하자.

**🔗 `noImplicitAny` 설정 후**

```typescript
getRanges() {
  for (const r of this.indices) {
    const low = r[0];
    //          ~~~~ Element implicitly has an 'any' type because
    //               type 'Number' has no index signature
    const high = r[1];
    //           ~~~~ Element implicitly has an 'any' type because
    //                type 'Number' has no index signature
    // ...
  }
}
```

타입 체크가 정상적으로 작동하고 있다. `indices`의 타입은 `number[][]` 이다.

## 📍 정리

처음에는 `noImplicitAny`를 로컬에만 설정하고 작업하는 것이 좋다. 빌드를 실패할 수 있기 때문이다. 타입 체커가 발생하는 오류의 개수를 `noImplicitAny` 관련된 작업의 진척도를 나타내는 지표로 활용할 수 있다.

타입 체크의 강도를 높이는 설정에는 여러 가지가 있다.

`strict: true` > `noImplicitAny` > `strictNullChekcs`

## 📍 요약

* `noImplicitAny` 설정을 활성화하여 마이그레이션의 마지막 단계를 진행해야 한다. `noImplicitAny` 설정이 없다면 타입 선언과 관련된 실제 오류가 드러나지 않는다.
* `noImplicitAny`를 전면 적용하기 전에 로컬에서부터 타입 오류를 점진적으로 수정해야 한다. 엄격한 타입 체크를 적용하기 전에 팀원들이 타입스크립트에 익숙해질 수 있도록 시간을 주자.
