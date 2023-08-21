# Refresh

생성자: 최예훈
생성 일시: 2023년 8월 21일 오후 1:43
태그: ABAP

[[ABAP STUDY] ALV REFRESH에 대해 :: 냥소리 하는 블로그 (tistory.com)](https://sidtalk.tistory.com/49)

![Untitled](Refresh%200c9e2eafdb994d8c81d4e453a0b530f2/Untitled.png)

[[SAP ABAP] REFRESH_TABLE_DISPLAY : 네이버 블로그 (naver.com)](https://m.blog.naver.com/howwithus/221710564129)

```abap
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
  ELSE. " 한 번 불러진 이후로는 refresh
        CASE edit_mode.

      DATA : ls_stable TYPE lvc_s_stbl.

      ls_stable-row = 'X'.
      ls_stable-col = 'X'.
    WHEN 0.
      CALL METHOD g_grid2->set_ready_for_input
        EXPORTING
          i_ready_for_input = 0." 1은 입력 가능 상태, 0은 입력 불가능 상태
      CALL METHOD g_grid2->refresh_table_display
        EXPORTING
          i_soft_refresh = 'X'
          is_stable =  ls_stable.
    WHEN 1.
      CALL METHOD g_grid2->set_ready_for_input
        EXPORTING
          i_ready_for_input = 1. " 1은 입력 가능 상태, 0은 입력 불가능 상태
      CALL METHOD g_grid2->refresh_table_display
        EXPORTING
          i_soft_refresh = 'X'
          is_stable =  ls_stable.
    WHEN OTHERS.
  ENDCASE.
  ENDIF.

ENDMODULE.                 " SET_ALV  OUTPUT
```

처음에 alv_display쪽에 넣었는데 set_table_for_first_display 뒤에 넣으면 이미 alv가 불러진 이후에 refresh되므로 의미 없고(필드 카탈로그가 이미 수정 가능하다고 도장 박아버리므로)

앞쪽에 넣으면 오류 발생

PBO에 넣어야 한다고 규정이 되어있는 듯하다.