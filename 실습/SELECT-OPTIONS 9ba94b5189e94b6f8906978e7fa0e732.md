# SELECT-OPTIONS

생성자: 최예훈
생성 일시: 2023년 7월 26일 오전 9:22
태그: 실습

```abap
TABLES : sflight.
DATA : gs_scarr TYPE scarr.
DATA : gv_val TYPE char20.
SELECT-OPTIONS :s_1 FOR sflight-carrid DEFAULT 'AC',
                s_2 FOR gs_scarr-carrid DEFAULT'AA*' OPTION EQ SIGN I,
                s_3 FOR gv_val DEFAULT '1111' TO '9999',
                s_4 FOR gv_val DEFAULT 'AAAA' TO  'ZZZZ' OPTION BT  SIGN E,
                s_5 FOR sflight-carrid MEMORY ID scl,
                s_6 FOR gv_val MATCHCODE OBJECT zcarrid,
                s_7 FOR sflight-carrid MODIF ID car,
                s_8 FOR sflight-carrid NO-DISPLAY,
                s_9 FOR sflight-carrid MODIF ID car,
                s_10 FOR sflight-carrid OBLIGATORY,
                s_11 FOR sflight-carrid NO-EXTENSION,
                s_12 FOR sflight-carrid NO INTERVALS,
                s_13 FOR sflight-carrid VISIBLE LENGTH 1.
```

![Untitled](SELECT-OPTIONS%209ba94b5189e94b6f8906978e7fa0e732/Untitled.png)