# HTTP

### 개념
HTML 같은 하이퍼미디어 문서를 전송하기 위한 프로토콜로

웹 서버와 클라이언트간 통신을 위해 설계 되었으며,

그 사이에서 데이터 교환에 사용 되는 문법, 절차 등을 정의한 규약이 http 이다.

TCP/IP 기반으로 되어있으며 request / response 구조로 구성된다.

클라이언트가 서버에 request를 보내면 서버가 response를 보내주는 형식

### 브라우저 URL 이동 과정

브라우저에 URL을 입력했을 때 해당 페이지를 화면에 띄워주는 과정

1. handling inputs : user 가 URL창에 text 입력

	브라우저 프로세스안에 UI Thread 가 유저가 입력한 text가 search query인지 URL인지를 판단한다.

	search query : search engine 으로 query를 보낼 준비

	URL : network thread로 URL을 전달할 준비

2. start navigation 
