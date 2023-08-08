# AT SELECTION-SCREEN

생성자: 최예훈
생성 일시: 2023년 8월 8일 오전 11:14
태그: 실습

[[SAP ABAP] AT SELECTION-SCREEN : 네이버 블로그 (naver.com)](https://m.blog.naver.com/howwithus/221467062587)

사용자가 스크린 1000에서 입력 필드 값이 변동되었을 때 실행되는 이벤트

필수값은 아니다

입력필드에 대해 어떤 이벤트를 넣고 싶을 때 사용한다.

프로그램을 시작하자마자 작동하는 이벤트이므로 권한체크에도 사용된다.

![Untitled](AT%20SELECTION-SCREEN%20bc2e9cc3745b45f08e18e2329bd42a18/Untitled.png)

## 예시

```abap
REPORT aaa

TABLES : SCARR.

PARAMETERS : P_MATNR TYPE MATNR.

SELECT-OPTIONS : S_CARRID FOR SCARR-CARRID.

INITIALIZAIONS.

AT SELECTION-SCREEN ON p_MATNR.
	IF P_MATNR NE 'AA'.
		MESSAGE 'TEST' TYPE 'I'.
		EXIT.
	ENDIF.
```

AA가 아닌 다른 값 입력 시 TEST 메시지 출력하게 됨