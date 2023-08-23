# er_ data_changed-MP_MOD_ROWS

생성자: 최예훈
생성 일시: 2023년 8월 23일 오후 1:46
태그: ABAP

![Untitled](er_%20data_changed-MP_MOD_ROWS%20284781759a194e7dafd65f830248cad2/Untitled.png)

MP_MOD_ROWS는 데이터의 구조를 참조해서 변화된 row 전체를 따오고

MT_MOD_CELLS는 LVC_T_MODI 타입의 구조로 변화된 데이터를 따온다.

```abap
CLASS lcl_event_receiver IMPLEMENTATION.
  METHOD handle_data_changed. " data_change 이벤트가 발생되면 아래 로직을 수행
    DATA : ls_makt TYPE makt, " 수정할 딕셔너리 테이블의 구조를 가진 구조체
           ls_outtab LIKE LINE OF it_list2.

		FIELD-SYMBOLS: <fs> TYPE table.

    ASSIGN er_data_changed->mp_mod_rows->* TO <fs>. " 바뀐 데이터들을 필드 심볼에 역참조(필드 심볼 it_list2의 구조로 바뀐 데이터 row를 저장) 
    LOOP AT <fs> INTO ls_outtab. " 바뀐 데이터들이 담긴 필드심볼에서 한 줄 씩 꺼냄
...

```

근데 er_ data_changed-MP_MOD_ROWS로 불러오는게 대체 뭔지

er_ data_changed는 무슨 구조인지 모르겠다.

→ 찾았당

[A Twossun Place :: A Twossun Place (tistory.com)](https://twossun.tistory.com/)

![Untitled](er_%20data_changed-MP_MOD_ROWS%20284781759a194e7dafd65f830248cad2/Untitled%201.png)

data reference기 때문에 역참조를 해줘서 원래 값을 불러와야 하는 것