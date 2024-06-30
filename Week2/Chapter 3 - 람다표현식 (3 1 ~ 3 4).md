# Chapter. 3 - 람다표현식 (3.1 ~ 3.4)

*깔끔한 코드로 동작을 구현하고 전달하는 Java 8의 새로운 기능인 람다 표현식*

람다표현식을

- **어떻게 만드는건지**
- **어떻게 사용하는건지**
- **어떻게 코드를 간결하게 만들 수 있는지**

확인한다.

이와 함께

- **인터페이스**
- **형식 추론**
- **메서드 참조**

기능을 확인한다.

## 3.1 람다란?

*람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한것.*

- 이름은 없지만 파리미터 리스트, 바디, 반환 타입, 발생 가능한 예외 목록을 가질 수 있다.

**람다의 특징**

- 익명 - 일반적인 메서드와 다르게 이름이 없다.
- 함수 - 메서드처럼 특정 클래스에 종속되지 않아 함수라고 불린다.
- 전달 - 람다 표현식을 메서드 인수로 전달하거나 변수로 저장이 가능하다.
- 간결성 - 익명 클래스처럼 쓸데없는 코드를 구현할 필요가 없다.

→ 람다가 기존 자바에 없던 기능을 제공하는 것은 아니지만 판에 박힌 코드를 구현할 필요가 없어진다.

**예시**

```kotlin
***before***
val byWeight = object : Comparator<Apple> {
    override fun compare(o1: Apple, o2: Apple): Int {
        return o1.weight - o2.weight
    }
}

***After***
val byWeight = Comparator<Apple> { o1, o2 -> o1.weight - o2.weight }
```

- **compare** 함수명이 없어졌다.

**구성**

람다 표현식은 **람다 파라미터**, **화살표**, **바디** 3 구간으로 이뤄진다.
아래와 같이 다양한 람다식을 작성할 수 있다.

```kotlin
val number = { -> 42 } // { 42 }, 인수가 없다면 화살표도 생략이 가능
number()

val add = { x: Int, y: Int -> x + y }
add(1, 2)

val length = { s: String -> s.length }
length("Hello")

val compareWight = { o1: Apple, o2: Apple -> o1.weight - o2.weight }
compareWight(Apple(RED, 100), Apple(GREEN, 150))

val filterOdd = { n: List<Int> -> n.filter { it % 2 == 1 } }
filterOdd(listOf(5))

val none = { -> {} } // { {} }
none()
```

**사례 유형**

```kotlin
**불리언 표현식**
- val booleanExpression = { list: List<String> -> list.isEmpty() }

**객체 생성**
- val createObject = { weight: Int, color: Color -> Apple(color, weight) }

**객체에서 소비**
- val userObject = { apple: Apple -> println(apple) }

**객체에서 선택/추출**
- val extractInObject = { apple: Apple -> apple.color }

**두 값을 조합**
- val combinationObjects = { a: Int, b: Int -> a * b }

**두 객체 비교**
- val compareObjects = 
		{ a: Apple, b: Apple, c: Apple -> a.color == b.color && b.color == c.color }
```

## 3.2 어디에, 어떻게 람다를 사용할까?

### 3.2.1 함수형 인터페이스가 무엇인가.

2장에서 Predicate<T> 인터페이스로 필터 메서드를 파라미터화 할 수 있었다. 바로 이 Predicate<T>가
함수형 인터페이스이다. Predicate<T>는 ***오직 하나의 추상 메서드만 지정하기 때문***이다.

```kotlin
fun interface Predicate<T> {
    fun filter(t: T): Boolean
}

val filter = Predicate<String> { string -> string == "String" }
```

즉, ***함수형 인터페이스는 정확히 하나의 추상 메서드를 지정하는 인터페이스***이다.

- 람다 표현식으로 추상 메서드 구현을 직접 전달이 가능하므로, 
람다의 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.
- 람다 표현식은 함수형 인터페이스를 구현한 클래스의 인스턴스인 셈이다.

### 3.2.2 함수 디스크립터(function descriptor)

*람다 표현식의 시그니처를 서술하는 메서드*를 함수 디스크립터라고 한다.

```kotlin
{ -> Unit }
- 인수와 반환값이 없는 시그니처

{ apple1: Apple, apple2: Apple -> apple1.weight + apple2.weight }
- Apple 인수 2개를 받아 int를 반환하는 시그니처
```

그런데 Java는 왜 함수형 인터페이스를 인수로 받는 메서드에만 람다 표현식을 사용할 수 있을까?

→ 언어를 더 복잡하게 만들지 않는 현재 방법을 선택함.

반면에, 코틀린은 더 폭넓게 람다 표현식이 사용 가능하다. 아래처럼 함수형 인터페이스가 아님에도 람다 표현식을 인자로 받는것이 가능하다.

```kotlin
fun <T> List<T>.customFilter(***predicate: (T) -> Boolean***): List<Int> {
    return this.customFilter(predicate)
}

fun filteringNumberList() {
    val oddNumberFilter = ***{ number: Int -> number % 2 == 1 }***
    val largeNumberFilter = ***{ number: Int -> number > 10 }***
    
    listOf(1, 2, 3)
        .customFilter { oddNumberFilter(it) }
        .customFilter { largeNumberFilter(it) }
}
```

## 3.3 람다 활용 : 실행 어라운드 패턴

*람다와 동작 파라미터화로 - 유연하고 간결한 코드를 구현하는데 도움을 주는 실용적인 예제를 확인한다.*

자원처리에 사용되는 순환 패턴(recurrent pattern)은 자원을 열고, 처리하고, 자원을 닫는 순서로 이뤄진다. 
setUp과 clearnUp 과정은 대부분 비슷하다는 것이다. 이러한 형식의 코드를 실행 어라운트 패턴이라고 부른다. 
(execute around pattern)

### 3.3.1 1단계, 동작 파라미터화를 기억하라!

```kotlin
fun processFile(): String {
    try {
        val br = BufferedReader(FileReader("data.txt"))
        return br.readLine()
    } catch (e: Exception) {
        throw RuntimeException(e)
    }
}
```

현재 위 코드는 한 번에 한 줄만 읽을 수 있다. 한번에 두 줄을 읽거나 가장 자주 나온 단어만 찾는다는 등 **동작**이 바뀌면 어떻게 해아할까?

→ 그렇다. 동작 파라미터화로 넘겨주면 된다.

### 3.3.2 2단계, 함수형 인터페이스를 이용해서 동작 전달

시그니처를 생각해보자. 
BufferedReader를 받아서 String을 반환하는 시그니처와 일치하는 함수형 인터페이스를 만들어야 한다.

```kotlin
fun interface BufferedReaderProcessor {
    fun process(bufferedReader: BufferedReader): String
}
```

### 3.3.3 3단계, 동작 실행

이제 BufferedReaderProcessor에 정의된 **process 메서드의 시그니처 (BufferedReader → String)**와 
일치하는 람다를 전달할 수 있다.

```kotlin
fun processFile(***p: BufferedReaderProcessor***): String {
    try {
        val br = BufferedReader(FileReader("data.txt"))
        return ***p.process(br)***
    } catch (e: Exception) {
        throw RuntimeException(e)
    }
}
```

### 3.3.4 4단계, 람다 전달

이제 람다를 이용해서 다양한 동작을 processFiler 메서드로 전달이 가능하다. 동작 파라미터화가 가능해진 것이다.

```kotlin
// 한줄 읽기
val readOneLine = { br: BufferedReader -> br.readLine() }

// 두줄 읽기
val readTwoLine = { br: BufferedReader -> br.readLine() + br.readLine() }

// 가장 많이 등장한 단어 찾기
val findWordMostUsed =
    { br: BufferedReader ->
        val map = mutableMapOf<String, Int>()
        br.readLines()
            .map { line ->
                line.split(" ").forEach { word ->
                    map[word] = map.getOrDefault(word, 0) + 1
                }
            }

        map.maxByOrNull { it.value }?.key ?: ""
    }

fun useProcessFileLambda() {
    processFile(readOneLine)
    processFile(readTwoLine)
    processFile(findWordMostUsed)
}
```

## 3.4 함수형 인터페이스 사용

자바 8 라이브러리 설계자들은 java.util.function 패키지로 여러가지 함수형 인터페이스를 제공한다. 
여기서 Predicate, Consumer, Function 인퍼페이스를 확인한다.

### 3.4.1 Predicate

앞에서 Predicate<T> 인터페이스는 test 추상 메서드를 정의하며 제네릭 타입 T를 받아 불리언을 반환했다. 
즉, T형식의 객체를 사용하는 불리언 표현식이 필요한 경우 **Predicate 인터페이스를 사용**하면 된다.

> java.util.function.Predicate 인터페이스
> 

```kotlin
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
}

// kotlin으로 작성하는 경우
fun interface MyPredicate<T> {
    fun test(t: T): Boolean
}
```

- default 메소드가 존재한다. 추상메소드는 test 단 하나만 존재.

```kotlin
val s = Predicate<String> { it == "java" }
        .or { it == "kotlin" }
        .and { it == "hello" }
        .and { it == "world" }
```

- and, or를 사용하여 위와같이 작성할 수 있다.

**사용 예시**

```kotlin
// Predicate 인터페이스를 사용한 경우
fun <T> filter1(list: List<T>, ***predicate: Predicate<T>***): List<T> {
    return list.filter { predicate.test(it) }
}

// 동일한 람다 표현식 시그니처를 함수 디스크립터로 작성한 경우
fun <T> filter2(list: List<T>, ***test: (T) -> Boolean***): List<T> {
    return list.filter { test(it) }
}

val onlyNoneEmptyString = Predicate<String> { it.isNotEmpty() }

val result1 = filter1(listOf("java", "kotlin", "hello", "world"), onlyNoneEmptyString)
val result2 = filter2(listOf("java", "kotlin", "hello", "world")) { it.length >= 2 }
```

- Predicate를 사용해도 되고 직접 람다 표현식의 시그니처를 작성해도 된다.
- Predicate는 이미 주어진 형식을 사용할 수 있도록 제공하는 것과 디폴트 메소드를 제공해준다는 점이 있는 것 뿐이다.

### 3.4.2 Consumer

> java.util.function.Consumer 인터페이스
> 

```kotlin
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}

// kotlin으로 작성하는 경우
fun interface MyConsumer<T> {
    fun accept(t: T)
}
```

- Consumer는 T를 받아서 void를 반환하는 함수형 인터페이스이다.

**사용예시**

```kotlin
val consumer1 = Consumer<String> { println(it) }
val consumer2 = object : Consumer<String> {
    override fun accept(t: String) {
        println(t)
    }
}
val consumer3 = { s: String -> println(s) }

// Consumer 함수형 인터페이스를 사용한 경우
fun <T> filter1(list: List<T>, ***consumer: Consumer<T>***) {
    return list.forEach { consumer.accept(it) }
}

// 동일한 람다 표현식 시그니처를 함수 디스크립터로 작성한 경우
fun <T> filter2(list: List<T>, ***consumer: (T) -> Unit***) {
return list.forEach { consumer(it) }
}

val result1 = filter1(listOf("hello", "world"), consumer1)
val result2 = filter1(listOf("hello", "world"), consumer2)
val result3 = filter2(listOf("hello", "world"), consumer3)
val result4 = filter2(listOf("hello", "world")) { println(it) }
```

### 3.4.3 Function

> java.util.function.Function 인터페이스
> 

```kotlin
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}

// kotlin으로 작성하는 경우
fun interface MyFunction<T, R> {
    fun apply(t: T): R
}
```

- T 타입을 인수로 받아서 R 타입으로 반환하는 Function 함수형 인터페이스이다.

**사용예시**

```kotlin
val function1 = Function<List<String>, Int> { it.sumOf { it.length } }
val function2 = object : Function<List<String>, Int> {
    override fun apply(t: List<String>): Int {
        return t.sumOf { it.length }
    }
}
val function3 = { list: List<String> -> list.sumOf { it.length } }

// Function 함수형 인터페이스를 사용한 경우
fun <T> filter1(list: List<T>, ***function: Function<List<T>, Int>***): Int {
    return function.apply(list)
}

// 동일한 람다 표현식 시그니처를 함수 디스크립터로 작성한 경우
fun <T> filter2(list: List<T>, ***function: (List<T>) -> Int***): Int {
    return function(list)
}

val result1 = filter1(listOf("hello", "world"), function1)
val result2 = filter1(listOf("hello", "world"), function2)
val result3 = filter2(listOf("hello", "world"), function3)
val result4 = filter2(listOf("hello", "world")) { it.sumOf { it.length } }
```

### 3.4.4 원사타입 기본형 특화

자바의 모든 형식은 참조형(reference type) Object, Integer, List 등 아니면 원시형(primitive type) int, double, byte, long 에 해당하게 된다.
하지만 제네릭 파라미터는 (Consumer<T>의 T) 참조형만 사용이 가능하다. 따라서 자바는 원시형을 참조형으로 변환하는 기능을 제공하는데 이러한 기능을 **박싱**, **언박싱**이라고 한다.
그리고 이를 프로그래머가 편리하게 코드를 작성할 수 있도록 **오토박싱** 기능을 제공한다.

하지만 이러한 박싱은 기본형을 감싼 래퍼 객체이며 힙 메모리에 저장된다. 또한 기본형을 가져올 때도 메모리를 탐색하는 과정이 필요하다. 
이러한 박싱 절차 없이 원시타입을 바로 함수형 인터페이스의 인수로 받을 수 있게 하는 특화 함수형 인터페이스가 존재한다.

일반적으로 특정 형식을 입력으로 받는 경우 IntPredicate, IntToDoubleFunction 등 형식명이 prefix로 붙게 된다.

> java.util.function.IntPredicate 인터페이스
> 

```kotlin
@FunctionalInterface
public interface IntPredicate {
    boolean test(int value);
}
```

**사용예시**

```kotlin
// Boxing이 발생하는 함수형 인터페이스
fun <T> filter1(list: List<T>, ***predicate: Predicate<T>***): List<T> {
    return list.filter { predicate.test(it) }
}

// Boxing이 발생하지 않는 함수형 인터페이스
fun filter2(list: List<Int>, ***intPredicate: IntPredicate***): List<Int> {
    return list.filter { intPredicate.test(it) }
}

val result1 = filter1(listOf(1, 2, 3), Predicate { it % 2 == 1 })
val result2 = filter2(listOf(1, 2, 3), IntPredicate { it % 2 == 1 })
val result3 = filter1(listOf(1, 2, 3)) { it % 2 == 1 }
val result4 = filter2(listOf(1, 2, 3)) { it % 2 == 1 }
```