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
➡️사용자가 읽는 도서 장르(카테고리) 추려내기: top 3까지 조회 / member-Mapper.xml
```
<select id="topThreeBookPlaner" resultMap="complit">
	  SELECT ROWNUM, CATE_NAME, CNT
	  FROM(SELECT
	  CATE_NAME, CNT
	  FROM (
		  SELECT COUNT(BOOK_CODE) AS CNT, CATE_CODE
		  FROM (
			  SELECT BC.BOOK_CODE, B.CATE_CODE
			  FROM BOOK B, BOOK_COMPLIT BC
		  	  WHERE B.BOOK_CODE = BC.BOOK_CODE
			  AND BC.MEM_ID = #{memId}
		  )
		  GROUP BY CATE_CODE	
	  ) B, BOOK_CATEGORY CA
	  WHERE B.CATE_CODE = CA.CATE_CODE
	  ORDER BY CNT DESC)
	  WHERE ROWNUM &lt; 4
   </select>
```
➡️사용자 취향 도서 추천: 사용자가 읽은 도서가 없을 경우, 도서관 전체 도서 중 인기 도서를 추천합니다.   
사용자가 읽은 도서가 있을 경우, 읽은 도서의 카테고리를 조회하여 나온 결과의 도서를 랜덤 추천합니다. 읽은 도서는 추천하지 않습니다. / memberController.java
```
// 독서 플래너 by 혜수 22-06-04 20:48 recommend List 조회 시 미작동 코드 정리. 추후 완독 도서가 너무 적을 때의 경우 추천 도서 조회도 생각하기.
	@GetMapping("/bookPlaner")
	public String bookPlaner(Model model, HttpSession session, BookComplitVO bookComplitVO) {
		//1.전체 데이터의 개수 조회
		bookComplitVO.setMemId(((MemberVO)(session.getAttribute("loginInfo"))).getMemId());
		int listCnt = memberService.selectBookPlanerCnt(bookComplitVO);
		bookComplitVO.setTotalCnt(listCnt);
		bookComplitVO.setDisplayCnt(5);
		//2.페이징 처리를 위한 세팅 메소드 호출
		bookComplitVO.setPageInfo();
		List<BookComplitVO> list = memberService.selectBookPlanerForPage(bookComplitVO);
    
    //완독 도서 있으면 읽은 도서의 카테고리 위주로 추천하며 읽은 도서는 제외하여 랜덤 조회함
    //완독 도서 없으면 도서관 전체 도서 중 추천수 높은 거 추천
		if(!(memberService.selectRecommendBook(bookComplitVO.getMemId())).isEmpty()) {
			model.addAttribute("rcdList", getRcdListRandom(memberService.selectRecommendBook(bookComplitVO.getMemId())));
			}
		else {
			model.addAttribute("rcdList", memberService.selectReadYet(bookComplitVO.getMemId()));
		}
		
		model.addAttribute("complitBookList", list); //독서기록 조회를 위한 리스트
		model.addAttribute("myTopThree", memberService.topThreeBookPlaner(bookComplitVO.getMemId())); //취향 카테고리 top3
		model.addAttribute("chartList", memberService.selectBookPlanerChart(bookComplitVO.getMemId())); //chart.js API 그래프
		
		return "mypage/book_planer";
	}
 ```
 member-Mapper.xml
 ```
 <!-- 내 취향 TOP3의 카테고리에서 내가 읽지 않은 책들 조회 -->
   <select id="selectRecommendBook" resultMap="complit">
	SELECT AL.BOOK_CODE, AL.TITLE, AL.WRITER
	FROM (SELECT BOOK_CODE, TITLE, WRITER
	    FROM BOOK
	    WHERE CATE_CODE 
	    IN(SELECT CATE_CODE
	        FROM (
	        SELECT ROWNUM, CATE_CODE, CATE_NAME, CNT
	        FROM(SELECT A.CATE_CODE, CATE_NAME, CNT
	            FROM (SELECT B.CATE_CODE, COUNT(C.BOOK_CODE) AS CNT
	                    FROM BOOK B, 
	                        (SELECT BOOK_CODE
	                            FROM BOOK_COMPLIT
	                            WHERE MEM_ID = #{memId}) C
	                    WHERE B.BOOK_CODE = C.BOOK_CODE
	                    GROUP BY B.CATE_CODE
	                    )A, BOOK_CATEGORY CA
	            WHERE A.CATE_CODE = CA.CATE_CODE
	            ORDER BY CNT DESC)
	        WHERE ROWNUM &lt; 4))) AL
	LEFT JOIN (
	    SELECT C.BOOK_CODE, B.TITLE, B.WRITER
	    FROM BOOK_COMPLIT C, BOOK B
	    WHERE C.BOOK_CODE = B.BOOK_CODE
	    AND C.MEM_ID = #{memId}) DONE
	ON AL.BOOK_CODE = DONE.BOOK_CODE
	WHERE DONE.BOOK_CODE IS NULL
   </select>
 ```


#### ✔️북적북적2(쌓아보기 버전)
➡️읽은 도서의 페이지 수를 UI 구현에 사용하기 위해 두께를 조절합니다. 100페이지 이하의 도서는 div가 너무 얇아질 수 있기 때문에 따로 조절했습니다.   
또 전체 회원의 독서량에 대한 상위 백분율을 조회합니다. / memberController.java
![화면 캡처 2022-06-04 193515](https://user-images.githubusercontent.com/94097773/171995493-f0454fa6-6e2c-426a-9d44-a0441ea06316.png)
```
   @GetMapping("/favChk")
	   public String favChk(HttpSession session, Model model) {
		  String memId = ((MemberVO)(session.getAttribute("loginInfo"))).getMemId();
		   
		 //북적북적 구현
		  List<BookComplitVO> list = memberService.selectBookPlaner(memId);
		  	for(int i = 0 ; i < list.size() ; i++) {
				if(list.get(i).getBookInfo().getBkPage() >= 100) { //100페이지 이상이면
					list.get(i).getBookInfo().setBkPage((int)(list.get(i).getBookInfo().getBkPage() * 0.025));
				}
				else {
					list.get(i).getBookInfo().setBkPage(2);
				}
			}
		  	model.addAttribute("highPct", memberService.selectComplitHighPct(memId));
		  	model.addAttribute("complitBookList", list);
		  	return "favor/book_planer_favorChk";
	  }
```
member-Mapper.xml
 ```
  <!-- 독서량 상위 백분율 -->
   <select id="selectComplitHighPct" resultMap="complit">
   		SELECT MEM_ID, CNT
			, TRUNC((RANK / (
			        SELECT COUNT(MEM_ID)
			        FROM BOOK_MEMBER
			        ) * 100), 0) AS HIGH_PCT
		FROM (SELECT DENSE_RANK() OVER (ORDER BY CNT DESC) AS RANK
		        , MEM_ID, CNT
		        FROM (
		            SELECT C.MEM_ID
		            , DECODE(B.CNT, NULL, 0, B.CNT) AS CNT
		        FROM (
		            SELECT MEM_ID
		            FROM BOOK_MEMBER
		            ) C LEFT OUTER JOIN (
		            SELECT MEM_ID, COUNT(COMPLIT_CODE) AS CNT
		            FROM BOOK_COMPLIT
		            GROUP BY MEM_ID
		            ) B
		        ON B.MEM_ID = C.MEM_ID
		        ORDER BY CNT DESC
		            )) 
		WHERE MEM_ID = #{memId}
		GROUP BY RANK, MEM_ID, CNT
		ORDER BY CNT DESC
   </select>
 ```
