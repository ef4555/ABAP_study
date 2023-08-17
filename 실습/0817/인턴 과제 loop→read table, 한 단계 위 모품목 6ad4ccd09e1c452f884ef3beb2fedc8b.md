# 인턴 과제 loop→read table, 한 단계 위 모품목 수량 가져오는 것 해결

생성자: 최예훈
생성 일시: 2023년 8월 17일 오후 1:02
태그: ABAP

일단 과제 완성 → 아님. 가라친 부분 발견 → 해결 8/16 15시

전에 loop를 쓸 때는 결과를 담을 테이블에 일단 정보를 다 넣고 그다음 한 방에 같은 정보를 가진 것들을 갱신하는 형식으로 만들었다.

이번에 read table로 바꾼 프로그램은

큰 loop 안에서 it_list 한 줄에 담긴 정보를 다른 테이블과 비교하면서 한 줄을 완성한 다음 결과 테이블에 넣었다.

처음에 read table로 바꿀 때 가장 어려웠던 점은 한방에 다 넣고 갱신하려니 다른 BOM 단계지만 같은 정보를 가진 것들이 여러개 있을 경우 갱신이 안된다는 것이었다. 

그런데 생각을 바꿔서 한 줄 씩 추출해서 전부 정보를 완전히 채우고 넣으니 해결되었다.

그리고 스탠다드 프로그램을 미처 생각하지 못하고 있었는데

오늘 점심에 스탠다드 BOM 프로그램이 중요하다는 말을 듣고 어제 봤던 CS12 프로그램을 찾아보니 

결과값이 올바르게 나오고 있는 지 이걸로 알 수 있었다.

내가 핑프이지 않았나 돌이켜보게 되었다. 

오늘 최종적으로 하위 자재들 한 단계 위 모품목 수량 가져오는 것 까지 해서 프로그램 제작 1차적으로 마무리 했다.

하위 자재들의 한 단계 위 모품목 수량 가져오는 것은

stpox와 cscmat을 뚫어져라 쳐다보고 이런저런 데이터 찾아보면서 연관성을 찾았다.

cscmat이 왜 필요한지 오늘 이제서야 알았다.

마지막 명세서 조건 때문에 그런것이었다.

cscmat이 정확히 무엇을 의미하는지도 존나 고민해서 알게 되었다.

이것을 스스로 알아가는게 과제의 의미였을까?

BOM 단계가 여러 단계인 샘플을 찾아다니고

디버거로 중간 BOM EXPL 단계를 까보면서 어떻게 상위 모품목에 매칭시킬 수 있는지 고민했다.

그리고 matnr_m 과 matnr_sub를 따로 칼럼을 만들어서

matnr_m에는 그 자재의 한 단계 위 모품목 자재의 자재번호를 넣고

matnr_sub에는 해당 자재의 자재코드를 넣었다.

최종 ALV에 출력될 때는 matnr_m은 출력되지 않는다.
read table 시에 key는 

```abap
READ TABLE it_cscmat WITH KEY index = it_stpox-ttidx. " 자재 카테고리(it_cscmat)에서 BOM 단계(INDEX: 범주 테이블 내 인덱스)를 키로 하여 모품목의 자재코드를 가져옴
    it_list-matnr_m = it_cscmat-matnr. " 구성부품의 그 위의 모품목 자재코드
    it_list-matnr_sub = it_stpox-idnrk. " 구성부품의 자재코드

((중략))

ELSEIF it_list-stufe NE 1.
      CLEAR it_stpox.
      READ TABLE it_stpox WITH KEY idnrk = it_list-matnr_m. " 하위 자재코드와 일치하는 정보를 it_stpox에서 불러옴
      it_list-bmeng    = it_stpox-menge. " 모품목의 수량
      it_list-bmein    = it_stpox-meins. " 모품목의 단위

    ENDIF.
```

범주 테이블 내 인덱스 기준으로 read table