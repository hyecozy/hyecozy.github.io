---
layout: single
title:  "늘봄 도서관 프로젝트"
---

>울산 KH 정보 교육원 파이널 팀 프로젝트   
>By. team 얼라(Alive)(본인 포함 총 4명)   
>2022.04 ~ 2022.05 (약 1개월)   
   
### 🍀개발 환경
Java, Spring, MyBatis, JQuery, JavaScript, HTML, CSS, Oracle DB, SVN
   
### 🍀간단 소개 및 전체 기능 보기
일반적인 도서관 기능(자료 검색, 대여, 반납)과 독서를 장려하기 위한 독서 모임, 독서 기록장, 독서 생활 관련 굿즈샵을 이용할 수 있는 도서관 사이트.   
[전체 기능 PDF 보러 가기(약 100장)](https://drive.google.com/file/d/1I8J19u-Cs55Ei4LsM0jNDEiPsE_OHnTk/view?usp=sharing)   
[전체 기능 시연 영상 보러 가기(약 20분)](https://drive.google.com/file/d/1Cc_98oDKJCAMzP0hIOuD19zgCjsNEUht/view?usp=sharing)   
   
### 🍀What I did
##### ✔️주요 기능 프로세스
![Untitled Diagram drawio](https://user-images.githubusercontent.com/94097773/171994414-76f382d5-5935-4b6c-be78-a3dd15ad73ca.png)
➡️북적북적 : 독서 계획, 기록을 위한 기능입니다. 사용자 입장을 고려했을 때 추가하고 싶어진 기능이었고 이것이 늘봄 도서관의 차별화를 보여줄 것이라 생각하여 주요 부분에 넣었습니다.   
북적북적이라는 이름과 쌓아보기 UI는 북적북적이라는 앱에서 가져왔습니다.  
➡️사용자 : 도서관 홈페이지에 필수적으로 들어가야 하는 기능이고 학원에서 배운 것을 전체적으로 쓸 수 있었습니다.
##### ✔️부가 기능, 역할
1. 메인화면: 사용자 지역 날씨(OpenWeatherMap API), TO DO LIST, 쿠키 이용 팝업창(연체 도서 안내)
2. TEMPLATE LAYOUT 구성, 디자인
3. 도서관 찾아오시는 길 카카오맵 API
4. 테이블 설계 및 수정

### 🍀UI & Code
#### ✔️북적북적1(독서기록장)
![화면 캡처 2022-06-04 192733](https://user-images.githubusercontent.com/94097773/171995297-77b616e0-0bd1-4071-b74a-d47fd67d3996.png)
![화면 캡처 2022-06-04 192811](https://user-images.githubusercontent.com/94097773/171995306-4ef18822-8739-4a23-8d68-eb5130cf3475.png)
※우측 하단의 TO READ LIST는 저 위치에 픽스되어 있으므로 캡처가 2번 됐습니다.
#### ✔️북적북적2(쌓아보기 버전)
![화면 캡처 2022-06-04 193515](https://user-images.githubusercontent.com/94097773/171995493-f0454fa6-6e2c-426a-9d44-a0441ea06316.png)
