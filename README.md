# **개발 가이드**



## 개발 샘플 소스

- 기본적인 목록(realgrid2), 상세, 등록, 수정, 파일첨부 등의 기능은 sample 패키지에 있는 소스참고.

  url : http://127.0.0.1:8080/sample/authorSampleList.do

  java : net.ecbank.sample 패키지 내 AuthorSampleXXXX.java

  jsp : /WEB-INF/jsp/sample 디렉토리 내 authorSampleXXXX.jsp

- 개발지원 메뉴의 해당 소스 참고

  

## Realgrid2

- 튜토리얼 및 API 사이트 : https://docs.realgrid.com/tutorials/realgrid-tutorial-start


- 기술지원 사이트 : https://forum.realgrid.com/categories/enmn89RvkpbLrDwH7

-  샘플: 개발지원 > 리얼그리드2 샘플

- 공통 자바스크립트 : ecRealGrid.js 파일 참고

  

## 공통 자바스크립트

**디렉토리** : src\main\webapp\resources\js

- app_common.js : 화면공통 관련
- ecBiz.js  : 팝업 등 업무공통
- ecConfig.js : 플러그인 설정 관련(inputmask, datepicker, validate 등)
- EcEgovMultiFiles.js : 파일업로드 관련
- ecJsComponent.js : ajax 처리 관련
- ecRealGrid.js : 리얼그리드 관련(초기설정, 간소화 함수 등)
- ecRealGridConfig.js : 리얼그리드 공통설정, 공통 콜백 등
- ecStatus.js : 각 업무데이타별 상태코드 정의
- ecUtil.js : 각종 유틸 함수

기타 라이브러리

- lowdash
- 

## DB Naming Rule

- 업무구분별 테이블 prefix

| 업무구분   | prefix | 비고, 예                  |
| ---------- | ------ | ------------------------- |
| 프레임워크 | EF_    | ecEgov Frameowrk, EF_CODE |
| 업무공통   | EC_    | ecEgov Common, EC_EMP     |

- DataBase Object별 prefix

| 구분     | prefix | 비고, 예    |
| -------- | ------ | ----------- |
| 시퀀스   | SQ_    | SQ_REG_ID   |
| 함수     | FN_    | FN_FORMAT   |
| 프로시저 | SP_    | SP_MIG_USER |

- 테이블명, 칼럼명 Naming은 파일공유서버 Y:\01. Phase2\02. 설계\05. 데이터모델정의\HDO_글로벌종합정보시스템_데이터표준사전_xxx.xlsx 엑셀 파일을 참고하여 작성하도록 한다.



## Java Source Package

| 구분       | package              | 비고. 예                            | JSP 디렉토리(WEB-INF/jsp/) |
| ---------- | -------------------- | ----------------------------------- | -------------------------- |
| higis업무  | higis.xxx            | higis 하위에 업무단위별 패키지 구성 | higis/xxx                  |
| ADMIN 기능 | net.ecbank.admin     |                                     | admin                      |
| 업무공통   | net.ecbank.biz       | 결재, 게시판, 공통, 공통배치        | biz                        |
| 프레임워크 | net.ecbank.framework | 프레임워크 관련 공통 소스           |                            |



## Java Source Naming

| 구분                        | Controller(&url)      | Service       | Dao           | Jsp            | sql Id        |
| --------------------------- | --------------------- | ------------- | ------------- | -------------- | ------------- |
| 등록 처리                   | **regist**Xxx         | registXxx     | insertXxx     |                | insertXxx     |
| 수정 처리                   | **update**Xxx         | updateXxx     | updateXxx     |                | updateXxx     |
| 저장 처리(등록,수정 처리시) | **save**Xxx           | saveXxx       | saveXxx       |                | saveXxx       |
| 목록데이타 조회 처리        | **select**Xxx**List** | selectXxxList | selectXxxList |                | selectXxxList |
| 상세데이타 조회 처리        | **select**Xxx         | selectXxx     | selectXxx     |                | selectXxx     |
| 삭제 처리                   | **delete**Xxx         | deleteXxx     | deleteXxx     |                | deleteXxx     |
| 등록화면 이동               | xxx**RegistPop**      |               |               | xxxViewPop.jsp |               |
| 수정화면 이동               | xxx**EditPop**        |               |               | xxxViewPop.jsp |               |
| 상세화면 이동               | xxx**ViewPop**        |               |               | xxxViewPop.jsp |               |
|                             |                       |               |               |                |               |



## Controller @ProgramInfo Annotaition

- Controller Method에는 @RequestMapping 과 함께 @ProgramInfo 어노테이션을 작성한다.
- name 속성은 필수, 
- desc: 부가설명 (default값: name)
- gid: 프로그램그룹(=해당 업무 패키지명), default값: url의 최상위 path로 지정됨.)

```
/**
* 작가 목록 조회 페이지
* @return
*/
@ProgramInfo(name="작가 목록조회 페이지", desc="작가 목록 조회 페이지로 이동", gid="sample")
@RequestMapping("/sample/authorSampleList.do")
public String authorList(ModelMap model) {
```



## Transaction 처리

- 기본적으로 Service Layer 메소드 레벨에서 선언적 Transaction 처리

  ```java
  @Service
  @Transactional(rollbackFor=Exception.class)
  public class AuthorSampleService extends BaseService {
  ```

- 별도 트랜잭션 분리가 필요한 메소드별로 트랜잭션 선언

  ```java
  @Transactional(propagation=Propagation.REQUIRES_NEW, rollbackFor = Exception.class)
  public Map<String, Object> saveContractServerSign(String ctrtNo, String ctrtOrd, String systemId) throws Exception {
  ```

- 주의) 동일 서비스 클래서 내에서 다른 메서드 호출은 트랜잭션 선언과 상관없이 동일 트랜잭션으로 묶인다. 따라서, 트랜잭션 분리가 필요한 메소드는 별도의 클래스의 메소드로 작성해서 호출되어야 함.
  1. AClass.a() -> AClass.b(): b method의 트랜잭션 선언과 상관없이 동일 트랜잭션으로 실행됨.
  2. AClass. a() -> CClass.c(): c method의 트랜잭션 선언에 따라 트랜잭션 적용.



## MyBatis Sql 작성

- 파일 위치 : src/main/resources/egovframework/mapper_{db}/{업무별}/{DaoName}\_SQL_mssql.xml

- xml namepace : <mapper namespace="{dao package}.{dao class name}"> 예) <mapper namespace="higis.cmn.SpreadJsManageDao">

- sql 최상단에 {sql 파일명} {sql id} {sql 제목} 라인을 주석으로 포함한다.

- sql 문은 가급적 1라인에 1칼럼으로 작성하고, 칼럼명 주석을 가능한 포함한다.

- 라인은 키워드 및 칼럼에 맞춰서 정렬하고, 칼럼 구분 ',' 는 라인 맨 앞으로 둔다.

```xml
<select id="selectAuthor" parameterType="Map" resultMap="author">
--Sample_SQL_mssql.xml selectAuthor 작가 상세 조회
SELECT AUTHOR_ID  /*작가ID*/
     , AUTHOR_NM  /*이름*/
     , TITLE      /*책제목*/
  FROM Z_AUTHOR A
     , Z_BOOK   B
     , (SELECT CODE_NM /*코드명*/
          FROM EF_CODE 
         WHERE CODE_ID = #{CODE_ID}
       ) C
 WHERE A.AUTHOR_ID = B.AUTHOR_ID
   AND A.AUTHOR_ID = #{AUTHOR_ID}
</select>

<update id="updateAuthor" parameterType="Map">
--Sample_SQL_mssql.xml updateAuthor 작가정보 수정
UPDATE Z_AUTHOR
   SET AUTHOR_NM        = #{AUTHOR_NM} /*작가명*/
     , BIRTH_DAY        = REPLACE(#{BIRTH_DAY},'-','') /*생년월일*/
 WHERE AUTHOR_ID = #{AUTHOR_ID}
</update>

<insert id="insertAuthor" parameterType="Map">
--Sample_SQL_mssql.xml insertAuthor 작가정보 등록
INSERT INTO Z_AUTHOR(
       AUTHOR_ID         /*작가ID*/
     , AUTHOR_NM         /*이름*/
)
VALUES(
       #{AUTHOR_ID}     /*작가ID*/
     , #{AUTHOR_NM}     /*이름*/
)
</insert>

```



#### 공통 쿼리 파라미터: 

- 로그인 세션정보, 환경정보 등의 변수는 프레임워크에서 아래의 파라미터로 자동 추가.(Dao 방식으로 처리 시.)

| 파라미터명       | Map type 파라미터  | BaseObjct 상속 Object 파라미터 | 비고             |
| ---------------- | ------------------ | ------------------------------ | ---------------- |
| 사용자ID         | #{SS_USER_ID}      | #{ssUserId}                    |                  |
| 사용자명         | #{SS_USER_NM}      | #{ssUserNm}                    |                  |
| 부서코드         | #{SS_DEPT_CD}      | #{ssDeptCd}                    |                  |
| 부서명           | #{SS_DEPT_NM}      | #{ssDeptNm}                    |                  |
| 직책명           | #{SS_JOB_NM}       | #{ssJobNm}                     |                  |
| 직급명           | #{SS_POS_NM}       | #{ssPosNm}                     |                  |
| 사용자 권한 배열 | #{SS_ROLE_LIST}    | #{ssRoleList}                  |                  |
| 사용언어         | #{SS_LANG}         | #{ssLang}                      | ko(default), en  |
| 실행환경구분     | #{SS_RUNTIME_MODE} | #{ssRuntimeMode}               | local, dev, prod |



## DB 공통함수

- 사용자 정의 공통함수 참고

```sql
--사업자번호 포맷
SELECT FN_FORMAT_BIZNO('1234567890') FROM DUAL; //123-45-67890
--전화번호 포맷
SELECT FN_FORMAT_TEL('01012345678') FROM DUAL; //010-1234-5678
--휴대폰번호 포맷
SELECT FN_FORMAT_MOBILE('01012345678') FROM DUAL; //010-1234-5678
--yyyymmdd 문자열을 yyyy-mm-dd 로 모맷
SELECT FN_FORMAT_YMD('20191020') FROM DUAL; //2019-10-20
--yyyymmddhhmiss 문자열을 yyyy-mm-dd hh:mi:ss 로 포맷
SELECT FN_FORMAT_YMDS('20191020121314') FROM DUAL; //2019-10-20 12:13:14

--전화번호 가운데 마스킹
SELECT FN_MASKING_TEL('01012345678')  FROM DUAL; //010-****-5678
--이메일 마스킹
SELECT FN_MASKING_EMAIL('jnjhanmail.net') FROM DUAL; //jnj**@hanmail.net
--이름 마스킹
SELECT FN_MASKING_NAME('홍길동') FROM DUAL; //홍*동

--날짜,숫자,포맷된문자들을 REPLACE 처리
SELECT FN_UNFORMAT('2020-10-20 23:45:30') FROM DUAL; //20201020234530

--YYYYMMDD 문자열을 시작 DATE 타입으로 리턴
SELECT FN_YMD_BGN_DATE('20200110') FROM DUAL; //2020-01-10 00:00:00
--YYYYMMDD 문자열을 끝 DATE 타입으로 리턴
SELECT FN_YMD_END_DATE('20200110') FROM DUAL; //2020-01-10 23:59:59
```



## 메세지 처리

- src\main\resources\egovframework\message\com\message-higis_ko.properties에 메세지 정의

- 데이타처리 및 업무공통 관련 메세지만 관리하고, 각 업무별 메세지는 프로그램 소스에서 개발 사용.

- jsp/Javascript (common.info.require={0}은 필수값입니다.)

  ```jsp
  alert('<spring:message code="common.info.require" arguments="이름"/>');
  ```

- Java

  ```
  throw new BizRuntimeException(messageSource, "common.info.require", new String[]{"이름"});
  ```



## 로깅 처리

- slf4j 로깅 api 사용

- 로깅 메세지는 {} 변수형식을 사용하여 구성

  ```java
  private static final Logger LOG = LoggerFactory.getLogger(SampleController.class);
  
  LOG.debug("test.prop: {}",  propertyService.getString("test.prop", "defalut val")); 
  
  ```



## 주석 처리

- 개발환경.md 파일 참고

  

## 폼 입력값 validation

- jquery validation library 사용 : https://jqueryvalidation.org/

  ```javascript
  var basicValidateOptions = {
  	rules :{
  		AUTHOR_NM: 	{required:true},
  		EMAIL:		{required:false,	isEmail:true},
  		DEBUT_YEAR:	{range:[1950,2020],	minlength:4}, //,integer:true
  		BIZ_NO:		{isBizNo:true},
  		MOBILE_PHONE:{required:true,isPhone:true}
  	},
      messages: {
      	AUTHOR_NM: { required: '<spring:message code="common.info.require" arguments="작가명"/>'},
          DEBUT_YEAR:{ required: '<spring:message code="common.info.require" arguments="데뷔년도"/>', range:"데뷔년도는 1950 ~ 2020입니다.'/>" }
      }
  };
  //저장
  function save(){
  	//폼입력값 유효성 체크
  	if (!validateForm("frm",basicValidateOptions)){
          return false;
      }
  }
  ```

- validation 공통 설정 및 추가 validation 메소드는 header.jsp 참고.



## 그리드 데이타 포맷 처리

- 데이타의 포맷은 기본적으로 화면에서 input mask(css), grid 포맷기능(displayCallback)을 이용하여 처리하고, 저장 시에는 unformat(fnGetParams() 공통javascript에서 처리됨)하여 서버로 전달한다.

- 필요에 따라 서버(sql)에서 포맷하여 조회할 경우나 언포맷하여 저장할 시에는 DB 공통 function을 활용한다.

- 그리드에서 일, 일시를 표시하는 경우 미리 정의한 renderType을 활용한다.


  ```javascript
  , {fieldName: "UNLOAD_ETA", header: {text: "대산ETA"}, dataType: "text", 	width:"80", rendererType:'yyyymmdd', editable:false}
  //유형.
  rendererType:'yyyymm' //2020-03
  rendererType:'yyyymmdd' //2020-03-02
  rendererType:'yyyymmhhmmss' //2020-03-02 12:40:56
  ```

  

## 권한관리

- 시스템 내의 업무역할 등에 따라 논리적인 권한을 정의한다. (시스템 관리 > 권한관리 > 권한관리)

- 사용자 또는 부서단위로 복수의 권한을 부여한다. (시스템 관리 > 권한관리 > 사용자 권한관리, 부서 권한관리)

- 로그한 사용자의 권한은 (사용자의 권한 + 소속 부서의 권한) 이다.

- 변경된 권한은 재로그인 시 적용된다.

  

## 서비스 접근제어 (권한에 따른 기능 제어)

- 기본적으로 모든 권한에 시스템 전체 접근 권한을 부여하고, 특정 기능에 한해서 권한을 정의하여 제어하는 개념으로 적용한다.

- 접근제어를 하고자 하는 업무기능의 서비스 URL을 시스템 관리 > 서비스 관리에 등록한다.

- 서비스 URL은 regular expression 형태로 등록하여  기능그룹 형태로 관리할 수 있다.

- 우선권한적용여부를 'Y'로 정의된 서비스들이 먼저 접근제어 처리되고, 만약 로그인 사용자가 해당 서비스에 권한이 없으면 후순위 서비스 접근제어 여부에 상관없이 "권한없음" 에러가 발생한다.

- 서비스 URL 패턴(regex)은 좁은 범위(더 세부적인) 의 표현이 우선 순위가 앞으로 오도록 정의하여야 한다.

- 서비스 접근제어 처리 순서는 우선권한적용(Y) -> 우선순위 순으로 처리된다.

- 등록된 서비스는 반드시 하나 이상의 권한과 맵핑되어야 시스템에 적용된다.(시스템 관리 > 권한관리 > 권한관리)

- 로그인하지 않고도 사용할 수 있는 서비스는 egov-com-access.xml 의 excludeList 에 정의한다.

- 권한에 따른 화면 상의 버튼 활성화 처리 등은 EgovUserDetailsHelper.hasUrlAccess("서비스url") 메소드로 활용하여 처리한다.

  (기타 공통기능의 '세션 및 권한정보 조회' 부분 참고.)

- 권한에 따른 화면 버튼 활성화 예제

  ```jsp
  <%@ page import="egovframework.com.cmm.util.EgovUser
  
  <c:if test="${EgovUserDetailsHelper.hasUrlAccess('/crudeOper/loadPayment/requestPo.do')}">
      <button type="button" class="btn" 		id="btnRequestPO">PO발급</button>
  </c:if>
  <c:if test="${EgovUserDetailsHelper.hasUrlAccess('/crudeOper/loadPayment/saveLoadPayment.do')}">
  	<button type="button" class="btn" 		id="btnSave">저장</button>
  </c:if>    
  ```

- 권한에 따른 그리드 셀 수정여부 예제

  ```jsp
  //수정권한 여부
  var hasAuthSave = <c:out value="${EgovUserDetailsHelper.hasUrlAccess(\'/crudeOper/loadPayment/saveLoadPayment.do\')}"/>;
  
  //그리드 수정여부 상시 처리 시
  , {fieldName: "OILKND_ID", header: {text: "유종"}, dataType: "text", 	width:"50",	editable:hasAuthSave}
  
  //그리드 수정여부 동적 처리 시
  gridView.setCellStyleCallback(function(grid, dataCell) {
  	var ret = {}
  	
  	if (some_condition){
  		ret.editable = true && hasAuthSave;
      }
  	return ret;
  });
  ```

- 특정 권한에만 그리드 숨김필드 보이기 예제

  ```jsp
  //숨김필드 표시여부(시스템관리자 또는 개발자)
  var isShowField = <c:out value="${EgovUserDetailsHelper.hasAnyRole('ROLE_ADMIN,ROLE_DEV')?true:false}"/>;
  
  , {fieldName: "PO_EAI_SEQ", header: {text: "PO I/F SEQ"},	dataType: "text",	width:"80", visible:isShowField}
  ```

- 런타임 모드(local, dev, prod) 에 따른 화면 제어 예제

  ```jsp
  <c:if test='${propertyService.getRuntimeMode()=="local"}'>
  	<button type="button" class="btn" 		id="btnTest">테스트</button>
  </c:if>
  ```

  

## 기타 공통 기능

세션 및 권한정보, 프로퍼티정보, 코드정보 등의 예제는 SampleController.infoView(), sample/infoView.jsp 소스 참고.

- 프로퍼티 조회 : 프로퍼티 정의는 globals.properties, context-properties.xml 파일

  - JSP : ${propertyService}  빈 참조 attribute 사용 또는 PropertiesUtil 클래스 사용.


  ```jsp
  <%@ page import="net.ecbank.fwk.util.PropertiesUtil" %>
  <li>프로퍼티조회 bean : 
  <c:out value='${propertyService}'/></li>
  <li>프로파일 조회 : 
  <c:out value='${fn:join(propertyService.getActiveProfiles(),",")}'/></li>
  <li>서버모드 : 						
  <c:out value='${propertyService.getRuntimeMode()}'/></li>
  <li>프로퍼티값 조회 : 				
  <c:out value='${propertyService.getString("test.prop")}'/></li>
  <li>프로퍼티값 조회(동일키 여러개) :
  <c:out value='${fn:join(propertyService.getStringArray("test.db.prop"),",")}'/></li>
  ```

  - Java : BaseController, BaseService, BaseDao 클래스에 정의된 propertyService 빈 사용


  ```java
  //프로퍼티 정보 조회
  log.debug("profiles: {}", Arrays.toString(propertyService.getActiveProfiles()));
  log.debug("로컬환경 여부:{}, 개발환경 여부:{}, 운영환경 여부:{}", propertyService.isLocalMode(), propertyService.isDevMode(), propertyService.isProdMode());
  log.debug("pageUnit: {}",   propertyService.getString("pageUnit", "defalut val"));  //context-properties.xml 에 있는 값
  ```

  - Java : 스프링 빈이 아닌 일반클래스에서 프로퍼티 조회 시 PropertiesUtil 클래스 사용


  ```java
  import net.ecbank.fwk.util.PropertiesUtil;
  
  PropertiesUtil.getString("key");
  ```

- 코드값 조회

  - JSP, Javascript : ${codeService} 빈 참조 attribute 사용


  ```jsp
  <script type="text/javascript">
  var cmbSttsValues = [${codeService.getCodeListAsString('TEST001').get("CODE")}];	//그리드 dropdown 값
  var cmbSttsLabels = [${codeService.getCodeListAsString('TEST001').get("CODE_NM")}];	//그리드 dropdown label.
  </script>
  
  <li>코드정보(map):      <c:out value='${codeService.getCode   ("COM001", "REGC02")}'/></li>
  <li>코드값(locale별값) :<c:out value='${codeService.getCodeVal("COM001", "REGC02")}'/></li>
  <!-- 'EF_PROP' 코드그룹에 정의된 프레임워크 및 시스템 설정값 관련 공통코드 조회-->
  <li>환경설정코드정보 :   <c:out value='${codeService.getSystemProperty("server.ip")}'/></li>
  <li>
  	코드selectbox :
  	<select name="codeBox">
  		<option value="">선택</option>
  		<c:forEach var="code" items='${codeService.getCodeList("COM001")}'>
  			<option value="${code.CODE}">${code.CODE_NM}</option>	
  		</c:forEach>
  	</select>
  </li>
  
  ```

  - Java : BaseController, BaseService, BaseDao 클래스에 정의된 codeService 빈 사용


  ```java
  //코드정보 조회
  log.debug("COM001 코드그룹 코드목록:");
  for(Map<String,Object> map : codeService.selectCodeList("COM001")) {
  	log.debug(map.toString());
  }
  log.debug("COM001-REGC02 코드정보(map):" + codeService.selectCode("COM001", "REGC02"));
  log.debug("COM001-REGC02 코드값:" + codeService.selectCodeVal("COM001", "REGC02"));
  ```

- 세션 및 권한정보 조회

  - JSP : EgovUserDetailsHelper 클래스 및 loginVO 세션 객체 사용


  ```jsp
  <%@ page import="egovframework.com.cmm.util.EgovUserDetailsHelper" %>
  
  <li>로그인여부: <%=EgovUserDetailsHelper.isAuthenticated() %></li>
  <li>사용자id : <%=EgovUserDetailsHelper.getUserId()%>,   ${loginVO.id}</li>
  <li>사용자명 : <%=EgovUserDetailsHelper.getUserName()%>, ${loginVO.name}</li>
  <li>부서코드 : ${loginVO.deptCd}</li>
  <li>부서명   : ${loginVO.deptNm}</li>
  <!--등록권한이 있는 경우 등록버튼 표시-->
  <%if(EgovUserDetailsHelper.hasUrlAccess("contract/contractTemplateList.do")){%>
  	<button id="btnRegist">등록</button>
  <%}%>
  ```

  - Java : EgovUserDetailsHelper 클래스 사용


  ```java
  //로그인 사용자 정보 조회
  LoginVO loginVO = (LoginVO)EgovUserDetailsHelper.getAuthenticatedUser();
  log.debug("로그인 사용자 정보 객체:{}", loginVO);
  		
  //로그인 사용자의 권한조회
  List<String> authorities = EgovUserDetailsHelper.getAuthorities();
  if (authorities!=null) {
  	log.debug("로그인 사용자 권한:{}",  Arrays.toString(authorities.toArray()));
  	log.debug("ROLE_ADMIN 권한여부:{}", EgovUserDetailsHelper.hasRole("ROLE_ADMIN"));
  	log.debug("ROLE_TEST 권한여부:{}",  EgovUserDetailsHelper.hasRole("ROLE_TEST"));
  }
  ```

- 공통팝업

  개발지원 > 업무공통 > 업무공통샘플 화면 소스 참고 (sample/bizPopSample.jsp)



## 문서번호 채번

- net.ecbank.fwk.common.service.DocNoSeqService 빈을 사용하여 문서번호 채번
  - 채번번호 규칙 :  PREFIX(옵션) + 일자(옵션,YYYY/YYYYMM/YYYYMMDD) + LPAD(일련번호,일련번호길이,'0')


- EF_DOC_NO_SEQ 테이블에 채번정보 등록 (Admin > 기준정보 > 문서번호 관리 메뉴)

  | 칼럼       | 이름               | 설명                                                         |
  | ---------- | ------------------ | ------------------------------------------------------------ |
  | TABLE_NAME | 채번ID             | 문서채번 구분값                                              |
  | NEXT_ID    | 다음 채번일련번호  | 신규 채번 일련번호. 채번 후 +1 UPDATE됨.                     |
  | RM         | 비고               | 문서채번정보 설명                                            |
  | PREFIX     | 접두어             | 옵션. 문서번호 제일 앞에 붙는 고정문자열                     |
  | YMD_FRMAT  | 채번일자포맷       | 옵션. 채번기준날짜형식 YYYY 년단위, YYYYMM 월단위 YYYYMMDD 일단위 |
  | ID_LENGTH  | 일련번호 자리수    | 미지정 시 10자리로 처리                                      |
  | NOW_YMD    | 현재 채번기준 일자 | 자동으로 UPDATE                                              |
  | NOW_DOC_NO | 현재 문서번호      | 마지막으로 채번된 문서번호.                                  |

- 각 업무단에서 insert 시점에 신규 문서번호 채번

```xml
@Autowired
DocNoSeqService docNoSeqService;

....
String newDocNo = docNoSeqService.getNextDocNo("채번ID");
....
```



## 메일 연동

- 업무별 메일 정보를 EF_MAIL_INFO 테이블에 등록한다.

  | 칼럼             | 칼럼명                     | 비고                                               |
  | ---------------- | -------------------------- | -------------------------------------------------- |
  | MAIL_ID          | 메일ID = 메일템플릿 파일명 | M_업무별코드+일련번호3자리 EX)M_CTRT001            |
  | MAIL_NM          | 메일명                     | 계약발송알림                                       |
  | DEF_SENDER_EMAIL | 디폴트 발신자 이메일       | 메일 전송 시 발신자를 지정하지 않을 경우 사용할 값 |
  | DEF_SENDER_NAME  | 디폴트 발신자 명           | 메일 전송 시 발신자를 지정하지 않을 경우 사용할 값 |
  | TEST_REC_EMAIL   | 테스트용 수신자 이메일     | 개발환경에서 메일 수신할 테스트용 이메일           |
  | DEF_TITLE        | 디폴트 메일제목            | 메일 전송 시 제목을 지정하지 않을 경우 사용할 값   |

- 메일 템플릿 HTML 파일을 메일ID와 동일한 파일명으로 src\main\resources\freemarker\mail 디렉토리에 생성한다.

  - 메일 템플릿 공통 변수
    - ${baseUrl} : 사이트 기본URL

- 메일 템플릿 라이브러리는 freemarker 사용

- 업무 프로그램에서 아래의 예제와 같이 메일을 전송한다.

  ```java
  // 메일전송정보 생성
  MailSendInfo mailSendInfo = new MailSendInfo();
  
  // 메일템플릿 ID
  mailSendInfo.setMailTemplateId("CTRT001");
  
  // 제목. 옵션 (제목을 지정하지 않으면 메일정보의 기본 제목으로 발송됨.)
  mailSendInfo.setTitle("계약전송알림");
  
  //수신자 정보, 수신자명은 옵션이나 필요하면 이에일주소와 동일한 순서로.
  String recvEmails = "abc@abc.com,def@abc.com";
  String recvNames  = "홍길동,이순신";
  
  //수신자 추가 (복수일 경우 콤마문자열로 처리 시)
  mailSendInfo.addRecipientStr(recvEmails, recvNames);
  
  //수신자 추가 (배열로 처리 시)
  //mailSendInfo.addRecipient(StringUtils.split(recvEmails,","), StringUtils.split(recvNames,","));
  //수신자 추가(1명씩 추가시)
  //mailSendInfo.addRecipient("abc@abc.com", "홍길동");
  
  // 메일 파라미터, 메일 템플릿에서 사용하는 변수명-값
  mailSendInfo.putCotentsParam("USER_NM",    "홍길동");
  mailSendInfo.putCotentsParam("CTRT_TITLE", "1월물품구매계약");
  
  // 메일에 첨부할 파일(옵션), 업로드된 첨부파일ID
  mailSendInfo.setAttachFileId("FILE_000000000000192");
  
  // 관련 업무데이타 참조값( 예:계약번호 )
  mailSendInfo.setRefVal1("CTRT00001");
  
  // 메일전송정보 테이블에 등록.(이후 메일배치에서 전송처리)
  Long mailSendSeq = mailService.registMailSendInfo(mailSendInfo); //메일전송일련번호 리턴
  LOG.debug("메일전송일련번호={}", mailSendSeq);
  ```

  

- 업무 프로그램에서 전송한 메일은 EF_MAIL_SEND 테이블에 저장되고, 메일전송 배치 프로그램에 의해 실제로 전송처리된다.

## 배치 개발

- 샘플 소스 : MailSendBatchService 참고

- 배치 주기는 java소스에서 @Scheduled 어노테이션에 cron 표현식으로 정의

- 시스템 관리 > 배치관리 > 배치정보 관리 에서 배치정보 등록(배치id, 배치명, 수행서버 등)

  - 수행서버 정보는 배치가 수행되어야 할 서버의 호스트명을 ','로 구분해서 입력.
  - 운영환경에서는 여러 서버에 등록되어 있으면 데이타 처리에 문제가 발행할 수 있으므로 1개 호스트만 등록
  - 기본 실행서버는 standby 서버에서 배치가 수행되도록 설정.

- 시스템 관리 > 배치관리 > 배치결과 관리에서 배치 수행결과 조회 및 재실행




## 파일 업로드

### Dext5 Upload

#### 환경설정

- http://www.dext5.com/ 에서 임시라이선스 발급
  - 사용도메인 : localhost,higis.oilbank.co.kr,devhigis.oilbank.co.kr,172.18.61.76,172.18.61.77
- 라이선스 등록
  - src\main\webapp\resources\plugin\dext5upload\config\dext5upload.config.xml

- 업로드 처리 핸들러(커스트마이징)
  - higis\src\main\webapp\plugin\dext5handler.jsp
- 업로드파일 DB 저장 Service
  - net.ecbank.fwk.common.service.DEXT5UploadService.java

#### 파일저장 방식이 각각 DB와 파일인 경우에 따른 차이 부분.

- DEXT5UploadService.saveFilesInfo(...) : 파일별로 파일정보 저장하는 부분에서 DB저장방식인 경우에는 업로된 물리파일 삭제 로직.

- FileManageDAO.insertFileInf(...) :  DB저장방식인 경우 물리파일의 byte를 CLOB필드 파라미터로 셋팅하는 로직.

- dext5handler.jsp : event.addOpenDownloadPathChangeEventListenerEx(다운로드 경로변경 이벤트) : DB로 파일저장하는 경우에는 다운로드용 임시파일 생성 그 경로를 넘겨줌.

- dext5handler.jsp : event.addOpenDownloadCompleteEventListenerEx (다운로드 완료 메소드) : DB로 파일저장하는 경우에는 다운로드용 임시생성파일 삭제하는 로직.

  

## DTO 및 Mpper 방식 개발

기존 application layer 간 데이타 이동 및 조회,저장 시 Map Data를 이용하던 것을 DTO 방식으로 적용하고, MyBatis Mapper Interface 방식을 적용하는 개발방법에 대한 추가적인 가이드.

#### 환경설정

##### Lombok 라이브러리 및 IDE 설정

- pom.xml : dependency 추가

```xml
		<dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
            <scope>provided</scope>
      	</dependency>
        
		<dependency>
		    <groupId>org.projectlombok</groupId>
		    <artifactId>lombok-maven-plugin</artifactId>
		    <version>1.18.20.0</version>
		    <scope>provided</scope>
		</dependency>
```

- C:\higis\eGovFrameDev-3.9.0-64bit\eclipse 디렉토리에 lombok.jar 파일 추가 후 아래 명령어로 설치

```shell
C:\higis\eGovFrameDev-3.9.0-64bit\eclipse > java -jar lombok.jar
```

##### Mybatis Mapper Scan 설정

- context-mapper.xml

```xml
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="net.ecbank,higis" />
	</bean>
```

#### DTO 개발

dto 클래스는 net.ecbank.fwk.common.BaseObject 를 상속하여 작성하고, 업무별 패키지 내의 domain 패키지에 생성한다.

getter/setter 메소드는 Lombok 어노테이션을 활용하여 적용한다.

검색조건은 별도의 xxxCondtion 클래스로 별도 작성하여 사용하는 것 권장한다.

- 일반 dto 클래스 : 그리드 목록 데이타 저장은 json 필드명의 Map<String, List> 멤버변수를 사용한다.

```java
@Data
@EqualsAndHashCode(callSuper=false)
public class Author extends BaseObject {
	/**
	 * 작가ID
	 */
	private long authorId;
	/**
	 * 작가명
	 */
	private String authorNm;
    
    /**
	 * 목록 저장 데이타 용.
	 * : 그리드에서 수정한 데이타가 각각 CREATED, DELETED, UPDATED key값의 List 데이타로 넘어온다.
	 */
	Map<String, List<Author>> saveList;
	
	/**
	 * 상세화면 책 목록 그리드 저장용.
	 */
	Map<String, List<Book>> saveBookList;]()
```

- 검색조건 dto : form, to 조건이 많으므로 별도의 클래스로 작성. 복수 셀렉트 값 처리는 String[] 배열을 리턴하는 추가 메소드 작성하여 사용.

```
@Data
@EqualsAndHashCode(callSuper=false)
public class AuthorCondition extends BaseObject {
	
	private String authorNm;
	private String debutYearBgn;
	private String debutYearEnd;
	private String registSttus;
	
	/**
	 * 복수 선택 selectbox 값을 배열로 리턴.
	 * @return
	 */
	public String[] getRegistSttusList() {
		if (StringUtils.isEmpty(this.registSttus)) {
			return null;
		}else {
			return StringUtils.split(this.registSttus,",");
		}
	}
```

- 복수의 테이블을 조인하여 여러 테이블의 칼럼들을 목록으로 보여주는 경우는 대응하는 별도 dto를 생성하여 구현하는 방식을 권장한다.

  (예: AuthorBookList)

#### Mapper Interface 개발

mapper interface 는 업무별 패키지 내 mapper 패키지에 생성하고, @Mapper (egovframework.rte.psl.dataaccess.mapper.Mapper) 어노테이션을 붙인다.

mapper interface는 업무별로 주요 테이블 그룹 단위로 생성한다.

```
@Mapper
public interface AuthorBookMapper {
	
	/**
	 * 작가 목록 조회
	 * @param condition
	 * @return
	 */
	List<Author> selectAuthorList(AuthorCondition condition);
	/**
	 * 작가 정보 조회
	 * @param authorId
	 * @return
	 */
	Author selectAuthor(String authorId);
	/**
	 * 작가 정보 수정
	 * @param author
	 */
	void updateAuthor(Author author);
```

#### Mybatis Sql 작성

sql 파일명은 "대응하는 Mapper Interface 명__SQL.xml" 로 명명하고, namespace는 대응하는 Mapper Interface full pacakge 경로로 사용한다.

칼럼 underscore  Naming과 result Dto 멤버변수의 camelCase Naming이 일치한다면 별도 resultMap을 정의할 필요는 없고, naming이 다르거나 데이타타입 변환의 명시가 필요한 경우는 resultMap을 정의한다.

```xml
<mapper namespace="net.ecbank.sample.mapper.AuthorBookMapper">
	
<select id="selectAuthorList" parameterType="net.ecbank.sample.domain.AuthorCondition" resultType="net.ecbank.sample.domain.Author">
	--AuthorBookMapper_SQL_oracle.xml selectAuthorList 작가목록 조회
	SELECT AUTHOR_ID AS AUTHOR_VIEW --그리드PUPUPLINK용
	     , AUTHOR_ID        --작가ID
	     , LAST_MOD_DT      --최종수정시점
         , LAST_MOD_ID      --최종수정자ID
         , NVL(U.USER_NM, LAST_MOD_ID) AS LAST_MOD_NM --최종수정자명
	  FROM Z_AUTHOR A LEFT OUTER JOIN EF_USER_VIEW U ON A.LAST_MOD_ID = U.USER_ID
	 WHERE 1=1
	 <if test='authorNm != null and authorNm != ""'>
       AND AUTHOR_NM LIKE '%'||#{authorNm}||'%'
     </if>
     <if test='debutYearBgn != null and debutYearBgn != ""'>
       AND DEBUT_YEAR <![CDATA[>=]]> #{debutYearBgn}
     </if>
     <if test='debutYearEnd != null and debutYearEnd != ""'>
       AND DEBUT_YEAR <![CDATA[<=]]> #{debutYearEnd}
     </if>
     <if test="registSttusList != null and registSttusList.length != 0" >
       AND REGIST_STTUS IN
       <foreach collection="registSttusList" item="item" index="index" separator="," open="(" close=")">
         #{item}
       </foreach>
     </if>
     ORDER BY AUTHOR_ID DESC
	</select>
```

Date 타입 칼럼의 데이타의 그리드 목록에 표시할 때는 해당 dto에 yyyymmddhhmmss 포맷의 string을 별도 선언하고, 그리드에서는 해당 필드명에서 적절한 renderer를 적용한다.(예: BaseObject의 frstCrtDt, lastModDt)

```java
@Data
public class BaseObject implements Serializable {  
    
    private LocalDateTime lastModDt;
    
    /**
	 * 최종수정일시 객체의 yyyyMMddhhmmss 포맷형식의 문자열 값
	 * @return
	 */
	public String getLastModDtStr() {
		if (this.lastModDt != null) {
			return lastModDt.format(DateTimeFormatter.ofPattern("yyyyMMddhhmmss"));
		}else {
			return "";
		}
	}
```

```javascript
,{fieldName: "lastModDtStr",header: {text: "최종수정일시"}, dataType: "text", width:"150", rendererType:'yyyymmddhhmmss', editable: false}
```

#### Controller 작성

@RequestBody에 해당 dto를 선언하여 request json body 데이타를 binding한다.

```java
    /**
	 * 작가 목록 데이타 조회
	 * @param condition
	 * @param request
	 * @param model
	 * @return
	 */
	@RequestMapping("/sample2/selectAuthorSampleList.do")
	@ResponseBody
	public JsonData selectAuthorList(@RequestBody AuthorCondition condition, HttpServletRequest request, ModelMap model) {

		JsonData jsonData = new JsonData();

		List<Author> dataList = authorSampleService.selectAuthorList(condition);
		jsonData.setPageRows(new HashMap(), dataList, dataList!=null ? dataList.size() : 0);

		return jsonData;
	}
	
	/**
	 * 작가 등록
	 * @param author
	 * @param request
	 * @param model
	 * @return
	 */
	@RequestMapping("/sample2/registAuthor.do")
	@ResponseBody
	public JsonData registAuthor(@RequestBody Author author, HttpServletRequest request, ModelMap model) {
		JsonData jsonData = new JsonData();

		Map<String,Object> resultMap = authorSampleService.registAuthor(author);
		jsonData.addFields("result", resultMap);

		return jsonData;
	}
```

목록 형태의 데이타는 해당 Dto에 List<?> 멤버변수를 선언하고 해당 멤버변수명을 키 값으로 json데이타를 받아 binding한다.

```
@Data
@EqualsAndHashCode(callSuper=false)
public class Author extends BaseObject {
	/**
	 * 목록 저장 데이타 용.
	 * : 그리드에서 수정한 데이타가 각각 CREATED, DELETED, UPDATED key값의 List 데이타로 넘어온다.
	 */
	Map<String, List<Author>> saveList;
```

```js
	var params = fnGetParams();
    $.extend(params, {"saveList":ecRealGrid.fn_getModifyData(gridView)});
```

```
	    Map<String, List<Author>> saveList = author.getSaveList();
		
		List<Author> updatedList = (List<Author>)MapUtils.getObject(saveList, "UPDATED");
		for (Author a : updatedList) {
			LOG.debug("UPDATED author={}", a);
		}
		
		List<Author> createdList = (List<Author>)MapUtils.getObject(saveList, "CREATED");
		for (Author a : createdList) {
			LOG.debug("CREATED author={}", a);
		}
		
		List<Author> deletedList = (List<Author>)MapUtils.getObject(saveList, "DELETED");
		for (Author a : deletedList) {
			LOG.debug("DELETED author={}", a);
		}
```



#### Service에서 Mapper호출

mapper 방식의 repostory 클래스를 호출할 때 쿼리 파라미터에 세션정보 및 시스템환경정보가 필요하다면 직접 myBatisParamAspect.addSessionParam 메소드로 추가한 후 호출한다. (파라미터 변수명은 이전 MyBatis Sql 작성 부분 참고. BaseObject 멤버변수에 정의됨.)

myBatisParamAspect는 상위 BaseService의 멤버변수로 @Autowired되어 있음.

```java
authorBookMapper.insertAuthor((Author)myBatisParamAspect.addSessionParam(author));
```

