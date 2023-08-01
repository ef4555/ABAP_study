# Class ALV part 2 : 데이터 올려보기

생성자: 최예훈
생성 일시: 2023년 8월 1일 오전 10:50
태그: 실습

```abap
REPORT  zppr_yh003.

DATA : con1_ref TYPE REF TO cl_gui_custom_container.
DATA : g_grid TYPE REF TO cl_gui_alv_grid.
DATA : gt_mara TYPE TABLE OF mara.

INCLUDE zppr_yh003_pbo.

START-OF-SELECTION.
  SELECT * FROM mara
    INTO  TABLE gt_mara UP TO 25 ROWS.

  CALL SCREEN 100.
```

```abap
*&---------------------------------------------------------------------*
*&  Include           ZPPR_YH003_PBO
*&---------------------------------------------------------------------*
Module init_con OUTPUT.
  IF con1_ref is INITIAL. " 컨테이너 오브젝트가 한 번 생성되었으면 다시 생성하지 않는다.
    " 화면에 한 번 조되었을 경우 REFRESH를 하더라도 인스턴스를 새로 생성할 필요가 없기 때문이다.
    CREATE OBJECT con1_ref
      EXPORTING
        container_name = 'CON1'.
    CREATE OBJECT g_grid
    EXPORTING
        i_parent = con1_ref.
  ENDIF.
  call METHOD g_grid->set_table_for_first_display
    EXPORTING
      i_structure_name = 'MARA'
    CHANGING
      it_outtab = gt_mara.
ENDMODULE.
```

```abap
ROCESS BEFORE OUTPUT.
  MODULE STATUS_0100.
  MODULE init_con.

PROCESS AFTER INPUT.
  MODULE USER_COMMAND_0100.
```

1. 화면에 추가할 ALV GRID 컨트롤의 인스턴스를 생성
2. ALV 필드를 생성하고 화면에 보여줄 데이터를 SELECT
3. set_table_for_first_display 메서드를 호출하여 화면에 보여주자.