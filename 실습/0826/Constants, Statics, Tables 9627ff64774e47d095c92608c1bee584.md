# Constants, Statics, Tables

생성자: 최예훈
생성 일시: 2023년 8월 26일 오후 10:39
태그: ABAP

### Constants

프로그램에서 자주 사용되는 값을 상수로 선언하여 사용 가능

상수로 선언하면 프로그램 내에서 바꿀 수 없다.

여러 곳에 사용되는 변수이고 변수 값을 변경해야 할 경우 한 번만 변경하면 되서 편리하다.

특정 숫자를 상수로 지정하여 코드의 가독성을 향상 시킬 수 있다.

```abap
CONSTANTS c.....
VALUE [val | IS INITIAL].

CONSTANTS : BEGIN OF crec,
...
						END OF crec.
```

### STATICS

원래는 서브루틴 구문 내에 선언된 Local 변수는 PERFORM 구문으로 호출될 때마다 초기화된다. 

Statics는 프로그램의 서브루틴에 사용되는 Local 변수의 값을 지속적으로 저장하여야 할 경우 사용된다.

FORM 구문 내의 STATICS로 선언된 Local 변수는 프로그램이 종료될 때까지 값을 유지하게 된다. 

```abap
STATICS f.

STATICS : BEGIN OF struc,
...
					END OF struc.
```

### TABLES

TABLES 구문을 사용하여 ABAP Dictionary talve, view, Structure를 선언하게 되면 ABAP Dictionary 타입과 같은 데이터 오브젝트를 생성하게 된다.

이것을 Table Work Object라고 부르며 프로그램 내에서 구조체처럼 사용할 수 있다.

```abap
TABLES : <dbtab>.
```

```abap
SELECT * FROM scarr.(INTO SCARR 없어도 됨)
```

테이블과 구조체 이름이 같으면 INTO SCARR 구문 생략