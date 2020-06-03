# Modern Java

### 동작 파라미터화

- 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록
- 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달함
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응하는 코드를 구현할 수 있으며. 엔지니어링 비용을 줄일 수 있음

### 전략 디자인 패턴 (Strategy Design Pattern)

- 각 알고리즘(전략이라 불리는)을 캡슐화하는 **알고리즘 패밀리를 정의**해둔 다음에 **런타임에 알고리즘을 선택**하는 기법
- 특정 계열의 알고리즘을 정의하고, 각 알고리즘을 캡슐화하며, 이 알고리즘들을 해당 계열 안에서 상호 교체가 가능하게 만듦
- 유연하고 재사용 가능한 객체지향 소프트웨어를 설계하기 위해 반복되는 디자인 문제를 해결

ex) 녹색인 사과를 선택하는 알고리즘, 150g 이상인 사과를 선택하는 알고리즘 등을 사과 선택 전략으로 캡슐화한다.

```java
//알고리즘 패밀리
public interface ApplePredicate{ 
	boolean test(Apple apple);
}

//전략 1
public class AppleHeavyWeightPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return apple.getWeight() >= 150;
	}
}

//전략 2
public class AppleGreenColorPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return GREEN.equals(apple.getColor());
	}
}
```

★ Predicate : 참 또는 거짓을 반환하는 함수로서, 선택 조건을 결정하는 함수형 인터페이스이다.

## 람다 (lambda)

- 메서드로 전달할 수 있는 익명 함수를 단순화한 것이다.
- 이름은 없지만, 파라미터 리스트, 바디, 반환 형식, 발생할 수 있는 예외 리스트는 가질 수 있다.
- 변수에 할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달할 수 있다.
- 함수형 인터페이스의 추상 메서드와 같은 시그니처를 갖는다. (함수형 인터페이스의 추상 메서드가 람다 표현식의 시그니처를 묘사한다.)
- 동작 파라미터 형식의 코드를 더 쉽게 구현할 수 있어, 코드가 간결하고 유연해진다.

### 람다의 특징

- 익명

    보통의 메서드와 달리 이름이 없으므로 익명이라고 표현

- 함수

    람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부름. 그러나 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함함.

- 전달

    람다 표현식을 메서드 인수로 전달하거나 변수로 저장

- 간결성

    익명 클래스처럼 자질구레한 코드를 구현할 필요 없음

### 람다 표현식

람다 표현식은 파라미터, 화살표, 바디로 이루어진다.

- 파라미터 리스트 (=람다 파라미터): Comparator의 compare 메서드 파라미터
- 화살표: 람다의 파라미터 리스트와 바디를 구분
- 람다 바디: 람다의 반환값에 해당하는 표현식

```java
/* 순서대로 파라미터 리스트, 화살표, 람다 바디 */
**(Apple a1, Apple a2)** -> **a1.getWeight().compareTo(a2.getWeight());**
```

### 함수형 인터페이스

- 정확히 하나의 추상 메서드를 지정하는 인터페이스
- 함수형 인터페이스에서 람다 표현식을 사용할 수 있다.
- 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.
- Comparator, Runnable, etc

+) 인터페이스는 디폴트 메서드(인터페이스의 메서드를 구현하지 않은 클래스를 고려해서 기본 구현을 제공하는 바디를 포함하는 메서드)를 포함할 수 있다. 많은 디폴트 메서드가 있더라도 **추상 메서드가 오직 하나면 함수형 인터페이스**다.

### 함수 디스크립터

- 람다 표현식의 시그니처를 서술하는 메서드 (=함수형 인터페이스의 추상 메서드 시그니처)
- Runnable 인터페이스는 인수와 반환값이 없는 시그니처이다. (run은 인수와 반환값이 없으므로)

### 실행 어라운드 패턴

: 자원 할당, 자원 정리 등과 같은 코드 중간에서 실행해야 하는 코드. 즉, 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 가진다.

1단계. 동작 파라미터화

2단계. 함수형 인터페이스를 이용하여 동작 전달

3단계. 동작 실행

4단계. 람다 전달

```java
//1. 동작 파라미터화
public String processFile() throws IOException{
	try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
		return br.readLine();
	}
}

//2. 동작 전달
public interface BufferedReaderProcessor{
	String process(BufferedReader b) throws IOException;
}

public String ProcessFile(BufferedReaderProcesser p) throws IOException{
	...
}

//3. 동작 실행
public String ProcessFile(BufferedReaderProcesser p) throws IOException{
	try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
		return p.process(br);
	}
}

//4. 람다 전달
String oneLine = processFile((BufferedReader br) -> br.readLine());
String twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

### 함수 인터페이스

: 다양한 람다 표현식을 사용하기 위해 공통의 함수 디스크립터를 기술한 것

- Predicate
    - T의 객체를 인수로 받아 boolean을 반환하는 test라는 추상 메서드를 정의
    - 따로 정의할 필요 없이 바로 사용할 수 있음
    - and, or과 같은 메서드도 있음
- Consumer
    - T 객체를 받아서 void를 반환하는 accept 추상 메서드를 정의
    - T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 사용
- Function
    - T를 인수로 받아서 제너릭 형식 R 객체를 반환하는 apply 추상 메서드를 정의
    - 입력을 출력으로 매핑하는 람다를 정의할 때 사용 (ex. 사과의 무게 정보를 추출하거나 문자열을 길이와 매핑)
- Supplier
    - T 형식의 시그니처를 갖는 추상 메서드 get 정의
- Callable
    - T 형식의 시그니처를 갖는 추상 메서드 call 정의

### 기본형 특화 (특화된 형식의 함수형 인터페이스)

제네릭 파라미터에는 참조형만 사용할 수 있다. 따라서 자바에는 기본형을 참조형으로 변환하는 기능을 제공한다.

기본형 - int, double, byte, char, ...

참조형 - Byte, Integer, Object, List, ...

boxing: 기본형 → 참조형

unboxing: 참조형 → 기본형

autoboxing: 박싱과 언박싱이 자동으로 이루어짐

자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱 동작을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공한다. = 기본형 특화 함수형 인터페이스

람다와 함수형 인터페이스

사용 사례 | 예시 | 대응하는 함수형 인터페이스 | 
---- | ---- | ---- |
boolean 표현 | (List<String> list) → list.isEmpty() | Predicate<List<String>>
객체 생성 | () → new Apple(10) | Supplier<Apple>
객체에서 소비 | (Apple a) → Sysout(a.getWeight()) | Consumer<Apple>
객체에서 선택/추출 | (String s) → s.length() | Function<String, Integer> 또는 ToIntFunction<String>
두 값 조합 | (int a, int b) → a * b | IntBinaryOperator
두 객체 비교 | (Apple a1, Apple a2) → a1.getWeight().compareTo(a2.getWeight()) | Comparator<Apple> 또는 BiFunction<Apple, Apple, Integer> 또는 ToIntBiFunction<Apple, Apple>


+) 함수형 인터페이스는 확인된 예외를 던지는 동작을 허용하지 않음. 따라서 예외를 던지는 람다 표현식을 만들려면 확인된 예외를 선언하는 함수형 인터페이스를 직접 정의하거나, 람다를 try/catch 블록으로 감싸야 함.

### 대상 형식

- **어떤 콘텍스트에서 기대되는 람다 표현식의 형식**을 대상 형식(target type)이라고 부른다.
- 할당문 콘텍스트, 메서드 호출 콘텍스트(파라미터, 반환값), 형변환 콘텍스트 등으로 **람다 표현식 형식을 추론**할 수 있다.
- 대상 형식이라는 특징 때문에 **같은 람다 표현식이라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.**
- 람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다.
- 대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로, **컴파일러는 람다의 시그니처도 추론**할 수 있다.
- 상황에 따라 명시적으로 형식을 포함하는 것이 좋을 때도 있고, 형식을 생략하는 것이 가독성을 향상시킬 때도 있다.

```java
//형식 추론 X
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());

//형식 추론
Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());
```

**람다 캡처링**

자유 변수(외부에서 정의된 변수)를 활용하는 행위

**클로저 (closure)**

함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스

### 메서드 참조

- 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.
- 특정 메서드만을 호출하는 람다의 축약형이다. 즉, 메서드를 어떻게 호출해야 하는지 설명을 참조하기보다는 메서드명을 직접 참조한다.
- 명시적으로 메서드를 참조함으로써 가독성을 높인다.
- 메서드명 앞에 구분자(::)를 붙이는 방식으로 메서드 참조를 활용한다.
- 메서드 참조는 콘텍스트의 형식과 일치해야 한다.

**메서드 참조를 만드는 방법**

1. 정적 메서드 참조

    예) Integer의 parseInt를 Integer::parseInt로 표현

2. 다양한 형식의 인스턴스 메서드 참조

    예) String의 length 메서드를 String::length로 표현

    예2) 람다 표현식 (String s) → s.toUpperCase() 를 String::toUpperCase로 표현

3. 기존 객체의 인스턴스 메서드 참조
    - 람다 표현식에서 현존하는 외부 객체 메서드를 호출할 때 사용

    예) () → expensiveTransaction.getValue()라는 람다 표현식을 expensiveTransaction::getValue로 표현

**생성자 참조**

ClassName::new처럼 클래스명과 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다.

+) Color(int, int, int)처럼 인수가 세 개인 생성자의 생성자 참조 사용하는 법

```java
//생성자 참조와 일치하는 시그니처를 갖는 함수형 인터페이스가 필요하나,
//위와 같은 시그니처를 갖는 함수형 인터페이스는 제공되지 않으므로 다음과 같은 함수형 인터페이스를 만든다.
public interface TriFunction<T, U, V, R>{
	R apply(T t, U u, V v);
}

TriFunction<Integer, Integer, Integer, Color> colorFactory = Color::new;
```

### 디폴트 메서드

- 간단한 여러 개의 람다 표현식을 조합해서 복잡한 람다 표현식을 만들 수 있도록 하는 메서드
- 추상 메서드가 아니므로 함수형 인터페이스의 정의를 벗어나지 않는다.

**Comparator의 reverse, thenComparing 메서드**

```java
//reverse: 비교자의 순서를 내림차순으로 정렬
inventory.sort(comparing(Apple::getWeight).reversed());

//thenComparing: 두 번째 비교자를 만듦
inventory.sort(comparing(Apple::getWeight)
				 .reversed()
				 .thenComparing(Apple::getCountry));
```

**Predicate의 negate, and, or 메서드**

```java
//negate: 특정 프레디케이트를 반전
Predicate<Apple> notRedApple = redApple.negate();

//and: 두 프레디케이트를 연결해 람다를 조합 (&&)
Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150);

//or: 또는 (||)
Predicate<Apple> redAndHeavyAppleOrGreen =
		redApple.and(apple -> apple.getWeight() > 150)
						.or(apple -> GREEN.equals(a.getColor()));
```

**Function의 andThen, compose 메서드**

```java
//andThen: 주어진 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달 g(f(x))
//compose: 인수로 주어진 함수를 먼저 실행한 다음, 그 결과를 외부 함수 인수로 제공 f(g(x))
Function<String, String> addHeader = Letter::addHeader;
Function<String, String> transformationPipeline = 
		addHeader.andThen(Letter::checkSpelling)
						 .andThen(Letter::addFooter);
```
