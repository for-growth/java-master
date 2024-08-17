# Chapter. 5 - 스트림 활용 (5.1 ~ 5.4)

stream을 활용해서 외부반복을 내부반복으로 변경할 수 있다는걸 확인했다.

```kotlin
// 내부반복
val vegetarianDishes = mutableListOf<Dish>()
for (dish in menu) {
    if (dish.isVegetarian()) {
        vegetarianDishes.add(dish)
    }
}

// 내부반복
val vegetarianDishes = menu.stream()
    .filter(Dish::isVegetarian)
    .collect(toList())
```

위처럼 데이터를 어떻게 처리할지는 스트림 API가 관리하므로 편리하게 데이터 처리가 가능하다.
스트림 API를 사용하면 내부 반복 뿐만아니라 병렬 실행 여부도 결정할 수 있다. 이러한 작업은 순차적인 반복을 단일 스레드에서 구현하는 외부 반복에서는 불가하다.

## 5.1 필터링

### 5.1.1 Predicate 필터링

stream의 filter 함수는 인자로 Predicate(boolean을 반환하는 함수)를 인수 받아서 일치하는 모든 요소를 포함하는 스트림을 반환한다.

```kotlin
val vegetarianDishes = menu.stream()
    .filter(Dish::isVegetarian) // boolean을 반환하는 함수
    .collect(toList())
    
---

class Dish(
    val name: String = "",
    val calories: Int = 0
) {
    fun isVegetarian(): Boolean {
        return true/false
    }
}
```

### 5.1.2 고유 요소 필터링

stream의 distinct 함수는 고유 요소로 이뤄진 스트림을 반환한다. 이때 고유 여부는 객체의 hashCode, equals로 판단이 이뤄진다.

```kotlin
val numbers = listOf(1, 2, 1, 3, 3, 2, 4)
val evenNumbers = numbers.stream()
    .filter { i -> i % 2 == 0 }
    .distinct()
    .collect(toList())
// 2, 4 만 필터 후 중복 제거되어 반환된다
```

## 5.2 스트림 슬라이싱

요소를 선택하거나 스킵하는 방법들에 대해 알아보자.

### 5.2.1 Predicate를 이용한 슬라이싱

***TakeWhile***

칼로리가 320 이하인 요리를 가져오고자 하면 어떻게 해야할까? → 본능적으로 filter를 사용할 수 있겠다 할것이다.

```kotlin
menu.stream().filter { dish -> dish.calories < 320 }.collect(toList())
```

이 경우에는 전체 요소를 루프를 돌면서 320 이하인 모든 요소에 predicate를 적용하게 된다. 하지만 만약 요소가 오름차순으로 정렬되어있다면 어떨까?
특정 요소가 320보다 크거나 같은 요소가 나오는 순간 loop를 종료해도 무방하다. 이때 사용 가능한 함수가 ***takeWhile***이다.

```kotlin
val lowCaloricDishes = menu.stream()
    .takeWhile { dish -> dish.calories < 320 } // false가 되는 순간 loop 탈출
    .collect(toList())
```

***DropWhile***

반대로 나머지 요소를 구할 때는 어떻게할까? 즉, 320 칼로리 이상인 요소만을 남기고 싶은 경우이다. 이때 ***dropWhile*** 메소드를 사용하면된다. predicate가 true 인 동안 요소를 계속 버리다가 false가 되는 순간 loop를 탈출한다. 물론 칼로리 기준으로 오름차순 정렬이 되어있어야 한다.

```kotlin
val lowCaloricDishes2 = menu.stream()
    .dropWhile { dish -> dish.calories < 320 }
    .collect(toList())
```

### 5.2.2 스트림 축소

요소들을 주어진 값 이하의 크기를 갖는 스트림을 반환하는 `limit(n)` 메소드를 지원한다.

```kotlin
val limited = menu.stream()
    .limit(3)
    .collect(toList())
```

### 5.2.3 요소 건너뛰기

첫 n개의 요소를 뽑는 대신 제외하는 메서드이다. `skip(n)`을 하면 앞에 n개를 제외하고 반한하며 전체 요소가 n보다 적으면 빈 stream을 반환한다.

```kotlin
val skipped = menu.stream()
    .skip(3)
    .collect(toList())
```

## 5.3 매핑

mapping은 특정 객체에서 특정 데이터를 선택하는 작업으로 transforming(변환)에 가까운 작업을 말한다.

### 5.3.1 스트림의 각 요소에 함수 적용하기

스트림은 함수를 인수로 받는 map 메서드가 존재한다. 인수로 제공된 함수는 각 요소에 적용되며 적용된 결과가 새로운 요소로 mapping 된다.

이때 기존 요소를 modify하는 것이 아니라 새로운 버전으로 transforming하는 것이므로 mapping이라는 단어를 사용한다.

아래는 메뉴의 이름만을 추출해서 반환하는 스트림 예시이다.

```kotlin
menu.stream()
    .map { it.name }
    .collect(toList())
```

### 5.3.2 스트림 평면화

[“Hello”, “World”] 라는 문자열 2개가 담겨있는 배열이 있을 때 중복없이 전체 단어만을 추출하려면 어떻게 해야할까? 즉 [”H”, “e”, “l”, “o”, “W”, “r”, “d”]가 되어야 한다.

이때, 아래처럼 map을 써서 처리하면 제대로 처리되지가 않는다.

```kotlin
menu.stream().map { it.name.split("") }.distinct().collect(toList())
```

- 이렇게 처리하면 각 요소인 Hello → [H, e, l, l, o], World → [W, o, r, l, d] 이렇게 별도의 배열이 만들어지고 이 둘을 distinct 비교하게 된다. [H, e, l, l, o] ↔[W, o, r, l, d]
  따라서 정상적으로 disctinct가 이뤄지지 않고 [[H, e, l, l, o], [W, o, r, l, d]] 가 반환이 된다.

### map과 Arrays.stream 활용

위 코드의 문제는 map내에서 반환하는 값들이 **Stream<String[]>** 이라는 것이다. 하지만 우리가 원하는 것은 **Stream<String>** 이므로 각 배열의 문자열을 스트림으로 만들어줘야 한다.
이를 위한 메서드로 `Arrays.stream()` 이 있다.

```kotlin
val result1: MutableList<Stream<String>> = menu.stream()
    .map { it.name.split("").toTypedArray() }
    .map { Arrays.stream(it) }
    .collect(toList())
```

단, 위처럼 처리하면 **List<Stream<String>>** 형식으로 반환이 된다.

***menu.stream()*** → `Stream { “hello”, “world” }`

***map { it.name.split(””).toTypeArray }*** → `Stream<String[]> { [”h”, “e”, “l”, “l”, “o”], [”w”, “o”, “r”, “l”, “d”] }`

***map { Arrays.stream(it) }*** → `Stream<Stream<String>> { Stream{ ”h”, “e”, “l”, “l”, “o”} , Stream{ ”w”, “o”, “r”, “l”, “d” } }`

***collect(toList()) →*** `List<Stream<String>> { Stream{ ”h”, “e”, “l”, “l”, “o”} , Stream{ ”w”, “o”, “r”, “l”, “d” } }`

따라서 **flatMap**을 이용해서 각 문자에 대해 stream을 생성해줘야한다.

***flatMap { Arrays.stream(it) }*** → `Stream<String>> { ”h”, “e”, “l”, “l”, “o”, ”w”, “o”, “r”, “l”, “d” }`

이렇게 되면 마지막에 collect를 수행하고 나면 `List<String> { “h”, “e”, … }` 로 구성되게 되므로 distinct를 정상적으로 수행할 수 있게 된다.

## 5.4 검색과 매칭

특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 Stream API에서 자주 사용된다.
allMatch, anyMatch, noneMatch, findFirst 등이 존재한다.

### 5.4.1 Predicate가 적어도 하나의 요소와 일치하는지 확인

```kotlin
if (menu.stream().anyMatch { it.isVegetarian() }) {
    println("The menu is (somewhat) vegetarian friendly!!")
}
```

- menu 스트림 중에서 적어도 하나라도 Predicate가 true인 요소가 있는지 체크

### 5.4.2 Predicate가 모든 요소와 일치하는지 검사

```kotlin
menu.stream().allMatch { dish -> dish.calories < 1000 }
```

- 스트림 내 모든 요소가 Predicate가 true인지 체크

참고로 allMatch, noneMatch, findFirst와 같은 스트림 메서드는 쇼트서킷 기법을 활용한다. 즉 하나의 요소라도 Predicate에서 일치하지 않거나 일치하면 전체 스트림을 탐색하지 않아도 종료하게 된다.

### 5.4.3 요소 검색

findAny 메서드는 현재 스트림에서 임의의 요소를 반환한다. 이때 스트림에 적합한 요소가 없으면 null이 반환될 수 있다. 이러한 경우 Optional로 받아서 nullable 객체로 표현할 수 있다.

### Optional이란?

optional은 값의 존재나 부재 여부를 표현하는 컨테이너 클래스이다. 즉 nullable 여부를 확인할 수 있는 객체이다.

- isPresent()
    - Optional에 값이 있으면 true, 없으면 false
- ifPresent(Consumer<T> block)
    - Consumer 함수형 인터페이스에는 T타입 인수를 받고 반환형이 void인 람다를 전달할 수 있다.
- get()
    - 값이 존재하면 반환, 없으면 NoSuchElementException을 일으킨다
- T orElse(other T)
    - 값이 있으면 반환, 없으면 기본값 other을 반환한다.

```kotlin
menu.stream()
    .filter { it.isVegetarian() }
    .findAny()
    .ifPresent { dish -> println(dish.name) }
```

위와같이 null이 아닌 경우에만 출력하도록 할 수 있다.

참고로 findFirst는 논리적인 아이템 순서가 있을 때 사용하고 findAny의 경우 병렬 실행에서 특별한 순서 조건이 없는 경우에 사용한다.