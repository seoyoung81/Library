# 📎 아이템 17 변경 관련된 오류 방지를 위해 readonly 사용하기

삼각수(1, 1+2, 1+2+3)를 출력하는 코드이다.

```typescript
function printTriangles(n: number) {
  const nums = [];
  for (let i = 0; i < n; i++) {
    nums.push(i);
    console.log(arraySum(nums));
  }
}
```

실행결과

```typescript
> printTriangles(5)
0
1
2
3
4
```

`arraySum`이 `nums`을 변경하지 않는다고 간주해서 문제가 발생했다. 다음과 같이 해결한다.

```typescript
function arraySum(arr: number[]) {
  let sum = 0, num;
  while ((num = arr.pop()) !== undefined) {
    sum += num;
  }
  return sum;
}
```

이 함수는 배열 안의 숫자들을 모두 합친다. 계산이 끝나면 원래 배열이 전부 비게 된다. 자바스크립트 배열은 내용을 변경할 수 있기 때문에, 타입스크립트에서도 오류 없이 통과하게 된다.

## 📍 readonly

오류의 범위를 좁히기 위해 `arraySum`이 배열을 변경하지 않는다는 선언을 해보자. `readonly` 접근 제어자를 사용하면 된다.

```typescript
function arraySum(arr: readonly number[]) {
  let sum = 0, num;
  while ((num = arr.pop()) !== undefined) {
      // 'readonly number[]' 형식에 'pop'속성이 없다.
    sum += num;
  }
  return sum;
}
```

**⭐️ `readonly number[]`는 타입이고, `number[]`와 구분되는 몇 가지 특징이 있다.**

1. 배열의 요소를 읽을 수 있지만, 쓸 수는 없다.
2. `length`를 읽을 수 있지만, 바꿀 수는 없다. (배열 변경)
3. 배열을 변경하는 `pop`을 비롯한 다른 메서드를 호출할 수 없다.

`number[]`는 `readonly number[]`보다 기능이 많기 때문에, `readonly number[]`의 서브타입이 된다. 따라서 변경 가능한 배열을 `readonly` 배열에 할당할 수 있다. 하지만 반대는 불가능하다.

```typescript
const a: number[] = [1, 2, 3];
const b: readonly number[] = a;
const c: number[] = b;
//    ~ Type 'readonly number[]' is 'readonly' and cannot be
//      assigned to the mutable type 'number[]'
// readonly number[] 타입은 readonly이므로 변경 가능한 number[] 타입에 할당될 수 없다.
```

타입 단언문 없이 `readonly` 접근제어자를 제거할 수 있다면 `readonly`는 쓸모 없으므로 오류가 발생하는게 맞다.

**⭐️ 매개변수를 readonly로 선언하면 다음과 같은 일이 생긴다.**

1. 타입스크립트는 매개변수가 함수 내에서 변경이 일어나는지 체크한다.
2. 호출하는 쪽에서는 함수가 매개변수를 변경하지 않는다는 보장을 받게 된다.
3. 호출하는 쪽에서 함수에 readonly 배열을 매개변수로 넣을 수도 있다.

자바스크립트에서는 명시적으로 언급이 없지만, 함수가 매개변수를 변경하지 않는다고 가정한다.&#x20;

→ 타입 체크에 문제를 일으킬 수 있다. (아이템30, 31) 명시적인 방법을 사용하는 것이 컴파일러와 사람에게 모두 좋다.

앞 예제를 고쳐보자. 배열을 변경하지 않으면 된다.

```typescript
function arraySum(arr: readonly number[]) {
  let sum = 0;
  for (const num of arr) {
    sum += num;
  }
  return sum;
}
```

**⭐️ 만약 함수가 매개변수를 변경하지 않는다면 readonly로 선언해야한다.**&#x20;

더 넓은 타입으로 호출할 수 있고, 의도치 않은 변경이 방지된다.

### readonly의 단점

#### 매개변수가 readonly로 선언되지 않은 함수를 호출해야 할 경우

만약 함수가 매개변수를 변경하지 않고도 제어가 가능하다면 `readonly`로 선언하면 된다. 하지만, 어던 함수를 `readonly`로 만들면, 그 함수를 호출하는 다른 함수들도 `readonly`로 만들어야 한다.&#x20;

→ 인터페이스를 명확히 하고 타입 안정성을 높일 수 있다.

→ 다른 라이브러리에 있는 함수를 호출하는 경우, 타입 선언을 바꿀 수 없으므로 타입 단언문(as number\[])을 사용해야 한다.

**⭐️ readonly를 사용하면 지역 변수와 관련된 모든 종류의 변경 오류를 방지할 수 있다.**

연속된 행을 가져와서 빈줄을 기준으로 구분되는 단락으로 나누는 기능을 하는 프로그램이다.

```typescript
function parseTaggedText(lines: string[]) : string[][] {
    const para : string[][] = [];
    const currPara : string[] = [];

    const addPara = () => {
        if (currPara.length) {
            para.push(currPara)  // 오류발생지점 1 (배열의 참조가 삽입)
            currPara.length = 0; // 오류발생지점 2 (배열을 비움)
        }
    }

    for (const line of lines) {
        if (!line) {
            addPara();
        } else {
            currPara.push(line);
        }
    }
    addPara();
    return para;
}

// output : [[],[],[]]
```

이 코드의 문제점은 별칭과 변경을 동시에 사용하여 발생했다.

`currPara`의 내용이 삽입되지 않고 배열의 참조가 삽입되었다. 따라서 `currPara` 의 길이가 0이 되면 `para`로 들어간 `currPara`의 길이 또한 0이 되어 빈 배열로 반환되었다. 따라서 이 코드는 새 단락을 `para`에 집어넣고 바로 지워 버린다.

`currPara`를 `readonly`로 선언하여 오류를 해결한다.

```typescript
function parseTaggedText(lines: string[]) : string[][] {
    const para : string[][] = [];
    const currPara : readonly string[] = [];

    const addPara = () => {
        if (currPara.length) {
            para.push(currPara)
          	// 1. readonly string[]속성의 인수는 string[]에 할당할 수 없습니다.
            currPara.length = 0;
          	// 2. 읽기 전용 속성으로 'length'에 할당할 수 없습니다.
        }
    }

    for (const line of lines) {
        if (!line) {
            addPara();
        } else {
            currPara.push(line);
          	// 3. readonly string[] 형식에 push 속성이 없습니다.
        }
    }
    addPara();
    return para;
}
```

`currPara`를 `let`으로 선언하고 변환이 없는 메서드를 사용함으로써 두 개의 오류를 고칠 수 있다.

```typescript
let currPara: readonly string[] = [];
// ...
currPara = [];    // 배열을 비움
// ...
currPara = currPara.concat((line));
```

`push`와 달리 `concat`은 원본을 수정하지 않고(3번 오류 해결), 새 배열을 반환한다. 선언부를 `const`에서 `let`으로 변경 후 (2번 오류 해결)`readonly`를 추가하여 한쪽의 변경 가능성을 또 다른 쪽으로 옮긴 것 이다.

`currPara` 변수는 가리키는 배열을 자유롭게 변경할 수 있지만, 그 배열 자체는 변경하지 못하게 된다.

⭐️ 1번 오류를 해결하는 3가지 방법

1. currPara의 복사본을 만든다.

```typescript
paragraphs.push([...currPara]);
```

currPara는 readonly로 유지되지만, 복사본은 원하는대로 변경이 가능하므로 오류가 사라진다.

2. `paragraphs`를 `readonly string[]`의 배열로 변경한다.

```typescript
const paragraphs: (readonly string[])[] = [];
```

3. 배열의 `readonly` 속성을 제거하기 위해 단언문을 사용한다.

```typescript
paragraphs.push(currPara as string[]);
```



**⭐️ readonly는 얕게 동작한다는 것에 유의해야 한다.**

`readonly string[][]` 에서 만약 객체의 `readonly` 배열이 있다면, 그 객체 자체는 `readonly`가 아니다.

```typescript
const dates: readonly Date[] = [new Date()];
dates.push(new Date()); // 'readonly Date[]' 형식에 'push' 속성이 없습니다.
dates[0].setFullyear(2000); // OK
```

`Readonly` 제너릭에도 해당된다.

```typescript
interface Outer {
    inner: {
        x: number
    }
}

const o: Readonly<Outer> = {inner: {x:0}}
o.inner = {x:1}; // 읽기 전용 속성이기 때문에 'inner'에 할당할 수 없습니다.
o.inner.x = 1;

// 타입 별칭을 만들면 어떻게 동작했는 지 알 수 있다.
type T = Readonly<Outer>
// Type T = {
//     readonly inner: {
//         x:number;
//     }
// }
```

중요한 점은 `readonly` 접근제어자는 inner에 적용되는 것이지 x는 아니라는 것이다. 제너릭을 만들면 깊은 `readonly` 타입을 사용할 수 있다. 그러나 제너릭은 만들기 까다로워 라이브러리를 사용하는 것이 낫다. (ts-essentials에 있는 DeepReadonly를 사용)

인덱스 시그니처에도 `readonly`를 사용할 수 있다. 읽기는 허용하되 쓰기를 방지하는 효과가 있다.

## 📍 요약

* 만약 함수가 매개변수를 수정하지 않는다면, readonly로 선언하는 것이 좋다. readonly 매개변수는 인터페이스를 명확하게 하며, 매개변수가 변경되는 것을 방지한다.
* readonly를 사용하면 변경하면서 발생하는 오류를 방지할 수 있고, 변경이 발생하는 코드도 쉽게 찾을 수 있다.
* const와 readonly의 차이를 이해해야 한다.
* readonly는 얕게 동작한다.
