## 💻 프로젝트 환경

- Java 17
- SpringBoot 2.7.7
- H2
- Spring Data JPA
- Gradle

---

## 👩‍ 구현 내용

### 기능

- [x] URL 입력폼 제공
- [x] 상세 결과 페이지(원래 URL, 단축된 URL, 요청 수) 제공
- [x] 잘못된 URL 을 단축 요청 시 사용자에게 오류 표시
- [x] 단축된 URL 은 7 Character 로 생성
- [x] 단축된 URL 요청시 원래 URL 로 리다이렉트

---

### API 문서

#### 1. URL 입력폼 화면 제공

| 메서드 | URL
:---: | :---: |
GET | /

---

#### 2. 단축된 URL 생성

| 메서드 | URL
:---: | :---:
POST | /shortener

| Response Parameter | 타입 | 설명
:---: | :---: | :---:
originUrl | String | 원래 URL
shortUrl | String | 단축된 URL
requestCount | Number | 요청 수

---

#### 3. 생성된 상세 결과 페이지 제공 (원래 URL, 단축된 URL, 요청 수)

| 메서드 | URL
:---: | :---: 
GET | /shortener

| Request Parameter | 타입 | 설명
:---: | :---: | :---:
originUrl | String | 원래 URL
shortUrl | String | 단축된 URL
requestCount | Number | 요청 수

---

#### 4. 단축된 URL 로 요청 시 원래 URL 로 리다이렉트

| 메서드 | URL
:---: | :---:
GET | /{shortUrl}

| Path Parameter | 타입 | 설명
:---: | :---: | :---:
shortUrl | String | 단축된 URL

---

### DB 테이블

| 컬럼 명 | 타입 | NULL 가능 | 키 | 설명
:---: | :---: | :---: | :---: | :---:
id | bigint | x | pk | Auto increment 로 생성한 ID
algorithm | varchar(20) | x | uk| URL 단축에 사용한 알고리즘
origin_url | varchar(2000) | x | uk | 프로토콜 정보를 제외한 원래 URL
short_url | varchar(7) | o | - | 단축된 URL
request_count | bigint | x | - | 요청 수

---

### 동작 화면

#### URL 입력폼 화면

![image](https://user-images.githubusercontent.com/68289543/210596993-25fbb2b4-a38b-4087-b5f1-415c6a27cde9.png)

---

#### URL 입력폼 화면 - 단축된 URL 생성 요청

![image](https://user-images.githubusercontent.com/68289543/210597113-3699b954-aacf-462f-a0c8-2142e7b50a5c.png)

---

#### 단축된 URL 생성 요청 후 상세 결과 페이지

![image](https://user-images.githubusercontent.com/68289543/210597413-b539d311-b3ca-4b4c-b58e-60850e78bf13.png)

---

#### 상세 결과 페이지 - 단축된 URL 클릭 시 원본 URL 로 이동

![gif-1](https://user-images.githubusercontent.com/68289543/210597475-b834e773-a70b-4e09-8223-6dab364fed13.gif)

---

#### 이미 DB에 존재하는 URL 인 경우 요청 수 증가

![git-2](https://user-images.githubusercontent.com/68289543/210597554-3629dc02-82dd-4671-9f61-7579abe84338.gif)

---

#### 잘못된 URL 을 입력하는 경우 사용자에게 오류 표시

![GIF-3](https://user-images.githubusercontent.com/68289543/210597608-88e5503d-7054-4205-b488-5294c24675c3.gif)

---

#### DB 에 없는 단축된 URL 요청을 한 경우 에러 페이지 이동

![gif-4](https://user-images.githubusercontent.com/68289543/210597617-666ca514-20e2-4d31-b0ec-798750d9aad8.gif)

---

## ✅ 고민
- URL 단축 알고리즘 어떤 방식으로 구현할 것인가?
  - 단축된 URL 을 7글자로 하는 것이 요구사항이었다. 왜 그런지 조사를 해봤더니 사람이 짧다고 느끼는 길이는 7글자 내외라는 [밀러의 법칙](https://ko.wikipedia.org/wiki/%EB%A7%88%EB%B2%95%EC%9D%98_%EC%88%98_%EC%B9%A0,_%EB%8D%94%ED%95%98%EA%B1%B0%EB%82%98_%EB%B9%BC%EA%B8%B0_%EC%9D%B4)이라는 것이 있었다. 이 법칙을 고려한 것인지 기존의 [bitly](https://bitly.com/), [shorturl](https://www.shorturl.at/) 서비스도 모두 7글자 이내로 단축된 URL 을 제공한다.
  - 원래 URL 에 대응되는 식별자로 UUID 를 생각해보았지만 7글자 이내라는 조건때문에 불가능했다. 그래서, DB 의 auto increment 를 이용하였다.
    - 7자리 이내이기 때문에 9,999,999 까지 밖에 이용하지 못한다. 서비스가 커진다고 생각했을 때 충분하지 않은 값이다. 10진법 말고 64진법인 Base64 를 고려하게 되었다. 다만, Base64 에서 사용되는 +, /, = 가 URL 형식에서는 사용할 수 없기에 알파벳과 숫자만으로 이루어진 Base62 를 선택하였다. 62진법을 이용하기에 최대 3,521,614,606,207 개의 URL 을 저장할 수 있다.
    - 현재는 identity 전략으로 생성된 id 값을 이용하여서 인코딩한다. id 값을 인코딩하여서 id 를 노출시키는게 옳은지에 대한 고민이 생겼다. 현재 데이터 상에서는 노출되서는 안되는 값이 없기에 id 를 노출해도 상관 없다고 판단하였다. 만약 문제가 된다면 시퀀스를 하나 만들어서 그것을 이용하면 될 것 같다.
    - auto increment 로 생성된 값을 인코딩하기에 단축되는 URL 의 길이가 고정되지 않는다. 하나의 표준처럼 형태를 고정해주는 것은 중요하기에 패딩을 하여 저장하기로 했다. 62진법에서 앞에 A가 붙는 일은 없으므로 부족한만큼 A를 붙이는 방식으로 패딩하였다.

- DB 에 저장하는 데이터 형태를 어떻게 할 것인가? 
  - 원래 URL
    - 사용자가 입력한 URL 에서 프로토콜이 다르더라도 나머지가 같다면 동일한 데이터로 판단하고 처리해야 중복된 URL 이  DB 에 저장되지 않는다. 또, DB 에 저장할 때 데이터의 용량은 작을 수록 좋다. 그러므로 URL 에서 프로토콜 정보를 제거하고 저장하기로 하였다.
    - naver.com 과 www.naver.com 은 같은 화면을 띄워준다. 이런 경우는 하나만 저장해야할까? 둘 다 저장하기로 했다. 엄밀히 말하면 naver.com 과 www.naver.com 은 동일한 URL 이 아니다. www.naver.com 은 naver.com 의 하위 도메인으로 네이버 측에서 사용자들의 편의를 위해서 www.naver.com 을 입력해도 naver.com 으로 갈 수 있게 지정해놓은 것이다. 대부분 사이트의 경우 www.주소.com 은 주소.com 을 향하게 지정되어 있지만, 아닌 경우가 있을 수 있으므로 동일하게 저장하지 않기로 하였다.
  - 단축된 URL
    - 원래 URL 을 인코딩한 값이 AAAAAAB 라고 했을 때, 이 값으로 원래 URL 로 가려면 shortURL.com/AAAAAAB 처럼 우리 서비스의 도메인을 붙여줘야한다. 서비스의 도메인 명은 변경될 수도 있으르로 DB 에 저장하는 것은 좋지 않다고 보고 인코딩한 값만을 저장해주기로 하였다.
  - 요청 수
    - 디폴트 값을 정해주는 것을 @Column 의 columnDefinition 속성으로 넣어주어서 DB 레벨에서 넣을지, 아니면 코드 레벨에서 생성자에 넣어줄지가 고민이었다. jpa 의 auto ddl 옵션을 쓰지 않는 경우도 있으니 코드 레벨에서 생성자에 디폴트 값을 넣어주는 방식을 이용했다. 또, 특정 DB 에 종속적으로 코드를 작성하기보다는 어플리케이션 레벨에서 디폴트 값을 넣어주고 DB 레벨에도 넣어주고 싶다면 DB 에서 테이블을 만들 때 넣어주는 것이 좋다고 판단했다.

- URL 파싱을 어디서 할 것인가? 
  - 사용자에게 입력 받은 URL 에서 프로토콜 정보를 제거하는 경우
    - URL 관련 정보를 변경하는 로직이기에 엔티티의 책임이라고 판단하였다. 
  - DB 조회한 단축된 URL 을 사용자가 사용할 수 있는 형태로 파싱하는 경우 
    - 우리 서비스 도메인을 붙여주는 작업이 필요했다. 로직의 위치를 서비스, 컨트롤러, 프론트 중 선택해야했다. 도메인 정보는 변경될 수 있는 정보이므로 백엔드 단에서 붙여주기보다는 프론트에서 붙여주는 것이 옳다고 생각하여 프론트의 책임으로 넘겼다.
    
- 잘못된 URL 어떻게 검증할 것인가? 
  - 이미 올바른 URL 형식인지를 검증해주는 @URL 이라는 어노테이션이 있지만, @URL 은 URL 형식만 검증해준다. URL 이 실제로 존재하는 도메인이라 접속이 가능한지를 검증해주지는 못한다. 생각해낸 방법은 입력된 URL 로 요청을 보내보고 성공할 때만 검증을 통과하게 하는 것이다. 
  - 입력된 URL 로 요청을 보내고 검증하는 코드가 길고 가독성이 떨어졌다. @URL 처럼 간편하게 검증하고 싶어서 @UrlValid 라는 검증 어노테이션을 만들었다. 
  
- 단축된 URL 생성 후 상세 페이지로 리다이렉트 시 어떤 식으로 정보를 넘길까?
  - 두 가지 방법을 생각했다. 첫번째는 RedirectAttributes 객체를 이용하는데 addAttributes 메서드를 이용하여서 param 으로 넘기는 방법이고, 두번째는 addFlashAttributes 메서드를 사용하여서 FlashMap 을 이용하여서 정보를 넘기는 방식이다. 리다이렉트 전에 세션에 잠깐 저장했다가 FlashMap 을 이용해서 데이터를 넘기고 새로고침을 되면 데이터가 날아가버리는 방식이다. 두번째 방식을 이용하기로 결정했다. 현재는 URL 에 노출되어도 문제가 되는 데이터가 없지만 URL 에 노출이 된다는 사실 자체가 불편했다.
    - 두번째 방식은 크게 두 가지 문제가 있을 것 같다. 첫번째는 서버가 여러 대인 경우 세션을 공유할 수 있도록 설정해줘야한다는 점, 두번째는 다른 도메인으로 리다이렉트될 시 세션을 공유할 수 없기에 사용할 수 없다는 점이다. 두 가지 문제 모두 현재 해당되지 않으므로 두번째 방식으로 결정했다.
---
## ✅ 개선점
- 로그를 찍어주지 않은 점이다. 프로그램 유지보수를 위해 로그는 중요하다. 출력할 로그의 형태, 로그 레벨을 고민해서 로그를 추가하자.
- URL 을 파싱해야하는 경우가 많았는데, 스프링에서 제공하는 컨버터, 포맷터를 이용해보자. 더 좋은 방식이 될 수도 있다.
---