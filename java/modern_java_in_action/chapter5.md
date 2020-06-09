### 스트림 연산

**중간 연산**

- 스트림을 반환하면서 다른 연산과 연결되는 연산
- 중간 연산을 이용해서 파이프라인 구성할 수 있음
- 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않음 **(게으름)**
- 중간 연산으로는 어떤 결과도 생성할 수 없음

**최종 연산**

- 스트림 파이프라인을 처리해서 스트림이 아닌 결과를 반환하는 연산
- 스트림 파이프라인을 실행해 결과를 만듦

### 스트링 활용법

**필터링**

- filter(): 프레디케이트를 인수로 받아 필터링
- distinct(): 고유 요소로 이루어진 스트림 반환. 중복을 필터링함

```java
List<Dish> vegetarianMenu = menu.stream()
                                .filter(Dish::isVegetarian) //채식 요리인지 확인
                                .collect(toList());

List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
       .filter(i -> i % 2 == 0)
       .distinct()
       .forEach(System.out::println);
//결과: 2,4
```

**스트림 슬라이싱 (Java 9)**

: 스트림의 요소를 선택하거나 스킵하는 방법

- 프레디케이트를 이용한 슬라이싱 (Java 9)
    - takeWhile: 프레디케이트를 인수로 받아 조건에 어긋난 요소가 나오면 스트림을 슬라이스함. 무한 스트림을 포함한 모든 스트림에 프레디케이트를 적용해 반복 작업을 중단시킴.
    - dropWhile: takeWhile과 정반대의 작업 수행. 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버림. 그 지점에서 작업을 중단하고 남은 모든 요소를 반환함.
- 스트림 축소
    - limit(n): 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환. 최대 n개 반환할 수 있음. 정렬되지 않은 스트림에도 사용 가능하며, limit의 결과 또한 정렬되지 않은 상태로 반환됨.
- 요소 건너뛰기
    - skip(n): 처음 n개 요소를 제외한 스트림을 반환. n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출하면 빈 스트림이 반환됨. limit(n)과 skip(n)은 상호 보완적인 연산을 수행함.

**매핑**

: 특정 객체에서 특정 데이터를 선택하는 기능

- 스트림의 각 요소에 함수 적용하기
    - map: 함수를 인수로 받아 각 요소에 함수를 적용한 결과가 새로운 요소로 매핑됨.

    ```java
    //다른 map 메서드를 연결(chaining)할 수 있음
    List<Integer> dishNameLengths = menu.stream()
                                    .map(Dish::getName)
                                    .map(String::length)
                                    .collect(toList());
    ```

- 스트림 평면화
    - flatMap: 각 배열을 스트림이 아닌 스트림의 콘텐츠로 매핑함. 즉, 하나의 평면화된 스트림을 반환함. 스트림의 각 값을 다른 스트림으로 만든 다음에, 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행함

    ```java
    //두 개의 숫자 리스트가 주어졌을 때 모든 숫자 쌍의 리스트 반환하는 코드
    List<Integer> numbers1 = Arrays.asList(1, 2, 3);
    List<Integer> numbers2 = Arrays.asList(3, 4);
    List<int[]> pairs = numbers.stream()
                               .flatMap(i -> numbers.stream()
                                                    .map(j -> new int[]{i, j})
                               )
                               .collect(toList());
    //결과: [(1,3),(1,4),(2,3),(2,4),(3,3),(3,4)]
    ```

**검색과 매칭**

: 특정 속성이 데이터 집합에 있는지 여부를 검색. anyMatch, allMatch, noneMatch 메서드는 쇼트서킷(자바의 &&, ||와 같은 연산) 기법을 활용

- 프레디케이트가 적어도 한 요소와 일치하는지 확인
    - anyMatch: 프레디케이트가 주어진 스트림에서 적어도 한 요소와 일치하는지 확인함.
- 프레디케이트가 모든 요소와 일치하는지 검사
    - allMatch: 모든 요소가 주어진 프레디케이트와 일치하는지 검사함.
    - noneMatch: allMatch와 반대 연산을 수행. 주어진 프레디케이트와 일치하는 요소가 없는지 확인함.
- 요소 검색
    - findAny: 현재 스트림에서 임의의 요소 반환. 다른 스트림연산과 연결해서 사용할 수 있음.

    ```java
    //채식 요리인 것 중에서 임의의 요소를 반환함.
    Optional<Dish> dish = menu.stream()
                              .filter(Dish::isVegetarian)
                              .findAny()
                              .ifPresent(dish -> System.out.println(dish.getName());
    //ifPresent()의 조건에 따라 값이 있으면 값을 반환하고, 없으면 아무 일도 일어나지 않음.
    ```

- 첫 번째 요소 찾기

    findFirst: 스트림에서 첫 번째 요소를 반환하는 메서드

    ※ 병렬 실행에서는 첫 번째 요소를 찾기 어렵다. 요소 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 findAny를 사용한다.

**리듀싱 (Reducing) = 폴드 (Fold)**

- 스트림 요소를 조합해서 더 복잡한 질의를 표현하는 방법
- 스트림의 모든 요소를 반복 조합해서 값을 도출한다.
- reduce는 **초깃값**과 두 요소를 조합해서 새로운 값을 만드는 **연산**(람다 표현식)이 필요하다.
- 초깃값을 받지 않도록 오버로드된 reduce도 있다. 이 reduce는 Optional 객체를 반환한다.
- 스트림의 모든 요소의 합계, 최댓값 혹은 최솟값을 구하는데 활용할 수 있다.

```java
//Java 8에서는 Integer 클래스에 두 숫자를 더하는 정적 sum 메서드를 제공한다.
int sum = numbers.stream().reduce(0, Integer::sum);

//최댓값과 최솟값을 구하는 코드 (내장 메서드 max, min 활용)
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

**상태 없음, 상태 있음**

상태 없는 연산: map, filter 등은 각 요소를 받아 0 또는 결과를 출력 스트림으로 보냄. 즉, 내부 상태를 저장하지 않는 연산임.

상태 있는 연산: sorted, distinct 등의 메서드는 새로운 스트림을 반환하기에 앞서 스트림의 모든 요소를 버퍼에 저장해야 함. 데이터 스트림의 크기가 크거나 무한일 때 문제가 생길 수 있음.

+) reduce, sum, max와 같은 연산은 결과를 누적할 내부 상태가 필요함. 스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 한정(bounded)되어 있음.

### 쇼트서킷

- 프레디케이트와 일치하지 않는 결과가 나오면 나머지 표현식이 결과와 상관없이 전체 결과는 거짓이 된다.
- anyMatch, allMatch, noneMatch, findFirst, findAny, limit 등의 연산은 **전체 스트림을 처리하지 않아도 결과를 반환**할 수 있다.
- **원하는 요소를 찾았으면 즉시 결과를 반환**한다.

### Optional

값의 존재나 부재 여부를 표현하는 컨테이너 클래스

값이 존재하는지 확인하고 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공함

- isPresent(): Optional이 값을 포함하면 참(true)을 반환하고, 값을 포함하지 않으면 거짓(false)을 반환한다.
- ifPresent(Consumer<T> block): 값이 있으면 주어진 블록을 실행한다.
- T get(): 값이 존재하면 값을 반환, 없으면 NoSuchElementException을 일으킨다.
- T orElse(T other): 값이 있으면 값을 반환하고, 없으면 기본값을 반환한다.

### 기본형 특화 스트림

- 숫자 스트림을 효율적으로 처리하도록 돕는 스트림. 박싱 비용을 피할 수 있도록 한다.
- 각각의 인터페이스는 sum, max와 같은 숫자 관련 리듀싱 연산 메서드를 제공한다.
- 필요할 때 다시 객체 스트림으로 복원하는 기능을 제공한다.
- **특화 스트림은 오직 박싱 과정에서 일어나는 효율성과 관련 있으며, 스트림에 추가 기능을 제공하지 않는다.**
- IntStream, DoubleStream, LongStream이 기본형 특화 스트림이다.

### 스트림 만들기

- 값으로 스트림 만들기
    - Stream.of: 임의의 수를 인수로 받는 정적 메서드

    ```java
    Stream<String> stream = Stream.of("Modern ", "Java ", "In ", "Action");
    stream.map(String::toUpperCase).forEach(System.out::println);
    ```

    - Stream.empty(): 스트림을 비우는 메서드
- null이 될 수 있는 객체로 스트림 만들기 (Java 9)

    Stream.ofNullable: 조건에 대응하는 값이 없을 때 null을 반환할 수 있도록 하는 메서드

    ```java
    Stream<String> values = Stream.of("config", "home", "user")
                                  .flatMap(key -> Stream.ofNullable(System.getProperty(key)));
    ```

- 배열로 스트림 만들기

    Arrays.stream: 배열을 인수로 받는 정적 메서드

    ```java
    int[] numbers = {2,3,5,7,11,13};
    int sum = Arrays.stream(numbers).sum();
    //합계: 41
    ```

- 파일로 스트림 만들기

    Files.lines: 주어진 파일의 행 스트림을 문자열로 반환하는 메서드

- 함수로 무한 스트림 만들기
    - 무한 스트림(unbounded stream): 크기가 고정되지 않은 스트림. 즉, 무한한 개수의 요소를 가진 스트림
    - 무한한 값을 출력하지 않도록 보통 limit(n) 함수를 함께 연결해서 사용함.
    - Stream.iterate: 초깃값과 람다를 인수로 받아 연속된 일련의 값을 생산함. 새로운 값을 생성하면서도 기존 상태를 바꾸지 않음 (불변 상태).
    - Stream.generate: 생산된 각 값을 연속적으로 계산하지 않음. Supplier<T>를 인수로 받아 새로운 값을 생산함. 만들어진 객체는 가변 상태임.

+) 스트림을 병렬로 처리하면서 올바른 결과를 얻으려면 불변 상태 기법을 고수해야 함.
