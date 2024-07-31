# 3️⃣ 타입 추론

타입스크립트는 타입 추론을 적극적으로 수행하며, 타입 추론은 수동으로 명시해야하는 타입을 줄여 주기 때문에, 코드의 전체적인 안정성이 향상된다. (초보자의 코드는 불필요한 타입 구문으로 도배되어 있을 것이다.)

타입 추론에서 발생할 수 있는 몇 가지 문제와 그 해법에 대해 알아보자. 타입스크립트가 어떻게 타입을 추론하는지, 언제 타입 선언을 작성해야하는지, 타입 추론이 가능하더라도 명시적으로 타입 선언을 작성하는 것이 필요한 상황은 언제인지 잘 이해할 수 있을 것이다.