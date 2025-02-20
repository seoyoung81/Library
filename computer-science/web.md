# 1️⃣ 웹(Web)

## 1. 웹 브라우저의 동작 방식

#### 🧷 다양한 웹 브라우저 Web Browser

* 웹 사이트에 접속할 때는 웹 브라우저 프로그램을 사용한다.
* 웹 브라우저의 기능
  * 웹 페이지를 서버에 요청 request하여 서버에 응답 response을 웹 문서 형태로 받기
  * 받은 웹 문서(HTML, CSS 등)을 렌더링 하여 모니터 화면에 웹 페이지를 표시
  *    Chrome, Firefox, Safari

#### 🧷 서버(Server)와 클라이언트(Client)

* 클라이언트가 요청을 보내면, 서버가 응답한다.
* 서버와 클라이언트 구조의 대표적인 예시가 웹 서비스
* 클라이언트는 일종의 고객으로,
* 서버로 요청을 보낸 뒤에 응답이 도착할 때까지 기다린다.
* 서버로부터 응답을 받으면, 서버의 응답을 처리하여 화면에 출력한다.

<figure><img src="../.gitbook/assets/image (18).png" alt="" width="375"><figcaption></figcaption></figure>

* 서버는 클라이언드로부터 받은 요청을 처리해 응답을 전송한다.
*   게임 서버, 모바일 서버, **웹 서버** 등

    <figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt="" width="375"><figcaption></figcaption></figure>
* 서버가 클라이언트에게 어떠한 파일을 보내주는 것인가?

#### 🧷 HTML(Hypertext Markup Language)

* HTML은 웹 문서를 작성하기 위해 사용하는 프로그래밍 언어
* 마크업(Markup): 웹 문서가 모니터 화면에서 보이는 형태를 결정하는 구조
* HTML 문서: `<HTML>` 태그로 시작, `</HTML>` 태그로 종료

#### 🧷 HTTP(Hypertext Transfer Protocol)

* 하이퍼텍스트를 전송하기 위해 개발된 프로토콜로 간편히 데이터를 전송하게 해준다.
* 웹 브라우저의 주소 표시줄에 URL(Uniform Resource Locator)을 입력한 뒤에 접속을 시도한다.
* URL은 인터넷에 존재하는 특정한 정보 자원의 종류와 위치를 나타내는 문자열
* http(protocol)://dowellcomputer.com(인터넷 주소)

#### 🧷 웹 (Web)

* HTTP 프로토콜을 이용해 수없이 많은 페이지를 링크를 타고 이동
* 웹에서는 많은 페이지가 거미줄 같은 형태를 갖는다.

#### 🧷 웹  브라우저의 구조

<figure><img src="../.gitbook/assets/image (2) (1).png" alt="" width="375"><figcaption></figcaption></figure>

#### 🧷 웹  브라우저의  동작 방식

* 웹 클라이언트는 웹 브라우저를 이용한다.
* 웹 브라우저에 주소를 입력하면 GET 방식으로 서버에 웹 문서를 요청한다.
* 웹 서버는 적절한 웹 문서를 찾아서 응답한다.
* 이후에 웹 브라우저는 문서를 화면에 표시한다.

1. 주소 입력 (http://www.gitbook.com) 후 엔터 (웹 클라이언트)
2. 웹 클라이언트가 웹 서버에 a.html 페이지 요청
3. 웹 서버는 a.html 해당 문서를 검색
4. a.html 페이지 응답
5. 웹 클라이언트가 a.html 화면 출력&#x20;

## 2. 쿠키(Cookie)와 세션(Session)

### 🔗 쿠키

* 사용자가 특정한 웹 사이트에 방문할 때, 사용자 컴퓨터에 저장하는 기록 파일
* 서버의 자원을 전혀 사용하지 않는다.
* 사용 예시: "아이디와 비밀번호를 저장하시겠습니까?"

### 🔗 세션

* 한 명의 사용자(브라우저)의 상태를 유지하는 기술
* 서버가 클라이언트에 고유한 Session ID를 부여하면, 클라이언트는 접속할 때마다 Session ID와 함께 요청
* 사용 예시: 웹 사이트에 한 번 로그인 하면, 다른 페이지로 이동해도 계속 접속 상태가 유지
* 만약 Session ID를 다른 클라이언트에게 탈취당하면, 다른 사람이 자신의 행세 가능

**세션 개요**

*  서버에서 가지고 있는 객체로, 특정 사용자의 로그인 정보를 유지하기 위해 사용할 수 있다.
* 웹 사이트에 로그인 한 뒤, 서버에서는 세션 ID에 따른 회원 ID 정보를 기록한다.
* 클라이언트는 해당 세션을 계속 유지한다. (메일함에 접속할 때도 세션 ID를 서버에 전송한다.)
*   세션은 자신이 누구인지 서버에 알려주는 역할

    <figure><img src="../.gitbook/assets/image (4).png" alt="" width="375"><figcaption></figcaption></figure>

**세션 인증 방식 예시**

1. 로그인 페이지 요청
2. 로그인 페이지 응답
3. 로그인 요청 (ID, Password)
4. 세션 생성 및 유지
5. 세션 ID (Session) 응답
6. 브라우저에 세션 정보 저장
7. 세션 ID와 함께 요청(글쓰기 등) 수행
8. 세션 ID를 통해 회원 ID 접근
9. 해당하는 회원에 대하여 서비스(글쓰기 처리) 수행

<figure><img src="../.gitbook/assets/image (5).png" alt="" width="375"><figcaption></figcaption></figure>

**세션 방식의 특징**

⭐️ 장점

* 클라이언트에게는 세션 ID(회원 식별 목적)을 제공하고, 회원에 대한 중요한 정보를 서버가 가지고 있다.
* 민감한 데이터를 클라이언트에게 직접 보내지 않는다.
* 클라이언트 브라우저가 가지고 있는 세션 ID 자체에는 개인정보를 포함하고 있지 않다.

⭐️ 단점

* 악의적인 공격자가 세션 ID를 탈취하여 사용자인 척 위장할 수 있다.
* 웹 서버에 세션 정보를 기록하고 있어야 하므로, 접속자가 많을 때 서버에 메모리 부하가 존재할 수 있다.

## 3. HTTP

* HyperText Transfer Protocol, 웹상에서 데이터를 주고받기 위한 프로토콜
* 웹 문서를 주고받기 위하여 사용
* 웹, 모바일 앱, 게임

### 🔗 HTTP 메서드 (Method)

* 클라이언트는 요청(request)의 목적에 따라 적절한 HTTP 메서드를 사용

| HTTP 메서드 | 설명          | 사용 예시            |
| -------- | ----------- | ---------------- |
| GET      | 데이터 조회를 요청  | 특정 페이지 접속, 정보 검색 |
| POST     | 데이터 생성을요청   | 회원가입, 글쓰기        |
| PUT      | 데이터 수정을 요청  | 회원 정보 수정         |
| DELETE   | 데이터 삭제를 요청  | 회원 정보 삭제         |

**HTTP 메서드 사용 예시**

* 특정한 웹 사이트에 접속하면, 기본적으로 GET 방식으로 호출
* 상태 코드 (status code)를 이용해 본인의 요청에 대한 결과를 응답 받을 수 있다.
* 웹 사이트는 HTML, JavaScript, CSS 코드를 반환하여 웹 브라우저는 이를 화면에 출력한다.

### 🔗 HTTP 상태관리와 세션

* HTTP는 상태를 저장하지 않는다. Stateless
* 클라이언트는 HTTP로 서버에 연결한 뒤에, 응답을 받으면 연결을 끊어버린다.
  * 서버 입장에서 접속 유지에 대한 요구가 적어, 불특정 다수를 대상으로 하는 서비스에 적합
* 예시로, 상품확인 → 장바구니 → 결제의 과정이 시스템적으로 상태 정보로 기록되지 않는다.
* 하지만, 세션을 이용해 원하는 기능이 수행되도록 한다.

### 🔗 Keep Alive 기능

* 무상태성으로 인한 문제점을 위해 고안된 기능
* 하나의 웹 사이트에 방문하면 대게 수십 개의 파일(CSS, Image, HTML, JS)를 제공한다.
* TCP 통신 과정에서 연결 수행/연결 해제 과정에서 리소스가 많이 소요된다.
* keep-alive는 이런 파일을 하나씩 받기 위하여 매번 연결을 맺고 끊는 것을 방지한다.
* HTTP 1.1 버전부터 지원



## 4. REST API

### 🔗 REST(Representational State Transfer) 등장 배경

* HTTP는 다양한 메서드(GET, POST, PUT, DELETE)를 지원
* 실제로, HTTP 메서드를 기존 설명에 맞게 사용하지 않더라도 프로그램 개발은 가능하지만 개발자 사이의 소통에 문제가 발생한다.
* 기준이 되는 아키텍처로 REST를 채택

**REST 이해하기**

특정한 자원에 대하여, 자원의 상태에 대한 정보를 주고받는 개발 방식

* 자원: URL를 이용
* 행위: HTTP 메서드를 이용
* 표현: 페이로드를 이용

ex. 회원가입

<figure><img src="../.gitbook/assets/image (1).png" alt="" width="188"><figcaption></figcaption></figure>

### 🔗 REST API

* API(Application Programming interface): 프로그램이 상호작용하기 위한 인터페이스
* REST API: REST 아키텍처를 따르는 API
* REST API 호출: REST 방식을 따르고 있는 서버에 특정한 요청(request)를 전송하는 행위

**REST API 연습하기**

* 목킹(moking): 어떠한 기능이 있는 것처럼 흉내내어 구현한 것
* 클라이언트 개발을 위해 간단히 서버 기능을 테스트할 때 사용
  *    DB 연동아 안되지만, 일단 기능이 있는 것처럼 사용



## 5. OAuth

* Google 로그인 기능을 떠올리자.
* 웹 서버에 Google 비밀번호를 제공하지 않고도, Google 계정의 일부 접근 궈한을 부여할 수 있다.
* SNS 간편 로그인 기능

#### **안전하지 않은 인증 방식 예시**

1. Google 로그인 요청, 데이터: Google 계정 정보가 서비스로 넘어간다.
2. 사용자의 Google 계정 로그인
3. 사용자 정보 얻기 (서비스)
4. 로그인 성공 응답 (사용자)

→ 사용자가 서버에게 Google 아이디와 패스워드를 알려줘야 한다.

#### **Acess Token 이용하기**

* 사용자가 설정한 권한에 대해서만 Google 정보에 접근할 수 있도록 한다.

1. 사용자에 대한 Acess Token 주기 (서비스 → 구글)
2. 사용자 정보 얻기 (구글 → 서비스)

### 🔗 OAuth 2.0 구성 요소

* Resource Owner: 특정한 서비스를 사용하려고 하는 사용자로, 개인정보(Resource)의 소유자
* Client: 특정한 개인 혹은 회사가 만든 서비스를 의미한다. 웹/앱 서버를 의미하지만, Client라고 부른다. → Google 입장에서는 Client
* Resource Server: 사용자의 개인정보를 가지고 있는 서버(Google, 카카오톡 등), Clinet는 Access Token을 Resource Server에 보내서 사용자의 개인정보를 얻는다.
* Authorization Server: 실질적으로 권한 부여 기능을 담당하는 서버. 사용자는 자신의 SNS 계정 정보를 넘겨 Authorization Code를 받는다. Client는 사용자로부터 받는 Authorization Code를 넘겨 Access Token을 받는다.

<figure><img src="../.gitbook/assets/image (1) (1).png" alt="" width="375"><figcaption></figcaption></figure>

## 6. JWT(JSON Web Token)

### 🔗 JSON 형식

* JavaScript Object Notation은 데이터를 주고받기 위해 사용하는 경량의 데이터 형식 중 하나
* 키(key)와 값(value)의 쌍으로 이루어진 데이터 객체를 사용

```json
{
    "id": "koko",
    "password" : "1234",
    "age" : 30,
    "job" : [
        "programmer",
        "dancer"
    ]
}
```

### 🔗 Token 인증 방식

1. 로그인 페이지 요청
2. 로그인 페이지 응답
3. 로그인 요청(ID, Password)
4. 토큰 생성
5. 토큰 응답
6. 토큰 저장
7. (나중에) 토큰과 함께 정보 요청
8.  토큰 검증 및 응답

    <figure><img src="../.gitbook/assets/image (2).png" alt="" width="188"><figcaption></figcaption></figure>

* 토큰에 요청한 사람의 정보가 포함되며, 서버는 DB를 조회하지 않고 토큰을 검증할 수 있다.
* 서버 내부에서는 비밀키(key) 하나만 가지고 있으면, 토큰 검증을 수행할 수 있다.

### 🔗 JWT (JSON Web Token)

* JWT는 인증에 필요한 정보를 암호화한 JSON 형식의 토큰
* JWT 토큰을 HTTP 헤더에 실어 서버가 클라이언트를 식별할 수 있도록 한다.
* 사용자가 인증을 수행하면, 서버는 다음의 정보를 가진 JWT 토큰을 발급한다.
*   구성 요소

    * Header:  사용할 해시 알고리즘 등의 메타 정보를 포함
    * Payload: 키(key)와 값(value) 형식으로 이루어진 정보(claim)의 구성 → 이 값을 서버로 전달
    * Signature: (헤더 + 페이로드 + 키(key)) 정보를 해싱하여 Client에게 함께 전달한다.

    `xxxxxx[Header].yyyyy[Payload].zzzzz[Signature]`
* ex.
  * Header: {"alg": "HS256", "typ": "JWT"}
  * Payload: {"sub": "user", "id": "admin"}
  * Signature: 위 두 내용에 대하여 적절한 서버 키(key) 값을 더해 해싱한 값

### 🔗 JWT를 이용한 인증

* 나중에 사용자(client)는 자신이 받았던 JWT 토큰을 다시 서버에 전달한다.
* 서버는 (헤더 + 페이로드 + 서버 내에 있는 키(Key))를 해싱한 값이 사용자로부터 전달받은 것과 일치하는지 체크한다.
* 이 과정에서 서버가 가지고 있는 비밀키(secret key)를 사용한다.

#### JWT 인증 원리

* 사용자는 서버가 처음에 부여했던 궈한만큼의 작업을 요청할 수 있다.
* 데이터를 변경하면 해시 값이 변경 → 악의적인 공격자가 Payload를 수정하는 것이 불가능
  * Payload 값이 변경되면, 서버의 키를 모르므로 서명 값이 일치하지 않아 서버가 위조 여부를 알 수 있다.

### 🔗 JWT 방식의 특징

#### 장점

* ﻿﻿세션 기반 인증 방식에 비해 서버(Server)가 DB에 세션 정보를 가지고 있을 필요가 없다.
* ﻿﻿각 해시 값이 어떤 Header와 Payload를 가지는지 일일이 서버 DB에서 저장할 이유가 없다.
* ﻿﻿서버에서 상태 정보를 저장하지 않아도 되므로, 무상태성(stateless)이 유지된다.
* ﻿﻿토큰 기반이므로, 서로 다른 웹 서버에 대해서도 동작할 수 있다. (웹 브라우저의 쿠키와 다른 점)

#### 단점

* ﻿﻿세션에 비하여 토큰 자체의 데이터 길이가 길다.
* ﻿﻿페이로드(payload)는 암호화되지 않으므로, 중요한 정보를 담기 적절하지 않을 수 있다.
* ﻿﻿토큰을 탈취당하는 경우 보안상의 문제가 발생할 수 있다. (때문에 토큰에 사용 기한을 부여한다.)

#### JWT 토큰의 유의사항

* Payload 자체는 중간자 공격에 의해 노출 될 수 있으므로, 페이로드에는 민감 정보를 넣지 않는다.
* 기본적으로 JWT의 목적의 정보 보호보다는
  * 위조 방지
  * 서버의 메모리 가용 이점 (DB 조회 필요 없음)

**실제 인증 방식**

1. ID와 Password를 이용해 로그인
2. 토큰 발급
3. Access Token을 이용해 API 요청
4. Access Token에 문제가 없다면 데이터 응답
5. 추후 Refresh Token으로 Access Token 재요청
6. Access Token 재발급
