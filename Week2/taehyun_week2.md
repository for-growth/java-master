# 3장. 람다 표현식

람다 표현식을 사용하여 더 깔끔하게 동작을 파라미터화할 수 있음

<br>

## 3.1 람다란 무엇인가?

- 메서드로 전달할 수 있는 익명 함수를 단순화 한것
- 이름 ❌, 파라미터 리스트 ⭕️ , 바디 ⭕️, 반환형식 ⭕️, 발생할 수 있는 예외 리스트 ⭕️
- 특징
    - **익명** : 보통 메서드와 달리 이름이 없음
    - **함수** : 특정 클래스에 종속되지 않으므로 함수하고 부름. 하지만 파라미터, 바디, 반환형식 예외 리스트를 포함
    - **전달** : 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있음
    - **간결성** : 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요없음
- 크게 3부분 으로 구성

    ```java
      (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
    //      람다 파라미터    화살표             람다 바디
    ```

- 헷갈릴 수 있는 람다 예시

    ```java
    () -> {}                          //⭕️ void 를 리턴하는 람다 표현식
    () -> "Raoul"                     //⭕️ 파라미터가 없으며 문자열을 반환하는 표현식
    () -> {return "Mario";}           //⭕️ 파라미터가 없으며 명시적으로 문자열을 반환하는 표현식
    (Integer i) -> return "Alan" + i; //❌ 중괄호가 빠짐
    (String i) -> {"Iron Man"}        //❌ 표현식이 아닌 구문이 와야함
    ```

<br>


## 3.2 어디에, 어떻게 람다를 사용할까?

- 함수형 인터페이스에서 람다를 사용할 수 있음

<br>

### 3.2.1 함수형 인터페이스

> 하나의 추상 메서드를 지정하는 인터페이스
>
- 디폴트 메서드를 가지고 있어도 추상메서드가 오직 하나면 함수형 인터페이스가 될 수 있음
- 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있음
- 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있음
- 덜 깔끔하지만 익명 내부 클래스로도 구현 가능
- `@FuntionalInterface` 로 표시

```java
    public static void main(String[] args) {

        Runnable runnable1 = () -> System.out.println("Hello World1");
        runnable1.run();

        Runnable runnable2 = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello World2");
            }
        };
        runnable2.run();

        process(() -> System.out.println("Hello World3"));
    }

    private static void process(Runnable runnable) {
        runnable.run();
    }
```

<br>

### 3.2.2 함수 디스크립터

- 함수형 인터페이스의 추상 메서드 시그니쳐는 람다 표현식의 시그니쳐를 가리킴
- 람다 표현식의 시그니쳐를 서술하는 메서드를 함수 디스크립터 라고 부름

  ex) `Runnable` 의 `run()`

- 람다 표현식은 변수에 할당하거나 함수형 인터페이스를 인수로 받는 메서드로 전달할 수 있으며, 함수형 인터페이스의 추상 메서드와 같은 시그니쳐를 가짐

<br>

## 3.3 람다 활용 : 실행 어라운드 패턴

- 실행 어라운드 패턴 : 설정 과정과 정리 과정을 갖는 형태

<br>

### 3.3.1 1단계 : 동작 파라미터를 기억하라

- 반복되는 동작은 파라미터와 하자

<br>

### 3.3.2 2단계 : 함수형 파라미터를 이용해서 동작 전달

- 함수형 인터페이스의 자리에 람다를 사용

```java
@FunctionalInterface
public interface BufferedReaderProcessor{
		String process(BufferedReader b) throws IOException;
}

//...

public String processFile(BufferedReaderProcessor p) throws IOException {
}
```

<br>

### 3.3.3 3단계 : 동작 실행

- 이제 위의 `BufferedReaderProcessor` 에 `process()` 메서드의 시그니쳐와 일치하는 람다를 전달할 수 있음
- `processFile()` 내부로 전달된 람다는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
	try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
		return p.process(br);
	}
}
```

<br>

### 3.3.4 4단계 : 람다 전달

- 이제 람다를 이용해서 다양한 동작을 `processFile()`  메서드로 전달할 수 있음

```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
String twoLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

<br>

## 3.4 함수형 인터페이스 사용

- 함수형 인터페이스의 추상 메서드 시그니쳐를 함수 디스크립터 라고 부름
- 다양한 람다 표현식을 사용하려면 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요

<br>

### 3.4.1 Predicate

- `test` 라는 추상 메서드를 정의하며 `test` 는 제네릭 형식 `T` 의 객체를 인수로 받아 `boolean` 을 리턴

```java
private static  <T> List<T> filter(List<T> list, Predicate<T> predicate) {
        List<T> result = new ArrayList<>();
        for (T t : list) {
            if (predicate.test(t)) {
                result.add(t);
            }
        }
        return result;
}

//...

List<Apple> result = filter(inventory, apple -> apple.getColor().equals("red"));
```

### 3.4.2 Consumer

- 제네릭 형식 `T` 의 객체를 받아 `void` 를 리턴하는 `accept` 추상 메서드를 정의
- `T` 형식의 객체를 받아서 어떤 동작을 수행하고 싶을때 사용

```java
private static <T> void print(List<T> list, Consumer<T> consumer) {
		list.forEach(consumer);
}

//...

inventory.forEach(System.out::println);
```

<br>

### 3.4.3 Function

- 제네릭 형식 `T` 의 객체를 받아 제네릭 형식 `R` 객체를 반환하는 추상 메서드 `apply` 를 정의
- 입력을 출력으로 매핑하는 람다를 정의할때 사용

```java
private static <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> result = new ArrayList<>();
    for (T t : list) {
        result.add(f.apply(t));
    }
    return result;
}

//...
List<ApplePie> applePieList = map(inventory, (apple -> {
        return new ApplePie(apple);
}));

```

<br>

### 기본형 특화

- 기본형을 입출력으로 사용하는 상황에서 오토 방식 동작을 피할 수 있는 함수형 인터페이스도 있음
- 보통 `참조형 + xxx` 형식의 이름을 가짐. `IntPredicate`, `LongBinaryOperator` …
- 람다와 함수형 인터페이스 예제
    - `boolean` 표현 : `Predicate<T>`
    - 객체 생성 : `Supplier<T>`
    - 객체에서 소비 : `Consumer<T>`
    - 객체에서 선택 / 추출 : `Function<T, R>`, `ToIntFunction<T>`
    - 두 값 조합 : IntBinaryOperator
    - 두 객체 비교 : `Comparator<T>`, `BiFunction<T, T>`, `ToIntBiFunction<T, T>`

<br>

### 사용 예시

```java
    public List<LocalDate> weekdayDate() {
        return decemberDate.stream()
                .filter(isWeekDay())
                .toList();
    }

    private Predicate<LocalDate> isWeekDay() {
        return date -> date.getMonthValue() == MONTH &&
                date.getDayOfWeek() != DayOfWeek.FRIDAY &&
                date.getDayOfWeek() != DayOfWeek.SATURDAY;
    }

    public List<LocalDate> weekendDate() {
        return decemberDate.stream()
                .filter(isWeekendDay())
                .toList();
    }

    private Predicate<LocalDate> isWeekendDay() {
        return date -> date.getMonthValue() == MONTH &&
                date.getDayOfWeek() == DayOfWeek.FRIDAY ||
                date.getDayOfWeek() == DayOfWeek.SATURDAY;
    }

    public List<LocalDate> specialDate() {
        return decemberDate.stream()
                .filter(isSpecialDay())
                .toList();
    }

    private Predicate<LocalDate> isSpecialDay() {
        return date -> date.getMonthValue() == MONTH &&
                date.getDayOfWeek() == DayOfWeek.SUNDAY ||
                date.getDayOfMonth() == CHRISTMAS.getDayOfMonth();
    }
```

<br>
<br>

