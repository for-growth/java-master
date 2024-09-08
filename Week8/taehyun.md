# 8장. 병렬 데이터 처리와 성능

# 8.1 컬렉션 팩토리

- 고정 크기의 List 를 만들었으면 요소 수정만 가능하고 새 요수 추가 및 삭제시 `UnsupportedOperationException`  발생

### UnsupportedOperationException 예외 발생

- 내부적으로 고정된 크기의 변화할 수 있는 배열로 구현되었기 때문에 발생

## 8.1.1 리스트 팩토리

- `List.of()` 를 사용하여 리스트를 만들 수 있음
    - 요소 추가시 `UnsupportedOperationException` 예외 발생
    - 컬렉션을 의도치 않게 변하는 것을 막을 수 있지만, 요소 자체가 변하는 것은 막을 수 없음
- 가변인수를 받을 수 있도록 설계하지 않은 이유는 배열 할당 배용과 가비지 컬렉션의 비용을 줄이기 위해

## 8.1.2 집합 팩토리

- `Set.of()` 로 바꿀 수 없는 집합을 만들 수 있음

## 8.1.3 맵 팩토리

- `Map.of()` 메서드로 키와 값을 번갈아 제공하는 방법으로 만들 수 있음
- 10 개 보마 많은  `K-V` 쌍을 가진 Map 이라면

  `Map.ofEntries(entry(key, value), entry(key, value) …)` 사용


# 8.2 리스트와 집합 처리

- List 와 Set 에 새로운 메서드 추가
    - `removeIf` : 프레디케이트를 만족하는 요소 제거. List 와 Set 을 구현하거나 그 구현을 상속받은 모든 클래스에서 이용 가능
    - `replaceAll` : 리스트에서 이용할 수 있는 기능으로 UnaryOperator 함수를 이용해 요소를 바꿈
    - `sort` : List 인터페이스에서 제공하는 기능으로 리스트를 정렬

## 8.2.1 removeIf 메서드

- `remove()` 메서드 사용시 `ConcurrentModificationException` 예외 발생
    - 두개의 객체가 컬렉션을 바꾸고 있기 때문
        - Iterator 객체가 `next()`, `hasNext()` 를 통해 질의
        - Collection 객체에서 요소 삭제
- `removeIf()` 로 개선

```java
transactions.removeIf(transaction -> transaction.getValue() == 1000);
```

## 8.2.2 replaceAll 메서드

- 스트림 API 요소를 이용해 리스트의 각 요소를 새로운 요소로 바꿀 수 있으나 새로운 컬렉션이 생성됨
- `replaceAll()` 메서드를 사용해 리스트의 각 요소를 새로운 요소로 바꿀 수 있음

```java
private static void replaceAll() {
		System.out.println(transactions);
		transactions.replaceAll(transaction -> new Transaction(null, 9999, 9999));
    System.out.println(transactions);
}

//[Transaction{trader=Trader{name='Brian', city='Cambridge'}, year=2011, value=300}, Transaction{trader=Trader{name='Raoul', city='Cambridge'}, year=2012, value=1000}, Transaction{trader=Trader{name='Raoul', city='Cambridge'}, year=2011, value=400}, Transaction{trader=Trader{name='Mario', city='Milan'}, year=2012, value=710}, Transaction{trader=Trader{name='Mario', city='Milan'}, year=2012, value=700}, Transaction{trader=Trader{name='Alan', city='Cambridge'}, year=2012, value=950}]
//[Transaction{trader=null, year=9999, value=9999}, Transaction{trader=null, year=9999, value=9999}, Transaction{trader=null, year=9999, value=9999}, Transaction{trader=null, year=9999, value=9999}, Transaction{trader=null, year=9999, value=9999}, Transaction{trader=null, year=9999, value=9999}]
```

# 8.3 맵 처리

## 8.3.1 forEach 메서드

- 맵에서 키와 값을 반복하며 하는 작업은 Map.Entry<K, V> 반복자를 이용해서 가능

```java
private static void forEach() {
        Map<Integer, Character> map = new HashMap<>();
        for (int i = 0; i < 10; i++) {
            map.put(i, (char)(65 + i));
        }

        map.forEach((key, value) -> System.out.println(key + " : " + value));
}
```

## 8.3.2 정렬 메서드

- 다음 두개의 유틸리티를 이용하여 맵의 항목을 값 또는 키를 기준으로 정렬할 수 있음
    - Entry.comparingByValue
    - Entry.comparingByKey

```java
private static void comparingBy() {
        Map<String, String> movie = Map.ofEntries(
                Map.entry("ironman", "tony stark"),
                Map.entry("avengers", "captain america"),
                Map.entry("spriderman", "peter parker"),
                Map.entry("black panther", "tChalla")
        );

        movie
                .entrySet()
                .stream()
                .sorted(Entry.comparingByKey())
                .forEachOrdered(entry -> System.out.println(entry));
}
    

//avengers=captain america
//black panther=tChalla
//ironman=tony stark
//spriderman=peter parker

```

- 많은 키가 같은 해시코드를 반환하는 상황이 되면 `O(n)` 의 시간이 걸리는 LinkedList 로 버킷을 반환해야 해서 이를 `O(log(n))` 을 반환하도록 변경

## 8.3.3 getOrDefault 메서드

- 찾으려는 키가 존재하지 않을 경우 기본값을 반환하는 방식으로 해결
- 첫번쨰 인수로 키를, 두번째 인수로 기본값을 받으며, 맵에 키가 존재하지 않으면 두 번쨰 인수로 받은 기본값을 반환

```java
private static void getOrDefault() {
		Map<String, String> movie = Map.ofEntries(
				Map.entry("ironman", "tony stark"),
        Map.entry("avengers", "captain america"),
        Map.entry("spriderman", "peter parker"),
        Map.entry("black panther", "tChalla")
    );

    System.out.println(movie.getOrDefault("batman", "ironman"));
		System.out.println(movie.getOrDefault("superman", "ironman"));
}
```

## 8.3.4 계산 패턴

- 키의 존재 여부에 따라 어떤 동작을 실행하고 결과를 저장해야하는 상황이 필요할때 사용할 수 있는 연산
    - `computeIfAbsent` : 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 값을 계산하고 맵에 추가
    - `computeIfPresent` : 재공된 키가 존재하면 새 값을 계산하고 맵에 추가
        - 정보를 캐시할 떄 사용 가능
    - `compute` : 제공된 키로 새 값을 계산하고 맵에 저장

## 8.3.5 삭제 패턴

- 제공된 키에 해당하는 맵 항목은 `remove()` 를 사용하여 제거 가능

## 8.3.6 교체 패턴

- `replaceAll()` : BiFunction 을 적용한 결과로 각 항목의 값을 교체. List 의 replaceAll 과 비슷
- `Replace()` : 키가 존재하면 앱의 값을 바꿈. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있음

## 8.3.7 합침

- 두 Map 을 `putAll()` 메서드를 사용해서 합칠 수 있음
- 중복된 키가 있을 경우 `merge()` 를 사용해 유연하게 합칠 수 있음

# 8.4 개선된 ConcurrentHashMap

- 동시성 친화적인 HashMap
- 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용
- 동기화된 HashTable 버전에 비해 읽기 쓰기 연산 성능이 월등

## 8.4.1 리듀스와 검색

- 새로운 연산 지원
    - forEach : 각 키,값 쌍에 주어진 액션을 실행
    - reduce : 모든 키, 값 쌍을 제공된 리듀스 함수를 이용해 결과로 합칩
    - search : 널이 아닌 값을 반환할 때까지 각 키, 값 쌍에 함수를 적용
- 키, 값으로 연산 : forEach, reduce, search
- 키로 연산 : forEachKey, reduceKeys, searchKeys
- 값으로 연산 : forEachValue, reduceValues, searchValues
- Map.Entry 객체로 연산 : forEachEntry, reduceEntries, searchEntries
- 이 연산들은 ConcurrentHashMap 의 상태를 잠그지 않고 연산을 수행
- 연산에 병렬성 기준값을 지정해야함

## 8.4.2 계수

- ConcurrentHashMap 클래스는 맵의 매핑 개수를 반환하는 mappingCount 메서드 제공
- 기존의 size 대신 mappingCount 를 사용해야 갯수가 int 범위를 넘어갈 때 대처 가능

## 8.4.3 집합 뷰

- ConcurrentHashMap 을 집합뷰로 반환하는 keySet 메서드 제공
- 맵을 바꾸면 집합도 바뀌고 반대도 마찬가지

# 8.5 마치며

- 자바9는 바꿀수 없는 리스트, 집합 맵을 만드는 메서드를 지원
- 이들 컬렉션 팩토리가 반환된 객체는 만들어진 다음 바꿀 수 없음
- List 인터페이스는 removeIf, replaceAll, sore 세가지 디폴트 메서드를 지원
- Set 인터페이스는 removeIf 디폴트 메서드를 지원
- Map 인터페이스는 자주 사용하는 패턴과 버그를 방지할 수 있도록 다양한 디폴트 메서드 지원
- ConcurrentHashMap 은 Map 에서 상속받은 새 디폴트 메서드를 사용함과 동시에 스레드 안정성 제공


<br>
<br>