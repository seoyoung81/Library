# 📎 아이템 36 해당 분야의 용어로 타입 이름 짓기

캐시 무효화와 이름 짓기는 컴퓨터 과학에서 가장 어려운 일!

## 📍 타입 이름의 중요성

엄선된 타입, 속성, 변수의 이름은 의도를 명확히 하고 코드와 타입의 추상화 수준을 높여준다. 잘못 선택한 타입 이름은 코드의 의도를 왜곡하고 잘못된 개념을 심어준다.

## 📍 예시

### 🔗 동물의 데이터베이스

```typescript
interface Animal {
  name: string;
  endangered: boolean;
  habitat: string;
}

const leopard: Animal = {
  name: 'Snow Leopard',
  endangered: false,
  habitat: 'tundra',
};
```

**❓ 이 코드의 네가지 질문**

1. name은 매우 일반적인 용어로 동물의 학명인지 일반적인 명칭인지 알 수 없다.
2. endangered 속성이 멸종위기를 표현하기 위한 boolean 타입인지 이상하다. 이미 멸종된 동물을 true로 해야하는 것인지 알 수 없다.
3. 서식지를 나타내는 habitat 속성은 너무 범위가 넓은 string 타입이며 서식지라는 뜻도 불분명하다.
4. 객체의 변수명이 leopard이지만, name 속성 값은 Snow Leopard이다. 객체의 이름과 속성의 name이 다른 의도로 사용된 것인지 알 수 없다.

**⭐️ 속성에 대한 정보를 명확하게 하기 위해선 작성한 사람에게 물어봐야한다. (회사에 없거나 작성자가 본인인 경우..)**

<pre class="language-typescript"><code class="lang-typescript"><strong>// 타입 선언의 이미가 분명한 경우
</strong><strong>interface Animal {
</strong>  commonName: string;
  genus: string;
  species: string;
  status: ConservationStatus;
  climates: KoppenClimate[];
}
type ConservationStatus = 'EX' | 'EW' | 'CR' | 'EN' | 'VU' | 'NT' | 'LC';
type KoppenClimate = |
  'Af' | 'Am' | 'As' | 'Aw' |
  'BSh' | 'BSk' | 'BWh' | 'BWk' |
  'Cfa' | 'Cfb' | 'Cfc' | 'Csa' | 'Csb' | 'Csc' | 'Cwa' | 'Cwb' | 'Cwc' |
  'Dfa' | 'Dfb' | 'Dfc' | 'Dfd' |
  'Dsa' | 'Dsb' | 'Dsc' | 'Dwa' | 'Dwb' | 'Dwc' | 'Dwd' |
  'EF' | 'ET';
const snowLeopard: Animal = {
  commonName: 'Snow Leopard',
  genus: 'Panthera',
  species: 'Uncia',
  status: 'VU',  // vulnerable
  climates: ['ET', 'EF', 'Dfd'],  // alpine or subalpine
};
</code></pre>

**⭐️ 개선점 3가지**

1. name은 더 구체적인 용어로 개선(commonName, genus, species)
2. endangered는 구체적인 기준을 바탕으로 ConservationStatus 타입의 status로 변경
3. habitat은 기후를 뜻하는 Climates로 변경

데이터를 훨씬 명확하게 표현하며 정보를 찾기 위해 사람에 의존할 필요가 없다.

코드로 표현하고자 하는 모든 분야에는 주제를 설명하기 위한 전문 용어들이 있다. 해당 분야에 이미 존재하는 용어를 사용하여 사용자와 소통에 유리하게 하고, 타입의 명확성을 올리자.

## 📍 타입, 속성, 변수 짓기의 3가지 규칙

1. 동일한 의미를 나타낼 때는 같은 용어 사용하기. 동의어를 사용하면 글을 읽을 때는 좋을 수 있지만 코드에서는 아니다. 의미적으로 구분 되어야 하는 경우에만 다른 용어를 사용하자.
2. data, info, thing, item, object, entity 같은 모호하고 의미 없는 이름은 피하자. 귀찮다고 의미없는 이름을 붙여서는 안된다. entity라는 용어가 해당 분야에서 특별한 의미를 가진다면 괜찮다.
3. 이름을 지을때는 포함된 내용이나 계산 방식이 아니라 데이터 자체가 무엇인지를 고려해야 한다. INodeList보다는 Dictionary가 더 의미있다. (구현의 측면이 아니라 개념적인 측면에서 디렉터리를 생각하게 된다.) → 좋은 이름은 추상화의 수준을 높이고, 의도치 않은 충돌의 위험성을 줄여준다.

## 📍 요약

* 가독성을 높이고, 추상화 수준을 올리기 위해서 해당 분야의 용어를 사용해야 한다.
* 같은 의미에 다른 이름을 붙이면 안된다. 특별한 의미가 있을 때만 용어를 구분해야 한다.
