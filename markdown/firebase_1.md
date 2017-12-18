### Firebase

Google IO 2016 에서 소개된, 웹과 모바일 환경에 필요한 기능을 제공하는 BaaS(Back-end as a Service).
기존의 경우, 백엔드 설계를 위해 서버 아키텍쳐 설계, 보안, 인증, DB 설계, API 설계 등 모든 서버의 설계 및 기능을 구현해야 했지만, firebase 를 도입하여 프론트 엔드 개발에 집중할 수 있다. 크게 아래와 같은 기능을 가지고 있다.
- backend 인증
- real time database
- Firebase Hosting

```
※ SPA(Single Page Application)

단일 페이지 웹 어플리케이션, 첫 페이지 로딩 후 화면 갱신 없이 모든 서비스가 한 화면에서 이루어지는 웹사이트. SPA로 구성할 경우, Firebase 로 백엔드로 구성한다면 프론트 엔드에 더욱 집중할 수 있다.
```
### Firebase 환경설정
- 프로젝트 생성 : [Firebase](https://firebase.google.com/) > Get Started (or Console로 이동) > 프로젝트 추가
프로젝트를 생성하면, Project main console로 이동한다. 현재 사용하고 있는 Pricing 정책, STABILITY, 분석 등 다양한 메뉴를 가지고 있다. 그리고 가장 메인에는 iOS, Android앱, 웹 앱에 대한 Firebase 추가버튼이 있는데, 웹앱을 클릭한다.

- 웹앱에 Firebase 추가 > Snippet 생성 > 복사하여 script 태그 앞에 붙여넣기

- 인증 설정 : Authentication > 로그인 방법 설정 | 이메일 템플릿(주소 인증, 비밀번호 재설정 등에 이용되는 이메일의 템플릿)

- 데이터베이스 설정 :
데이터베이스는 실시간 클라이언트 데이터베이스이며, 모든 것은 json으로 저장된다.(key-value) 연결되어 있는 모든 클라이언트에게 실시간 동기화가 되고, 오프라인시에도 캐쉬 데이터 처리하여 작동된다.
	- Database > 규칙 : 해당 데이터베이스의 읽기, 쓰기의 권한 규칙과 데이터 구조, 인덱스 생성 방법 등을 설정. 테스트할 수 있는 시뮬레이터 또한 제공된다.

	- Database > 사용량 : DB 사용량

	- Databse > 백업 : 데이터 백업 기능. blaze 요금제부터 사용 가능.

### Firebase 로컬 개발 환경 설정
- [node.js 설치](https://nodejs.org/en/)
- Firebase CLI 설치 및 log-in
	- 설치 : `$ npm install firebase-tools -g`

### Firebase Android 설정 및 환경 설정
1. Firebase Android Platform 설정
2. Anroid Project 설정
3. Authentication (/w Google)