---
coverY: 0
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 📓 리팩터링 2판

## 1.1 자, 시작해보자!&#x20;

#### 리팩터링의 시작

**무엇을 수정할지 찾기 어렵다면?**

* 코드를 수정할 때, 먼저 프로그램의 작동 방식을 더 쉽게 파악할 수 있도록 코드를 여러 함수와 프로그램 요소로 재구성한다.
* 프로그램이 새로운 기능을 추가하기에 편한 구조가 아니라면, 먼저 기능을 추가하기 쉬운 형태로 리팩터링하고 나서 원하는 기능을 추가한다.

**리팩토링이 필요한 이유?**

* 함수의 복잡도가 증가하는 경우
* 복잡해질 수록 변경이 어렵다

**리팩터링의 첫 단계**

* 코드 영역을 검사해줄 테스트 코드 마련하기
* 테스트 결과를 보고하는 자가 진단 테스트 만들기 (초록불/빨간불)



### 리팩토링 할 예시 코드

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  const format = new Intl.NumberFormat("en-US", 
                                       { style: "currency", currency: "USD", 
                                        minimumFractionDigits: 2 }).format;
  
  for (let perf of invoice.performaces) {
    const play = plays[perf.playID];
    let thisAmount = 0;
    
    switch (play.type) {
      case "tragedy":
        thisAmount = 40000;
        if (perf.audience > 30) {
          thisAmount += 1000 * (perf.audience - 30);
        }
        break;
      case "comedy":
        thisAmount = 30000;
        if (perf.audience > 20) {
          thisAmount += 1000 + 500 * (perf.audience - 20);
        }
        thisAmount += 300 * perf.audience
        break;
      default:
        throw new Error(`알 수 없는 장르: ${play.type}`);
      }
	  
    // 포인트를 적립한다
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if ("comedy" === play.type) volumeCredits += Math.Floor(perf.audience / 5 );
    
    // 청구 내역을 출력한다.
    result += ` ${play.name}: ${format(thisAmount/100)} (${perf.audience}석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount/100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}                                 
```

#### `statement()`함수의 테스트는 어떻게 구성하면 될까?

1. 이 함수가 문자열을 반환하므로, 미리 문자열 형태로 작성하여 준비해둔다
2. statemen()가 반환한 문자열과 준비해둔 정답 문자열을 비교한다
3. 테스트 프레임워크를 이용하여 모든 테스트를 단축키 하나로 실행할 수 있도록 설정해둔다.

#### `statement()`함수 쪼개기

> 긴 함수의 각각의 부분으로 나눌 수 있는 지점을 찾는다

`switch문`: 코드를 분석해서 얻은 정보는 다시 분석하지 않아도 되므로 코드 조각을 별도 함수로 추출하여 파악한 정보를 코드에 반영한다: `amountFor(aPerformane)` **함수 추출하기**: 코드 조각을 함수로 추출할 때 실수를 최소화 해주는 절차

* 유효 범위를 벗어나는 변수, 새 함수에서 곧바로 사용할 수 없는 변수가 있는지 확인한
  * perf, play: 필요하지만 값을 변경하지 않아 매개변수로 전달
  * thisAmount: 값이 변하는 변수를 초기화하는 코드를 추출한 함수에 넣어야 한다

```javascript
function amountFor(perf, play) { // 값이 바뀌지 않는 변수는 매개변수로 전달
  let thisAmount = 0;	// 변수를 초기화하는 코드
	switch (play.type) {
      case "tragedy":
        thisAmount = 40000;
        if (perf.audience > 30) {
          thisAmount += 1000 * (perf.audience - 30);
        }
        break;
      case "comedy":
        thisAmount = 30000;
        if (perf.audience > 20) {
          thisAmount += 1000 + 500 * (perf.audience - 20);
        }
        thisAmount += 300 * perf.audience
        break;
      default:
        throw new Error(`알 수 없는 장르: ${play.type}`);
      }
	  return thisAmound; // 함수 안에서 값이 바뀌는 변수 변환
```

statement()에서는 thisAmount 값을 채울 때 방금 추출한 amountFor() 함수를 호출한다

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  const format = new Intl.NumberFormat("en-US", 
                                       { style: "currency", currency: "USD", 
                                        minimumFractionDigits: 2 }).format;
  
  for (let perf of invoice.performaces) {
    const play = plays[perf.playID];
    let thisAmount = amountFor(perf, play); // 추출한 함수를 이용
    
    // 포인트를 적립한다
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if ("comedy" === play.type) volumeCredits += Math.Floor(perf.audience / 5 );
    
    // 청구 내역을 출력한다.
    result += ` ${play.name}: ${format(thisAmount/100)} (${perf.audience}석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount/100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}                                 
```

수정 후 바로 컴파일하고 테스트를 통해 확인한다. 리팩터링은 프로그램 수정을 작은 단계로 나눠 진행한다. 중간에 실수하더라도 버그를 쉽게 찾을 수 있다. 문제 없는 것을 확인하고 커밋을 진행한다.

#### 함수 추출 후 변수 이름 더 명확하게 바꾸기

1. 반환값

```javascript
function amountFor(perf, play) {
  let result = 0;	// 명확한 이름으로 변경
	switch (play.type) {
      case "tragedy":
        result = 40000;
        if (perf.audience > 30) {
          result += 1000 * (perf.audience - 30);
        }
        break;
      case "comedy":
        result = 30000;
        if (perf.audience > 20) {
          result += 1000 + 500 * (perf.audience - 20);
        }
        result += 300 * perf.audience
        break;
      default:
        throw new Error(`알 수 없는 장르: ${play.type}`);
      }
	  return result;
```

함수의 반환 값으로 `result`를 사용하면 변수의 역할을 쉽게 알 수 있다

2. 인수

```javascript
function amountFor(aPerformance, play) {	// 명확한 이름으로 변경
  let result = 0;
	switch (play.type) {
      case "tragedy":
        result = 40000;
        if (perf.audience > 30) {
          result += 1000 * (perf.audience - 30);
        }
        break;
      case "comedy":
        result = 30000;
        if (perf.audience > 20) {
          result += 1000 + 500 * (perf.audience - 20);
        }
        result += 300 * perf.audience
        break;
      default:
        throw new Error(`알 수 없는 장르: ${play.type}`);
      }
	  return result;
```

매개변수 이름에 접두어로 타입 이름을 적어둔다. 역할이 뚜렷하지 않은 경우 부정 관사를 붙인다. 변수 이름을 바꿔 명확성을 높이는 것은 매우 좋다.

> 타입스크립트를 사용하는 경우에도 좋은 리팩터링 방식인가?

3. 변수 제거 **play 변수 제거하기**

* aPerformance는 루프 변수에서 오기 때문에 반복문을 한 번 돌 때마다 값이 변경 된다
* play는 개별 공연에서 얻기 때문에 매개변수로 전달할 필요가 없다 ➡️ **임시 변수를 질의 함수로 바꾸기**

```javascript
function playFor(aPerformance) {
  return plays[aPerformance.playID];
}
```

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  const format = new Intl.NumberFormat("en-US", 
                                       { style: "currency", currency: "USD", 
                                        minimumFractionDigits: 2 }).format;
  
  for (let perf of invoice.performaces) {
    const play = playFor(perf); // 우변을 함수로 추출
    let thisAmount = amountFor(perf, play); 
    
    // 포인트를 적립한다
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if ("comedy" === play.type) volumeCredits += Math.Floor(perf.audience / 5 );
    
    // 청구 내역을 출력한다.
    result += ` ${play.name}: ${format(thisAmount/100)} ({$perf.audience}석)\n`;
    totalAmount += thisAmount;
  }
  result += `총액: ${format(totalAmount/100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

변수 인라인 적용하기

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  const format = new Intl.NumberFormat("en-US", 
                                       { style: "currency", currency: "USD", 
                                        minimumFractionDigits: 2 }).format;
  
  for (let perf of invoice.performaces) {
 	// 인라인 된 변수 제거
    let thisAmount = amountFor(perf, playFor(perf)); 
    
    // 포인트를 적립한다
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if ("comedy" === playFor(perf).type) volumeCredits += Math.Floor(perf.audience / 5 ); // 변수 인라인
    
    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${format(thisAmount/100)} (${perf.audience}석)\n`;
    totalAmount += thisAmount; // 변수 인라인
  }
  result += `총액: ${format(totalAmount/100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

amountFor()에 함수 선언 바꾸기를 적용해서 play 매개변수를 제거할 수 있게 되었다.

1. playFor()를 사용하도록 amountFor()를 수정한다

```javascript
function amountFor(aPerformance, play) {	
  let result = 0;
	switch (playFor(aPerfomance).type) { // play를 playFor() 호출로 변경
      case "tragedy":
        result = 40000;
        if (perf.audience > 30) {
          result += 1000 * (perf.audience - 30);
        }
        break;
      case "comedy":
        result = 30000;
        if (perf.audience > 20) {
          result += 1000 + 500 * (perf.audience - 20);
        }
        result += 300 * perf.audience
        break;
      default:
        throw new Error(`알 수 없는 장르: ${playFor(aPerfomance).type}`);
      }
	  return result;
```

2. play 매개변수를 삭제한다. thisAmount 변수를 인라인한다.

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  const format = new Intl.NumberFormat("en-US", 
                                       { style: "currency", currency: "USD", 
                                        minimumFractionDigits: 2 }).format;
  
  for (let perf of invoice.performaces) {
    // 포인트를 적립한다
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if ("comedy" === playFor(perf).type) volumeCredits += Math.Floor(perf.audience / 5 ); // 변수 인라인
    
    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${format(amountFor(perf)/100)} (${perf.audience}석)\n`;
    totalAmount += amountFor(perf); // thisAmount 변수를 인라인
  }
  result += `총액: ${format(totalAmount/100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

```javascript
function amountFor(aPerformance) {	
  let result = 0;
	switch (playFor(aPerfomance).type) { // play를 playFor() 호출로 변경
      case "tragedy":
        result = 40000;
        if (perf.audience > 30) {
          result += 1000 * (perf.audience - 30);
        }
        break;
      case "comedy":
        result = 30000;
        if (perf.audience > 20) {
          result += 1000 + 500 * (perf.audience - 20);
        }
        result += 300 * perf.audience
        break;
      default:
        throw new Error(`알 수 없는 장르: ${playFor(aPerfomance).type}`);
      }
	  return result;
```

> 이전 코드는 루프를 한 번 돌 때마다 공연을 조회했는데 리팩터링한 코드에서는 세번이나 조회한다. 성능에 큰 영향은 없지만 제대로 리팩터링 된 코드베이스는 그렇지 않은 코드보다 성능을 개선하기가 훨씬 수월하다

#### 적립 포인트 계산 코드 추출하기

`statement()` 함수 결과

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  const format = new Intl.NumberFormat("en-US", 
                                       { style: "currency", currency: "USD", 
                                        minimumFractionDigits: 2 }).format;
  
  for (let perf of invoice.performaces) {
    
    // 포인트를 적립한다
    volumeCredits += Math.max(perf.audience - 30, 0);
    // 희극 관객 5명마다 추가 포인트를 제공한다.
    if ("comedy" === playFor(perf).type) volumeCredits += Math.Floor(perf.audience / 5 );
    
    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${format(amountFor(perf)/100)} (${perf.audience}석)\n`;
    totalAmount += amountFor(perf); // thisAmount 변수를 인라인
  }
  result += `총액: ${format(totalAmount/100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

`volumeCredits`는 반복문을 돌 때마다 값을 누적해야 하기 때문에 추출한 함수에서 `volumeCredits`의 복제본을 초기화한 뒤 계산결과를 반환하게 리팩터링 하자

```javascript
function volumeCreditsFor(perf) { // 새로 추출한 함수
  let voluemCredits = 0;
  volumeCredits += Math.max(perf.audience - 30, 0);
  if ("comedy" === playFor(perf).type) 
    volumeCredits += Math.Floor(perf.audience / 5 );
  return volumeCreits;
}
```

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  const format = new Intl.NumberFormat("en-US", 
                                       { style: "currency", currency: "USD", 
                                        minimumFractionDigits: 2 }).format;
  
  for (let perf of invoice.performaces) {
    voluemCredits += volumeCreditsFor(perf); // 추출한 함수를 이용해 값을 누적

    
    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${format(amountFor(perf)/100)} (${perf.audience}석)\n`;
    totalAmount += amountFor(perf); // thisAmount 변수를 인라인
  }
  result += `총액: ${format(totalAmount/100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

추출한 함수의 변수명 바꾸기

```javascript
function volumeCreditsFor(aPerformance) { 
  let result = 0;
  volumeCredits += Math.max(aPerformance.audience - 30, 0);
  if ("comedy" === playFor(aPerformance).type) 
    volumeCredits += Math.Floor(aPerformance.audience / 5 );
  return result;
}
```

#### format 변수 제거하기

`statement()` 함수

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  const format = new Intl.NumberFormat("en-US", 
                                       { style: "currency", currency: "USD", 
                                        minimumFractionDigits: 2 }).format;
  
  for (let perf of invoice.performaces) {
    voluemCredits += volumeCreditsFor(perf); 

    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${format(amountFor(perf)/100)} (${perf.audience}석)\n`;
    totalAmount += amountFor(perf); // thisAmount 변수를 인라인
  }
  result += `총액: ${format(totalAmount/100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

임시변수는 자신이 속한 부분에서만 의미가 있어서 루틴이 길고 복잡해지기 쉽다. `format` 임시변수를 함수를 직접 선언하여 바꿔보자.

```javascript
function format(aNumber) {
  return new Intl.NumberFormat("en-US", 
                 { style: "currency", currency: "USD", 
          		   minimumFractionDigits: 2 }).format;
}
```

format을 임시 변수가 아닌 함수 호출로 대체한다.

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  for (let perf of invoice.performaces) {
    voluemCredits += volumeCreditsFor(perf); 

    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${format(amountFor(perf)/100)} (${perf.audience}석)\n`;
    totalAmount += amountFor(perf);
  }
  result += `총액: ${format(totalAmount/100)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

**함수 선언 바꾸기**

```javascript
function usd(aNumber) {	// 함수 이름 변경
  return new Intl.NumberFormat("en-US", 
                 { style: "currency", currency: "USD", 
          		   minimumFractionDigits: 2 }).format(aNumber/100);	// 단위 변환 로직도 이 함수 안으로 이동
}
```

```javascript
function format(aNumber) {
  return new Intl.NumberFormat("en-US", 
                 { style: "currency", currency: "USD", 
          		   minimumFractionDigits: 2 }).format;
}
```

format을 임시 변수가 아닌 함수 호출로 대체한다.

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  for (let perf of invoice.performaces) {
    voluemCredits += volumeCreditsFor(perf); 

    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${usd(amountFor(perf))} ($perf.audience}석)\n`;
    totalAmount += amountFor(perf);
  }
  result += `총액: ${usd(totalAmount)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

긴 함수를 작게 쪼개는 리팩터링은 이름을 잘 지어야만 효과가 있다.

#### volumeCredits 변수 제거하기

volumeCredits 변수는 반복문을 한 바퀴 돌 때마다 값을 누적하기 때문에 **반복문 쪼개기**로 volumeCredits 값이 누적되는 부분을 따로 빼낸다.

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let voluemCredits = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  
  for (let perf of invoice.performaces) {

    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${usd(amountFor(perf))} ($perf.audience}석)\n`;
    totalAmount += amountFor(perf);
  }
  // 값 누적 로직을 별도 for 문으로 분리
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);
  }
  
  result += `총액: ${usd(totalAmount)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

> 값 누적 로직을 분리하는게 리팩터링???

**문장 슬라이드하기** volumeCredits 변수를 선언하는 문장을 반복문 바로 앞으로 옮긴다

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  
  for (let perf of invoice.performaces) {

    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${usd(amountFor(perf))} ($perf.audience}석)\n`;
    totalAmount += amountFor(perf);
  }
  
  let volumeCredits = 0; // 변수 선언(초기화)을 반복문 앞으로 이동
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);
  }
  
  result += `총액: ${usd(totalAmount)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

`volumeCredits` 값 갱신과 관련한 문장들을 모아두면 **임시 변수를 질의 함수로 바꾸기**가 수월해진다. `volumeCredits` 값 계산 코드를 **함수로 추출**하는 착업부터 한다

```javascript
function totalVolumeCredits() {
  let voluemCredits = 0;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);
  }
  return volulmeCredits;
}
```

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  
  for (let perf of invoice.performaces) {

    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${usd(amountFor(perf))} ($perf.audience}석)\n`;
    totalAmount += amountFor(perf);
  }
  
  let volumeCredits = totalVolumeCredits(); // 값 계산 로직을 함수로 추출
  
  result += `총액: ${usd(totalAmount)}\n`;
  result += `적립 포인트: ${volumeCredits}점\n`;
  return result;
}              
```

함수 추출 후 `volumeCredits` 변수를 인라인 한다.

```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  
  for (let perf of invoice.performaces) {

    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${usd(amountFor(perf))} ($perf.audience}석)\n`;
    totalAmount += amountFor(perf);
  }
  
  result += `총액: ${usd(totalAmount)}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;
}              
```

반복문의 중복은 성능에 미치는 영향이 적다. 리팩터링으로 인한 성능 문제는 **특별한 경우가 아니라면 일단 무시하자**. 리팩터링 때문에 성능이 떨어진다면 하던 리팩터링을 마무리하고 나서 성능 개선을 하자

`volumeCredits` 변수를 제거하는 작업의 단계를 아주 잘게 나눴다는 점을 주목하자.

>

1. **반복문 쪼개기**: 변수 값을 누적시키는 부분을 분리한다.
2. **문장 슬라이드하기**: 변수 초기화 문장을 변수 값 누적 코드 바로 앞으로 옮긴다.
3. **함수 추출하기**: 적립 포인트 게산 부분을 별도 함수로 추출한다.
4. **변수 인라인하기**: volumeCredits 변수를 제거한다.

`totalAmount` 도 똑같은 절차로 제거한다.

1. 반복문 쪼개기

```javascript
function appleSauce() {
  let totalAmount = 0;
for (let perf of invoice.performaces) {
	totalAmount += amountFor(perf);
}
  return totalAmount;
}
```

```javascript
function statement(invoice, plays) {
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  
  for (let perf of invoice.performaces) {
    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${usd(amountFor(perf))} ($perf.audience}석)\n`;
  }
  
  let totalAmount = appleSauce();	// 함수 추출 & 임시 이름 부여
  
  result += `총액: ${usd(totalAmount)}\n`;
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;
}    
```

2. totalAmount 변수 인라인 후, 이름을 의미있게 고치기

```javascript
function statement(invoice, plays) {
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  
  for (let perf of invoice.performaces) {
    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${usd(amountFor(perf))} ($perf.audience}석)\n`;
  
  result += `총액: ${usd(totalAmount())}\n`;	// 변수 인라인 후 함수 이름 바꾸기
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;
}    
```

```javascript
function totalAmount() {	// 함수이름 바꾸기
  let totalAmount = 0;
for (let perf of invoice.performaces) {
	totalAmount += amountFor(perf);
}
  return totalAmount;
}
```

3. 추출한 함수 안에서 쓰인 이름들도 내 코딩 스타일에 맞게 변경하기

```javascript
function totalAmount() {	
  let result = 0;	// 변수 이름 바꾸기
for (let perf of invoice.performaces) {
	totalAmount += amountFor(perf);
}
  return result;
}
```

### 1.5 중간 점검: 난무하는 중첩 함수

> 리팩터링 결과를 살펴보자

```javascript
function statement(invoice, plays) {
  let result = `청구 내역(고객명): ${invocie.customer})\n`;
  
  for (let perf of invoice.performaces) {
    // 청구 내역을 출력한다.
    result += ` ${playFor(perf).name}: ${usd(amountFor(perf))} ($perf.audience}석)\n`;
  
  result += `총액: ${usd(totalAmount())}\n`;	
  result += `적립 포인트: ${totalVolumeCredits()}점\n`;
  return result;
}    
  
function totalAmount() {	
  let result = 0;	
for (let perf of invoice.performaces) {
	totalAmount += amountFor(perf);
}
  return result;
}
  
// 중첩 함수 시작
function totalVolumeCredits() {
  let voluemCredits = 0;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);
  }
  return volulmeCredits;
}
  
function usd(aNumber) {	
  return new Intl.NumberFormat("en-US", 
                 { style: "currency", currency: "USD", 
          		   minimumFractionDigits: 2 }).format(aNumber/100);
}

function volumeCreditsFor(aPerformance) { 
  let result = 0;
  volumeCredits += Math.max(aPerformance.audience - 30, 0);
  if ("comedy" === playFor(aPerformance).type) 
    volumeCredits += Math.Floor(aPerformance.audience / 5 );
  return result;
}

function playFor(aPerformance) {
  return plays[aPerformance.playID];
}

  
function amountFor(aPerformance) {	
  let result = 0;
	switch (playFor(aPerfomance).type) { 
      case "tragedy":
        result = 40000;
        if (perf.audience > 30) {
          result += 1000 * (perf.audience - 30);
        }
        break;
      case "comedy":
        result = 30000;
        if (perf.audience > 20) {
          result += 1000 + 500 * (perf.audience - 20);
        }
        result += 300 * perf.audience
        break;
      default:
        throw new Error(`알 수 없는 장르: ${playFor(aPerfomance).type}`);
      }
	  return result;
	}
}

```

최상위 `statement()` 함수는 일곱줄 뿐이며, 출력할 문장을 생성하는 일만 한다. 계산하는 로직은 모두 여러 개의 보조 함수로 빼냈다. 결과적으로 각 계산 과정은 물론 전체 흐름을 이해하기가 훨씬 쉬워졌다.
