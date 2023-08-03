# “NO INTERVALS." 옵션

생성자: 최예훈
생성 일시: 2023년 8월 3일 오전 11:32
태그: ABAP

```abap
SELECT-OPTIONS SO_CAR FOR MARA-ERNAM NO INTERVALS.

**AT SELECTION-SCREEN ON VALUE-REQUEST FOR SO_CAR-LOW.**
```

"NO INTERVALS." 옵션을 사용하지 않았다면, **`SO_CAR-LOW.`**와 **`SO_CAR-HIGH.`**는 범위의 하한과 상한 값을 나타냅니다. 이 경우 **`SO_CAR-LOW.`**는 범위의 시작값을 나타내며 **`SO_CAR-HIGH.`**는 범위의 끝값을 나타냅니다.

예를 들어, 사용자가 **`SO_CAR-LOW.`**에 "A"를 입력하고 **`SO_CAR-HIGH.`**에 "D"를 입력한다면, 이는 "A"부터 "D"까지의 범위를 나타내며 해당 범위 내의 값을 선택하려는 의도를 나타냅니다.

반면에 "NO INTERVALS." 옵션을 사용한 경우, **`SO_CAR-LOW.`**와 **`SO_CAR-HIGH.`**는 개별 값을 나타냅니다. 이 경우 사용자가 **`SO_CAR-LOW.`**와 **`SO_CAR-HIGH.`** 각각에 개별 값을 입력하면, 해당 값들을 독립적으로 선택하려는 의도를 나타냅니다.

정리하자면, "NO INTERVALS." 옵션을 사용하지 않은 경우 **`SO_CAR-LOW.`**와 **`SO_CAR-HIGH.`**는 범위의 하한과 상한 값을 나타내며, "NO INTERVALS." 옵션을 사용한 경우 각각은 개별 값을 나타냅니다.