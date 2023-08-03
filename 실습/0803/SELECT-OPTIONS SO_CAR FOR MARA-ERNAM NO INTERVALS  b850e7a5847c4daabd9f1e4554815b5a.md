# SELECT-OPTIONS SO_CAR FOR MARA-ERNAM NO INTERVALS. 구문

생성자: 최예훈
생성 일시: 2023년 8월 3일 오전 11:31
태그: ABAP

**`NO INTERVALS`** 옵션은 ABAP의 **`SELECT-OPTIONS`** 문에서 사용되는 옵션 중 하나입니다. 이 옵션은 선택 옵션(Selection Option)을 정의할 때 범위를 지정할 수 없음을 나타냅니다.

**`SELECT-OPTIONS`** 문은 데이터베이스 테이블의 필드에 대한 범위를 나타내는 Range 변수를 생성하는 데 사용됩니다. 이때 **`NO INTERVALS`** 옵션을 사용하면 사용자가 필터 범위를 지정하는 대신 개별 값만을 선택할 수 있습니다. 즉, 범위를 나타내는 것이 아니라 하나의 값만을 선택할 수 있게 됩니다.

예를 들어, **`SELECT-OPTIONS SO_CAR FOR MARA-ERNAM NO INTERVALS.`** 문을 사용하면 사용자는 **`SO_CAR`** 옵션에 특정 필드(**`MARA-ERNAM`**)의 개별 값을 선택할 수 있습니다. 범위를 지정하는 것은 불가능하며, 여러 개의 개별 값만 선택할 수 있습니다. 만약 범위를 지정하고자 한다면 **`NO INTERVALS`** 옵션을 사용하지 않고 단순한 **`SELECT-OPTIONS SO_CAR FOR MARA-ERNAM.`** 문을 사용하면 됩니다.