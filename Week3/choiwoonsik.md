# Chapter. 4 - 스트림 소개

컬렉션은 거의 모든 자바 애플리케이션에서 사용하는 기능으로 데이터를 그룹화하고 처리하는 자료구조이다.
하지만 컬렉션은 아직 완벽한 모든 기능을 제공하지 못하는데 이를 보완하기 위해 나온것이 스트림이다.

## 4.1 스트림이란?

스트림은 Java 8 API에 새로 추가된 기능이다. 스트림을 이용하면 선언형으로 데이터를 처리하는 임시 구현 코드 없이 질의를 통해 컬렉션 데이터를 처리할 수 있다.

**예제 데이터**

```kotlin
class Dish(
    val name: String = "",
    val calories: Int = 0
)

val menu = listOf(
    Dish("Java", 100),
    Dish("Kotlin", 200),
    Dish("C++", 300),
    Dish("C#", 400),
    Dish("Python", 500),
    Dish("Go", 600),
    Dish("JavaScript", 700),
    Dish("TypeScript", 800),
    Dish("Ruby", 900),
    Dish("Rust", 1000),
)
```

- 위 데이터들이 존재할 때 400칼로리 이하의 메뉴 이름을 가져오는 코드를 작성할 때 Stream이 없을 때 구현코드와 이후 코드를 비교해보자.

**AS-IS**

```kotlin
// stream이 없던 과거 시절
fun asIs() {
    val lowCaloricDishes = mutableListOf<Dish>()

    for (dish in menu) {
        if (dish.calories < 400) {
            lowCaloricDishes.add(dish)
        }
    }
    Collections.sort(lowCaloricDishes, object : Comparator<Dish> {
        override fun compare(o1: Dish, o2: Dish): Int {
            return o1.calories - o2.calories
        }
    })
    val lowCaloricDishesName = mutableListOf<String>()
    for (dish in lowCaloricDishes) {
        lowCaloricDishesName.add(dish.name)
    }
}
```

**TO-BE**

```kotlin
// stream 사용
fun toBe() {
    val lowCaloricDishesName: List<String> = menu
        .stream() // parallelStream
        .filter { dish -> dish.calories < 400 }
        .sorted(comparing(Dish::calories))
        .map(Dish::name)
        .collect(toList())
}

// kotlin 확장함수 사용
fun toBe() {
    val lowCaloricDishesName = menu
      .filter { dish -> dish.calories < 400 }
      .sortedBy { it.calories }
      .map { it.name }
}
```

- stream으로 훨씬 간략화되며 임시 쓰레기 변수를 생성할 일도 없어졌다.
- **parallelStream**을 사용하면 얼마나 많은 스레드가 생성되고 성능이 좋아지는지는 7장에서 얘기한다.
- 선언형 코드를 통해 구현할 수 있게 되었다. 동작을 어떻게 할지 코드를 작성하지 않고 단순히 “400 칼로리 이하만 골라”, “요리 이름만 줘” 이런식으로 요구사항만 지정할 수 있게 됐다.

filter와 같은 연산은 고수준의 블록과 같아서 파이프라인을 형성할 수 있다. 즉 원하는 연산을 블록 쌓듯 구현이 가능하다. 
또한 내부적으로 멀티코어 아키텍처를 최대한 활용할 수 있게 구현되어 있어서 데이터 처리 병렬화를 할 때 스레드와 락을 걱정하지 않아도 된다.

Stream API의 특징을 정리하면 아래와 같다.

- 선언형 - 더 간결하고 가독성이 좋아지고
- 조립할 수 있음 - 유연성이 좋아지며
- 병렬화 - 성능이 좋아진다.

## 4.2 스트림 시작하기

스트림이란 - ***“데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소”***이다.
위 정의에서 단어 하나씩 이해해 보자.

**연속된 요소**

- 컬렉션처럼 스트림도 연속된 값 집합의 인터페이스를 제공하는데 관점의 차이가 있다.
- 컬렉션은 자료구조로서 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룬다.
    - ArrayList, LinkedList, HashMap, TreeMap …

**소스**

- 스트림은 데이터 제공 소스로부터 데이터를 소비한다.
- 정렬된 컬렉션으로부터 스트림을 생성하면 해당 요소들의 순서는 유지된다.

**데이터 처리 연산**

- 함수형 프로그래밍에서 지원하는 연산 + 데이터베이스 질의와 비슷한 연산을 지원한다.
- filter, map, reduce, find, match, sort 등으로 데이터 조작이 가능하다.
- 순차적 or 병렬적으로 실행할 수 있다.

### 특징

**파이프라이닝**

- 스트림 연산은 스트림 연산끼리 연결해서 하나의 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다.
- 이러한 특징 덕분에 laziness(지연 연산), short circuit(쇼트 서킷)과 같은 최적화 효과를 얻을 수 있다.

**내부 반복**

- 반복자 (for, while 등)를 통한 명시적인 반복이 아닌 내부 반복을 지원한다.

**예제**

```kotlin
fun innerLoop() {
    val threeHighCaloricDishNameList = menu
        .stream()
        .filter { it.calories > 300 }
        .map { it.name }
        .limit(3)
        .collect(toList())
}
```

앞에서 정리한걸 바탕으로 분석해보자.

- 데이터 소스
    - menu로 여러 메뉴들이 소스이다.
    - 이들은 소스로서 연속된 요소를 스트림에 제공한다.
- 데이터 처리 연산
    - filter, map, limit, collect로 이어지는 일련의 처리 연산을 제공한다.
- 파이프라인
    - collect를 제외한 연산들은 스트림 자신을 반환하여 서로 파이프라인을 형성할 수 있도록 한다.

## 4.3 스트림과 컬렉션

컬렉션, 스트림 둘다 ***연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공***한다.

시각적인 예시를 들어보자.

- DVD
    - DVD는 어떤 일련의 영화들이 저장된 컬렉션이다.
- 스트리밍
    - 스트리밍은 전체를 저장하지 않고 앞의 일부분을 미리 받아서 제공한다. 즉, 전체를 받을 필요 없이 미리 받은 데이터로부터 제고잉 가능하다.

즉, **데이터를 언제 계산**하느냐가 중요한 포인트이다.

**컬렉션**

- 컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 유연한 자료구조이다.
- 컬렉션은 요소를 추가/삭제할 수 있다, 이런 연산을 위해서는 모든 요소를 메모리에 저장해야하고 추가하려는 요소 또한 미리 계산이 되어야 한다.

**스트림**

- 요청할 때만 요소를 계산하는 고정된 자료구조이다.
- 스트림은 요소를 추가하거나 제거할 수 없다.
- 요소에 대한 조작없이 요청한 값만 추출해내는 것이 핵심이다.

둘의 비교는 6장에서 무제한의 소수(1, 3, 5, 7, 11 …)를 만들어보는 예제를 통해 더 알아본다.

### 4.3.1 딱 한번만 탐색할 수 있다.

스트림은 한 번만 탐색할 수 있다. 즉 한번 탐색된 스트림의 요소는 소비되어 날라간다. 
만약 한번 탐색된 요소를 다시 탐색하고자 한다면 기존 컬렉션 처럼 초기 데이터 소스로부터 다시 새로운 스트림을 생성해줘야한다.

```kotlin
val s = listOf(1, 2, 3, 4, 5).stream()
s.forEach { println(it) }
s.forEach { println(it) } // <<--- 이미 소비되었거나 스트림이 닫혀있다.
```

### 4.3.2 외부 반복과 내부 반복

컬렉션 인터페이스를 사용하려면 사용자가 for문으로 직접 요소를 반복해야한다.

```kotlin
for (dish: Dish in menu) {
	println(dish.name)
}
```

하지만 스트림을 사용하면 내부 반복을 통해 처리할 수 있다.

```kotlin
menu
.stream()
.forEach { println(it.name) }
```

컬렉션의 외부 반복은 마치 아래 예시와 같다.

- 방을 청소하자 → *바닥에 가방이 있다 치우자 → 바닥에 옷이 있다 치우자 → 바닥에 … 이 있다 치우자* → 방 청소 끝
- 방을 청소하자 → *바닥에 …이 있다 **모두** 치우자* → 방 청소 끝

즉 컬렉션은 요소를 **하나씩 가져와서 처리**하는 구조이다.

## 4.4 스트림 연산

```kotlin
val lowCaloricDishesName1: List<String> = menu
    .stream() // <<-- 스트림 얻기
    .filter { dish -> dish.calories < 400 } // <<-- 중간 연산
    .sorted(comparing(Dish::calories)) // <<-- 중간 연산
    .map(Dish::name) // <<-- 중간 연산
    .collect(toList()) // <<-- 최종 연산
```

스트림 연산은 크게 중간 연산과 최종 연산으로 구분할 수 있다.

### 4.4.1 중간 연산

filter, sorted와 같은 중간 연산은 스트림을 반환한다. 따라서 여러 중간 연산을 연결에서 질의를 구성할 수 있다.
이때의 큰 특징은 각 연산들이 파이프라인을 실행하기 전까지 아무 연산도 실행하지 않는다는 것이다. 즉 Lazy 지연 연산이 가능하다.

중간 연산을 모두 합친 후 합쳐진 중간 연산을 최종 연산으로 한번에 처리하기 때문이다.

filter, map, limit 연산을 실행하는 경우 

- limit 연산을 통해 해당 제한 조건이 먼저 수행된다. 이는 limit 연산과 short circuit 기법 덕분이다. (short circuit은 5장에서 자세히 설명)
- filter, map 각각은 다른 연산이지만 하나의 과정으로 병합되어 처리된다. 이러한 기법을 loop fusion 이라고 한다.

### 4.4.2 최종 연산

최종 연산은 스트림 파이프라인에서 결과를 도출해낸다. 보통 Int, List, Unit 등 스트림이 아닌 타입으로 반환된다.

```kotlin
menu
.stream()
.forEach { println(it.name) } // Unit 반환
```