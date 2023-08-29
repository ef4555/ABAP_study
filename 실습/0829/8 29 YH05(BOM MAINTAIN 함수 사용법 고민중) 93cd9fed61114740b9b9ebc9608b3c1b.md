# 8/29 YH05(BOM MAINTAIN 함수 사용법 고민중)

생성자: 최예훈
생성 일시: 2023년 8월 29일 오후 5:15
태그: 실습

## MAIN

```abap
*&---------------------------------------------------------------------*
*& Report  ZPPR_BOM_YH05
*&
*&---------------------------------------------------------------------*
*&
*&
*&---------------------------------------------------------------------*
* yh03과 달라진 점
*class 부분 정리
*AVL 툴바 버튼 추가

REPORT  zppr_bom_yh05.

INCLUDE zppr_bom_yh05_top.
INCLUDE zppr_bom_yh05_cls. " 클래스 정의
INCLUDE zppr_bom_yh05_scr. " 1000번 스크린(조회 조건 화면)
INCLUDE zppr_bom_yh05_pbo. " 스크린 100번의 PBO
INCLUDE zppr_bom_yh05_pai. " 스크린 100번의 PAI
INCLUDE zppr_bom_yh05_frm. " 서브루틴 정리

*----------------------------------------------------------------------*
* INITIALIZATION                                                       *
*----------------------------------------------------------------------*
INITIALIZATION.
  PERFORM init_before_run. " 초기값을 넣어주어야 할 때 사용

*----------------------------------------------------------------------*
* START OF SELECTION                                                   *
*----------------------------------------------------------------------*
START-OF-SELECTION.

  PERFORM init_data. " 사용할 인터널 테이블, 구조체 초기화
  PERFORM select_data. " 조회값에 기반하여 BOM 정보 불러옴

*----------------------------------------------------------------------*
* END OF SELECTION                                                     *
*----------------------------------------------------------------------*
END-OF-SELECTION.
  CALL SCREEN 0100. " 조회 결과 ALV 를 담을 화면 100번 불러옴
```

## TOP

```abap
*&---------------------------------------------------------------------*
*&  Include           ZPPR_BOM_YH05_TOP
*&---------------------------------------------------------------------*

INCLUDE zppr_bom_yh04_zkutil.

TABLES : marc, " 자재에 대한 플랜트 데이터
         mast, "  BOM 연결 자재
         mara, " 일반 자재 데이타
         stko, " BOM 헤더
         stpo, " BOM 품목
         t001w, " 플랜트 상세 정보
         zpp001yh. " 생성한 CBO 테이블

TYPES : BEGIN OF result_type,

        matnr TYPE mara-matnr, " 상위자재코드
        maktx TYPE makt-maktx, " 자재내역
        werks TYPE marc-werks, " 플랜트
        stlan TYPE stpox-stlan, " 용도
        stlal TYPE mast-stlal, " 대체 BOM

        stlst TYPE stko-stlst, " BOM 상태
        mtart TYPE mara-mtart, " 자재 그룹
        stufe TYPE stpox-stufe, " 레벨
        matnr_m TYPE mara-matnr, " 구성부품의 상위 모품목 자재코드
        matnr_sub TYPE mara-matnr, " 구성부품의 자재코드
        ojtxp TYPE stpox-ojtxp, " 구성부품 내역

        menge TYPE stpox-menge, " 수량
        meins TYPE stpox-meins, " 단위
        datuv TYPE stpox-datuv, " 효력 시작일
        bmeng TYPE stko-bmeng, " 기준수량
        bmein TYPE stko-bmein, " 단위

        ausch TYPE stpox-ausch, " 구성부품스크랩
        sanka TYPE stpox-sanka, " 원가계산관계도

        stlnr TYPE stko-stlnr, " BOM

        posnr TYPE stpox-posnr, " BOM 품목번호

        index(18) TYPE c,
        linecolor(4) TYPE c, " 라인 색상 지정

        END OF result_type.

DATA : it_base TYPE TABLE OF result_type, " 초기화면 조건과 mara, marc, makt, mast 테이블을 이용해 BOM정보를 취득하여 저장할 인터널 테이블 (대상 필드는 화면 레이아웃의 테이블/필드 참조)
       it_base_str TYPE result_type,  " it_base는 헤더라인 없으므로 it_base_str 구조체 생성해주어야 함
       gt_list TYPE TABLE OF result_type WITH HEADER LINE. " 모든 자재에 대한 결과값을 담아 출력할 인터널 테이블

FIELD-SYMBOLS <it_base_fs> LIKE it_base_str. " 필드 심볼 선언
*-----------------------------------------------------------------
* 컨테이너, 그리드 관련 변수
DATA : con1_dock TYPE REF TO cl_gui_docking_container, " 도킹 컨테이너
       g_grid1 TYPE REF TO cl_gui_alv_grid, " 그리드
       g_grid2 TYPE REF TO cl_gui_alv_grid. " 그리드2

DATA : gv_splitter TYPE REF TO cl_gui_splitter_container,
       con1 TYPE REF TO cl_gui_container, " TOP-OF-PAGE 들어갈 컨테이너
       con2 TYPE REF TO cl_gui_container, " ALV 들어갈 컨테이너
       g_dyndoc_id TYPE REF TO cl_dd_document.

*-----------------------------------------------------------------
* ALV 레이아웃 관련 변수
DATA : gs_variant TYPE disvariant, " ALV 레이아웃 저장하기 위한 변수

       gs_layout TYPE lvc_s_layo, " ALV 레이아웃 지정
       gt_fieldcat TYPE lvc_t_fcat, " ALV 필드 카탈로그 인터널 테이블
       gs_fieldcat TYPE lvc_s_fcat. " ALV 필드 카탈로그 구조체

FIELD-SYMBOLS <gt_fieldcat_fs> LIKE gs_fieldcat. " 필드 심볼 선언

*------------------------------------------------------------------
* PAI 관련 변수
DATA : ok_code TYPE sy-ucomm. " 버튼 입력 받기 위한 변수

DATA : gv_edit_control TYPE int4 VALUE 0. " 조회/수정모드 나타내는 변수

** EDIT 관련 변수
DATA : gt_modified_rows TYPE TABLE OF result_type, " 수정한 row를 담을 변수
       gv_report_name TYPE sdydo_text_element VALUE 'BOM 조회',
       gv_edit_color(4) TYPE c VALUE 'C500'. " EDIT시 색 변경

DATA : gt_convert_itab TYPE TABLE OF zpp001yh,
       gt_changed_itab TYPE TABLE OF zpp001yh.
```

## CLASS

```abap
*&---------------------------------------------------------------------*
*&  Include           ZPPR_BOM_YH05_CLS
*&---------------------------------------------------------------------*
*----------------------------------------------------------------------*
*       CLASS lcl_event DEFINITION
*----------------------------------------------------------------------*
*
*----------------------------------------------------------------------*
CLASS lcl_event DEFINITION.
  PUBLIC SECTION.
    METHODS : top_of_page
         FOR EVENT top_of_page OF cl_gui_alv_grid " cl_gui_alv_grid에 정의된 TOP-OF-PAGE 이벤트를 감지
                  IMPORTING e_dyndoc_id. " 이벤트 정의 시 IMPLEMENTATION구절에서 사용할 수 있는 e_dyndoc_id.라는 인스턴스 생성함.(명시하지 않아도 상관 X지만 명시 하지 않으면 메소드에서 사용불가)

*----------------------------------
    METHODS: handle_data_changed
          FOR EVENT data_changed OF cl_gui_alv_grid " " cl_gui_alv_grid에 정의된 data_changed 이벤트를 감지
                   IMPORTING er_data_changed sender. " 이벤트 정의 시 IMPLEMENTATION구절에서 사용할 수 있는 er_data_changed라는 인스턴스 생성.(명시하지 않아도 상관 X지만 명시 하지 않으면 메소드에서 사용불가)
*----------------------------------

    METHODS : handle_double_click
          FOR EVENT double_click OF cl_gui_alv_grid "  cl_gui_alv_grid 클래스에서 이벤트 따옴
                   IMPORTING e_row
                             e_column.

    METHODS : handle_hotspot_click
          FOR EVENT hotspot_click OF cl_gui_alv_grid "  cl_gui_alv_grid 클래스에서 이벤트 따옴
                   IMPORTING e_row_id
                             e_column_id.

    METHODS : handle_toolbar " 툴바 정의
          FOR EVENT toolbar OF cl_gui_alv_grid
                   IMPORTING e_object
                             e_interactive.

    METHODS : handle_user_command  " 임의로 생성한 툴바 동작 정의
          FOR EVENT user_command OF cl_gui_alv_grid
                    IMPORTING e_ucomm.
ENDCLASS.                    "lcl_event DEFINITION

*----------------------------------------------------------------------*
*       CLASS lcl_event IMPLEMENTATION
*----------------------------------------------------------------------*
*
*----------------------------------------------------------------------*
CLASS lcl_event IMPLEMENTATION.
  METHOD top_of_page.
    PERFORM set_top_page USING e_dyndoc_id. " 정의하는 부분에서 IMPORTING했던 e_dyndoc_id.
  ENDMETHOD.                    "top_of_page

  METHOD handle_data_changed. " data_change 이벤트가 발생되면 아래 로직을 수행
    CLEAR : gt_changed_itab.
    DATA : lv_num(18) TYPE c,
           ls_outtab TYPE result_type.

    FIELD-SYMBOLS: <fs> TYPE table.

*   데이터 변화가 일어났을 때
*   변경된 데이터는 ALV의 output인 gt_list의 result_type으로 되어있음
*   따라서 CBO 테이블의 형식으로 변형해서 저장해야 한다.

*   er_data_changed : ALV 그리드의 수정 내역을 저장하기 위해 이벤트가 생성하는 인스턴스
*   er_data_changed의 mp_mod_rows 메서드를 작동. 변화된 row가 담긴 인터널 테이블의 data refrence가 나온다.
    ASSIGN er_data_changed->mp_mod_rows->* TO <fs>. " 변경된 row 전체를 <fs>에 할당
*    이 row를 CBO 테이블 형식으로 변환
    LOOP AT <fs> INTO ls_outtab.
      MOVE-CORRESPONDING ls_outtab TO zpp001yh.
      zpp001yh-mandt = sy-mandt.
      zpp001yh-menge = zpp001yh-menge * 1000.
      zpp001yh-lines = sy-tabix.
      APPEND zpp001yh TO gt_changed_itab. " 데이터 변경이 감지되면 변경된 데이터를 gt_changed_itab에 저장함.
    ENDLOOP.

    BREAK-POINT.

  ENDMETHOD.                    "handle_data_changed

  METHOD handle_double_click.
    IF gv_edit_control = 0.
      gv_edit_control = 1.

    ELSEIF gv_edit_control = 1.
      gv_edit_control = 0.

    ENDIF.

*    PERFORM col_line.
    PERFORM refresh_alv.
    PERFORM display_top_of_page.

  ENDMETHOD.                    "handle_double_click

  METHOD handle_hotspot_click.
* 설정 안함
  ENDMETHOD.                    "handle_hotspot_click

  METHOD handle_toolbar. " 툴바 설정
    PERFORM toolbar_part USING e_object e_interactive.
  ENDMETHOD.                    "handle_toolbar

  METHOD handle_user_command. " 툴바 버튼 구현
    PERFORM user_command_toolbar USING e_ucomm.
  ENDMETHOD.                    "handle_toolbar

ENDCLASS.                    "lcl_event IMPLEMENTATION

DATA : gr_varifier TYPE REF TO lcl_event .
```

## SCR

```abap

*&---------------------------------------------------------------------*
*&  Include           ZPPR_BOM_YH05_SCR
*&---------------------------------------------------------------------*
SELECTION-SCREEN BEGIN OF BLOCK sel_01 WITH FRAME TITLE text-001.

PARAMETERS : p_datuv LIKE stko-datuv DEFAULT sy-datum, " 효력 시작일
             p_werks LIKE marc-werks OBLIGATORY DEFAULT 'M960'. " 플랜트

SELECT-OPTIONS :  s_matnr FOR mara-matnr VISIBLE LENGTH 10, " 자재코드
                  s_stlan FOR mast-stlan VISIBLE LENGTH 10, " 용도
                  s_stlal FOR mast-stlal, " 대체 BOM "
                  s_mtart FOR mara-mtart DEFAULT 'ZFRT'. " 자재 유형

SELECTION-SCREEN END OF BLOCK sel_01.

SELECTION-SCREEN BEGIN OF BLOCK sel_02 WITH FRAME TITLE text-002.

PARAMETERS : p_vari LIKE disvariant-variant.

SELECTION-SCREEN END OF BLOCK sel_02.

* 레이아웃 선택
AT SELECTION-SCREEN ON VALUE-REQUEST FOR p_vari.
  gs_variant-report = sy-repid.
  CALL FUNCTION 'REUSE_ALV_VARIANT_F4'
    EXPORTING
      is_variant    = gs_variant
      i_save        = 'A'
    IMPORTING
      es_variant    = gs_variant
    EXCEPTIONS
      not_found     = 1
      program_error = 2
      OTHERS        = 3.

  IF sy-subrc = 0.
    p_vari = gs_variant-variant.
  ENDIF.
```

## PBO

```abap
*&---------------------------------------------------------------------*
*&  Include           ZPPR_BOM_YH05_PBO
*&---------------------------------------------------------------------*

MODULE STATUS_0100 OUTPUT.
  SET PF-STATUS 'B100'.
  SET TITLEBAR 'YHTITLE'.
ENDMODULE.                 " STATUS_0100  OUTPUT

MODULE set_alv OUTPUT.

  IF con1_dock is INITIAL. " 처음 ALV가 생성되었을 때.

    PERFORM alv_obj. " 컨테이너와 그리드 생성
    PERFORM alv_layout_setting. " ALV 레이아웃 설정
    PERFORM set_fieldcat. " 필드 카탈로그 설정
    PERFORM alv_display. " ALV 출력

  ELSE. " 데이터 바뀌어서 refresh 될 때

    PERFORM refresh_alv. " ALV 새로고침
    PERFORM display_top_of_page. " TOP-OF-PAGE 새로고침

  ENDIF.

ENDMODULE.                 " SET_ALV  OUTPUT
```

## PAI

```abap
*&---------------------------------------------------------------------*
*&  Include           ZPPR_BOM_YH05_PAI
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0100  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_0100 INPUT.
  DATA : lv_answer TYPE c.

  CASE ok_code.
    WHEN 'BACK' OR 'CANC'.
      LEAVE TO SCREEN 0. " 이전 화면으로 가세옹

    WHEN 'EXIT'.
      PERFORM exit_check.

    WHEN 'SAVE'. " 저장 버튼 눌렀을 때
      PERFORM data_save. " CBO 테이블에 저장.

    WHEN 'BOMEDIT'. " 변경 버튼 눌렀을 때
      PERFORM bom_edit. " BOM 변경사항 반영

    WHEN 'EDITMODE'.
      gv_edit_control = 1. " 수정 모드로 변경
*        PERFORM col_line." 라인의 색 변경

    WHEN 'REPORTMODE'.
      gv_edit_control = 0. " 조회 모드로 변경
*        PERFORM col_line." 라인의 색 변경

    WHEN OTHERS.
      MESSAGE 'ERROR' TYPE 'E'.

  ENDCASE.

ENDMODULE.                 " USER_COMMAND_0100  INPUT
```

## SUBROUTINE

```abap
*&---------------------------------------------------------------------*
*&  Include           ZPPR_BOM_YH05_FRM
*&---------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Form  SELECT_DATA
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM select_data.

  PERFORM get_werks_name. " 작업장 이름 불러옴
  PERFORM sel_bom_data. " 조건값에 맞는 BOM 정보 조회

  LOOP AT it_base ASSIGNING <it_base_fs>. " 입력받은 자재번호 range를 순회하면서 조회
    PERFORM bom_expl. " 하위 BOM 조회
  ENDLOOP.

  PERFORM msg. " 조회중 메시지 출력
ENDFORM.                    " SELECT_DATA
*&---------------------------------------------------------------------*
*&      Form  INIT_DATA
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM init_data .
  CLEAR :  con1_dock, g_grid1,
           gs_variant, gs_layout,
           gs_fieldcat, gt_fieldcat,
           gt_list, gt_list[],
           it_base, it_base_str.

ENDFORM.                    " INIT_DATA

*&---------------------------------------------------------------------*
*&      Form  get_werks_name
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM get_werks_name .
  SELECT name1 INTO CORRESPONDING FIELDS OF t001w
    FROM t001w
    WHERE werks = p_werks.
  ENDSELECT.
ENDFORM.                    " GET_WERKS_NAME
* 사용할 인터널 테이블, 구조체 초기값 넣어줌
FORM init_before_run.

ENDFORM.                    "INIT_BEFORE_RUN

*&---------------------------------------------------------------------*
*&      Form  loading
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM loading.

*  DATA : lv_text(50) TYPE c.

*  lv_text = 'BOM 정보를 조회중입니다... '.
*
*  CALL FUNCTION
*    'SAPGUI_PROGRESS_INDICATOR'
*    EXPORTING
*      text   = lv_text
*    EXCEPTIONS
*      OTHERS = 0.

ENDFORM.                    "loading

*&---------------------------------------------------------------------*
*&      Form  sel_bom_data
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
FORM sel_bom_data .

  SELECT b~matnr c~maktx a~werks a~stlan a~stlal a~stlnr INTO CORRESPONDING FIELDS OF TABLE it_base
   FROM mast AS a INNER JOIN mara AS b
                      ON a~matnr = b~matnr
                  INNER JOIN makt AS c
                      ON a~matnr = c~matnr
   WHERE a~werks = p_werks
       AND a~matnr IN s_matnr " 입력받은 자재코드
       AND a~stlan IN s_stlan " 입력받은 용도
       AND a~stlal IN s_stlal " 입력받은 대체 BOM
       AND b~mtart IN s_mtart  " 입력받은 자재 그룹
       AND c~spras = sy-langu. " 현재 시스템 언어

  SORT it_base BY stlnr. " stlnr 기준으로 정렬
  DELETE ADJACENT DUPLICATES FROM it_base COMPARING stlnr. "  stlnr(BOM)기준으로 중복 제거

  assist=>counting_total = LINES( it_base ). " zkutil의 assist include 활용
  assist=>counting_init( ).

ENDFORM.                    " SEL_BOM_DATA
*&---------------------------------------------------------------------*
*&      Form  MSG
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM msg .
  IF it_base IS NOT INITIAL. " it_base 인터널 테이블이 비어있지 않을 경우 = 조회값이 있을경우
    DATA : total TYPE i.
    total =  LINES( gt_list[] ). " 결과값의 총 row 수

    DATA : msg TYPE string.
    CONCATENATE `정상조회 되었습니다.     BOM 조회 결과 : `  `( `  INTO msg RESPECTING BLANKS.
    MESSAGE s001(00) WITH msg total '건)'. " total은 i 타입이라서 concatenate 불가

  ELSE.
    MESSAGE '조회할 데이터가 없습니다.' TYPE 'I'. " 조회값이 없습니다 팝업 출력
    LEAVE LIST-PROCESSING. " 리스트 처리를 중단하고 빠져나옴(조회 페이지로 다시 감)
  ENDIF.
ENDFORM.                    " MSG
*&---------------------------------------------------------------------*

*&---------------------------------------------------------------------*
*&      Form  BOM_EXPL
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM bom_expl.

  assist=>counting( 'BOM정보를 조회중입니다…' ).

  DATA : it_stpox TYPE TABLE OF stpox WITH HEADER LINE WITH NON-UNIQUE KEY idnrk, " BOM EXPL 결과 저장할 인터널 테이블
         ls_topmat TYPE cstmat, " 최상위 자재 정보 저장할 구조체
         it_cscmat TYPE TABLE OF cscmat WITH HEADER LINE, " BOM 카테고리를 저장할 인터널 테이블
         it_list TYPE SORTED TABLE OF result_type WITH HEADER LINE WITH UNIQUE KEY matnr. " 제품 기준으로 하위품목을 가져와서 저장할 인터널 테이블 SORTEDs

  CLEAR : it_list, it_list[], it_stpox, it_stpox[], ls_topmat, it_cscmat, it_cscmat[]. " 서브루틴에서 사용할 테이블들, 구조체들 초기화.

  CALL FUNCTION 'CS_BOM_EXPL_MAT_V2'
    EXPORTING
      capid                 = 'PP01' " 어플리케이션
      datuv                 = p_datuv " 효력시작일
      emeng                 = 0
      mehrs                 = 'X' " 다중 BOM 지시자 - 없으면 단일 레벨만 읽는다
      mtnrv                 = <it_base_fs>-matnr " 상위 자재 번호(모자재)
      werks                 = p_werks " 플랜트
      stlan                 = <it_base_fs>-stlan " BOM 용도
      stlal                 = <it_base_fs>-stlal " 대체 BOM
    IMPORTING
      topmat                = ls_topmat " 최상위 자재 번호를 저장
    TABLES
      stb                   = it_stpox[] " BOM EXPL 결과 저장
      matcat                = it_cscmat[] " 자재 카테고리
    EXCEPTIONS
      alt_not_found         = 1
      call_invalid          = 2
      missing_authorization = 4
      no_bom_found          = 5
      no_plant_data         = 6
      no_suitable_bom_found = 7
      conversion_error      = 8
      OTHERS                = 9.

* 4에서 취득한 it_stpox 기준으로 for all entries in 사용 하여, 제품 기준으로 하위 품목을 가지고 온다.120-0
* it_list = 가져온 하위 품목이 담긴 리스트
  SELECT ma~matnr kt~maktx
      INTO CORRESPONDING FIELDS OF TABLE it_list
      FROM mara AS ma INNER JOIN marc AS mc
                             ON mc~matnr = ma~matnr
                      INNER JOIN makt AS kt
                             ON ma~matnr = kt~matnr
    FOR ALL ENTRIES IN it_stpox
    WHERE ma~matnr = it_stpox-idnrk
    AND kt~spras = sy-langu.

*  SORT it_list BY matnr. " BINARY SEARCH 사용하기 위해 정렬

  LOOP AT it_stpox. " BOM EXPL에서 얻은 하위 자재 정보 순회하면서

    READ TABLE it_list WITH KEY matnr = it_stpox-idnrk. " 자재코드를 KEY로 하여 구성 부품 정보 가져와서 it_list의  matnr과 매칭하여 it_list의 헤더 라인에 정보를 담는다.

    " 이후 이 it_list 헤더라인에 원하는 정보를 담아서 수정
    it_list-werks = <it_base_fs>-werks. " 최상위 모품목 작업장
    it_list-matnr = <it_base_fs>-matnr. " 최상위 모품목 자재코드
    it_list-ojtxp = it_list-maktx.    " 구성부품 이름
    it_list-maktx = <it_base_fs>-maktx. " 최상위 모품목 이름
    it_list-stlan = <it_base_fs>-stlan. " 최상위 모품목 용도
    it_list-stlal = <it_base_fs>-stlal. " 최상위 모품목 대체 BOM
    it_list-stlst = ls_topmat-stlst. " 조회 제품의 대체 BOM
    it_list-stufe = it_stpox-stufe.    " 구성부품 레벨

    it_list-stlnr = it_stpox-stlnr.
    it_list-posnr = it_stpox-posnr.
*    it_list-ojtxp = it_stpox-ojtxp.    " 구성부품 이름(BOM 수정하고 저장하면 바로 반영 안되는 문제로 인해 바꿈 - 왜 bom expl이 갱신이 안될까?)

    READ TABLE it_cscmat WITH KEY INDEX = it_stpox-ttidx. " 자재 카테고리(it_cscmat)에서 BOM 단계(INDEX: 범주 테이블 내 인덱스)를 키로 하여 모품목의 자재코드를 가져옴
    it_list-matnr_m = it_cscmat-matnr. " 구성부품의 그 위의 모품목 자재코드
    it_list-matnr_sub = it_stpox-idnrk. " 구성부품의 자재코드
    it_list-menge = it_stpox-mngko.    " 구성부품수량
    it_list-meins = it_stpox-meins.    " 구성부품 단위
    it_list-mtart = it_stpox-mtart.    " 구성부품 용도
    it_list-datuv = it_stpox-datuv.    " 구성부품 효력 시작일
    it_list-stlal = it_stpox-stlal.    " 구성부품 대체 BOM
    it_list-ausch = it_stpox-ausch.    " 구성 부품 스크랩
    it_list-sanka = it_stpox-sanka.    " 원가 계산 관련 지시자

* 1레벨의 경우에만 기준수량을 가져온다."최상위 인경우 bom 헤더 수량을 가지고 온다
    IF it_list-stufe EQ 1.
      it_list-bmeng = ls_topmat-bmeng. " 기준 수량
      it_list-bmein = ls_topmat-bmein. " BOM의 기본 단위

* 1레벨이 아닌 경우 그 위의 모품목 수량을 가지고 온다.
    ELSEIF it_list-stufe NE 1.
      CLEAR it_stpox.
      READ TABLE it_stpox WITH KEY idnrk = it_list-matnr_m." 하위 자재코드의 모품목과 일치하는 정보를 it_stpox에서 불러옴
      it_list-bmeng    = it_stpox-menge. " 모품목의 수량
      it_list-bmein    = it_stpox-meins. " 모품목의 단위
    ENDIF.

    APPEND it_list TO gt_list[]. " it_list 헤더라인에 담긴 정보를 it_list2 인터널 테이블에 추가.

  ENDLOOP.

ENDFORM.                    " BOM_EXPL
*&---------------------------------------------------------------------*
*&      Form  SEL_BOM_DATA
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*

*&---------------------------------------------------------------------*
*&      Form  alv_obj
*&---------------------------------------------------------------------*
*       컨테이너, 그리드 오브젝트 생성
*----------------------------------------------------------------------*
FORM alv_obj .
  CREATE OBJECT con1_dock
    EXPORTING
      repid     = sy-repid " 컨트롤에 추가하고자 하는 프로그램 ID
      dynnr     = sy-dynnr " 컨트롤에 추가하고자 하는 스크린 번호
      side      = con1_dock->dock_at_bottom
      extension = 2000.

* 컨테이너 분할
* 1.도킹 컨테이너를 분할
* 2.분할한 row에 컨테이너 할당
  CREATE OBJECT gv_splitter
    EXPORTING
      parent  = con1_dock " 도킹 컨테이너를 분할
      rows    = 2      " 몇개의 행으로 나눌것인지
      columns = 1.   " 몇개의 열로 나눌것인지

  " TOP-OF-PAGE가 들어갈 컨테이너 con1
  CALL METHOD gv_splitter->get_container
    EXPORTING
      row       = 1 " gv_splitter로 나눈 컨테이너 row중 1번 row에 con1 할당
      column    = 1
    RECEIVING
      container = con1.

* ALV가 들어갈 컨테이너 con2
  CALL METHOD gv_splitter->get_container
    EXPORTING
      row       = 2 " gv_splitter로 나눈 컨테이너 row중 2번 row에 con2 할당
      column    = 1
    RECEIVING
      container = con2.

* 전체 컨테이너 높이 할당.
  CALL METHOD gv_splitter->set_row_height
    EXPORTING
      id     = 1
      height = 10.

* ALV
  CREATE OBJECT g_grid2 " ALV grid가 들어갈 컨테이너 지정.
    EXPORTING
      i_parent = con2.

*----------------------------------------------
* grid에 핸들러를 설정
  CREATE OBJECT g_dyndoc_id.

  CREATE OBJECT gr_varifier. " 이벤트 핸들러

  SET HANDLER gr_varifier->top_of_page FOR g_grid2. " top_of_page 이벤트를 감지하는 핸들러 g_grid2에 설정

*****----------------------
  SET HANDLER gr_varifier->handle_data_changed FOR g_grid2. " data_changed 이벤트를 감지하는 핸들러 g_grid2에 설정
*****----------------------

  SET HANDLER gr_varifier->handle_double_click FOR g_grid2. " double_click 이벤트를 감지하는 핸들러 g_grid2에 설정
  SET HANDLER gr_varifier->handle_hotspot_click FOR g_grid2. " hotspot_click 이벤트를 감지하는 핸들러 g_grid2에 설정
  SET HANDLER gr_varifier->handle_toolbar FOR g_grid2. " handle_toolbar 이벤트를 감지하는 핸들러 g_grid2에 설정
  SET HANDLER gr_varifier->handle_user_command FOR g_grid2. " handle_user_command 이벤트를 감지하는 핸들러 g_grid2에 설정

ENDFORM.                    " ALV_OBJ
*&---------------------------------------------------------------------*
*&      Form  ALV_LAYOUT_SETTING
*&---------------------------------------------------------------------*
*       ALV 레이아웃 요소 설정
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM alv_layout_setting.

  DATA : lv_title(20) TYPE c.
  CONCATENATE '플랜트' ' ' p_werks ' ' t001w-name1 INTO lv_title RESPECTING BLANKS. " respecting blanks를 해줘야 자동 trim 안생김
  gs_layout-zebra = 'X'.
*  gs_layout-cwidth_opt = 'X'. " 칼럼 길이 자동 조절
  gs_layout-grid_title =  lv_title. " 플랜트 이름
  gs_layout-smalltitle =  'X'. " ALV 타이틀을 작은 글씨로
  gs_layout-sel_mode =  'A'. " sel_mode를 설정
  gs_layout-info_fname = 'LINECOLOR'.
*  gs_layout-edit = 'X'. " ALV 전체 수정 가능하게 변경

ENDFORM.                    " ALV_LAYOUT_SETTING

*&---------------------------------------------------------------------*
*&      Form  SET_FIELDCAT
*&---------------------------------------------------------------------*
*       필드 카탈로그 레이아웃 설정
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM set_fieldcat .

  PERFORM append_field_catalog USING :
        'MATNR'        'X'  '상위 자재 코드'    ' '     ' '    ' '    'X'     ' '    ' '     ' ',
        'MAKTX'        ' '  '자재 내역'        '40'    ' '    'X'    ' '     ' '    ' '     ' ',
        'WERKS'        ' '  '플랜트'           ' '     ' '    ' '   ' '      ' '    ' '     ' ',
        'STLAN'        ' '  '용도'            ' '     ' '    'C'    'X'     ' '    ' '     ' ',
        'STLAL'        ' '  '대체 BOM'        ' '     ' '    'C'    'X'     ' '    ' '     ' ',
        'STLST'        ' '  'BOM 상태'        ' '     ' '    'C'    ' '     ' '    ' '     ' ',
        'MTART'        ' '  '자재 그룹'        ' '     ' '    'R'    ' '     ' '    ' '     ' ',
        'STUFE'        ' '  '레벨'            ' '     ' '    'C'    ' '     ' '    ' '     ' ',
        'MATNR_SUB'    ' '  '구성부품'         ' '     ' '    'C'    'X'     ' '    ' '     ' ',
        'OJTXP'        ' '  '구성부품 내역'     '40'    'L'    ' '     ' '    ' '    ' '     ' ',
        'MENGE'        ' '  '수량'            '30'    ' '    'R'    ' '     ' '    ' '     ' ',
        'MEINS'        ' '  '단위'            ' '     ' '    'C'    ' '     ' '    ' '     ' ',
        'DATUV'        ' '  '효력시작일'       '15'    ' '    'C'     ' '     ' '   '____/__/__'  ' ',
        'BMENG'        ' '  '기준수량'         '25'    ' '    'R'    ' '     ' '     ' '         ' ',
        'BMEIN'        ' '  '단위'            ' '     ' '    'L'    ' '     ' '     ' '         ' ',
        'AUSCH'        ' '  '구성부품스크랩'     ' '     ' '    'C'    ' '     ' '    ' '          ' ',
        'SANKA'        ' '  '원가계산관계도'     ' '     ' '    'L'    ' '     ' '    ' '          ' '.

ENDFORM.                    "SET_FIELDCAT

*&---------------------------------------------------------------------*
*&      Form  append_field_catalog
*&---------------------------------------------------------------------*
*       필드 카탈로그에 데이터 입력
*----------------------------------------------------------------------*
*      -->P_FNAME       text
*      -->P_KEY         text
*      -->P_OUTPUTLEN   text
*      -->P_EDIT        text
*      -->P_EMPHASIZE   text
*      -->P_CFIELDNAME  text
*      -->P_JUST        text
*      -->P_ZERO        text
*      -->P_DECIMALS_O  text
*----------------------------------------------------------------------*
FORM append_field_catalog USING p_fname
                                p_key
                                p_coltext
                                p_outputlen
                                p_emphasize
                                p_just
                                p_zero
                                p_decimals_o
                                p_edit_mask
                                p_hotspot.

  CLEAR gs_fieldcat.
  gs_fieldcat-fieldname = p_fname. " ALV에 나타낼 필드 이름
  gs_fieldcat-key = p_key." 고정 할 지?
  gs_fieldcat-coltext = p_coltext." 열 헤딩
  gs_fieldcat-outputlen = p_outputlen." 열 길이
  gs_fieldcat-emphasize = p_emphasize. " 열을 색상으로 강조 표시
  gs_fieldcat-just = p_just. " 정렬 L R C
  gs_fieldcat-no_zero = p_zero. " 옆에 0 제거
  gs_fieldcat-decimals_o = p_decimals_o. " 표시되는 소숫점 정의
  gs_fieldcat-edit_mask = p_edit_mask. " 날짜 표시 형식을 ____/__/__바꿈
  gs_fieldcat-hotspot = p_hotspot. " 칼럼에 hotspot 설정

  CASE gs_fieldcat-fieldname.
    WHEN 'MENGE'. " 수량필드일 때 변경 기능 추가, 단위 필드 설정
      gs_fieldcat-edit = abap_true. " 수정 가능하게 할 지
      gs_fieldcat-ref_table = zpp001yh.
      gs_fieldcat-ref_field = 'MEINS'.
      gs_fieldcat-qfieldname = 'MEINS'.

    WHEN 'BMENG'. " 단위 필드 설정
      gs_fieldcat-ref_table = zpp001yh.
      gs_fieldcat-ref_field = 'BMEIN'.
      gs_fieldcat-qfieldname = 'BMEIN'.
  ENDCASE.

  APPEND gs_fieldcat TO gt_fieldcat.

ENDFORM.                    "append_field_catalog
*&---------------------------------------------------------------------*
*&      Form  ALV_DISPLAY
*&---------------------------------------------------------------------*
*       ALV diplay에 필요한 설정 입력
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM alv_display .
  gs_variant-report = sy-repid. " 현재 프로그램
  gs_variant-username = sy-uname. " 현재 유저네임

*  PERFORM col_line." 출력하는 ALV row의 색 지정
  PERFORM display_top_of_page.

* 처음에는 조회모드
  CALL METHOD g_grid2->set_ready_for_input
    EXPORTING
      i_ready_for_input = gv_edit_control." 1은 입력 가능 상태, 0은 입력 불가능 상태

* 데이터 출력 ALV 부분
  CALL METHOD g_grid2->set_table_for_first_display
    EXPORTING
      is_variant      = gs_variant " 레이아웃을 저장할 수 있게 함
      i_save          = 'A'
      i_default       = 'X'
      is_layout       = gs_layout
    CHANGING
      it_outtab       = gt_list[]
      it_fieldcatalog = gt_fieldcat.

* 데이터가 변경되면 변경된 값이 인터널테이블에 저장됨
  CALL METHOD g_grid2->register_edit_event
    EXPORTING
      i_event_id = cl_gui_alv_grid=>mc_evt_modified.

  PERFORM free_data. " 조회 시 사용되었던 데이터 메모리 점유 공간 반환

ENDFORM.                    " ALV_DISPLAY

*&---------------------------------------------------------------------*
*&      Form  SET_TOP_PAGE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_G_DYNDOC_ID  text
*----------------------------------------------------------------------*
FORM set_top_page  USING  e_dyndoc_id  TYPE REF TO cl_dd_document.
  " TOP-OF-PAGE에 나타낼 데이터
  DATA: lv_today TYPE sdydo_text_element,
        lv_nowtime TYPE sdydo_text_element,
        lv_plant TYPE sdydo_text_element.

  lv_today = sy-datum. " 조회 일자(현재 날짜)
  lv_nowtime = sy-uzeit. " 조회 시간(현재 시간)
  lv_plant = t001w-name1. " 작업장명

  CONCATENATE '현재 시간 : ' lv_today+0(4) '년 ' lv_today+4(2) '월 ' lv_today+6(2) '일' '   ' " 현재 날짜
               lv_nowtime+0(2) ':' lv_nowtime+2(2) ':' lv_nowtime+4(2) " 현재 시간
               INTO lv_today RESPECTING BLANKS.

*  e_dyndoc_id->add_text(
*      text         = lv_name
**      sap_style    = cl_dd_area=>key
**      sap_fontsize = cl_dd_area=>large
**      sap_color    = cl_dd_area=>list_heading_int
*    ).

  CALL METHOD e_dyndoc_id->add_text
    EXPORTING
      text      = lv_today
      sap_color = cl_dd_area=>list_group_int. "list_heading_int 민트색.
  "list_group_int 주황색.
  "list_negative_int 빨간색.
  e_dyndoc_id->new_line( ).
  e_dyndoc_id->new_line( ).
  CALL METHOD e_dyndoc_id->add_text
    EXPORTING
      text = '최예훈'.

  e_dyndoc_id->new_line( ).
  e_dyndoc_id->new_line( ).

  DATA : lo_table TYPE REF TO cl_dd_table_element,
         lo_key TYPE REF TO cl_dd_area,
         lo_info TYPE REF TO cl_dd_area,
         lv_text TYPE sdydo_text_element.

  CALL METHOD e_dyndoc_id->add_table
    EXPORTING
      no_of_columns = 2
      width         = '500'
      border        = '1'
    IMPORTING
      table         = lo_table.

  CALL METHOD lo_table->add_column
    IMPORTING
      column = lo_key.

  CALL METHOD lo_table->add_column
    IMPORTING
      column = lo_info.

  lv_text = gv_report_name.
  lo_key->add_text(
                    text = lv_text
                    sap_emphasis = cl_dd_area=>strong
                    sap_fontsize = cl_dd_document=>large
  ).
  lo_key->add_gap(
                    width = '1'
  ).

  lo_info->add_text(
                    text = lv_plant
  ).

  lo_info->add_gap(
                    width = '3'
  ).

  e_dyndoc_id->merge_document( ).

* 이거 안해주면 안보임
  CALL METHOD e_dyndoc_id->display_document
    EXPORTING
      reuse_control      = 'X'
      reuse_registration = 'X'
      parent             = con1.

ENDFORM.                    " SET_TOP_PAGE
*&---------------------------------------------------------------------*
*&      Form  SET_TOP_OF_PAGE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM display_top_of_page .
  IF gv_edit_control = 0.
    gv_report_name = 'BOM 조회'.
  ELSEIF gv_edit_control = 1.
    gv_report_name = 'BOM 수정'.
  ENDIF.

*  TOP-OF-PAGE 초기화
  CALL METHOD g_dyndoc_id->initialize_document.
*    EXPORTING
*      NO_MARGINS = 'X' " TOP-OF-PAGE에 들어갈 text 마진 없앰
*      background_color = cl_dd_area=>col_textarea.
*      FIRST_TIME = 'X'. " 처음 시작 시에만 초기화

* TOP-OF-PAGE
  CALL METHOD g_grid2->list_processing_events
    EXPORTING
      i_event_name = 'TOP_OF_PAGE'
      i_dyndoc_id  = g_dyndoc_id.

ENDFORM.                    " SET_TOP_OF_PAGE
*&---------------------------------------------------------------------*
*&      Form  REFRESH_ALV
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM refresh_alv .
  DATA : ls_stable TYPE lvc_s_stbl.

  ls_stable-row = 'X'.
  ls_stable-col = 'X'.
  CALL METHOD g_grid2->set_ready_for_input
    EXPORTING
      i_ready_for_input = gv_edit_control." 1은 입력 가능 상태, 0은 입력 불가능 상태

  CALL METHOD g_grid2->refresh_table_display
    EXPORTING
      is_stable = ls_stable.
ENDFORM.                    " REFRESH_ALV
*&---------------------------------------------------------------------*
*&      Form  COL_LINE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM col_line .
  FIELD-SYMBOLS : <gt_list_fs> LIKE LINE OF gt_list.
  CASE gv_edit_control.
    WHEN 1. " 수정 모드일 때

*       modify는 퍼포먼스를 떨어뜨리기 때문에 필드 심볼이 유용
      LOOP AT gt_list ASSIGNING <gt_list_fs> WHERE stufe EQ 1.
        <gt_list_fs>-linecolor = ''.
      ENDLOOP.

    WHEN 0. " 조회 모드일 때

      LOOP AT gt_list ASSIGNING <gt_list_fs> WHERE stufe EQ 1.
        <gt_list_fs>-linecolor = 'C500'.
      ENDLOOP.

    WHEN OTHERS.
  ENDCASE.
ENDFORM.                    " COL_LINE

*&---------------------------------------------------------------------*
*&      Form  toolbar_part
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_OBJECT       text
*      -->P_INTERACTIVE  text
*----------------------------------------------------------------------*
FORM toolbar_part  USING  p_object TYPE REF TO cl_alv_event_toolbar_set
                          p_interactive.

  DATA : lw_button TYPE stb_button.
* 세로 구분선 추가
  CLEAR lw_button.
  MOVE 3 TO lw_button-butn_type.
  APPEND lw_button TO p_object->mt_toolbar.

* 버튼 추가
  CLEAR lw_button.
  lw_button-function  = 'EDITTOOL'. " 이것을 handle_user_command가 감지
  lw_button-icon      = icon_change. " 아이콘 지정
  lw_button-butn_type = 0.
  lw_button-quickinfo = '수정 모드 전환'.
  lw_button-text      = '수정'.                               "#EC NOTEXT
  APPEND lw_button TO p_object->mt_toolbar. " append로 하면 alv 툴바 맨 뒤에 생김
* INSERT lw_button TO e_object->mt_toolbar index 1. " insert로 하면 ALV 툴바 인덱스 자리로 생김, 1로 하면 맨 앞

* 세로 구분선 추가
  CLEAR lw_button.
  MOVE 3 TO lw_button-butn_type.
  APPEND lw_button TO p_object->mt_toolbar.

* 버튼 추가
  CLEAR lw_button.
  lw_button-function  = 'BOMEDIT'. " 이것을 handle_user_command가 감지
*  lw_button-icon      = icon_change. " 아이콘 지정
  lw_button-butn_type = 0.
  lw_button-quickinfo = 'BOM 변경 반영'.
  lw_button-text      = '변경'.                               "#EC NOTEXT
  APPEND lw_button TO p_object->mt_toolbar. " append로 하면 alv 툴바 맨 뒤에 생김

* 세로 구분선 추가
  CLEAR lw_button.
  MOVE 3 TO lw_button-butn_type.
  APPEND lw_button TO p_object->mt_toolbar.

* 버튼 추가
  CLEAR lw_button.
  lw_button-function  = 'REPORT'.
  lw_button-icon      = icon_display. " 아이콘 지정
  lw_button-butn_type = 0.
  lw_button-quickinfo = '조회 모드 전환'.
  lw_button-text      = '조회'.                               "#EC NOTEXT
  APPEND lw_button TO p_object->mt_toolbar. " append로 하면 alv 툴바 맨 뒤에 생김

*   버튼 추가
  CLEAR lw_button.
  lw_button-function  = 'SAVE'. " 이것을 handle_user_command가 감지
  lw_button-icon      = icon_system_save. " 아이콘 지정
  lw_button-butn_type = 0.
  lw_button-quickinfo = '저장하기'.
  lw_button-text      = '저장'.                               "#EC NOTEXT
  INSERT lw_button INTO p_object->mt_toolbar INDEX 1. " insert로 하면 ALV 툴바 인덱스 자리로 생김, 1로 하면 맨 앞

* 세로 구분선 추가
  CLEAR lw_button.
  MOVE 3 TO lw_button-butn_type.
  APPEND lw_button TO p_object->mt_toolbar.

ENDFORM.                    " TOOLBAR_PART
*&---------------------------------------------------------------------*
*&      Form  EXIT_CHECK
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM exit_check .
  CALL FUNCTION 'POPUP_TO_CONFIRM_STEP' " 데아터의 손실 가능성이 있는 작업을 수행할 때 사용되는 함수
        EXPORTING
          textline1 = '프로그램을 종료하시겠습니까?' " 팝업창의 본문 내용
          start_column        = 30 " 팝업창 위치
          start_row           = 6
          defaultoption       = 'Y' " 디폴트 설정 값을 Y 값으로 한다.
          titel     = '프로그램 종료' " 팝업창의 제목
        IMPORTING
          answer    = lv_answer. "팝업창에 입력한 값을 저장

  CHECK lv_answer NE 'N'. " N을 입력하지 않을 시 = Y를 입력했을 시
  LEAVE PROGRAM. " 프로그램 종료
ENDFORM.                    " EXIT_CHECK
*&---------------------------------------------------------------------*
*&      Form  DATA_SAVE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM data_save .

  DELETE FROM zpp001yh. " 전체 삭제
*  ALV 에서 데이터를 입력할 때 mc_evt_modifried로 지정하면 셀에서 포커스가 빠져나가야 Data Changed 이벤트를 탄다.
*  이를 하지 않고 저장을 해 버릴 경우 데이터를 인식하지 못하고 저장이 되게 된다.
*  아래의 로직을 추가하면 강제로 Data Changed Event 가 수행되면서 Internal Table 에 ALV 의 변경된 데이터가 반영되게 된다.

  DATA : l_valid TYPE c. " 데이터 변화가 있는지 없는지 여부를 저장할 변수. 데이터 변화가 있으면 'X'

*   처음 조회될 때는 g_grid2가 빈 값이었다가 채워졌으므로 작동함
*   그 이후로는 데이터가 변경되면 작동
  CALL METHOD g_grid2->check_changed_data " check_changed_data메서드 실행
    IMPORTING
        e_valid = l_valid. " e_valid : 데이터 변화가 있는지?
  IF l_valid IS NOT INITIAL. " 데이터 변화가 있으면 = l_valid가 'X'면
    PERFORM convert_cbo_table. " 현재 ALV 상의 데이터를 CBO 테이블 형식으로 변환.
    PERFORM update_database. " 데이터베이스에 바뀐 데이터 저장하는 서브루틴 실행

  ENDIF.
ENDFORM.                    " DATA_SAVE

*&---------------------------------------------------------------------*
*&      Form  UPDATE_DATABASE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM update_database . " 실제로 데이터베이스에 저장하는 서브루틴
  DATA : ls_stable TYPE lvc_s_stbl.
  ls_stable-row = 'X'. ": 현재 선택된 row
  ls_stable-col = 'X'. ": 현재 선택된 col

  TRY.
    MODIFY zpp001yh FROM TABLE gt_convert_itab. " CBO 테이블에 저장
    IF sy-subrc EQ 0.
      COMMIT WORK.
      MESSAGE '저장되었습니다.' TYPE 'I'.
    ELSE.
      ROLLBACK WORK.
      "DB저장 시 에러가 발생하였습니다.
      MESSAGE s031(zmpp) DISPLAY LIKE 'E'.

    ENDIF.
  ENDTRY.

**   gt_modified_rows는 이벤트 핸들러가 data_change를 감지하여 만들어놓음
*  MODIFY makt FROM TABLE gt_modified_rows. " 변경한 정보가 담긴 gt_modified_rows를 원래 데이터베이스 테이블에 저장
*  IF sy-subrc = 0.
*    MESSAGE '저장되었습니다.' TYPE 'I'.
*
*    CLEAR : gt_list, gt_list[]. " 결과값을 보여줄 인터널 테이블을 초기화한다.
*    PERFORM select_data. " 데이터가 변경되었으므로 다시 데이터를 불러온다.
*
*    CALL METHOD g_grid2->refresh_table_display " ALV를 새로고침한다.
*      EXPORTING
*        is_stable = ls_stable. " 새로고침 해도 현재 선택된 위치 유지

*  ENDIF.
ENDFORM.                    " UPDATE_DATABASE
*&---------------------------------------------------------------------*
*&      Form  USER_COMMAND_TOOLBAR
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_E_UCOMM  text
*----------------------------------------------------------------------*
FORM user_command_toolbar  USING p_ucomm.
  ok_code = p_ucomm.
* OK_CODE를 강제로 생성하여 PBO가 다시 작동하게 함
* ALV에서 일어나는 이벤트는 OK_code에 할당되지 않고 E_UCOMM에만 할당되어 스크린의 PAI를 일으키지 않기 때문에
* 따로 메소드를 사용하여 스크린의 PAI를 일으켜야 한다.
  CASE ok_code.
    WHEN 'EDITTOOL'.
      CALL METHOD cl_gui_cfw=>set_new_ok_code
        EXPORTING
          new_code = 'EDITMODE'.
    WHEN 'REPORT'.
      CALL METHOD cl_gui_cfw=>set_new_ok_code
        EXPORTING
          new_code = 'REPORTMODE'.
    WHEN 'SAVE'.
      CALL METHOD cl_gui_cfw=>set_new_ok_code
        EXPORTING
          new_code = 'SAVE'.
    WHEN 'BOMEDIT'.
      CALL METHOD cl_gui_cfw=>set_new_ok_code
        EXPORTING
          new_code = 'BOMEDIT'.
    WHEN OTHERS.
  ENDCASE.

ENDFORM.                    " USER_COMMAND_TOOLBAR
*&---------------------------------------------------------------------*
*&      Form  FREE_DATA
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM free_data .
  FREE : it_base, it_base_str.
ENDFORM.                    " FREE_DATA
*&---------------------------------------------------------------------*
*&      Form  BOM_EDIT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM bom_edit .
* 미완성

  DATA: ls_stko1       TYPE stko_api01,
        ls_stko2       TYPE stko_api02,
        lt_stpo        TYPE TABLE OF stpo_api01,
        ls_stpo        TYPE stpo_api01,
        lv_warning     LIKE capiflag-flwarning,
        lv_bom_no      LIKE stko_api02-bom_no,
        lv_datuv(10).
  DATA:lv_matnr        TYPE mara-matnr.
  DATA:ls_stko         TYPE stko_api02,
       lt_stpo2        TYPE TABLE OF stpo_api02,
       ls_stpo2        TYPE stpo_api02,
       lt_stpo3        TYPE TABLE OF stpo_api03,
       ls_stpo3        TYPE stpo_api03.

  CLEAR:ls_stko1 ,ls_stko2 ,lt_stpo,lt_stpo[],lt_stpo3,lt_stpo3[],ls_stpo3,
      ls_stpo,lv_warning ,lv_bom_no.

  lv_datuv = p_datuv.

  PERFORM convert_cbo_table.

  LOOP AT gt_convert_itab INTO zpp001yh.
    ls_stpo3-item_no = zpp001yh-posnr.
    ls_stpo3-component = zpp001yh-matnr_sub.
    ls_stpo3-comp_qty = zpp001yh-menge.
    ls_stpo3-comp_unit = zpp001yh-meins.

  ENDLOOP.

  CLEAR : zpp001yh, ls_stko1.
  LOOP AT gt_changed_itab INTO zpp001yh.
    ls_stko1-base_quan  = zpp001yh-bmeng.
    ls_stko1-base_unit  = zpp001yh-bmein.
    ls_stko1-bom_status  = zpp001yh-stlst.

    CALL FUNCTION 'CSAP_MAT_BOM_MAINTAIN'
      EXPORTING
        material           = zpp001yh-matnr      "자재 (모품목)
        plant              = zpp001yh-werks      "플랜트 (모품목)
        bom_usage          = zpp001yh-stlan      "BOM용도 (모품목)
        valid_from         = lv_datuv             "효력시작일
        i_stko             = ls_stko1              "BOM 헤더데이터
        fl_commit_and_wait = 'X'
        fl_complete        = 'X'
      IMPORTING " 함수 호출 후의 결과 값을 저장할 변수
        fl_warning         = lv_warning " 경고
        o_stko             = ls_stko2
      TABLES
        t_stpo             = lt_stpo3              "구성부품데이터
      EXCEPTIONS
        error              = 1
        OTHERS             = 2.

    LOOP AT lt_stpo3 INTO ls_stpo3.

    ENDLOOP.

  ENDLOOP.

ENDFORM.                    " BOM_EDIT
*&---------------------------------------------------------------------*
*&      Form  CONVERT_CBO_TABLE
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*  -->  p1        text
*  <--  p2        text
*----------------------------------------------------------------------*
FORM convert_cbo_table .
  DATA : lt_itab LIKE TABLE OF zpp001yh," CBO 테이블의 타입을 가지는 lt_itab 인터널 테이블 생성
         lv_num(18) TYPE c.
*  zpp001yh는 working area로 TOP에서 선언

  LOOP AT gt_list.
    MOVE-CORRESPONDING gt_list TO zpp001yh.
    zpp001yh-mandt = sy-mandt.
    zpp001yh-lines = sy-tabix.
    APPEND zpp001yh TO lt_itab.
  ENDLOOP.

  MOVE lt_itab TO gt_convert_itab. " Global 인터널 테이블 gt_convert_itab에 저장

ENDFORM.                    " CONVERT_CBO_TABLE
```