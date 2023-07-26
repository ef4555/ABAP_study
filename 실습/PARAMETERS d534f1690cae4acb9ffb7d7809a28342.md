# PARAMETERS

생성자: 최예훈
생성 일시: 2023년 7월 26일 오전 9:09
태그: 실습

```abap
DATA : l_fname(20) TYPE c.

PARAMETERS :
p_1 DEFAULT 'A', "기본값 정의
p_2 TYPE char10,
p_3 TYPE c LENGTH 3 DEFAULT '123', " 최대 3자리 입력 가능이고 기본 값 123
p_4 TYPE p DECIMALS 2 DEFAULT '123.456789', "기본값은 123.456789이고 보여지는 값은 소수점 둘째자리까지 123.45
p_5 LIKE sflight-carrid, " 선택 옵션 리스트 제공
p_6 MEMORY ID scl,
p_7 MATCHCODE OBJECT zcarrid,
p_8 MODIF ID mid,
p_9 NO-DISPLAY, " 안보임
p_10 DEFAULT 'a' LOWER CASE, " 소문자의 입력도 가능하게 함
p_11 OBLIGATORY, " 이 값이 필수 입력 칸임을 의미
p_12 AS CHECKBOX,
p_13 RADIOBUTTON GROUP radi, " radi 그룹중 하나 선택
p_13_2 RADIOBUTTON GROUP radi, " radi 그룹중 하나 선
p_14(10) VISIBLE LENGTH 3 DEFAULT '1234567890', " 최대 10자리 입력 가능이고 3자리만 보이고 기본값은 1234567890
p_15 LIKE sflight-carrid VALUE CHECK, " 뭐가 다르지
p_16 LIKE (l_fname), " 동적으로 입력 가능, l_fname은 위에서 최대 20자리 데이터로 지정
p_17 LIKE sflight-carrid AS LISTBOX VISIBLE LENGTH 20, " 선택 옵션 리스트 제공, 보이는 글자수는 20자
p_18 AS CHECKBOX USER-COMMAND abc. "abc는 사용자 정의 이벤트
```

## 해설

![Untitled](PARAMETERS%20d534f1690cae4acb9ffb7d7809a28342/Untitled.png)

![Untitled](PARAMETERS%20d534f1690cae4acb9ffb7d7809a28342/Untitled%201.png)

## 설명

**●DEFAULT f**

- Data 문의 Value 절과 동일
- 변수의 초깃값을 설정하는 것
- 프로그램을 실행할 경우 기본값이 화면에 표시되면서 값을 다시 입력받을 수 있게 함.

**●  TYPE type**

- 변수의 형을 선언

**● DECIMALS dec**

- 소수점 이하의 자리 수를 정의하며 적용 가능한 변수형은 P type임

**● LIKE g**

- Data 문에서의 Like 옵션과 동일하며 g의 변수형을 그대로 인용하는 것
- g가 문자형이면 선언된 변수도 문자형이 되는 것

**● MEMORY ID pid**

- Global Memory 변수 pid를 선언한 변수에 Assign 하는 형태

**● MATCHCODE OBJECT mobj**

- Matchcode Object를 변수에 할당하는 것(4문자)

**● MODIF ID modid**

- 변수들의 속성(입력 가능, 불가능, 등등)들을 일괄적으로 변경하기 위하여 Group으로 묶어주는 것
- AT SELECTION-SCREEN OUTPUT EVENT에서 사용할 수 있음

**● NO-DISPLAY**

- 프로그램을 실행하여도 화면에 나타나지 않지만 내부적으로 값을 지정할 수도 있음
- 단지 화면에 나타나지 않는다는 점을 제외하고는 다른 변수와 동일함

**● LOWER CASE**

- 소문자의 입력도 가능하게 함

● **OBLIGATORY**

- 필수적으로 값을 입력 받고. 입력하지 않으면 오류로 처리 함.

**● AS CHECKBOX**

- CheckBox의 형태로 변수의 값을 입력하게 함.

**● RADIOBUTTON GROUP radi**

- 라디오 버튼의 그룹으로 설정하며 화면에는 RadioButton의 형태로 값을 입력 받음.

**● FOR TABLE dbtab**

- 자주 사용되는 형태는 아니며 Logical Database Programming의 경우에 사용되는 형태
- 이 Option을 사용하면 datab 테이블과 관련되어 사용하게 됨

**● FOR NODE node**

- LDB를 사용할 경우에 Nodes로 선언하여 사용함

**● AS SEARCH PATTERN**

- Search Help Selection을 사용하여 데이터를 찾고자 할 때 사용

**● VALUE-REQUEST**

- F4 Key를 이용하여 값을 찾을 수 있음
- 보통 ABAP Dictionary의 형태를 이용하는 Like 문을 사용

**● HELP-REQUEST**

- F1 Key를 이용하여 도움말을 사용

**● VISIBLE LENGTH vlen**

- 화면에 보이는 변수의 길이를 정의하는 것
- 선언에는 20자리지만 이 값을 5로 하면 화면에는 5자리만 보임

**● VALUE CHECK**

- Foreign Key로 연결한 것과 동일하며 입력한 값을 Check 하는 기능

● **USER-COMMAND ucom**

- CheckBox 또는 RadioButton으로 정의된 변수들에만 사용할 수 있음
- RadioButton을 누르거나 CheckBox에 값을 Check 하면 정의된 ucom의 Code 값으로 처리해야 할 내용을 처리할 수 있음.