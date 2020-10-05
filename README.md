freeboard

로그인, 게시글, 댓글 기능이 들어간 간단한 게시판 프로젝트 입니다.

설계부터 구현까지 진행하는 과정을 담았습니다.  



---

## 목차

1. 개발환경
2. 설계
   - 기능분석
   - 도메인 모델, 테이블 설계
   - 화면 설계
3. 기능 구현
4. 웹 계층 개발
5. 단위 테스트 작성

---

## 1. 개발환경

- 운영체제: Window OS (10)
- IDE: IntelliJ IDEA Community (2020.1.4)
- Language: Java (jdk8)
- Framework: SpringBoot (2.3.2 RELEASE)
- WAS: Apache Tomcat 8.5
- Build: Gradle 4.10.2
- Front: Thymeleaf, JavaScript
- Plugins: Lombok
- 형상관리: Git - Git Bash, GitHub Desktop
- 단위테스트: JUnit4

---

## 2. 설계

#### - 기능분석

1. 회원기능   
   -회원가입   
   -로그인/로그아웃   
   -회원정보 수정            

2. 게시판   
   -게시글 조회   
   -게시글 등록   
   -게시글 수정   
   -게시글 삭제   
   -게시글 검색   
   -페이징바    

3. 댓글   
   -댓글 등록   
   -댓글 수정   
   -댓글 삭제    

   

   *기타 요구사항

    -로그인하지 않으면 게시글은 조회만 가능하다.
    -게시글 검색은 작성자(닉네임)/제목/제목+내용으로 검색할 수 있다.
    -게시글을 열람할 때 조회수도 증가한다.



#### 도메인 모델, 테이블 설계



<img src="https://github.com/ksan41/freeboard/blob/master/img/Untitled%20Diagram.png?raw=true" width="600px" height="550px" title="550px" alt="tables"></img><br/>



> 회원은 여러 게시글을 작성할 수 있다. (1대 다)
> 하나의 게시글에는 여러개의 댓글이 달릴 수 있다. (1대 다)



#### *도메인

-회원(member)

-게시글(board)

-댓글(reply)

#### *보조 클래스

-회원권한(Role) : Enum

-게시글상태(BoardStatus) : Enum // 게시글, 댓글 같이 사용

-검색조건(BoardSearch) : Enum // 게시글 검색용 검색조건, 카테고리(작성자, 제목, 제목+내용)



#### 화면 설계

**https://ovenapp.io/view/EmXhOdyDCMGCrPOgfWhHYBhLDwwEqyak/ALD8x**

카카오 오븐 사용



-홈 화면

<img src="https://github.com/ksan41/freeboard/blob/master/img/oven/1.PNG?raw=true" width="900px" height="400px" title="550px" alt="tables"></img>

​																				로그인 정보 없을 시



<img src="https://github.com/ksan41/freeboard/blob/master/img/oven/2.PNG?raw=true" width="900px" height="400px" title="550px" alt="tables"></img>

​																						로그인 시



-회원가입 페이지

<img src="https://github.com/ksan41/freeboard/blob/master/img/oven/3.PNG?raw=true" width="800px" height="400px" title="550px" alt="tables"></img>

-로그인 페이지

<img src="https://github.com/ksan41/freeboard/blob/master/img/oven/4.PNG?raw=true" width="750px" height="500px" title="550px" alt="tables"></img>

-마이페이지

<img src="https://github.com/ksan41/freeboard/blob/master/img/oven/5.PNG?raw=true" width="750px" height="500px" title="550px" alt="tables"></img>

​																				마이페이지 첫 화면

<img src="https://github.com/ksan41/freeboard/blob/master/img/oven/6.PNG?raw=true" width="750px" height="500px" title="550px" alt="tables"></img>

​																				각 변경버튼 클릭 시

-게시글 상세페이지

<img src="https://github.com/ksan41/freeboard/blob/master/img/oven/7.PNG?raw=true" width="750px" height="500px" title="550px" alt="tables"></img>

-게시글 작성페이지, 수정페이지

<img src="https://github.com/ksan41/freeboard/blob/master/img/oven/8.PNG?raw=true" width="750px" height="500px" title="550px" alt="tables"></img>

​																					수정페이지

총 7개 페이지로 구성



# 3. 기능구현

### A. Member(회원) 기능 개발

**SpringSecutiry 사용.

-SecurityConfig

  :  Service에서 비밀번호를 암호화할 수 있도록 BCryptEncoder를 Bean으로 등록했다.

​     SessionFilter를 등록해 front에서 사용자 정보를 "user"로 접근할 수 있도록 했다.

​     사용자의 권한에 따라 url 접근 제한을 두었다.

​     로그인, 로그아웃 설정을 해주었다.



- 회원가입 기능

  -회원가입 시 사용할 MemberDto에 사용자로부터 아이디, 비밀번호, 닉네임을 입력받아 컨트롤러로 전달.

  -서비스에서 BCryptPasswordEncoder를 통해 비밀번호를 암호화 한 뒤, 아이디 중복검사를 수행한다.

   (중복 아이디가 있을 경우엔 등록처리 하지 않고 0l을 반환)

- 회원정보 수정

  -컨트롤러에서 사용자 정보를 받아와 서비스에 넘긴다. 비밀번호와 닉네임을 각각 선택적으로 변경할 수 있으므로, 각각 값이 있을 경우에만 변경처리 하도록 했다.



---

### B. Board(게시글) 기능 개발

**Querydsl 사용

​	: JPARepository로 처리하기 복잡한 쿼리를 수행하기 위해 사용.

​	 간단한 기능처리는 JPARepository를 사용했다.



- 게시글 조회

  -화면에서 선택한 boardId(식별자)를 받아와 서비스로 넘긴다.

  -이때 조회수도 증가해야 하기 때문에 Board 엔티티를 조회해 온 뒤 조회수를 1 증가시키고

    조회해온 Board를 반환한다.

- 게시글 등록

  -사용자 아이디, 제목, 내용을 컨트롤러에서 받아와 서비스에 넘겼다.

  -서비스에서는 사용자 아이디로 Member 엔티티를 조회해오고, 각 정보를 담아 Board 객체를 생성 후

   BoardRepository(JPA Repository) save 메소드 실행.

- 게시글 수정

  -boardId, 제목과 내용을 받아와 서비스에 넘긴 후, Board 엔티티 내에 작성해둔 update 메소드를 수행한다.

- 게시글 삭제

  -boardId를 받아와 서비스에 넘긴 후, Board 엔티티 내에 작성해둔 delete 메소드를 수행한다.

  (실제로 삭제되지는 않고 boardStatus값만 'DELETE'로 변경해 화면에서 보이지 않도록 했다.)

- 게시글 전체조회, 게시글 검색

  -페이징을 구현하기 위해 page값(기본 0)을 받아오도록 하고, 검색조건(String. 기본 ""로 뒀다.)과 검색어를 받아온다.

  -컨트롤러에서 조건검사를 통해 넘어온 검색조건 값에 맞는 BoardSearch(검색조건 enum)값을 생성해준다.

   검색 값이 없을 경우엔 null로 둔다.

   페이징을 위한 PageRequest 객체를 생성한다.

   PageRequest, 검색 조건, 키워드를 서비스로 넘긴다.

  -서비스에선 받아온 값들을 BaordQueryRepository(query dsl을 사용하기 위해 따로 만든 Repository)에 넘긴다.

  -JPQLQuery에서 기본적으로 boardStatus값이 'VISIBLE'인 모든 게시글을 조회한다.

  여기에, where메소드에서 각각 BoardSearch값에 따라 조건이 추가되는 메소드를 수행한다.

  (각각 BoardSearch(검색조건)의 값을 비교해, 일치한다면 그에 따른 조회 쿼리를 추가해주고, 일치하지 않는다면 null을 반환해 해당 쿼리가 생성되지 않는다.)

  밑에서는 orderBy로 정렬조건과 offser,limit으로 페이징 조건을 추가해준다.

  마지막으로 쿼리 결과값을 페이징 조건에 맞게 PageImpl객체로 반환한다.

  

---

### C. Reply(댓글) 기능 개발

**Query dsl 사용

-댓글 조회시 조건추가를 위해 사용했다.



- 댓글 조회

  -해당 게시글의 댓글을 모두 조회해온다. 게시글의 식별자 boardId를 받아온다.

  게시글의 댓글개수가 0일 경우엔 빈 문자열을 반환. 

  0보다 클 경우에만 댓글 조회 요청을 수행한다.

  -바로 ReplyQueryRepository의 메서드를 실행하도록 했다.

   해당 게시글의 댓글을 날짜순 List로 받아온다.

   컨트롤러에서는 조회된 데이터를 ObjectMapper로 한번 감싼 뒤, String으로 받는다.

  -댓글 데이터를 Json으로 넘길 것이므로, 미리 Reply 엔티티에서 날짜 정보를 원하는 포맷으로 설정해주었다.

- 댓글 작성

  -게시글 식별자 boardId와 회원 아이디(댓글 작성자), 댓글내용을 받아와 서비스에 넘겨준다.

  -서비스에선 넘어온 데이터들로 Reply객체를 생성, JPARepository의 save메서드를 수행해준다.

   게시글 식별자로 Board엔티티를 불러와서 게시글 댓글 개수도 1 증가시켜준다.

- 댓글 수정

  -댓글 식별자 replyId, 수정할 내용을 받아와 서비스에 넘겨준다.

  -해당 Reply 엔티티를 조회해온 뒤, Reply엔티티에 작성해둔 메서드 update를 수행해준다.

    (updateDate를 현재 날짜로 업데이트하고, 댓글 내용을 변경한다.)

- 댓글 삭제

  -댓글 식별자 replyId를 받아와 ObjectMapper로 한번 감싼 뒤 서비스 요청한다.

  -해당 Reply엔티티를 불러온 뒤, Reply엔티티에 작성해둔 메서드 delete를 수행해준다.

  (status값을 'DELETE'로 변경한 뒤, 댓글 내용을 "삭제된 댓글입니다."로 변경한다.)

# 4. 웹 계층 개발





# 5. 단위 테스트 작성

각 도메인 서비스의 메서드들을 테스트.

JUnit4 사용



- MemberServiceTest (회원 기능 테스트)

  - 회원가입 테스트

  ​       : 아이디, 비밀번호, 닉네임이 주어졌을때 정상적으로 가입이 되는지 테스트한다.

  - 중복 아이디 체크 테스트

    : 가입 시 입력받은 아이디 중복확인 기능을 테스트한다.

  - 회원정보 수정 테스트

    : 새 비밀번호, 닉네임이 주어졌을 때 정상적으로 회원정보 수정이 되는지 테스트한다.

      비밀번호는 저장 시 암호화 되기 때문에 BCryptPasswordEncoder의 matches메소드로

      비교한다.

- BoardServiceTest (게시판 기능 테스트)

  - 글 등록 테스트

    :  회원 아이디, 제목, 내용이 주어졌을 때 정상적으로 게시글이 등록 되는지 테스트한다.

  - 게시글 삭제 테스트

    :  주어진 게시글 번호(pk)에 해당하는 게시글이 삭제되는지 테스트한다.

       (실제로는 status값이 DELETE로 변경된다.)

  - 게시글 수정 테스트

    : 게시글 번호와 수정할 제목과 내용이 주어졌을 때 정상적으로 게시글이 수정되는지 

      테스트한다.

- ReplyServiceTest (댓글 기능 테스트)

  - 댓글 작성 테스트

    :  게시글 번호, 회원아이디(댓글 작성자), 댓글 내용이 주어졌을 때 정상적으로 댓글이 등록 

       되는지 테스트한다. 해당 게시글의 댓글 갯수도 1 증가했는지 확인한다.

  - 댓글 수정 테스트

    : 회원아이디(댓글 작성자), 새 댓글 내용이 주어졌을 때 정상적으로 댓글이 수정되는지 확인한다.

  - 댓글 삭제 테스트

    : 댓글 아이디(pk)가 주어졌을 때 해당 댓글이 삭제되는지 테스트한다.

      (삭제 시 댓글 상태값이 DELETE로 변경되며, 내용은 "삭제된 댓글입니다."로 변경된다.)