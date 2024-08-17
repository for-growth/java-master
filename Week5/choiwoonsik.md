# Chapter. 5 - 스트림 활용 (5.5 ~ 5.9)

## 5.5 리듀스

“메뉴의 모든 칼로리의 합계를 구하시오", “메뉴에서 칼로리가 가장 큰 요리는?” 과 같이 스트림 요소를 조합해서 더 복잡한 질의를 표현하기 위한 메서드이다.

즉, 리듀스는 스트림의 모든 요소를 반복 처리해서 값으로 결과를 도출해낼 때 사용한다.

### 5.5.1 요소의 합

for-each 반복문을 통한 리스트 내 숫자 요소의 총 합을 구하는 코드를 보자.

```kotlin
val numbers = listOf(1, 2, 3, 4, 5)
var sum = 0
for (x: Int in numbers) {
    sum += x
}
```

위 코드를 보면 

- sum 변수 초기값 0
- 리스트의 모든 요소를 조합하는 연산 “+”

이렇게 2개의 파라미터를 사용했다. 이러한 리스트 연산의 총 합을 구하기 위해 반복적으로 해당 코드를 구현하지 않을 수 있다면 효율적일 것이다.

이를 reduce를 사용함으로서 반복적인 패턴을 추상화할 수 있다.

```kotlin
val sum2 = numbers.stream().reduce(0) { a, b -> a + b }
```

위 코드에서는

- 초기값 0
- 두개의 요소를 조합해서 새로운 값을 만드는 BinaryOperator<T>
    
    → 함수형 인터페이스로 T, T 를 받아서 T를 반환하는 람다식을 받을 수 있다.
    
    → 따라서, { a, b → a * b } 등 활용이 가능하다.
    

를 인수로 넘겨주었다.

위 reduce에서 처리되는 람다식을 형상화하면 아래와 같다.

```kotlin
초기값: 0, 스트림: { 1, 2, 3, 4, 5 }

0 -> **0 + 1**
			 1 -> **1 + 2**
							3 -> **3 + 4**
											7 -> **7 + 5**
															12.
```

reduce 입장에서 보면 stream의 요소를 하나씩 소비하면서 람다의 기능을 수행한다고 이해하면 된다.
즉, 초기값 0을 갖고 스트림의 첫번째 요소 1을 소비해서 + 연산을 수행하고 새로운 결과 1을 만든다. 
새로운 결과 1을 갖고 스트림의 두번째 요소 2를 소비해서 + 연산을 수행하고 새로운 결과 3을 만든다.
…
이렇게 reduce를 통해 스트림의 요소를 소비하게 되는 것이다.

초기값을 넘겨주지 않고 reduce를 수행할 수도 있는데, 이렇게 되면 stream의 첫번째 요소를 기반으로 소비가 처리된다. 다만 이 경우 stream이 비어있다면 초기값이 없어서 반환값이 null이 될 수 있다. 따라서 반환 타입이 `Optional<Int>`임을 인지하자.

### 5.5.2 최댓값과 최솟값

reduce는 두 인수를 받는다고 정리할 수 있다.

- 초기값 (없으면 스트림의 첫번째 값)
- 스트림의 두 요소를 합쳐서 하나의 값으로 만드는데 사용할 람다

따라서 요소를 차근차근 더하는것 뿐만 아니라 최소값, 최대값을 갱신하면서 구하는 것 또한 가능하다.

```kotlin
numbers.stream().reduce(0L) { a, b -> max(a, b) }
numbers.stream().reduce { a, b -> min(a, b) }
```

```kotlin
초기값: 없음, 스트림: { 4, 9, 1, 10, 3 }

4 -> max(4, 9)
					9 -> max(9, 1)
										9 -> max(9, 10)
															 10 -> max(10, 3)
																					 10 <-- 최종 반환 값.
```

이를 활용하면 map + reduce를 조합하여 사용이 가능한데 이를 맵 리듀스 (map reduce) 패턴이라고 한다. 예를들어 map과 reduce를 사용해서 전체 요소의 개수를 구할 수 있다.

```kotlin
numbers.stream().map { 1 }.reduce { a, b -> a + b }
```

이러한 방식은 쉽게 병렬화가 가능하다는 특징이 있고 구글에서 사용하며 유명해졌다.

```kotlin
numbers.parallelStream().map { 1 }.reduce { a, b -> a + b }
```

이러한 병렬처리는 7장에서 fork/join framework를 이용하는 방법을 정리한다. 또한 6장에서는 collect 메서드를 이용해서 더 복잡한 형태의 reduce를 소개한다. 이를 통해 요리를 종류별로 그룹핑할 때 Map으로 리듀스가 가능하다.

## 5.6 실전연습

**데이터셋**

```kotlin
class Trader(
    val name: String,
    val city: String
)

class Transaction(
    val trader: Trader,
    val year: Int,
    val value: Int
)

val minsoo = Trader("Minsoo", "Pyeongchon")
val jinwoo = Trader("Jinwoo", "Gawchen")
val gildong = Trader("Gildong", "Pyeongchon")
val younghee = Trader("Younghee", "Pangyo")
val woonsik = Trader("Woonsik", "Gawchen")
val donghyun = Trader("Donghyun", "Pyeongchon")
val jongjin = Trader("Jongjin", "Indeogwon")

val transactions = listOf(
    Transaction(minsoo, 2011, 300),
    Transaction(jinwoo, 2012, 1000),
    Transaction(gildong, 2011, 400),
    Transaction(younghee, 2012, 710),
    Transaction(woonsik, 2012, 700),
    Transaction(donghyun, 2012, 950),
    Transaction(jongjin, 2011, 950)
)
```

**예제**

1. 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하시오
    
    ```kotlin
    transactions.stream()
    		.filter { it.year == 2011 }
        .sorted(Comparator.comparingInt(Transaction::value))
        .collect(toList())
    ```
    
2. 거래자가 근무하는 모든 도시를 중복 없이 나열하시오
    
    ```kotlin
    transactions.stream()
        .map { it.trader.city }
        .distinct()
        .collect(toList())
    ```
    
3. 판교에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오
    
    ```kotlin
    transactions.stream()
        .map { it.trader }
    		.filter {it.city == "Pangyo"}
    		.sorted(Comparator.comparing(Trader::name))
    		.collect(toList())
    ```
    
4. 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오
    
    ```kotlin
    transactions.stream()
        .map { it.trader.name }
        .distinct()
        .sorted()
        .collect(toList())
    ```
    
5. 평촌에 거래자가 있는가
    
    ```kotlin
    transactions.stream()
        .anyMatch { it.trader.city == "Pyeongchon" }
    ```
    
6. 평촌에 거주하는 거래자의 모든 트랜잭션값을 출력하시오
    
    ```kotlin
    transactions.stream()
    		.filter { it.trader.city == "Pyeongchon" }
    		.map { it.trader }
        .forEach { println(it) }
    ```
    
7. 전체 트랜잭션 중 최대값은 얼마인가
    
    ```kotlin
    val reduce: Int? = transactions.stream()
        .map { it.value }
        .reduce { t1, t2 -> max(t1, t2) }
        .orElse(null)
    ----
    transactions.stream()
        .max(compareBy(Transaction::value))
    ```
    
8. 전체 트랜잭션 중 최소값은 얼마인가
    
    ```kotlin
    val reduce: Int? = transactions.stream()
        .map { it.value }
        .reduce { t1, t2 -> min(t1, t2) }
        .orElse(null)
    ----
    transactions.stream()
        .min(compareBy(Transaction::value))
    ```
    

## 5.7 숫자형 스트림

```kotlin
menu
    .stream()
    .map { it.calories }
    .reduce(0) { a, b -> a + b }
```

우리는 위처럼 reduce를 사용해서 스트림 모든 요소의 합을 구할 수 있었다. 하지만 여기에는 Integer를 기본형으로 unboxing하는 비용이 소모된다.
단순히 아래처럼 직접 sum() 메서드를 호출할 수는 없을까?

```kotlin
menu
    .stream()
    .map { it.calories }
    ~~.sum()~~ <---- 불가능.
```

이러한 코드가 되면 편하겠지만 불가능하다. 이유는 map 메서드가 Stream<T>을 생성하기 때문이다. 
즉, Stream<Dish>와 같은 요소만 있다면 sum이라는 연산이 불가능하기 때문이다. 따라서 이를 위해서는 
**숫자형 스트림**을 만들어줘야하는데 이를 위해 **기본형 특화 스트림(primitive stream specialization)**을 제공한다.

### 5.7.1 기본형 특화 스트림

자바8에서는 세 가지 기본형 특화 스트림을 제공한다. boxing 비용을 피할 수 있도록 int, double, long 요소에 특화된 스트림을 제공한다.

### 숫자 스트림으로 매핑

mapToInt, mapToDouble, mapToLong 세 가지 메서드를 이용해서 숫자 특화 스트림으로 변환할 수 있다. 
이를 통해 map과 완전히 동일한 기능을 제공하면서도 ***Stream<T>가 아닌 특화 타입의 스트림 (IntStream)를 반환***한다. (≠ Stream<Integer>)

이렇게 반환된 IntStream 등을 통해서 제공하는 max, min, average 등 다양한 유틸리티 메서드를 사용할 수 있다.

### 객체 스트림 복원하기

숫자 스트림을 만든 후에 다시 원상태의 특화되지 않은 스트림으로 복원이 가능할까? 
IntStream의 map 연산은 “int를 인수로 받아서 int로 반환”하는 람다를 인수로 받는다. 이때 boxed()를 통해 다시 Stream<Integer>로 변경이 가능하다.

```kotlin
val intStream: IntStream = menu
    .stream()
    .mapToInt { it.calories }

val streamInteger: Stream<Int> = intStream.boxed()
```

### 기본값: OptionalInt

최대값을 구하는 스트림에서 0이라는 기본값을 주는 경우를 생각해보자.

- 스트림이 없어서 0인 것 vs. 최대값이 0인 것

이 둘을 어떻게 구별할 수 있을까?

이러한 값의 존재 여부를 포함하는 것을 위해 **컨테이너 클래스 Optional**을 소개했다. 이 또한 특화 타입의 Optional을 제공한다.

- OptionalInt, OptionalDouble, OptionalLong

이를 통해 아래와 같은 IntStream 요소 내 최대값을 구할 수 있다. 이때 값이 없다면 null 반환이 가능.

```kotlin
val optionalMax: OptionalInt = menu
    .stream()
    .mapToInt { it.calories }
    .max()
val max: Int? = optionalMax.orElseGet(null)
```

### 5.7.2 숫자 범위

말그대로 특정 범위의 숫자를 이용할 때 사용하는 메서드이다. 프로그래밍을 하면 for 문 등을 통해 범위를 지정하는 경우가 잦을 것이다.
예를들어 1 ~ 100 사이의 숫자를 생성한다고 가정하자. 자바 8의 IntStream, LongStream에서는 range(), rangeClosed라는 2가지 정적 메서드를 제공한다.
이 둘다 시작값, 종료값 인수를 받는다. 참고로 range는 시작, 끝 요소 둘다 exclusive, rangeClosed는 둘다 include라는 특징이 있다.

```kotlin
val eventNumbers = IntStream.rangeClosed(1, 100)
    .filter { n -> n % 2 == 0 }
eventNumbers.count()
```

### 5.7.3 숫자 스트림 활용 : 피타고라스 수

앞에서 소개한 숫자 스트림과 스트림 연산을 통해서 활용을 해본다. 피타고라스 수 스트림을 만들어본다.

### 피타고라스 수

다들 아는 그 공식

- a^2 + b^2 = c^2

이 공식에 부합하는 (a, b, c) 정수가 피타고라스 수이다.

### 세 수 표현하기

일단, c를 구하려면 아래 수식이 충족되어야 한다.

$$
sqrt(a^2 + b^2) = c(정수)
$$

따라서 sqrt(a^2 + b^2) % 1 == 0 이 되면 c에 값이 올 수 있다는 것이다.

이를 코드로 변환하면 아래처럼 표현할 수 있다.

```kotlin
val a: Double = 1.0
IntStream.range(1, 100)
    .filter { b -> sqrt(a * a + b * b) % 1.0 == 0.0 }
```

이제 a, b, c를 모아보자.

```kotlin
val a = 1.0
IntStream.range(1, 100)
    .filter { b -> sqrt(a * a + b * b) % 1.0 == 0.0 }
    .map { b: Int -> arrayOf(a.toInt(), b, sqrt(a * a + b * b).toInt()) }
```

이렇게 하려고 하면 map에서 오류가 발생하는데 그 이유를 봐보자.

수의 범위를 IntStream으로 생성해서 filter로 조건이 부함하는 IntStream만 넘겼다.
필터링 된 IntStream에 대해 map을 처리하게 되는데 이 경우 람다의 반환 타입이 Int가 되어야 IntStream으로 반환이 가능하다.
하지만 현재 위 map 내 람다에서는 Int를 받아서 Array<Int>를 반환하므로 오류가 표시되는 것이다.

따라서 ***Stream<T> 처리가 가능하도록 boxed()처리***를 해야한다.

```kotlin
val a = 1.0
IntStream.range(1, 100)
    .filter { b -> sqrt(a * a + b * b) % 1.0 == 0.0 }
    .boxed()
    .map { b: Int -> arrayOf(a.toInt(), b, sqrt(a * a + b * b).toInt()) }
```

마지막으로 a를 넘겨주도록 수정하고, a 를 넘길 때 마다 생성되는 배열 목록을 flatten 하여 모든 요소를 하나의 Stream으로 합치고 collect를 통해 하나의 리스스트로 변환해주면 된다.

```kotlin
val pythagoreanTripleList: MutableList<Array<Int>> = 
IntStream
		.rangeClosed(1, 100)
		.boxed()
		.flatMap { a ->
		    IntStream.range(a, 100)
		        .filter { b -> sqrt(a.toDouble() * a + b * b) % 1.0 == 0.0 }
		        .boxed()
		        .map { b: Int -> 
				        arrayOf(a.toInt(), b, sqrt(a.toDouble() * a + b * b).toInt()) 
		        }
		}
		.collect(toList())
```

아래 코드를 통해 출력해볼 수 있다.

```kotlin
pythagoreanTripleList
    .stream()
    .limit(10)
    .forEach { println("${it[0]}, ${it[1]}, ${it[2]}") }
```

## 5.8 스트림 만들기

### 5.8.1 값으로 스트림 만들기

Stream.of() 메서드를 통해 스트림을 만들 수 있다.

```kotlin
Stream.of("hello", "world", "java", "kotlin")
```

Stream.empty() 메서드를 통해 비어있는 스트림을 선언할 수 있다.

```kotlin
val emptyStream: Stream<String> = Stream.empty()
```

### 5.8.2 null이 될 수 있는 객체로 스트림 만들기

요소가 nullable할 때 스트림으로 만드는 메서드도 별도로 존재한다.

```kotlin
val notNullStream: Stream<String> = 
		Stream.of("hello", "world", "java", "kotlin")
val nullableStream: Stream<String?> = 
		Stream.of("hello", "world", "java", "kotlin", null)
    .flatMap { Stream.ofNullable(it) }
```

### 5.8.3 배열로 스트림 만들기

배열을 인수로 받은 경우에는 Arrays.stream() 메서드를 통해 스트림으로 만들 수 있다.

```kotlin
val numberArray = arrayOf("hello", "world", "java", "kotlin")
val sum = Arrays.stream(numberArray)
    .mapToInt() { it.length }
    .sum()
```

### 5.8.4 파일로 스트림 만들기

### 5.8.5 함수로 무한 스트림 만들기

Stream 의 정적 메서드를 통해서 무한 스트림을 만들 수 있다.

- `Stream.interate`
- `Stream.generate`

무한 스트림은 고정된 컬렉션이 아닌 크기가 고정되지 않은 스트림을 만들 수 있다. interate, generate는 요청할 때마다 주어진 함수를 통해서 값을 무한정 만들게 된다.
따라서 reduce, filter, sort 와 같은 메서드를 활용할 수 없고 takeWhile, limit 등을 활용해서 종료 조건을 충족시켜줘야한다.

### iterate 메서드

```kotlin
Stream.iterate(0) { n -> n + 2 }
    .limit(10)
    .forEach { println(it) }
```

- interate 첫번째 인수인 0은 seed 이다.
- 두번째 인수는 람다식으로 n을 받아서 n + 2를 반환한다. 따라서 0을 받으면 2, 2를 반으면 4를 반환한다.
- limit 10개까지 스트림 요소를 받고 각 요소를 forEach 구문에서 출력하게 된다.

위 스트림을 실행하면 0, 2, 4, 6 .. 18 로 총 10개의 짝수가 출력된다. 이렇듯 iterate는 기본적으로 기존 결과에 의존해서 순차적으로 연산을 수행한다. 이전 수행 결과를 베이스로 수행이 이뤄지게 된다. 이러한 스트림을 ***unbound stream*** 이라고 표현한다.

자바 9의 iterate 메소드는 Predicate를 지원한다. 이를 이용하여 0에서 시작해서 100보다 크면 숫자 생성을 중단하는 iterate를 구현할 수 있다.

```kotlin
IntStream.iterate(0, { n -> n < 100 }, { n -> n + 4 })
        .forEach { println(it) }
```

```kotlin
public static IntStream iterate(int seed, IntPredicate hasNext, IntUnaryOperator next)
```

- iterate의 인수를 보면 2번째 인수로 IntPredicate가 추가된걸 볼 수 있다. 이 Predicate 통해 언제까지 작업을 수행할 것인지 기준을 삼게 된다.

**이때, 위와 같은 Predicate 대신 filter를 사용해서도 같은 효과를 얻을 수 있을까?**

```kotlin
IntStream.iterate(0) { n -> n + 4 }
      .filter { n -> n < 100 } // <<----- filter로 스트림 생성 중단이 가능할까?
      .forEach { println(it) }
```

→ 불가능하다. 왜냐하면 filter 메서드는 해당 스트림의 작업을 언제 중단해야할지 알 수가 없기 때문이다. 
계속 생성되는 스트림에서 100 이하의 요소만 받고 그 뒤에 생성되는 요소를 무시할 뿐 중단하지는 않게 된다.

따라서 filter 대신 ***takeWhile*** 을 이용하는 것이 해법이다.

### generate 메서드

generate는 interate와 다르게 **생산된 각 값을 연속적으로 계산하지 않는다**. 
generate는 **Supplier<T>를 인수로 받아서 새로운 값을 생산**한다.

```kotlin
Stream.generate(void)
      .limit(10)
      .forEach { println(it) }
<출력>
kotlin.Unit
kotlin.Unit
...

---

Stream.generate(Math::random)
      .limit(10)
      .forEach { println(it) }
<출력>
0.6890988885892316
0.197260048329719
...
```

**Supplier**

```kotlin
public static<T> Stream<T> generate(Supplier<? extends T> s) { ... }

---

@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

위처럼 Supplier라는 함수형 인터페이스를 인수로 받는 것이므로 아래와 같이 오버라이딩할 수 있다.
람다로 넘겨주고자 하면 변수를 밖으로 빼서 처리할 수 있다.

```kotlin
Stream.generate(
    object : Supplier<Int> {
        var i = 0
        override fun get(): Int {
            return i++
        }
    }
)
    .limit(10)
    .forEach { println(it) }
<출력>
0
1
2
3
4
5
6
7
8
9

---

var i = 0
Stream.generate { i++ }
    .limit(10)
    .forEach { println(it) }
<출력>
위와 동일.
```