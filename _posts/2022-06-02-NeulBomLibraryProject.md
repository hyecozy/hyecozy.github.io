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

### 🍀기능 
#### 🟢북적북적1(독서기록장)
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


#### 🟢북적북적2(쌓아보기 버전)
➡️새 창으로 열립니다.   
읽은 도서의 페이지 수를 UI 구현에 사용하기 위해 두께를 조절합니다. 100페이지 이하의 도서는 div가 너무 얇아질 수 있기 때문에 따로 조절했습니다.   
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

#### 🟢회원1 (회원가입)
![화면 캡처 2022-06-05 204824](https://user-images.githubusercontent.com/94097773/172049042-f07ec7ff-6654-450e-a15d-903b2640c9f4.png)   
➡️유효성 검사를 합니다. (JQuery validate)   
전체: DB에서 NOT NULL로 설정돼 있는 데이터는 필수 입력값으로 받습니다.   
프로필 사진의 경우, 따로 설정하지 않으면 MemberController에서 기본 프로필 이미지로 자동 저장됩니다.   
아이디: 중복 확인 버튼을 클릭하여 기존 아이디와 겹치지 않는지 확인하며, 길이의 제한이 있습니다.
비밀번호: 비밀번호 확인으로 사용자가 정확한 비밀번호를 입력했는지 확인합니다.   
전화번호: 3-4자의 숫자를 입력했는지 확인합니다.   
➡️주소 입력 시, 다음 우편번호 API를 사용합니다.   
   
join_form.js   
아이디 
```
/*아이디 중복 체크 */
function checkId(){
const newId = document.getElementById('inputId1').value;
$.ajax({
		url: '/member/checkId', //요청경로
		type: 'post',
		data: {'memId':newId}, //필요한 데이터 '데이터이름':값
		success: function(result) {
			if(result === 1){
				$('.id-unavailable').css("display", "inline-block");
				$('.id-available').css("display", "none");
			}
			else if(result === 0) {
				$('.id-available').css("display", "inline-block");
				$('.id-unavailable').css("display", "none");
			}
			
		},
		error: function() {
			
			alert('실패');
		}
	});
}
```

```
/*유효성 검사*/
$('#joinForm').validate({
	debug: false,
	groups:{
		username1: 'memTell1 memTell2',
		username2: 'memEmail1 memEmail2'
	},
	rules: {
		memId: {
		required: true,
		minlength: 5,
		maxlength: 12
		},
		memPwdCheck: { 
		required: true,
        equalTo: '#inputPwd'
         },
		memName: {
		required: true
		},
		memBirth: {
		required: true
		},
		memGender:{
		required: true
		},
		memTell1: {
		digits: true,
		required: true,
		minlength: 3,
		maxlength: 4
		},
		memTell2:{
		digits: true,
		required: true,
		minlength: 4,
		maxlength: 4
		},
		memEmail1:{
		required: true
		},
		memEmail2:{
		required: true
		},
		memAddr:{
		required: true
		}
      },
	messages: {
  	  memId: {
			required: '필수 입력 항목입니다.',
            minlength: '5자 이상 입력해 주셔야 해요.',           
            maxlength: '12자 이하로 입력해 주셔야 해요.'            
         },
	memPwdCheck: {
		required: '필수입력 항목입니다.',
		equalTo: '위에 입력하신 비밀번호랑 일치하지 않아요😥'
		},
	memName:{
		required: '필수입력 항목입니다.',
		},
	memBirth: {
		required: '필수입력 항목입니다.'
		},
	memGender:{
		required: '필수입력 항목입니다.'
		},
	memTell1: {
		digits: '올바른 전화번호 표기 형식이 아닙니다.',
		required: '필수입력 항목입니다.',
		minlength: '3~4자리의 숫자를 입력해 주세요.',
		maxlength: '4자리의 숫자를 입력해 주세요.'
		},
	memTell2: {
		digits: '올바른 전화번호 표기 형식이 아닙니다.',
		required: '필수입력 항목입니다.',
		minlength: '3~4자리의 숫자를 입력해 주세요.',
		maxlength: '4자리의 숫자를 입력해 주세요.'
		},
	memEmail1: {
		required: '필수입력 항목입니다.'
		},
	memEmail2: {
		required: '필수입력 항목입니다.'
		},
	memAddr:{
		required: '필수입력 항목입니다.'
		}
      },
	errorElement:'div',
	errorPlacement: function(error,element){
		if($(element).attr('id') == 'inputTell1' || $(element).attr('id') == 'inputTell2'){
			error.insertAfter($('#joinTell'));
		}
		else if($(element).attr('id') == 'inputEmail1' || $(element).attr('id') == 'inputEmail2'){
			error.insertAfter($('#joinEmail'));
		}
		else{
			error.insertAfter(element);
		}
		error.css('color', 'red');
		error.css('font-size', '12px');
		error.css('margin-top', '2px');
	  },
      submitHandler: function(form) {
		$('#inputTell1').attr('name', 'memTell');
		$('#inputTell2').attr('name', 'memTell');
		$('#inputEmail1').attr('name', 'memEmail');
		$('#inputEmail2').attr('name', 'memEmail');
		removeSpecData(form);
        form.submit();   //유효성 검사를 통과시 전송
      }
   });

```

#### 🟢회원2 (회원탈퇴)
![화면 캡처 2022-06-05 211337](https://user-images.githubusercontent.com/94097773/172049860-072bfe18-c95e-444e-a107-daad4f7da401.png)   
➡️회원 탈퇴 페이지에 가기 위해서는 비밀번호를 입력해야 하며, 잘 입력할 경우 위의 화면으로 이동합니다.   
탈퇴 안내 후 한번 더 클릭하면 탈퇴 아이디로 전환되면서 로그아웃 됩니다.   
탈퇴 아이디로는 로그인 할 수 없습니다.   
   
#### 🟢회원2 (로그인)
![화면 캡처 2022-06-05 211747](https://user-images.githubusercontent.com/94097773/172050108-b61007f9-4788-4091-a74b-9bed588423cd.png)
![화면 캡처 2022-06-05 212016](https://user-images.githubusercontent.com/94097773/172050123-90783c21-8503-4944-9afc-f4bbf6577cd9.png)   
➡️Session을 이용하여 로그인한 사용자의 정보를 1시간 동안 저장합니다.  
로그인 시, 화면 상단에 프로필 영역이 생깁니다.   
memberController.java
```
 // 로그인
   @ResponseBody
   @PostMapping("/login")
   public int login(String memId, String memPwd, HttpSession session) {
      MemberVO loginMem = memberService.login(memId);
      int result = 1;
      if (loginMem != null && pwEncoder.matches(memPwd, loginMem.getMemPwd())) {
    	  
    	  if(loginMem.getIsDelete().equals("Y")) {
    		  result = 2;
    	  }
    	  else {
    		  session.setAttribute("loginInfo", loginMem);
    		  //session.setMaxInactiveInterval(60*60);
    		  result = 0;
    	  }
      }
      return result;
   }
```
   
#### 🟢회원3 (내 정보 조회/수정)
![화면 캡처 2022-06-05 212856](https://user-images.githubusercontent.com/94097773/172050436-cada0b54-4d72-4115-ad6a-5cd7d2b77b4f.png)   
➡️Boot Strap의 Modal을 이용하여, 수정하기 버튼을 누르면 Modal창이 뜹니다.   
기본 정보와 보안 정보의 경우, 정보 입력 도즁 Model창을 입력칸이 리셋됩니다.   
my_page_detail.js
```
//모달 닫고 열었을 때 정보 다 사라져있도록
function showPopup(){
	$('#myPageDetail-basic .modal-body-top-right input').each(function(index, element){
		$(element).val('');
	});
	
	$('#myPageDetail-basic').modal('show');
}
```
![화면 캡처 2022-06-05 213452](https://user-images.githubusercontent.com/94097773/172050724-74631ea9-db86-4ce1-bd54-f8ea07c880db.png)   
➡️기본정보 수정 Modal창의 경우, 프로필 사진을 변경/삭제 시 썸네일로 확인할 수 있습니다. 여기서 삭제는 기본 프로필로 변경되는 것을 의미합니다.
```
//기본정보 변경 시 변경되는 프로필 이미지 미리보기
function previewFile(){
	let preview = document.getElementById('thumbnail');
	let file = document.querySelector('input[type=file]').files[0];
	let reader = new FileReader();
	
	reader.addEventListener("load", function(){
		preview.src = reader.result;
	}, false);

	if(file){
		reader.readAsDataURL(file);
	}
}

//프사 삭제 시 프로필 이미지 미리보기
function deleteProfileImage(){
	const hiddenMemImg = document.querySelector('#basicForm input[name="memImage"]');
	
	let preview = document.getElementById('thumbnail');
	preview.src = '/resources/images/member/profile_sample.jpg';
	hiddenMemImg.value = 'profile_sample.jpg';
	
}
```
기본 정보 수정
memberController.java
```
//기본 정보 수정
   @PostMapping("/updateBasicInfo")
   public String updateBasicInfo(Model model, MemberVO memberVO, MultipartHttpServletRequest multi
         , HttpSession session, RedirectAttributes re) {
	  memberVO.setMemTell(memberVO.getMemTell().replace(",", "-"));
      MultipartFile file = multi.getFile("file");
      if(!file.getOriginalFilename().equals("")) {
         String uploadPath = "D:\\dev\\workspaceSTS\\LIBRARY\\src\\main\\webapp\\resources\\images\\member\\";
         
         try {
            String memOriginName = file.getOriginalFilename();
            String memAtImgName = System.currentTimeMillis() + "_" + file.getOriginalFilename();
            file.transferTo(new File(uploadPath + memAtImgName));
            MemberImageVO vo = new MemberImageVO();
            vo.setMemOriginName(memOriginName);
            vo.setMemAtImgName(memAtImgName);
            vo.setMemId(memberVO.getMemId());
            memberService.updateMemImage(vo);
            
            memberVO.setMemImage(memAtImgName);
            
         } catch (IllegalStateException e) {
            //업로드 예외 발생시
            e.printStackTrace();
         } catch (IOException e) {
            //파일 입출력 예외 발생시
            e.printStackTrace();
         }
         
      }
      //기본 정보 수정은 했는데 프로필은 바꾸지 않았을 경우
      else if(file.getOriginalFilename().equals("")){
         //프로필 삭제를 했을 경우 (profile_sample로 바꿀 경우)
         if(memberVO.getMemImage().equals("profile_sample.jpg")){
        	 MemberImageVO vo = new MemberImageVO();
        	 vo.setMemOriginName("profile_sample.jpg");
        	 vo.setMemAtImgName("profile_sample.jpg");
        	 vo.setMemId(memberVO.getMemId());
        	 memberService.updateMemImage(vo);
        	 memberVO.setMemImage("profile_sample.jpg");
        	// memberService.joinMember(memberVO);
        	// memberService.insertMemberImage(vo);
         }
         else {
        	 memberVO.setMemImage(memberService.selectMemAtImgName(memberVO.getMemId()));
         }
      }
    
```
#### 🟢회원4 (아이디/비밀번호 찾기)
![화면 캡처 2022-06-05 214110](https://user-images.githubusercontent.com/94097773/172050955-684ada3c-38cc-4e89-b0b3-14cfe24256cd.png)   
➡️유효성 검사 만족 시, 사용자 정보에 저장된 이메일로 임시 비밀번호가 전송됩니다.      
memberController.java
```
   @ResponseBody
   @PostMapping("/findPwd")
   public void findPwd(MemberVO memberVO) {
      // 임시 비번 보낼 이메일 조회
      String memEmail = memberService.selectEmail(memberVO);
      // 임시 비밀번호 생성 소문자 + 대문자 + 숫자 포함 8자리
      String tempPwd = getTempPwd();
      
      //-------------------비밀번호 암호화-----------------------//
      String encodePw = pwEncoder.encode(tempPwd);
      memberVO.setMemPwd(encodePw);
      
     // memberVO.setMemPwd(tempPwd);
      memberService.updateTempPwd(memberVO);
      try {
         MimeMessage message = mailSender.createMimeMessage();
         MimeMessageHelper messageHelper;
         messageHelper = new MimeMessageHelper(message, true, "UTF-8");
         messageHelper.setFrom("surfurlove@gmail.com");
         messageHelper.setTo(memEmail);
         messageHelper.setSubject(memberVO.getMemName() + "님 늘봄 도서관 비밀번호 잃어버리셨죠?");
         messageHelper.setText("임시 비밀번호는 <" + tempPwd + ">입니다. 로그인 후 새 비밀번호로 변경해 주세요.");
         mailSender.send(message);

      } catch (MessagingException e) {
         // TODO Auto-generated catch block
         e.printStackTrace();
      }

   }
```
➡️임시 비밀번호를 가지고 있는 사용자가 로그인 시, 메인화면에 비밀번호를 수정하라는 alert을 받습니다.   
home.jsp
```
 <c:if test="${sessionScope.loginInfo.isPwdTemp eq 'Y' }">
	<script type="text/javascript">
        if (!confirm("새로운 비밀번호로 설정해 주세요.")) {
            alert("언제든지 내 정보 상세보기에서 수정 가능합니다.");
        } 
        else {
            location.href='/member/myPageDetail';
        }
	</script>
</c:if>
```
확인을 누르면 내 정보 상세 페이지로 이동합니다. 

### 🍀프로젝트 하면서 느낀 점
필요한 건 그때그때 수정하고, 빨리 시작을 하는 게 좋을 거라는 생각이 추후 많은 수정을 하게 만들었습니다. 언제나 변수는 있기 때문에 100% 완벽한 계획은 없겠지만, 그래도 계획 시 충분한 시간을 들여야 한다는 것을 느꼈습니다.   
   
### 🍀팀으로 협업하면서 느낀 점
각자 할 것에 집중하다 보니 의사소통의 때를 놓칠 때가 종종 있었습니다. 형상 관리 툴로 SVN을 사용하면서 의사소통을 꼼꼼히 하기 시작하였습니다. 또 팀원이 맡은 모듈의 코드를 재사용할 때 누가 봐도 알아보기 쉽도록 주석을 잘 달아 두는 것이 좋겠다는 것 또한 알게 되었습니다.

### 🍀수정하고 싶은 부분
유효성 검사 부분 보충 / 북적북적 도서 랜덤 추천 수정
   
   
