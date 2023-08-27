# SAP Memory, ABAP Memory

생성자: 최예훈
생성 일시: 2023년 8월 27일 오후 3:45
태그: ABAP

## 외부 세션, 내부 세션

사용자가 SAP GUI에 로그인 하면 하나의 사용자 터미널 세션이 생성된다.

하나의 터미널 세션으로 6개의 윈도우를 실행가능하다.

각각의 윈도우는 Memory 영역을 공유하고 있다.

외부 세션 = 메인 세션 은 6개의 윈도우 작업 창을 의미한다

하나의 외부 세션은 최대 20개의 내부 세션을 가질 수 있다.

내부 세션은 하나의 외부 세션 내에서 실행중인 프로그램이 다른 프로그램을 호출할 때 생성되는 내부적인 세션이다. 

## SAP Memory, ABAP Memory

### SAP Memory

사용자 터미널 세션 내의 모든 외부 세션에서 접근할 수 있는 Global Memory이다.

즉, 사용자가 한 GUI 로그인 화면을 이용해 로그인한 상태에서 실행된 모든 프로그램에서 공유되는 메모리 영역

```abap
GET / SET PARAMETER
```

구문을 이용해서 파라미터를 전달

### ABAP Memory

같은 외부 세션 내에서 수행되고 있는 모든 프로그램에서 공유되는 메모리

즉, ABAP 메모리는 개별 외부 세션에 하나만 존재한다.

개별 프로그램은 각자의 내부 세션을 가지며, 내부 세션은 스택에 쌓인다.

```abap
EXPORT obj1 .. objn TO MEMORY ID key.
IMPORT obj ... objn FROM MEMORY ID key.
```

## SAP Memory 데이터 넘겨주기

메모리 파라미터를 넘겨주어 INPUT FEILD에 값을 입력하는 것을 SPA/GPA 기술이라 한다.

프로그램 간 데이터를 넘겨주려면 SPA/GPA 기술이 필요하다

SET/GET 구문을 이용한다

쓰이는 경우

만약 사용자가 구매 오더를 생성한 후에 공급업체 코드를 잘 못 입력한 경우 다시 구매 오더 리스트를 조회하여 해당 오더를 찾을 수도 있지만 구매오더 생성 시에 구매 오더 메모리 파라미터에 해당 값을 저장하면 바로 구매오더 번호를 확인할 수 있다.

그리고 프로그램에서 다른 프로그램을 호출할 경우 Selection Screen을 건너뛰고 데이터를 조회할 수 있다.

물론 이 경우 SUBMIT 구문을 이용 할 수도 있다.

SAP 메모리에 저장된 값을 확인하려면 디버깅 모드에서 SAP MEMORY를 조회하면 된다.

디버거에서 <GO TO> → [SYSTEM AREAS] → [SAP MEMORY]

### 활용(TYPE-1)

```abap
REPORT Z11_02.

TABLES sbook.

START-OF-SELECTION.
	WRITE: 'DOUBLE click a list'.

SKIP.
SELECT * FROM sbook UP TO 5 ROWS.

WRITE : / sbook-carrid, sbook-connid,
					sbook-fldate, sbook-bookid.

* HIDE 구문은 필드의 값을 HIDE 메모리 영역으로 저장하는 역할
* WRITE 구문 바로 아래에 기술
HIDE : sbook-carrid, sbook-connid,
				sbook-fldate, sbook-bookid.

AT LINE-SELECTION.
	SET PARAMETER ID: 'CAR' FIELD sbook-carrid,
										 'CON' FIELD sbook-connid.

CALL FUNCTION 'Z11_03'AND SKIP FIRST SCREEN.
" 조회 리스트 라인을 더블클릭하게 되면 AT LINE-SELECTION 영역이 실행되고 파라미터에 값이 저장된다.

```

```abap
REPORT Z11_03.

PARAMETERS : p_car TYPE sbook-carrid MEMORY ID car,
							p_con TYPE sbook-connid MEMORY ID con.

WRITE : P_CAR, P_CON.
```

SM93에서 트랜젝션 코드 생성 

### 활용(TYPE M)

스크린 페인터 필드 속성창 PARAMETER ID 필드에 메모리 ID 명을 입력

화면 필드에 파라미터 값을 보이려면 GET PARAMETER 체크박스 체크

값을 입력하였을 때 파라미터 값을 변경하려면 SET PARAMETER 체크박스를 선택

만약 필드 속성이 FROM dict(ABAP dictionary 속성을 상속받음) 속성이 설정되어 있고 DATA Element에 파라미터가 설정되어 있다면 자동으로 파라미터가 추가된다.

### SAP 메모리 파라미터 생성

두 가지 방법이 존재한다.

1. SE80에서 EDIT Object를 이용해 파라미터 ID 생성
2. SM30 트랜젝션을 이용해 테이블을 생성

### SAP 메모리 할당

```abap
SET PARAMETER ID <pid> FIELD <f>. 
```

필드 <f>를 <pid>에 저장하려면 이 구문을 사용

pid는 최대 20문자 길이까지 가능

pid가 이미 값이 지정되어 있다면 겹쳐쓰게 된다.

pid가 생성되어 있지 않다면 ABAP Editor상에서 더블클릭하여 생성가능

### SAP 메모리 읽기

```abap
GET PARAMETER ID <pid> FIELD <f>. 
```

pid에 저장된 값을 필드 <f>에 저장하게 된다

pid  메모리 id가 존재하지 않는다면 시스템 변수 sy-subrc에 4가 할당되고 존재한다면 0이 할당된다.

## ABAP Memory 데이터 넘겨주기

EXPORT 구문을 이용하려 데이터 오브젝트(필드. 인터널테이블, 구조체)를 ABAP 메모리에 저장할 수 잇따.

파라미터 ID를 생성해야 하는 SAP 메모리와 다르게 ABAP 메모리를 사용하는 경우 메모리 ID명을 프로그램 레벨에서 임의로 지정할 수 있다.

EXPORT/IMPORT 구문을 사용한다.

내부 세션에서만 데이터가 공유되므로 다른 외부 세션의 프로그램에 데이터를 넘겨줄 수 없다. → 새 창 띄워서 작업할 때는 메모리 영역이 사라진다.

다른 프로그램을 호출할 때 데이터 오브젝트(인터널 테이블 등)을 넘겨주기 위해 사용된다.

다른 프로그램을 호출할 때 CALL TRANSACTION, SUBMIT, CALL DIALOG를 사용하면 내부 세션이 생성되고 ABAP 메모리에 존재하는 데이터기ㅏ 공유된다.

```abap
EXPORT obj1 .. objn TO MEMORY ID key.
IMPORT obj ... objn FROM MEMORY ID key.
```

```abap
FREE MEMORY [ID key].
* ID key 를 생략할 경우 전체 메모리 삭제
```

### 활용

```abap
TABLES : sflight.

DATA : gt_sflight TYPE TABLE OF sflight.

SELECT *
INTO CORRESPONDING FIELD OF TABLE gt_sflight
FROM sflight
WHERE carrid = 'AA'.

EXPORT gt_sflight TO MEMORY ID 'TEST_ID'.

CALL TRANSACTION 'Z11_06'
AND SKIP FIRST SCREEN.
```

EXPORT 구문에서 TEST_ID라는 ID 명으로 gt_sflight 인터널 테이블을 ABAP 메모리 영역에 저장 그리고 Z11_06 호출

```abap
REPORT Z11_06

TABLES : sflight.

DATA : gt_sflight TYPE TABLE OF sflight.

IMPORT gt_sflight FROM MEMORY IF 'TEST_ID'.
```

IMPORT  구문으로 ABAP MEMORY 의 데이터를 인터널 테이블에 저장