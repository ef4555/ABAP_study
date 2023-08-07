# SubSCREEN

생성자: 최예훈
생성 일시: 2023년 8월 7일 오후 2:39
태그: 실습

# 서브스크린 구현

```abap
*&---------------------------------------------------------------------*
*& 모듈풀               ZPPR_YH011
*&
*&---------------------------------------------------------------------*
*&
*&
*&---------------------------------------------------------------------*

PROGRAM  ZPPR_YH011.

DATA : gv_repid TYPE sy-repid,
       gv_sname TYPE sy-dynnr.

DATA : ok_code TYPE sy-ucomm.
DATA : save_ok TYPE sy-ucomm.

INCLUDE ZPPR_YH011_STATUS_0100O01.

INCLUDE ZPPR_YH011_USER_COMMAND_010I01.
```

```abap
*----------------------------------------------------------------------*
***INCLUDE ZPPR_YH011_STATUS_0100O01 .
*----------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  STATUS_0100  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE STATUS_0100 OUTPUT.

  SET PF-STATUS '011'. " 화면에 있는 버튼들 정의, 버튼의 이벤트 이름 등록
  SET TITLEBAR '100_TITLE'. " 화면의 타이틀
  IF gv_repid is INITIAL.
      gv_repid = 'ZPPR_YH011'.
      gv_sname = '0110'.
    ENDIF.
ENDMODULE.                 " STATUS_0100  OUTPUT
```

```abap
*----------------------------------------------------------------------*
***INCLUDE ZPPR_YH011_USER_COMMAND_010I01 .
*----------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0100  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE USER_COMMAND_0100 INPUT.

SAVE_OK = OK_CODE. " save_OK로 값 받아서 써야 한다.서브스크린은 OK_CODE안됨
CASE SAVE_OK.
  WHEN 'BACK' OR 'CANC'.
      LEAVE TO SCREEN 0. " 이전 화면으로 가세옹
  WHEN 'EXIT'.
      LEAVE PROGRAM. " 초기 화면으로
  WHEN 'SUB1'.
    gv_repid = 'ZPPR_YH011'.
    gv_sname = '0110'.
  WHEN 'SUB2'.
    gv_repid = 'ZPPR_YH011'.
    gv_sname = '0120'.
  WHEN OTHERS.
ENDCASE.
ENDMODULE.                 " USER_COMMAND_0100  INPUT
```

100번 화면(OK_CODE 넣어줌)

```abap
PROCESS BEFORE OUTPUT.

 MODULE STATUS_0100.
 CALL SUBSCREEN s_area INCLUDING gv_repid gv_sname.
*
PROCESS AFTER INPUT.
  CALL SUBSCREEN s_area.
  MODULE user_command_0100.
```

![Untitled](SubSCREEN%20776e2043dfa8442bbf21bc2616a49ad0/Untitled.png)

110번 화면(서브스크린으로 생성, 서브스크린으로 보여주고 싶은 레이아웃 짜면 됨)

```abap
PROCESS BEFORE OUTPUT.
* MODULE STATUS_0110.
*
PROCESS AFTER INPUT.
* MODULE USER_COMMAND_0110.
```

120번 화면(서브스크린으로 생성, 서브스크린으로 보여주고 싶은 레이아웃 짜면 됨)

```abap
PROCESS BEFORE OUTPUT.
* MODULE STATUS_0120.
*
PROCESS AFTER INPUT.
* MODULE USER_COMMAND_0120.
```