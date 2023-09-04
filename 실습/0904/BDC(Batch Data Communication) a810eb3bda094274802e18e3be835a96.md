# BDC(Batch Data Communication)

생성자: 최예훈
생성 일시: 2023년 9월 4일 오후 12:42
태그: ABAP

### BDC

데이터를 SAP포멧에 맞는 형태로 생성한 후 SAP로 데이터를 전송하는것

### 왜 필요한가?

물론 사용자가 직접 테이블 데이터를 변경할 수 있지만(se11)

SAP에 사용되는 데이터는 관계형 DB를 기본으로 하기 때문에 데이터 특성을 고려하여 여러 테이블에 분산되어 저장되고 있다.

따라서 하나의 테이블의 데이터를 변경하는 것이 관계가 있는 다른 테이블에 어떤 영향을 주는지 알기 어렵기 때문에 

BDC를 사용하여 데이터를 변경한다.

### 사용법 (Call Transaction 이용)

```abap
CALL TRANSACTION <tcode>
			USING <bdc_tab>
			OPTIONS FROM <ctu_params>
			MODE <mode>
			UPDATE <update>
	MESSAGES INTO MESSTAB.
```

<tcode> : 트랜젝션 코드

<bdc_tab> : BDCDATA 구조의 인터널 테이블을 트랜젝션에 전달

<ctu_params> : 옵션 설정

### 예시

```abap
FORM evnt_hotspot_click1  USING    p_e_row_id TYPE lvc_s_row
                                   p_e_column_id TYPE lvc_s_col
                                   p_es_row_no TYPE lvc_s_roid.

  FIELD-SYMBOLS: <ls_data> LIKE LINE OF gt_list.
  DATA: lt_bdc TYPE bdcdata_tab.  " 레코딩 정보를 담는 인터널 테이블
  DATA: ls_param TYPE ctu_params. " 트랜젝션 호출 시 옵션

  ls_param-dismode = 'E'.   " 에러가 발생할 경우에만 화면 조화
  ls_param-updmode = 'S'.   " 동기 업데이트
  ls_param-nobinpt = 'X'.   " BATCH INPUT MODE 사용안함

  CLEAR: lt_bdc[].
  CASE p_e_column_id-fieldname.
    WHEN 'MATNR'.     " MATNR 칼럼이 클릭되었을 경우

      READ TABLE gt_list ASSIGNING <ls_data> INDEX p_e_row_id-index.   " 결과 테이블에서 클릭된 row의 데이터를 필드심볼에 할당
      IF <ls_data> IS ASSIGNED.
        PERFORM dynpro TABLES lt_bdc USING:
          'X'  'SAPLCSDI'         '0100',
          ' '  'BDC_OKCODE'        '/00',
          ' '  'RC29N-MATNR'      <ls_data>-matnr,
          ' '  'RC29N-WERKS'      <ls_data>-werks,
          ' '  'RC29N-STLAN'      <ls_data>-stlan,
          ' '  'RC29N-STLAL'      <ls_data>-stlal,
          ' '  'RC29N-DATUV'      <ls_data>-datuv.

        CALL TRANSACTION 'CS03' USING lt_bdc OPTIONS FROM ls_param.   " BDCDATA 구조의 인터널 테이블을 CS03 트랜잭션에 전달하여 최상위 모품목의 정보 조회

        UNASSIGN <ls_data>.
      ENDIF.
  ENDCASE.

ENDFORM.                    " EVNT_HOTSPOT_CLICK1
```

```abap
FORM dynpro TABLES output STRUCTURE bdcdata
                      USING
                            p_dynbegin
                            p_1
                            p_2.

  CLEAR output.
  IF p_dynbegin = 'X'.
    output-dynbegin = 'X'.   " BDC 화면 시작
    output-program = p_1.    " BDC 모듈 풀
    output-dynpro = p_2.     " BDC 화면번호
  ELSE.
    output-fnam = p_1.       " 필드 이름
    output-fval = p_2.       " BDC 필드
    CONDENSE output-fval.
  ENDIF.

  APPEND output.

ENDFORM.                    " DYNPRO
```

### 레코딩 작업

BDC프로그램을 만들기 위해 트랜젝션 레코딩을 하여 전송한 데이터가 어떤 트랜젝션을 수행해야 하는지 결정