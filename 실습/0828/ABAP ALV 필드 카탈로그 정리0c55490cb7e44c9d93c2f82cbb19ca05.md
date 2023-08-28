# ABAP ALV 필드 카탈로그, 레이아웃 정리

생성자: 최예훈
생성 일시: 2023년 8월 28일 오후 12:41
태그: ABAP

[alv 변수 선언 및 필드 속성 지정[LIST] : 네이버 블로그 (naver.com)](https://m.blog.naver.com/longwhiter/30038795580)

- 필요한 변수 선언

```abap
*----------------------------------------------------------------------*
* [ALV] ALV 용 변수 선언 : ALV 사용을 위한 모든 변수 선언.
*----------------------------------------------------------------------*
TYPE-POOLS: slis.

DATA: s_fieldcat TYPE slis_fieldcat_alv,
      t_field    TYPE slis_t_fieldcat_alv.

* STATUS
DATA: gt_extab  TYPE slis_t_extab.
DATA: gv_status TYPE slis_formname VALUE 'STANDARD'.

* Layout
DATA: gs_layout TYPE slis_layout_alv.

* Fieldcat (-> 주요 사용)
DATA: gt_fieldcat    TYPE slis_t_fieldcat_alv,
      gt_fieldcat_ln LIKE LINE OF gt_fieldcat.

*-  ALV 데이타 COLOR SETTING
*DATA: t_colinfo_tab TYPE slis_t_specialcol_alv WITH HEADER LINE.

DATA: lt_color          TYPE slis_t_specialcol_alv,
      ls_color          TYPE slis_specialcol_alv.

* Special Group
DATA: gt_sp_group    TYPE slis_t_sp_group_alv,
      gt_sp_group_ln LIKE LINE OF gt_sp_group.

* Sort (-> 주요 사용)
DATA: gt_sort    TYPE slis_t_sortinfo_alv,
      gt_sort_ln LIKE LINE OF gt_sort.

* Variant
DATA: gv_save,
      gs_variant TYPE disvariant.

* Events (-> 주요 사용)
DATA: gt_events    TYPE slis_t_event,
      gt_events_ln LIKE LINE OF gt_events.

* Events Exit
DATA: gt_event_exit    TYPE slis_t_event_exit,
      gt_event_exit_ln LIKE LINE OF gt_event_exit.

* Filter
DATA: gt_filter    TYPE slis_t_filter_alv,
      gt_filter_ln LIKE LINE OF gt_filter.

* Sel hide
DATA: gs_sel_hide              TYPE slis_sel_hide_alv,
      gs_sel_hide-t_entries_ln LIKE LINE OF gs_sel_hide-t_entries.

* USER COMMAND
DATA: gv_user_command  TYPE slis_formname VALUE 'USER_COMMAND'.
DATA: r_ucomm TYPE sy-ucomm.
DATA: rs_selfield TYPE slis_selfield.

* Current Program 저장 변수
DATA: gv_cprog TYPE sy-repid VALUE sy-repid.

* Double Click Event에서 사용할 변수
DATA : gv_line_cnt    TYPE slis_selfield-tabindex,    "라인 넘버
       gv_line_value  TYPE slis_selfield-value,       "field 값
       gv_fieldname   TYPE slis_selfield-fieldname,
       gv_endsum      TYPE slis_selfield-endsum,
       gv_refresh     TYPE slis_selfield-refresh,
       gv_col_stable  TYPE slis_selfield-col_stable,
       gv_row_stable  TYPE slis_selfield-row_stable.
```

- 필드 카탈로그

```abap
************* *  필드 Category 선언(Sample)********************************
*  gt_fieldcat_ln-fieldname    = 'IDNO'.
*  gt_fieldcat_ln-row_pos      = '1'.       "필드 순서(행)
*  gt_fieldcat_ln-col_pos      = '1'.       "필드 순서(렬)
*  gt_fieldcat_ln-outputlen    = '6'.       "필드 사이즈
*  gt_fieldcat_ln-just         = 'L'.       "또는 'C', 'R'
*  gt_fieldcat_ln-fix_column   = 'X'.       "칼럼 고정
*  gt_fieldcat_ln-key          = 'X'.       "키 칼럼 설정
*  gt_fieldcat_ln-emphasize    = 'C300'.    "필드강조:Cxxx로 설정
*  gt_fieldcat_ln-currency     = 'KRW'.     "통화
*  gt_fieldcat_ln-cfieldname   = 'WAERS'.   "수량 또는 통화 참조필드
*  gt_fieldcat_ln-do_sum       = 'X'.       "합계
*  gt_fieldcat_ln-no_sum       = 'X'.       "합계 구하기 금지
*  gt_fieldcat_ln-input        = 'X'.       "필드 수정 가능
*  gt_fieldcat_ln-icon         = 'X'.       "아이콘사용
*  gt_fieldcat_ln-symbol       = 'X'.       "심벌의 사용
*  gt_fieldcat_ln-hotspot      = 'X'.       "필드에 접근하면 손모양 표현
*  gt_fieldcat_ln-no_sign      = 'X'.       "음수(-) 부호 표현 금지
*  gt_fieldcat_ln-no_zero      = 'X'.       "필드에 '0' 표현 금지
*  gt_fieldcat_ln-lzero        = 'X'.       "NUMC 타입 필드의 '0' 표현
*  gt_fieldcat_ln-no_out       = 'X'.       "화면에서 Hidden.
*  gt_fieldcat_ln-tech         = 'X'.       "화면에서 Hidden.
*  gt_fieldcat_ln-datatype     = 'NUMC'.    "데이터 타입 변경
*  gt_fieldcat_ln-reptext_ddic = '번호'.    "55Byte 사용 가
*  gt_fieldcat_ln-edit_mask    = 'RR___Kg'. "RR:오른쪽정렬,LL:왼쪽정렬,
*  _:한자리 Char, : :구분자, ==conv:출력 변환
*  APPEND gt_fieldcat_ln TO gt_fieldcat.
```

- 레이아웃

```abap
****gs_layout******************************************************************

 
* gs_layout-colwidth_optimize    = 'X'.     "칼럼명 Size 최적화
  gs_layout-zebra                = 'X'.
* gs_layout-detail_initial_lines = 'X'.     "세부항목 필드 처리
* gs_layout-key_hotspot          = 'X'.
* gs_layout-no_vline             = 'X'.     "칼럼 사이 경계선 Hidden
  gs_layout-no_colhead           = 'X'.     "칼럼 헤드(전체) Hidden
  gs_layout-box_fieldname        = 'ZMARK'. "체크박스 필드 설정.
* gs_layout-lights_fieldname     = 'X'.     "신호등(1:Red,2:Yell,3:Gre)
* gs_layout-lights_condense      = 'X'.     "subtotal에 신호등표시
* gs_layout-totals_text          = '총합계'.
* gs_layout-subtotals_text       = '소계'.
* gs_layout-no_sumchoice         = 'X'.     "SUM 가능 필드를 SUM에서제외
* gs_layout-no_totalline         = 'X'.     "Total Recodr Hidden
* gs_layout-no_totalline         = 'X'.     "Total Recodr Hidden
* gs_layout-no_subtotals         = 'X'.     "Sun Total 결과 Hidden
* gs_layout-totals_only          = 'X'.     "Total 결과만 보여주기
* gs_layout-numc_sum             = 'X'.     "NUMC 타입만 Sum 수행
* gs_layout-detail_popup         = 'X'.     " show detail in popup

** 세로줄 삭제
*  gs_layout-no_vline          = 'X'.

** TOTAL RECORD를 보여주지 않는다.
*  gs_layout-no_totalline      = 'X'.

** CHECKBOX
*  GS_LAYOUT-BOX_FIELDNAME     = 'MARK'.

** numc 타입 Sum 가능
  gs_layout-numc_sum          = 'X'.

** 입력 불가능
*  gs_layout-no_input          = 'X'.

** double Click시 아래의 Event가 발생한다.
*  GS_LAYOUT-F2CODE            = '&OL0'.

** KEY FIELD가 좌우 SCROLL시 고정되어 있지 않음.
*  gs_layout-no_keyfix         = 'X'.

** SORT와 SUBTOTAL 팝업에서 그룹과 UL 구분가능
*  gs_layout-lights_fieldname  = 'STATUS'.

** SUM 가능 필드에 SUM 수행하지 않음.
*  gs_layout-no_sumchoice      = 'X'.

** 원하는 line 색깔 변경, PERFORM change_linecolor함께 사용
  gs_layout-info_fieldname    = 'COLOR'.

** 조회 조건을 볼 수 있다.
  gs_layout-get_selinfos = 'X'.

  gs_layout-info_fieldname =      'LINE_COLOR'.

 

 

** sort ********************************************************

  gt_sort_ln-spos      = '1'.           "정렬대상 필드 순서
  gt_sort_ln-fieldname = 'MATNR'.       "정렬대상 필드명
  gt_sort_ln-up        = 'X'.           "오름차순
* gt_sort_ln-down      = ' '.           "내림차순
* gt_sort_ln-SUBTOT    = ' '.           "합계
  APPEND gt_sort_ln TO gt_sort.
```

- 예시 코드
    
    ```abap
    ************* *  필드 Category 선언(Sample)********************************
    *  gt_fieldcat_ln-fieldname    = 'IDNO'.
    *  gt_fieldcat_ln-row_pos      = '1'.       "필드 순서(행)
    *  gt_fieldcat_ln-col_pos      = '1'.       "필드 순서(렬)
    *  gt_fieldcat_ln-outputlen    = '6'.       "필드 사이즈
    *  gt_fieldcat_ln-just         = 'L'.       "또는 'C', 'R'
    *  gt_fieldcat_ln-fix_column   = 'X'.       "칼럼 고정
    *  gt_fieldcat_ln-key          = 'X'.       "키 칼럼 설정
    *  gt_fieldcat_ln-emphasize    = 'C300'.    "필드강조:Cxxx로 설정
    *  gt_fieldcat_ln-currency     = 'KRW'.     "통화
    *  gt_fieldcat_ln-cfieldname   = 'WAERS'.   "수량 또는 통화 참조필드
    *  gt_fieldcat_ln-do_sum       = 'X'.       "합계
    *  gt_fieldcat_ln-no_sum       = 'X'.       "합계 구하기 금지
    *  gt_fieldcat_ln-input        = 'X'.       "필드 수정 가능
    *  gt_fieldcat_ln-icon         = 'X'.       "아이콘사용
    *  gt_fieldcat_ln-symbol       = 'X'.       "심벌의 사용
    *  gt_fieldcat_ln-hotspot      = 'X'.       "필드에 접근하면 손모양 표현
    *  gt_fieldcat_ln-no_sign      = 'X'.       "음수(-) 부호 표현 금지
    *  gt_fieldcat_ln-no_zero      = 'X'.       "필드에 '0' 표현 금지
    *  gt_fieldcat_ln-lzero        = 'X'.       "NUMC 타입 필드의 '0' 표현
    *  gt_fieldcat_ln-no_out       = 'X'.       "화면에서 Hidden.
    *  gt_fieldcat_ln-tech         = 'X'.       "화면에서 Hidden.
    *  gt_fieldcat_ln-datatype     = 'NUMC'.    "데이터 타입 변경
    *  gt_fieldcat_ln-reptext_ddic = '번호'.    "55Byte 사용 가
    *  gt_fieldcat_ln-edit_mask    = 'RR___Kg'. "RR:오른쪽정렬,LL:왼쪽정렬,
    *  _:한자리 Char, : :구분자, ==conv:출력 변환
    *  APPEND gt_fieldcat_ln TO gt_fieldcat. 
    
    ****gs_layout******************************************************************
    
     * gs_layout-colwidth_optimize    = 'X'.     "칼럼명 Size 최적화
      gs_layout-zebra                = 'X'.
    * gs_layout-detail_initial_lines = 'X'.     "세부항목 필드 처리
    * gs_layout-key_hotspot          = 'X'.
    * gs_layout-no_vline             = 'X'.     "칼럼 사이 경계선 Hidden
      gs_layout-no_colhead           = 'X'.     "칼럼 헤드(전체) Hidden
      gs_layout-box_fieldname        = 'ZMARK'. "체크박스 필드 설정.
    * gs_layout-lights_fieldname     = 'X'.     "신호등(1:Red,2:Yell,3:Gre)
    * gs_layout-lights_condense      = 'X'.     "subtotal에 신호등표시
    * gs_layout-totals_text          = '총합계'.
    * gs_layout-subtotals_text       = '소계'.
    * gs_layout-no_sumchoice         = 'X'.     "SUM 가능 필드를 SUM에서제외
    * gs_layout-no_totalline         = 'X'.     "Total Recodr Hidden
    * gs_layout-no_totalline         = 'X'.     "Total Recodr Hidden
    * gs_layout-no_subtotals         = 'X'.     "Sun Total 결과 Hidden
    * gs_layout-totals_only          = 'X'.     "Total 결과만 보여주기
    * gs_layout-numc_sum             = 'X'.     "NUMC 타입만 Sum 수행
    * gs_layout-detail_popup         = 'X'.     " show detail in popup
    
    ** sort ********************************************************
    
      gt_sort_ln-spos      = '1'.           "정렬대상 필드 순서
      gt_sort_ln-fieldname = 'MATNR'.       "정렬대상 필드명
      gt_sort_ln-up        = 'X'.           "오름차순
    * gt_sort_ln-down      = ' '.           "내림차순
    * gt_sort_ln-SUBTOT    = ' '.           "합계
      APPEND gt_sort_ln TO gt_sort.
    
    * 원하는 line 색깔 변경
      gS_layout-coltab_fieldname = 'TABCOLOR'.
    
      LOOP AT gt_itab ASSIGNING <ls_outtab>.
        CLEAR lt_color.
        IF <ls_outtab>-rate < 97.
          ls_color-fieldname = 'RATE'.
          ls_color-color-col = 6.
          ls_color-color-int = 1.
          ls_color-color-inv = 0.
          ls_color-nokeycol  = 'X'.
          APPEND ls_color TO lt_color.
        ENDIF.
        <ls_outtab>-tabcolor = lt_color.
     ENDLOOP.
    
      CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
        EXPORTING
          i_callback_program       = g_repid
          i_callback_top_of_page   = 'TOP-OF-PAGE'
          i_callback_user_command  = g_user_command
          i_callback_pf_status_set = g_callback_pf_status_set
          is_layout                = gt_layout
          it_sort                  = gt_sort[]
          it_fieldcat              = gt_fieldcat
    *      it_events                = gt_events[]
    *      is_variant               = gs_variant
    *      i_save                   = gv_save
    *      i_default                = 'X'
        TABLES
          t_outtab                 = gt_itab.
    ```
    

## 목록

| FieldName | 소속 | 설명 |
| --- | --- | --- |
| ROW_POS |  | ALV 제어: 출력라인 (INTERNAL USE) |
| COL_POS | Display | Display되는 Column의 순서생략시 입력순서대로 표시된다 |
| TABNAME | Control | ALV의 Output Table의 Name을 넣어준다.그런데 없어도 상관은 없다.왜? 어차피 1개의 ALV는 1개의 FieldCatalog를 취하니까. |
| FIELDNAME | Control(필수) | ALV의 Output Table의 FieldName을 넣어준다.여기가 Output Table 과 Catalog가 Match되는 방법이다 |
| F4AVAILABL |  | 필드에 출력도움말 유무점검오류 : "No input help is available" 발생시ref_table / ref_field를 Clear하면 된다. |
| REF_TABLE | ABAP Dic(활용) | ABAP Dic으로 부터 모든 데이타를 활용하게 설정할경우(DOMAIN값을 참조할때도 반드시 설정되어 있어야 한다.)ABAP Dic 참조하기위한 [테이블명]을 넣어준다. |
| REF_FIELD | ABAP Dic(활용) | ABAP Dic으로 부터 모든 데이타를 활용하게 설정할경우(DOMAIN값을 참조할때도 반드시 설정되어 있어야 한다.)ABAP Dic 참조하기위한 테이블의 [Field명]을 넣어준다. |
| DOMNAME | ABAP Dic(활용) | 윗 4개가 일치하지 않는경우,DOMNAME이 어뚱한 놈을 바라보면 POPUP은 뜨는데 유효성 검사가 어뚱하게 걸리는 현상이 나타난다. |
| CURRENCY | ABAP Dic(참조) | 보여주려는 [숫자]의 소속이 [통화금액]인 경우[고정통화]를 쓰는 경우 통화단위를 넣어준다. ex) [KRW] |
| CFIELDNAME | ABAP Dic(참조) | 보여주려는 [숫자]의 소속이 [통화금액]인 경우[통화]를 가지고 있는 [FieldName]을 넣어준다.=> [FieldName]이 FieldCatalog의 Member로 등록되어있어야 함.(만약에 없는경우, [Data]수정시 오류가 발생함. ) |
| QUANTITY | ABAP Dic(참조) | 보여주려는 [숫자]의 소속이 [단위수량]인 경우[고정단위]를 쓰는 경우 수량단위를 넣어준다. ex) [Kg] |

| QFIELDNAME | ABAP Dic(참조) | 보여주려는 [숫자]의 소속이 [단위수량]인 경우[단위]를 가지고 있는 [FieldName]을 넣어준다.=> [FieldName]이 FieldCatalog의 Member로 등록되어있어야 함.(만약에 없는경우, [Data]수정시 오류가 발생함. ) |
| --- | --- | --- |
| INTTYPE | Control(필수) | ABAP 데이타유형 (C,D,N,...) |
| JUST | Display(추천) | 화면표시 정렬 ( Left , Center , Right ) |
| OUTPUTLEN | Display(추천) | 화면표시 문자의 열 너비Layout-cwidth_opt = 'X' 설정시 무력화 된다. |
| COLTEXT (40) | Display(필수) | Column Header의 텍스트(오직 1개의 종류만 표시할때) |
| SCRTEXT_L  (40) | Display(선택) | Column Header의 텍스트(폭이 넓은경우 표시) |
| SCRTEXT_M (20) | Display(선택) | Column Header의 텍스트(폭이 중간인 경우 표시) |
| SCRTEXT_S (10) | Display(선택) | Column Header의 텍스트(폭이 좁은경우 표시) |
| TOOLTIP(40) | Display(선택) | Column의 말풍선 |
| NO_OUT | Display(Variant설정) | 초기화면 조회시에는 보이지 않도록Variant설정시에는 Layout설정에 나타난다. |
| TECH | Display(Variant설정) | 화면에서 아예 보이지 않는다. |
| HOTSPOT | Display(선택)/Event연동 | 웹브라우저의 HyperLink처럼 동작하게 한다. Click Event처리 |

| EDIT(가장 중요한 설정항목중 하나임) | Display(편집) | 해당 Field에 대해서 편집설정을 나타낸다.Layout-edit = 'X' 설정시 무력화 된다.(하위개념임)그밑에 Cell-Style의 Enable/Disable이 있음 |
| --- | --- | --- |
| CHECKBOX | Display(편집) | 체크박스로 보여준다. |
| DRDN_HNDL | Display(편집) | Dropdown List Handle/ 정수값을 입력해 주어야 한다1개의 Field는 1개의 Dropdown List만을 보여줄때 사용 |
| DRDN_FIELD | Display(편집) | Dropdown List Field/ FieldName을 입력해 주어야 한다1개의 Field는 다수의 Dropdown List를 보여줄수 있다. |
| DRDN_ALIAS | Display(편집) | Dropdown List Handle( X 설정요망 )User에게 코드를 보여주면 가독성이 떨어진다.따라서 [코드:설명]이렇게 보여주기 원할때 사용한다 |
| CHECKTABLE | Display(편집) | ABAP Dic에 의해 CheckTable이 설정되면 자동으로 F4활성유효성 검사등이 수행된다.이때 이값에 "!"을 넣어주면 임의편집을 가능하게할수있다 |
| KEY | Control(선택) | 키필드로 선택(배경색이 자동으로 파랑색, Key역활을 시작함)1. 필수 입력사항이다.2. 한번 입력된 내용은 수정될 수 없다.3. Key Value는 Unique해야 한다. |
| KEY_SEL | Control(선택) | KEY로 설정한후화면상의 Display하지않음, Variant에도 안나옴 |
| FIX_COLUMN | Control(선택) | Column고정.  Hsplit 기능을 사용할 때 사용한다. |
| ROUND | Display(숫자) | 보여주려는 [숫자]의 소수점을 주어진 round 수만큼좌측이로 이동한다.결과숫치 =  기존숫자 / (10^(round) ) 의 역활을 한다.숫치입력시 자동으로 나누기가 시행되므로 사용에 주의할것Ex) 100.00 [Round = 2] => 1.00 |
| NO_SIGN | Display(숫자) | 보여주려는 [숫자]의 부호를 표시하지 않는다. |

| NO_ZERO | Display(숫자) | 보여주려는 [숫자]가 0인경우 표시하지 않는다[숫자]의 소속이 [통화금액][단위수량]인경우 자동으로 설정 |
| --- | --- | --- |
| DECIMALS_O | Display(숫자) | 보여주려는 [숫자]의 소숫점 자리수 설정.Decimals 설정을 같이 늘려주지 않으면 입력오류가 발생됨 |
| DECIMALS | Display(숫자) | 입력되는 [숫자]의 소숫점 자리수 설정. |
| LZERO | Display(숫자) | Data Type : N , ABAP Type : Numeric문자형 숫자이므로 앞쪽에 빈공간에 "0"을 채워준다. |
| DO_SUM | Display(숫자)/Sort연동 | set_table_for_first_display의 Sort설정이 되었을경우 중간합을 보여준다.'X' 설정시 [총합]'C' 설정시 [평균]을 보여준다. |
| EXPONENT |  | ALV 제어: 부동표현에 대한 지수 |
| ICON | Display(문자) | 아이콘으로 출력할때 |
| SYMBOL | Display(문자) | ALV 제어: 기호로 출력 |
| EDIT_MASK | Display(문자) | 화면 Display시 편집마스크. Ex) "____/__/__" |
| EMPHASIZE | Display(강조) | Column(Field)를 시스템이 지정한 색상으로 강조한다= 'C600' (개발자가 색상지정), = 'X' (시스템이 색상지정) |
| NO_SUM |  | ALV 제어: 열값에 관한 집합처리없음 |

| TECH |  | ALV 컨트롤: 기술적 필드 |
| --- | --- | --- |
| CONVEXIT |  | 변환루틴( Domain에 Exit가 걸린경우 해당정보를 제공한다 ) |
| SELTEXT |  | ALV 제어: 다이얼로그 기능에 대한 열식별자 |
| NO_CONVEXT |  | ALV 제어: 화면 Display시 변환종료를 고려하지 마십시오 |
| ROLLNAME |  | ALV 제어: F1 도움말의 데이터 요소Elment의 이름을 넣어주면된다. [Dynamic itab]이 설정기준이 된다. |
| DATATYPE |  | ABAP Dictionary 데이타유형 |
| INTLEN |  | 내부길이 (바이트단위) |
| LOWERCASE |  | 소문자 사용/금지 |
| REPTEXT |  | 헤딩 |
| HIER_LEVEL |  | ALV 컨트롤: 내부 사용 |
| REPREP |  | ALV 컨트롤: 값은 rep./rep.intf에 대한 선택 기준입니다. |
| DOMNAME |  | 도메인이름 |
| SP_GROUP |  | 그룹 키 |
| DFIELDNAME |  | ALV 제어: 데이터베이스의 열 그룹에 대한 필드 이름 |
| COL_ID |  | ALV 제어: 열 ID |
| AUTO_VALUE |  | ALV 제어: 자동값복사 |
| VALEXI |  | 고정값 존재 |
| WEB_FIELD |  | ALV 제어: 내부 테이블 필드의 필드 이름 |
| HREF_HNDL |  | 정수 |
| STYLE |  | ALV 제어: 스타일 |
| STYLE2 |  | ALV 제어: 스타일 |
| STYLE3 |  | ALV 제어: 스타일 |
| STYLE4 |  | ALV 제어: 스타일 |
| DRDN_FIELD |  | ALV 제어: 내부 테이블 필드의 필드 이름 |
| NO_MERGING |  | 문자 필드 길이 1 |
| H_FTYPE |  | ALV 트리 제어: 기능유형 (합계, 평균, 최대, 최소, ...) |

| COL_OPT |  | 선택 열 최적화 엔트리 |
| --- | --- | --- |
| NO_INIT_CH |  | 문자 필드 길이 1 |
| TXT_FIELD |  | ALV 제어: 내부 테이블 필드의 필드 이름 |
| ROUNDFIELD |  | ALV 제어: ROUND 사양을 가진 필드이름 |
| DECMLFIELD |  | ALV 제어: DECIMALS 사양을 가진 필드이름 |
| DD_OUTLEN |  | ALV 제어: 문자의 출력길이 |
| COLDDICTXT |  | ALV 제어: DDIC 텍스트 참조 결정 |
| SELDDICTXT |  | ALV 제어: DDIC 텍스트 참조 결정 |
| TIPDDICTXT |  | ALV 제어: DDIC 텍스트 참조 결정 |
|  |  |  |
| TECH_COL |  | ALV 컨트롤: 내부 사용 |
| TECH_FORM |  | ALV 컨트롤: 내부 사용 |
| TECH_COMP |  | ALV 컨트롤: 내부 사용 |
| HIER_CPOS |  | ALV 제어: 계층구조적 열위치 |
| H_COL_KEY |  | 트리 컨트롤: 열 이름/항목 이름 |
| H_SELECT |  | 트리제어의 열을 선택할 수 있는지의 여부 지시 |

| DD_ROLL |  | Data element (의미도메인) |
| --- | --- | --- |
| DRAGDROPID |  | ALV 제어: 끌어옮기기 오브젝트에 대한 끌어옮기기 조정 |
| MAC |  | 문자 필드 길이 1 |
| INDX_FIELD |  | 정수 |
| INDX_CFIEL |  | 정수 |
| INDX_QFIEL |  | 정수 |
| INDX_IFIEL |  | 정수 |
| INDX_ROUND |  | 정수 |
| INDX_DECML |  | 정수 |
| GET_STYLE |  | 문자 필드 길이 1 |
| MARK |  | 문자 필드 길이 1 |