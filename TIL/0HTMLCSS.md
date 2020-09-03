# 1. HTML
 - Hypertext Markup Langauge
 - 웹페이지 DOM 구조를 태그를 이용해 표현하는 방법
 - head : html 정보를 나타내는 부분 , body : 실제 구조 로 구성
 - Semantic HTML : HTML5로 넘어오면서 태그의 의미가 중요함. 적절한 태그 사용
 ## tag
  - <h1> .. <h6> : 제목태그 : 위아래 자동으로 공백
  - <p> : 문단태그 : 위아래 자동으로 공백 생성
  - <a href=""> : 링크 태그
  - <img src="" alt=""> : 이미지 태그
  - <table> <thead> <tbody> <th> <tr> <td> : 테이블 태그
  - <ul> , <ol> : 리스트 태그
  
 ## block과 inline, inline-block
  - block : 한 줄을 통째로 차지 EX) div
  - inline : 내용물의 크기만큼 공간을 차지(padding margin width height X) EX) span
  - inline-block : inline과 block 특징을 모두. css로 설정
    - 줄바꿈X, block처럼 크기지정가능
	
 ## Semantic HTML
  - 의미있는 HTML tag 사용
  - header, footer, main, nav 등
  - header : 문서의 제목 등 메타 정보 표기
  - footer : 사이트 정보 등 부가정보 표기
  - main : 페이지의 body 부분. html 하나당 한번만 사용
  - nav : 네비게이션 바
  - section, article : 본문 영역을 나눌 때 사용.
  
 ## form
  - 서버로 정보를 보낼 때 사용
  - <form action="" method="GET"> 형식으로 사용
  - <input> : form 안에 input 태그를 이용해 정보 입력받음
    - type : text, radio, checkbox
    - placeholder : 기본적으로 표시할 정보
	- name : form 을 전송했을 때 서버에서 받는 변수(req.body.~)
	- id : label과 연결할 값
	- label : 각 input이 어떤 역할을 하는지 나타냄 > for 속성으로 input id 입력
	EX) <label for="email"><input type="email" name="email" id="email" placeholder="dd@naver.com" /></label>
	
  ## html entity
   - &nbsp : 띄어쓰기
   - &lt : <
   - &amp : &
   - &quot : "
   
  ## [기타 태그](https://www.zerocho.com/category/HTML&DOM/post/5825983aaff5c7001827996f)