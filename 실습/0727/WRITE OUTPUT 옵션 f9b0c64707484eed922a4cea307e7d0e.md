# WRITE OUTPUT 옵션

생성자: 최예훈
생성 일시: 2023년 7월 27일 오후 1:58
태그: ABAP

```abap
DATA: l_color TYPE char25 VALUE 'Color Test'.
DATA: l_intens TYPE char25 VALUE 'Intensified test'.
DATA: l_inverse TYPE char25 VALUE 'Inverse test'.
DATA: l_hotspot TYPE char25 VALUE 'hotspot test'.
DATA: l_input TYPE char25 VALUE 'Input test'.
DATA: l_frame TYPE char25 VALUE 'Frame test'.

* 필드의 색을 변경
* 1 파랑, 2 노랑 3 초록 4 초록 5 빨강 6 보라 7 주황 
FORMAT COLOR 5 ON.
WRITE : / l_color.
FORMAT COLOR OFF.

* 색상이 강조됨(blue)
FORMAT INTENSIFIED ON.
WRITE : / l_intens.
FORMAT INTENSIFIED OFF.

* 색상이 반전
FORMAT INVERSE ON.
WRITE : / l_inverse.
FORMAT INVERSE OFF.

* 클릭 = 더블클릭(핫스팟 기능)
FORMAT HOTSPOT ON.
WRITE : / l_hotspot.
FORMAT HOTSPOT OFF.

* 입력필드로 변환
FORMAT INPUT ON.
WRITE : / l_input.
FORMAT INPUT OFF.

FORMAT FRAMES ON.
WRITE : / l_frame.
FORMAT FRAMES OFF.

FORMAT RESET.
```

![Untitled](WRITE%20OUTPUT%20%E1%84%8B%E1%85%A9%E1%86%B8%E1%84%89%E1%85%A7%E1%86%AB%20f9b0c64707484eed922a4cea307e7d0e/Untitled.png)