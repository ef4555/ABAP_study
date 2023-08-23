# 데이터 변화되었을때 data_changed 로직 정리

생성자: 최예훈
생성 일시: 2023년 8월 23일 오전 10:59
태그: ABAP

[[한설날][ABAP] 클래스 'CL_GUI_ALV_GRID' 이벤트 'DATA_CHANGED' (tistory.com)](https://seollal.tistory.com/686)

[[SAP ABAP] DATA_CHANGED : 네이버 블로그 (naver.com)](https://m.blog.naver.com/howwithus/221708021504)

```abap
CLASS lcl_event_receiver IMPLEMENTATION.
	METHOD handle_data_changed. " data_change 이벤트가 발생되면 아래 로직을 수행
	    DATA : ls_makt TYPE makt, " 수정할 딕셔너리 테이블의 구조를 가진 구조체
	           ls_outtab LIKE LINE OF it_list2.
	    FIELD-SYMBOLS: <fs> TYPE table.
	    " er_data_changed
	    ASSIGN er_data_changed->mp_mod_rows->* TO <fs>. " 바뀐 데이터들을 필드 심볼에 저장(필드 심볼 it_list2의 구조로 바뀐 데이터 row를 저장) "
	    LOOP AT <fs> INTO ls_outtab. "
...
```

### er_data_changed의 구조

이벤트가 발생했을 때 

er_data_changed가 생김

이것은 

TYPE REF TO CL_ALV_CHANGED_DATA_PROTOCOL인 인스턴스

![Untitled](%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%92%E1%85%AA%E1%84%83%E1%85%AC%E1%84%8B%E1%85%A5%E1%86%BB%E1%84%8B%E1%85%B3%E1%86%AF%E1%84%84%E1%85%A2%20data_changed%20%E1%84%85%E1%85%A9%E1%84%8C%E1%85%B5%E1%86%A8%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5%20a662818d8af94d16a3adfc22d8f0c5fe/Untitled.png)

```abap
ASSIGN er_data_changed->mp_mod_rows->* TO <fs>. 
er_data_changed중에서 변화된 row담긴 인터넬 테이블 꺼내와서 필드심볼<fs>에 역참조
```

![Untitled](%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%92%E1%85%AA%E1%84%83%E1%85%AC%E1%84%8B%E1%85%A5%E1%86%BB%E1%84%8B%E1%85%B3%E1%86%AF%E1%84%84%E1%85%A2%20data_changed%20%E1%84%85%E1%85%A9%E1%84%8C%E1%85%B5%E1%86%A8%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5%20a662818d8af94d16a3adfc22d8f0c5fe/Untitled%201.png)

![Untitled](%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%20%E1%84%87%E1%85%A7%E1%86%AB%E1%84%92%E1%85%AA%E1%84%83%E1%85%AC%E1%84%8B%E1%85%A5%E1%86%BB%E1%84%8B%E1%85%B3%E1%86%AF%E1%84%84%E1%85%A2%20data_changed%20%E1%84%85%E1%85%A9%E1%84%8C%E1%85%B5%E1%86%A8%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5%20a662818d8af94d16a3adfc22d8f0c5fe/Untitled%202.png)