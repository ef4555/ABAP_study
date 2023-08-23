# 데이터 변화가 일어났을때 ALV refresh하는건 구식 방법이래~

생성자: 최예훈
생성 일시: 2023년 8월 23일 오후 4:00
태그: ABAP

[A Twossun Place :: A Twossun Place (tistory.com)](https://twossun.tistory.com/)

![Untitled](%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%92%E1%85%AA%E1%84%80%E1%85%A1%20%E1%84%8B%E1%85%B5%E1%86%AF%E1%84%8B%E1%85%A5%E1%84%82%E1%85%A1%E1%86%BB%E1%84%8B%E1%85%B3%E1%86%AF%E1%84%84%E1%85%A2%20ALV%20refresh%E1%84%92%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB%E1%84%80%E1%85%A5%E1%86%AB%20%E1%84%80%20efa6ac8299c6469e90a7c8a52ee44585/Untitled.png)

![Untitled](%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%92%E1%85%AA%E1%84%80%E1%85%A1%20%E1%84%8B%E1%85%B5%E1%86%AF%E1%84%8B%E1%85%A5%E1%84%82%E1%85%A1%E1%86%BB%E1%84%8B%E1%85%B3%E1%86%AF%E1%84%84%E1%85%A2%20ALV%20refresh%E1%84%92%E1%85%A1%E1%84%82%E1%85%B3%E1%86%AB%E1%84%80%E1%85%A5%E1%86%AB%20%E1%84%80%20efa6ac8299c6469e90a7c8a52ee44585/Untitled%201.png)

[[SAP ABAP] CL_ALV_CHANGED_DATA_PROTOCOL(MODIFY_CELL, GET_CELL_VALUE) : 네이버 블로그 (naver.com)](https://m.blog.naver.com/howwithus/221708057694)

[DATA_CHANGE 메소드/이벤트 : 네이버 블로그 (naver.com)](https://m.blog.naver.com/l_yh6/221876162312)

이것이 최신.

```abap
CLASS lcl_event_receiver_101 DEFINITION.
  PUBLIC SECTION.
    METHODS : handle_data_changed_101
      FOR EVENT data_changed OF cl_gui_alv_grid
      IMPORTING er_data_changed.
ENDCLASS.

CLASS lcl_event_receiver_101 IMPLEMENTATION.
  METHOD handle_data_changed_101.
    DATA : lv_row       TYPE i,
           ls_mod_cells TYPE lvc_s_modi,
           ls_carrid    TYPE sflight-carrid.

    "변경된 Cell 의 값을 Read 하고, 해당 라인의 다른 값을 Modify
    LOOP AT er_data_changed->mt_good_cells INTO ls_mod_cells.
      AT NEW row_id.
        CLEAR : lv_zstat, lv_ztrst.
        CALL METHOD er_data_changed->get_cell_value
          EXPORTING
            i_row_id    = ls_mod_cells-row_id
            i_fieldname = 'CARRID'
          IMPORTING
            e_value     = lv_carrid.

        CALL METHOD er_data_changed->modify_cell
          EXPORTING
            i_row_id    = ls_mod_cells-row_id
            i_fieldname = 'CONNID'
            i_value     = '변경할 값'.
      ENDAT.

    ENDLOOP.

  ENDMETHOD.
ENDCLASS.
```