# CSCMAT - 자재 카테고리

생성자: 최예훈
생성 일시: 2023년 8월 17일 오후 5:02
태그: ABAP

cscmat의 INDEX는 

![Untitled](CSCMAT%20-%20%E1%84%8C%E1%85%A1%E1%84%8C%E1%85%A2%20%E1%84%8F%E1%85%A1%E1%84%90%E1%85%A6%E1%84%80%E1%85%A9%E1%84%85%E1%85%B5%20364c6cb03b684a17a9b49d60bdacec8b/Untitled.png)

![Untitled](CSCMAT%20-%20%E1%84%8C%E1%85%A1%E1%84%8C%E1%85%A2%20%E1%84%8F%E1%85%A1%E1%84%90%E1%85%A6%E1%84%80%E1%85%A9%E1%84%85%E1%85%B5%20364c6cb03b684a17a9b49d60bdacec8b/Untitled%201.png)

![Untitled](CSCMAT%20-%20%E1%84%8C%E1%85%A1%E1%84%8C%E1%85%A2%20%E1%84%8F%E1%85%A1%E1%84%90%E1%85%A6%E1%84%80%E1%85%A9%E1%84%85%E1%85%B5%20364c6cb03b684a17a9b49d60bdacec8b/Untitled%202.png)

```abap
CALL FUNCTION 'CS_BOM_EXPL_MAT_V2'
    EXPORTING
      capid                 = 'PP01' " 어플리케이션
      datuv                 = p_datuv " 효력시작일
      emeng                 = 0
      mehrs                 = 'X' " 다중 BOM 지시자 - 없으면 단일 레벨만 읽는다
      mtnrv                 = it_base_str-matnr " 상위 자재 번호(모자재)
      werks                 = p_werks " 플랜트
      stlan                 = it_base_str-stlan " BOM 용도
      stlal                 = it_base_str-stlal " 대체 BOM
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
```

'CS_BOM_EXPL_MAT_V2' 함수의 결과로 나온 stpox 테이블에서 ttidx 값과 같다

BOM 분기점? 포함범위? 단계? 를 의미한다.

ttidx와 index를 연결하여 자재의 한 단계 위 모품목을 알 수 있다.

![Untitled](CSCMAT%20-%20%E1%84%8C%E1%85%A1%E1%84%8C%E1%85%A2%20%E1%84%8F%E1%85%A1%E1%84%90%E1%85%A6%E1%84%80%E1%85%A9%E1%84%85%E1%85%B5%20364c6cb03b684a17a9b49d60bdacec8b/Untitled%203.png)