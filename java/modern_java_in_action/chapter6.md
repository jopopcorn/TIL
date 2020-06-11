## Collect

- 다양한 스트림 요소 누적 방법을 인수로 갖는 최종 연산
- 다양한 요소 누적 방식은 Collector 인터페이스에 정의되어 있다.

## Collector

- 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되어 있다.
- collect로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의한다.
- Collectors 클래스에서 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 정적 팩토리 메서드를 제공한다.
- Collectors에서 제공하는 정적 팩토리 메서드의 기능은 크게 리듀싱과 요약, 요소 그룹화, 요소 분할로 구분할 수 있다.
- Collector 인터페이스에 정의된 메서드를 구현해서 커스텀 컬렉터를 개발할 수 있다.

- 리듀싱
    - 컬렉터로 스트림의 모든 항목을 하나의 결과로 합칠 수 있다(컬렉션으로 재구성할 수 있다).

        ex) counting(): 요소 갯수를 카운팅하는 메서드

    - 최대, 최솟값을 계산할 수 있다.

        ex) Collectors.maxBy - 스트림의 최댓값을 구하는 메서드, Collectors.minBy - 스트림의 최솟값 구하는 메서드

- 요약
    - 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산을 요약 연산이라고 한다. (리듀싱 기능)
    - Collectors.summingInt: 객체를 int로 매핑하는 함수를 인수로 받고, int로 매핑한 컬렉터를 반환하는 메서드

        ex) summingInt 외에도 summingDouble, summingLong 메서드도 같은 방식으로 동작함

    - 평균값 계산 등의 연산도 요약 기능으로 제공된다.

        ex) averagingInt, averagingLong, averagingDouble 등 다양한 형식의 숫자 집합의 평균 계산

    - 위의 연산들 중 두 개 이상의 연산을 한 번에 수행해야 할 때 사용하는 팩토리 메서드도 제공한다.

        ex) summarizingInt: 요소 수, 합계, 평균, 최대 및 최솟값을 int 형식으로 한 번에 수집함

    - joining: 스트림의 각 객체에 toString 메서드를 호출해 추출한 모든 문자열을 하나의 문자열로 연결해서 반환한다. 이 메서드는 내부적으로 StringBuilder를 이용해서 문자열을 하나로 만든다. 또한 요소 사이에 구분 문자열을 넣을 수 있도록 오버로드된 joining 팩토리 메서드도 있다.

    ```java
    String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
    ```

    - 범용 팩토리 메서드인 reducing 팩토리 메서드로도 위의 연산을 정의할 수 있다. 그러나 **가독성이 중요하므로 범용 팩토리 메서드보다는, 특화된 컬렉터를 사용하는 것이 좋다.**

    ```java
    //동일한 연산, 다른 코드
    int totalCalories = menu.stream().collect(reducing(0, 
                                      Dish::getCalories, (i, j) -> i + j));

    int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
    ```

- 그룹화
    - Collectrors.groupingBy: 스트림의 요소를 그룹화하는 팩토리 메서드. 다수준 그룹화도 제공한다.
    - n수준 그룹화의 결과는 n수준 트리 구조로 표현되는 n수준 맵이 된다.
    - 어떤 함수를 기준으로 스트림이 그룹화되면, 그 함수는 분류 함수라고 부른다.
    - 복잡한 분류 기준이 필요한 상황에서는 메서드 참조를 분류 함수로 사용할 수 없다.
    - filtering: 각 그룹의 요소와 필터링 된 요소를 재그룹화하는 메서드.
    - mapping: 매핑 함수와 각 항목에 적용한 함수를 모으는 데 사용하는 또 다른 컬렉터를 인수로 받는 메서드.
    - flatMapping: 평면화된 매핑을 제공하는 메서드. 리스트가 아닌 집합으로 그룹화해 중복 태그를 제거할 수 있음.
    - collectingAndThen: 컬렉터가 반환한 결과를 다른 형식으로 활용할 수 있는 메서드.
    - toList: 스트림의 모든 항목을 리스트로 수집하는 메서드.
    - toSet: 스트림의 모든 항목을 중복 없는 집합으로 수집하는 메서드.
    - toCollection: 스트림의 모든 항목을 발행자가 제공하는 컬렉션으로 수집.
- 분할
    - 분할 함수라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능이다.
    - 결과적으로 그룹화 맵은 최대 (참 아니면 거짓의 값을 갖는 ) 두 개의 그룹으로 분류된다.
    - partitioningBy: 스트림의 요소를 분할하는 메서드. 반환한 맵 구현은 참과 거짓만 포함하므로 간결하고 효과적이다.
