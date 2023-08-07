# 모듈풀 프로그램(MARA CRUD)

생성자: 최예훈
생성 일시: 2023년 8월 7일 오후 2:02
태그: 실습

## MAIN

```abap
*&---------------------------------------------------------------------*
*& 모듈풀               ZPPR_YH009
*&
*&---------------------------------------------------------------------*
*&
*&
*&---------------------------------------------------------------------*

PROGRAM  ZPPR_YH009.

TABLES : MARA. " MARA 테이블의 구조를 가진 구조체를 생성. MARA라는 이름으로 구조체처럼 사용할 수 있음 MARA에 헤더라인을 생성하는 것과 유사
* Table Work Area라고 한다. 91p

DATA : ok_code TYPE sy-ucomm. " 유저가 버튼을 클릭했을 때 이벤트를 저장할 변수 선언, 버튼 정의

INCLUDE ZPPR_YH009_PBO. " 화면을 보여주기 위한 준비(버튼 정의, 프로그램 타이틀)
INCLUDE ZPPR_YH009_PAI. " 유저가 이벤트를 실행했을 경우 일어날 이벤트

* 중간에 화면에 표시되는 것들에 관한 로직은 SCREEN(0100)에 존재
```

## PBO

```abap
*----------------------------------------------------------------------*
***INCLUDE ZPPR_YH009_PBO .
*----------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  STATUS_0100  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE status_0100 OUTPUT.
  SET PF-STATUS '100'. " 화면에 있는 버튼들 정의, 버튼의 이벤트 이름 등록
  SET TITLEBAR '100_TITLE'. " 화면의 타이틀
  MARA-ERSDA = sy-datum. " 날짜 초기값을 오늘 날짜로 지정

ENDMODULE.                 " STATUS_0100  OUTPUT
*&---------------------------------------------------------------------*
*&      Module  SCREEN_INIT  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE screen_init OUTPUT. " 특정 필드를 출력 전용으로 변경
  LOOP AT SCREEN.
    IF screen-group1 EQ 'DAT'.
      screen-input = sy-datum.
      MODIFY SCREEN.
    ENDIF.
  ENDLOOP.
ENDMODULE.                 " SCREEN_INIT  OUTPUT
```

## PAI

```abap
*----------------------------------------------------------------------*
***INCLUDE ZPPR_YH009_PAI .
*----------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0100  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_0100 INPUT.

  DATA : gv_answer(1) TYPE c.

  CASE ok_code.
    WHEN 'BACK' OR 'CANC'.
      LEAVE TO SCREEN 0. " 이전 화면으로 가세옹
    WHEN 'EXIT'.
      LEAVE PROGRAM. " 초기 화면으로

    WHEN 'CREA'.
      INSERT mara.
      IF sy-subrc = 0.
        MESSAGE '생성 성공' TYPE 'S'.
      ELSE.
        MESSAGE '생성 실패' TYPE 'E'.
      ENDIF.
      CLEAR mara." 안해주면 해당 데이터가 계속 인풋값으로 남아있음

    WHEN 'DISP'.
      SELECT SINGLE * FROM mara
        WHERE ernam = mara-ernam.
      IF sy-subrc = 0.
        MESSAGE '해당 데이터 존재' TYPE 'S'.
      ELSE.
        MESSAGE '해당 데이터가 없습니다.' TYPE 'E'.
      ENDIF.
      CLEAR mara. " 안해주면 해당 데이터가 계속 인풋값으로 남아있음

    WHEN 'UPDA'.
      UPDATE mara.
      IF sy-subrc = 0.
        MESSAGE '업데이트 성공' TYPE 'S'.
      ELSE.
        MESSAGE '업데이트 실패' TYPE 'E'.
      ENDIF.
      CLEAR mara. " 안해주면 해당 데이터가 계속 인풋값으로 남아있음

    WHEN 'DELE'.
      SELECT SINGLE * FROM mara
        WHERE ernam = mara-ernam.
                               .
      CALL FUNCTION 'POPUP_TO_CONFIRM_LOSS_OF_DATA' " 데아터의 손실 가능성이 있는 작업을 수행할 때 사용되는 함수
        EXPORTING
          textline1 = '삭제하시겠습니까?' " 팝업창의 본문 내용
          START_COLUMN        = 30 " 팝업창 위치
          START_ROW           = 6
          DEFAULTOPTION       = 'Y' " 디폴트 설정 값을 Y 값으로 한다.
          titel     = '경고' " 팝업창의 제목
        IMPORTING
          answer    = gv_answer. "팝업창에 입력한 값을 저장

      CHECK gv_answer NE 'N'. " N을 입력하지 않을 시 = Y를 입력했을 시

      DELETE mara. " 데이터 삭제
      CLEAR mara.
      MESSAGE '삭제 성공' TYPE 'S'.

  WHEN OTHERS.

ENDCASE.

ENDMODULE.                 " USER_COMMAND_0100  INPUT
```

## 화면

```abap
* 0100

PROCESS BEFORE OUTPUT.
 MODULE STATUS_0100.
* MODULE screen_init.

PROCESS AFTER INPUT.
 MODULE USER_COMMAND_0100.

* 스크린 레이아웃에서 조회 화면 구성함
```

![Untitled](%E1%84%86%E1%85%A9%E1%84%83%E1%85%B2%E1%86%AF%E1%84%91%E1%85%AE%E1%86%AF%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%86%B7(MARA%20CRUD)%200755fa117bb64a8f8cd101d8cc6cc45b/Untitled.png)

![Untitled](%E1%84%86%E1%85%A9%E1%84%83%E1%85%B2%E1%86%AF%E1%84%91%E1%85%AE%E1%86%AF%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%86%B7(MARA%20CRUD)%200755fa117bb64a8f8cd101d8cc6cc45b/Untitled%201.png)

![Untitled](%E1%84%86%E1%85%A9%E1%84%83%E1%85%B2%E1%86%AF%E1%84%91%E1%85%AE%E1%86%AF%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%86%B7(MARA%20CRUD)%200755fa117bb64a8f8cd101d8cc6cc45b/Untitled%202.png)