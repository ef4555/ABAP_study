# TOP-OF-PAGE 넣는법(Class ALV)

생성자: 최예훈
생성 일시: 2023년 8월 18일 오후 4:46
태그: ABAP

[ERP Tech Lab(SAP) : 네이버 블로그 (naver.com)](https://blog.naver.com/howwithus)

## 방법

split 컨테이너를 만든다.

반갈한 다음 아래쪽에 내가 만든 도킹 컨테이너를 넣고 위에 TOP-OF-PAGE를 넣는다

1. CLASS 지정
2. OBJECT 생성하고 컨테이너 할당해줌( con1, con2)
3. con2(아래쪽) 컨테이너에 GRID 넣어줌(alv_display로 보여줄 것)
4. 서브루틴에 TOP-OF-PAGE 내용 설정해줌

```abap

  CALL METHOD g_dyndoc_id->initialize_document.

  CALL METHOD g_grid2->list_processing_events
    EXPORTING
      i_event_name = 'TOP_OF_PAGE'
      i_dyndoc_id  = g_dyndoc_id.

```

1. 출력

## 코드

### 클래스 include

```abap
*----------------------------------------------------------------------*
*       CLASS lcl_event DEFINITION
*----------------------------------------------------------------------*
*
*----------------------------------------------------------------------*
CLASS lcl_event DEFINITION. "정의부
  PUBLIC SECTION.
    METHODS : top_of_page
      FOR EVENT top_of_page OF cl_gui_alv_grid
      IMPORTING e_dyndoc_id.
ENDCLASS.                    "lcl_event DEFINITION

*----------------------------------------------------------------------*
*       CLASS lcl_event IMPLEMENTATION
*----------------------------------------------------------------------*
*
*----------------------------------------------------------------------*
CLASS lcl_event IMPLEMENTATION. "구현부
  METHOD top_of_page.
    PERFORM set_top_page USING e_dyndoc_id.
  ENDMETHOD.                    "top_of_page
ENDCLASS.                    "lcl_event IMPLEMENTATION

DATA : g_handler TYPE REF TO lcl_event.
```

### CREATE Object

```abap
FORM alv_obj .
  CREATE OBJECT con1_dock
    EXPORTING
      repid     = sy-repid " 컨트롤에 추가하고자 하는 프로그램 ID
      dynnr     = sy-dynnr " 컨트롤에 추가하고자 하는 스크린 번호
      SIDE         = con1_dock->DOCK_AT_BOTTOM
      extension = 2000.

* 화면 분할
  CREATE OBJECT gv_splitter
      EXPORTING
        PARENT  = con1_dock
        ROWS    = 2      " 몇개의 행으로 나눌것인지
        COLUMNS = 1.   " 몇개의 열로 나눌것인지

* top-of-page
  CALL METHOD gv_splitter->GET_CONTAINER
    EXPORTING
      ROW         = 1
      COLUMN    = 1
    RECEIVING
      CONTAINER = con1.

* ALV
  CALL METHOD gv_splitter->GET_CONTAINER
    EXPORTING
      ROW         = 2
      COLUMN    = 1
    RECEIVING
      CONTAINER = con2.

  CALL METHOD gv_splitter->SET_ROW_HEIGHT
    EXPORTING
      ID      = 1
      HEIGHT  = 10.

* ALV
    CREATE OBJECT g_grid2
      EXPORTING
        i_parent = con2.

  CREATE OBJECT g_dyndoc_id.
  CREATE OBJECT g_handler.

  SET HANDLER g_handler->top_of_page FOR g_grid2.

  CALL METHOD g_grid2->list_processing_events
    EXPORTING
      i_event_name = 'TOP_OF_PAGE'
      i_dyndoc_id = g_dyndoc_id.
```

### Add Form

```abap
FORM SET_TOP_PAGE  USING  e_dyndoc_id  TYPE REF TO cl_dd_document.

  DATA: today TYPE SDYDO_TEXT_ELEMENT,
        plant TYPE SDYDO_TEXT_ELEMENT.

  today = sy-datum.
  plant = t001w-name1.

  CONCATENATE '조회일 : ' today+0(4) '년 ' today+4(2) '월 ' today+6(2) '일' INTO today RESPECTING BLANKS.

  e_dyndoc_id->add_text(
      text         = 'BOM 조회 레포트'
*      sap_style    = cl_dd_area=>key
      sap_fontsize = cl_dd_area=>large
*      sap_color    = cl_dd_area=>list_heading_int
    ).

  e_dyndoc_id->new_line(
    ).
  e_dyndoc_id->new_line(
    ).

  CALL METHOD e_dyndoc_id->add_text
    exporting
     text         = today
     sap_color    = cl_dd_area=>list_group_int. "list_heading_int 민트색.
                                                "list_group_int 주황색.
                                                "list_negative_int 빨간색.
  e_dyndoc_id->new_line(
    ).

  CALL METHOD e_dyndoc_id->add_text
    exporting
     text         = '최예훈'
     sap_color    = cl_dd_area=>list_heading_int.

    e_dyndoc_id->new_line(
    ).
   CALL METHOD e_dyndoc_id->add_text
    exporting
     text         = plant.

* 이거 안해주면 안보임
  CALL METHOD e_dyndoc_id->display_document
    EXPORTING
      reuse_control = 'X'
      parent        = con1.
ENDFORM.                    " SET_TOP_PAGE
```

### alv diplay할 때 TOP-OF-PAGE display

```abap
FORM alv_display .
  gs_variant-report = sy-repid. " 현재 프로그램
  gs_variant-username = sy-uname. " 현재 유저네임

* 데이터 출력 ALV 부분
  CALL METHOD g_grid2->set_table_for_first_display
    EXPORTING
      is_variant      = gs_variant " 레이아웃을 저장할 수 있게 함
      i_save          = 'A'
      i_default       = 'X'
      is_layout       = gs_layout
    CHANGING
      it_outtab       = it_list2[]
      it_fieldcatalog = gt_fieldcat.

  CALL METHOD g_dyndoc_id->initialize_document.
*    EXPORTING
*      background_color = cl_dd_area=>col_textarea.

  CALL METHOD g_grid2->list_processing_events
    EXPORTING
      i_event_name = 'TOP_OF_PAGE'
      i_dyndoc_id  = g_dyndoc_id.

ENDFORM.                    " ALV_DISPLAY
```