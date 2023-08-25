# LOCK Object

생성자: 최예훈
생성 일시: 2023년 8월 25일 오후 1:04
태그: ABAP

[ABAP 자료실 - ABAP [Dictionary] Lock Object (sapjoy.co.kr)](http://sapjoy.co.kr/abappds/56937)

## LOCK Object란

아무다 ABAP DICTIONARY의 테이블을 동시에 편집하다보면 데이터의 일관성에 오류가 생길 수 있다

따라서 한번에 접근하는 프로그램을 제한하여 얘 끝나면 쟤 프로그램 접근할 수 있게 하는 개념이 바로 Lock

**지금 프로그램이 사용하는 테이블 전체를 잠글 수도 있고** 

**지금 프로그램이 사용하는 row단위로 잠글 수도 있다.**

DBMS는 데이터 변경 요청이 오면 물리적으로 테이블 데이터에 Lock을 설정한다

데이터에 접근하려는 사용자는 Lock 설정이 해지될 때까지 기다려야 한다. 

DB COMMIT이 수행되는 DB LUW의 마지막에 DBMS는 LOCK 설정을 해지하게 된다.

![Untitled](LOCK%20Object%20026ad3dad1b94f63a411aa2ed78af1b4/Untitled.png)

개별 스크린이 종료될 때 마다 내부적으로 DB COMMIT이 수행되기 때문에 개별 스크린마다 LOCK을 설정하고 해지하는 작업이 필요하다

데이터베이스 레벨의 LOCK 설정은 여러 개의 스크린에서 데이터 변경 작업을 해야하는 SAP에서는 충분한 메커니즘을 제공하지 못하기 때문에 LOCK Object가 필요하다.

SAP LOCK Object는 Lock Object를 생성하여 개별 프로그램 마다 Lock을 설정하거나 해지하는 구문을 추가해야 한다. 그리고 사용자가 프로그램을 실행하여 데이터에 접근할 때 LOCK이 설정되어 있는지 확인하는 스크립트를 추가해야 한다. 

![Untitled](LOCK%20Object%20026ad3dad1b94f63a411aa2ed78af1b4/Untitled%201.png)

LOCK Object는 SE11에서 생성/변경/조회할 수 있는 ABAP DICTIONARY로서

여러 프로그램이 같은 데이터에 접근 할 수 있도록 동기화 하는 역할을 담당한다.

프로그램 1이 데이터를 수정할 때 프로그램 2는 변경 중인 데이터에 접근하지; 못하도록 함

이 때 프로그램 1은 해당 데이터에 LOCK을 설정하게 된다. 

LOCK 메커니즘이 제공하는 기능

1. 프로그램이 데이터를 읽고 변경한 후에 다른 프로그램에 완료하였다고 전한다.
2. 프로그램이 다른 프로그램에 의해 변경 중인 상태의 데이터를 읽는 것을 방지한다.

LOCK을 설정하려면 ABAP DICTINARY에 LOCK Object가 생성되어있어야 한다.

LOCK Object가 활성화될 때 LOCK을 설정하고 해지하는 다음 두 개의 Function module이 시스템에 의해 자동으로 생성된다

```abap
ENQUEUE_<LockObjectname>
EDQUEUE_<LockObjectname>
```

### LOCK 절차

![Untitled](LOCK%20Object%20026ad3dad1b94f63a411aa2ed78af1b4/Untitled%202.png)

1. Request lock
    
    프로그램에서 데이터에 대한 LOCK을 요청한다. 
    
2. Enter lock
    
    LOCK 데이터의 조건이 LOCK TABLE에 입력된다. 
    
3. Set lock
    1. 프로그램 요청에 의해 lock이 설정된다. 이 상태는 프로그램이 unlock을 수행하거나 프로그램이 종료되는 시점까지 유지된다. 
    2. 모든 lock은 프로그램에 의해 설정되므로 해당 프로그램에서 해지해야 한다.
    3. 데이터가 잠긴 상태에서 다른 프로그램에서 데이터에 접근하면 이 요청은 Lock table에 저장된다.
    4. 작업하고자 하는 데이터를 정확하게 설정하기 위해 lock object는 테이블의 key 필드로 구분될 수 있도록 설정하는 것이 바람직하다.
4. Access Data
    
    lock을 설정한 프로그램에서만 해당 데이터에 접근하여 작업할 수 있다. 
    
    ![Untitled](LOCK%20Object%20026ad3dad1b94f63a411aa2ed78af1b4/Untitled%203.png)
    
    ZAEMPCERT 테이블에 Lock Object를 생성하면 자동으로 Lock Module 함수가 만들어진다. 그리고 이 함수를 프로그램에서 호출하면 EMP_NO 필드가 ‘0001’인 데이터는 잠기게 된다.
    
    이렇게 Lock이 설정된 데이터를 관리하기 위한 Lock 프로세스는 모든 프로그램의 협조가 필요하다.
    
    Lock 이 설정되었을 때 다른 프로그램에서 데이터에 접근하여 작업하기 위해서는 Lock을 중지시켜 달라는 요청을 할 수 있다.
    
    이러면 Lock Mode에 따라서 다른 프로그램의 요청을 수락 혹은 거절을 할 수 있다.
    
    ![Untitled](LOCK%20Object%20026ad3dad1b94f63a411aa2ed78af1b4/Untitled%204.png)
    

## LOCK  Object 생성하는 방법

1. 어떤 테이블에 대해 LOCK Object 생성
2. Lock Object에 필요한 테이블 추가(Primary Table)
3.  Primary Table과 외부 키로 연결된 필요한 테이블 추가 가능
    1. 외부 키로 연결된 데이터들이 함께 잠기게 할 수 있다.
        
        ![Untitled](LOCK%20Object%20026ad3dad1b94f63a411aa2ed78af1b4/Untitled%205.png)
        
4. Lock Parameter탭에서 key 설정. 기본적으로 PK가 선택되어 있다.
5. 생성 완료
6. Object Lock을 생성하여 활성하면
    
    ```abap
    ENQUEUE_<LockObjectname>
    EDQUEUE_<LockObjectname> 가 생성된다. 
    ```
    
    LOCK이 걸려있는지 확인하는 코드 SM12 - LOCK이 걸린 상태의 프로그램 목록 조회 가능
    
    - 이지아밥 캡쳐
        
        ![Untitled](LOCK%20Object%20026ad3dad1b94f63a411aa2ed78af1b4/Untitled%206.png)
        
        ![Untitled](LOCK%20Object%20026ad3dad1b94f63a411aa2ed78af1b4/Untitled%207.png)
        
        ![Untitled](LOCK%20Object%20026ad3dad1b94f63a411aa2ed78af1b4/Untitled%208.png)
        

## LOCK Object 사용 예시

```abap
CALL FUNCTION 'ENQUEUE_EZ_ZEMP'
	EXPORTIONG
		MODE_ZEMP = 'E'
		MANDT = SY-MANDT
		EMP_NO = '0001'
		X_EMP_NO = ' '
		_SCOPE = '2'
		_WAIT = ' '
		_COLLECT = ' '.

IF sy-subrc eq 0.
	WRITE 'data is Locked'.
ENDIF.
```

실행하게 되면

데이터가 잠겼다는 메시지를 화면에 보여준다.

SAP Lock 메커니즘을 활용하고자 하면 반드시 모든 프로그램에 Lock 함수를 추가하여야 한다.

Lock은 같은 프로그램 내에서만 설정과 해지가 가능하므로 A 프로그램에서 설정한 Lock을 B 프로그램에서 해지할 수 없다

따라서 항상 같은 프로그램 내에서 ENQUEUE와 DEQUEUE 함수가 존재하도록 프로그래밍 하거나

ENQUEUE 함수를 호출 후 DEQUEUE_ALL 함수를 호출하여 주는 것도 좋은 방법이다.

```abap
CALL FUNCTION 'DEQUEUE_ALL'
	EXPORTIONG
		_SUNCHRON = ' '
```

테이블 단위로 LOCK을 하는 경우

```abap
DEQUEUE_ALL

ENQUEUE_E_TABLE 

    [Table 단위로 Lock을 설정하는 경우]
DEQUEUE_E_TABLE

    [Table 단위로 unLock을 설정하는 경우]
```

## ENQUEUE_READ

하나의 생산 오더를 여러 명이 동시에 변경하는 일과 같이 SAP 시스템에는 같은 데이터를 변경하는 일이 자주 발생한다.

ENQUEUE_READ 함수를 이용하여 해당 데이터가 잠겨 있는 상태인지 체크할 수 있다.

표준 프로그램은 항상 데이터를 변경할 때 Lock을 설정하기 때문에 ENQUEUE_READ를 사용하여 Lock 여부를 체크하는 습관이 필요하다.

```abap
ㅍ [LOCK의 상태를 확인할 때 사용]
결국 이 함수를 써먹기 위해 이부분을 공부한것이다.

Lock의 설정과 해제를 반복하는 상황에서
내가 속한 프로그램이 Update를 하기전

지금 변경하려는 Key값에 Lock이 설정되어

있는지를 먼저 확인후 갱신을 시행해야함.
```

```abap
CALL FUNCTION 'ENQUEUE_READ'
*   EXPORTING
*     GCLIENT                     = SY-MANDT
*     GNAME                       = ' '
*     GARG                        = ' '
*     GUNAME                      = SY-UNAME
*     LOCAL                       = ' '
*     FAST                        = ' '
*     GARGNOWC                    = ' '
*   IMPORTING
*     NUMBER                      =
*     SUBRC                       =
    TABLES
      enq                         =
*   EXCEPTIONS
*     COMMUNICATION_FAILURE       = 1
*     SYSTEM_FAILURE              = 2
*     OTHERS                      = 3
            .
  IF sy-subrc <> 0.
* MESSAGE ID SY-MSGID TYPE SY-MSGTY NUMBER SY-MSGNO
*         WITH SY-MSGV1 SY-MSGV2 SY-MSGV3 SY-MSGV4.
  ENDIF.
```

Lock되어있는 테이블의 데이터 리스트를 반환해준다. 잠금 오브젝트를 활용하지 않더라도  lock 상태인 전체 리스트 오브젝트를 조회할 수 있다.

![Untitled](LOCK%20Object%20026ad3dad1b94f63a411aa2ed78af1b4/Untitled%209.png)

## EDIT LOCK

지금 수정하고 있는 프로그램을 다른 사람이 수정하지 못하도록 하는 것

Lock Object와는 다른 개념으로 알고 있자. 참고

sm12에서 조회하면 현재 수정중인 프로그램 뜬다.