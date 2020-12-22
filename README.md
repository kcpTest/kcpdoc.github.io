---
description: 이 문서는 가맹점 사이트에서 연동 가이드에 따라 PG결제 연동 시스템을 개발하는 방법에 대해 설명합니다.
---

# 개요 및 연동 전 준비사항

**문서개요**

* 가맹점에서 구현해야 하는 API 명세 가이드
* 서비스 설명 및 NHN KCP 권고 사항 설명
* 각 샘플 페이지 해당 역할 및 파라미터 상세 안내

**독자**

* 이 문서의 독자는 NHN KCP PG결제를 연동하는 가맹점 사이트 개발자입니다.

**문의처**

이 문서의 내용에 오류가 있거나 내용과 관련한 의문 사항이 있으시면 아래 연락처로 문의 바랍니다.

* NHN KCP 기술지원팀 : 1544-8661 혹은 support@kcp.co.kr

## 연동 전 준비사항

### 1.공통 안내 및 권장사항

* KCP 인코딩 방식은 **EUC-KR** 형식을 사용하며, 이에 따른 리턴 값도 **EUC-KR**로 제공합니다.
* 결제로그\(log\)는 기본적으로 관리를 권장합니다. \(※ 오류 거래 건에 대한 추적이 필요할 수 있음\)
* 통신방식은 샘플소스 기준으로 bin\pp\_cli \(실행파일\)를 통해 가맹점 서버와 소켓통신 합니다.\( JSP의 경우 lib\jPpcliE.jar 파일 \)
* 방화벽은 KCP와 결제 통신을 위해 TCP SOCKET을 사용합니다.
* 결과 요청 및 처리에 관한 변수처리는 기관의 정책이나 신규 서비스 출시 등에 따라 변경이 될 수 있으니 연동매뉴얼을 업데이트하여 관리하시기 바랍니다.
*  **NHN KCP 결제 모듈에는 데이터베이스 연동 작업을 위한 기능은 포함되어있지 않습니다.**

  \(데이터베이스 연동을 위한 지불 결과 데이터만 제공하며 데이터베이스 처리에 관한 부분은 일체 가맹점에서 관리\)

* 결과 요청 및 처리에 관한 변수처리는 기관의 정책이나 신규 서비스 출시 등에 따라 변경이 될 수 있으니 연동매뉴얼을 업데이트하여 관리하시기 바랍니다.
* 설치 Directory 는 절대 Web으로 접근할 수 있는 경로에 설치하지 마십시오.

  결제서비스 관련 폴더 및 파일들은 보안상 절대 Web을 통해서 접근하지 않도록 관리해주시기 바랍니다.

* 결제 창 호출 시 요청하는 site\_cd 와 승인 요청 시 site\_cd 가 다를 경우 정산 시 불이익을 받을 수 있습니다.
* 반드시 결제 인증 및 승인 시 동일한 site\_cd 로 요청하시기 바랍니다.

**※특히 사이트 키 값, 결제거래번호, 환경설정파일의 경우 가맹점과 NHN KCP간의 보안을 유지하는 값이므로 Web 상에서 확인될 수 없도록 관리 바랍니다.**

### 2.결제 창 호출 가능한 운영체제 및 브라우저 안내

① PC버전 OS : 표준WEB\(Windows & Mac\), Plugin\(Windows only\)  
② PC버전 Browser : IE\(Ver. 6 ~\), Firefox \(Ver. 5.0 ~\), Chrome \(Ver. 16.0 ~\), Safari\(Ver. 5.0 ~\), Opera \(Ver. 10~\)  
③ 모바일버전 OS : Android, IOS  
④ 모바일버전 Browser : 스마트폰 기본브라우저

**※ 방화벽 설정 정보**

**\[PC, Mobile 공통\]**

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#xC5F0;&#xACB0;&#xB300;&#xC0C1; PORT</th>
      <td style="text-align:left"><b>8090</b>
      </td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th style="text-align:left">&#xC5F0;&#xACB0;&#xB300;&#xC0C1; &#xB3C4;&#xBA54;&#xC778;</th>
      <td style="text-align:left">
        <p>paygw.kcp.co.kr (&#xC2E4; &#xACB0;&#xC81C;)</p>
        <p>testpaygw.kcp.co.kr (&#xD14C;&#xC2A4;&#xD2B8; &#xACB0;&#xC81C;)</p>
      </td>
    </tr>
  </tbody>
</table>

**\[Mobile 전용\]**

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>&#xC5F0;&#xACB0;&#xB300;&#xC0C1; PORT/&#xB3C4;&#xBA54;</b>
      </th>
      <td style="text-align:left">
        <p>smpay.kcp.co.kr 443/80 (&#xC2E4; &#xACB0;&#xC81C;)
        </p>
        <p>testsmpay.kcp.co.kr 443  (&#xD14C;&#xC2A4;&#xD2B8; &#xACB0;&#xC81C;)
        </p>
      </td>
    </tr>
  </thead>
  <tbody></tbody>
</table>

* NHN KCP 결제는 TCP Socket 통신으로 이루어지며, 추가로 모바일버전의 경우 결제 창 호출 전 SOAP 통신을 통해 주요 주문데이터 검증을 위해 거래등록을 진행합니다. 아래 도메인, PORT 에 대해 방화벽 허가를 해주셔야 합니다.



### 2.API 규격 안내 및 특수문자

해당 항목에서는 결제 시스템을 구축하기 위한 파라미터 소개 및 규격 안내 페이지입니다. 제공해 드리는 샘플소스와 함께 해당 파라미터를 참고하시어 연동하시기 바랍니다.

**데이터 타입**

| Column | Type | Description |
| :--- | :--- | :--- |
| 1 | Number | 숫자형 Integer : \(0~9\) |
| 2 | String | 문자형 String : Alpha numeric \(A-Z; 0-9; UTF-8 characters\) |
| 3 | Etc | 기타 Etc |

**사용불가 특수문자**

| 콤마 | 엠퍼센트 | 세미콜론 | 뉴 라인 | 역 슬래쉬 | 파이프 라인 | 작은 따옴표 | 큰 따옴표 | 부등호 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **,** | & | ; | \n | \ | \| | ' | " | &lt; |

※ 스크립트 언어 불가 ex\) &lt;script&gt;. eval;\(\(.\*\)\)

※ \u0000 : unicode null

요청 페이지에서 데이터를 입력할 때, 위와 같은 특수 문자를 입력할 경우 오류가 발생할 수 있습니다. 위 특수 문자 목록을 반드시 참고하여, 데이터를 입력할 때에 반드시 체크해 주시기 바랍니다.

### 3.보안지침서

하기 사항을 지키지 않아 발생하는 보안관련 사고에 대해서는 KCP에서 책임을 질 수 없는 부분이오니 반드시 결제모듈 연동 시 철저하게 가맹점 관리 부탁 드립니다.

**site\_key 관리 강화**

