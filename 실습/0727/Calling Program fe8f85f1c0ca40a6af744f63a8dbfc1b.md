# Calling Program

생성자: 최예훈
생성 일시: 2023년 7월 27일 오후 3:35
태그: ABAP

## 프로그램 호출

프로그램이 복잡한 기능을 수행할 경우 기능별로 프로그램을 나누어서 개발하는 방법이 있다(모듈화)

이렇게 구성된 패키지 프로그램은 메인 프로그램에서 사용자의 요구에 따라 각각의 프로그램을 호출해서 사용한다. 

![Untitled](Calling%20Program%20fe8f85f1c0ca40a6af744f63a8dbfc1b/Untitled.png)

SUBMIT 구문과 CALL TRANSACTION 구문의 가장 큰 차이점은 프로그램과 트랜잭션을 호출한다는 점이다

SUBMIT AND RETURN 구문과 CALL TRANSACTION 구문을 이용하여 다른 프로그램을 호출한 다음에 프로그램을 빠져나오면 이전 리스트로 돌아와야 한다. 이러한 작업을 문제없이 수행하려면 SUBMIT 과 AND RETURN을 함께 추가하면 된다. 다음과 같은 예제가 프로그램에 포함되어 있다면,

```abap
REPORT ZTEST.
SUBMIT 'program' AND RETURN.
IMPORT g_carrid FROM MEMORY ID 'T_CID'.
```

프로그램을 호출한 후 원래 프로그램으로 복귀할 때 IMPORT 구문부터 수행된다. CALL TRANSACTION 구문도 마찬가지이다.