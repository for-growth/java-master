# Chapter 8. 컬렉션 API 개선

## 8.1 컬렉션 팩토리

자바에서 다음처럼 코드를 짜게 되면 삽입 삭제 등이 자유로운 컬렉션 리스트를 만들 수 있다. 
하지만, 해당 코드를 짜기 위해 요소 별로 나열해야 하기에 코드의 양이 늘어난다.

```Java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```

Arrays.asList() 같은 팩토리 메서드를 사용하면 코드의 양을 줄일 수 있다.
하지만, 내부적으로 고정 크기의 리스트가 생성되어 요소를 갱신만 가능하고 새 요소를 추가하거나 삭제하는 것이 불가능하다.

```Java
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
friends.set(0, "Richard");
// friends.add("Thibaut"); // UnsupportedOperationException이 발생
```

집합의 경우에는 Arrays.asSet()이라는 팩토리 메서드가 없기 때문에 다른 방법이 필요하다.
이땐 리스트를 셋으로 바꿔주는 생성자를 사용하거나 Stream을 통해 만들 수 있다. 
하지만 이렇게 만든 Set은 불필요한 객체를 할당한다. 그리고 결과는 앞과 달리 요소의 수정이 가능하다. 

```Java
Set<String> friends = new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibaut"));
Set<String> friendsByStream = Stream.of("Raphael", "Olivia", "Thibaut")
                             .collect(Collectors.toSet());
friendsByStream.add("Olaf");
```

---

### 8.1.1 리스트 팩토리
List.of라는 팩토리 메서드를 사용해 간단하게 리스트를 만들 수 있는데 
해당 객체는 불변 객체라 요소 추가도 불가능하고 set()으로 내용을 바꾸는 것도 불가능하다.

```Java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends);
// friends.add("Chih-chun"); // UnsupportedOperationException 발생!
// friends.set(0, "Olaf"); // UnsupportedOperationException 발생!
```

※ 오버로딩 vs 가변인수

가변인수를 아래처럼 사용하면 인수의 개수와 관계없이 다중요소를 입력 받을 수 있다.

```Java
static <E> List<E> of(E... elements)
```

다중 요소를 통해 간단하게 API를 정의할 수 있지만 List API에는 아래와 같은 메서드들이 정의돼 있다.

```Java
static <E>. List<E> of(E e1, E e2, E e3, E e4)
static <E>. List<E> of(E e1, E e2, E e3, E e4, E e5)
```

이렇게 정의된 이유는 가변 인수 방식은 내부에서 추가 배열을 할당해서 리스트로 감싼 뒤 가비지 컬렉션하는 비용을 지불한다.
따라서 자바에서는 이 비용에 대한 최적화로 따로 고정된 숫자의 요소를 가진 API를 메서드로 정의한다.


### 8.1.2 집합 팩토리
List.of와 비슷하게 Set.of를 통해서 바꿀 수 없는 집합을 만들 수 있다.

```Java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
```

중복된 원소가 만약에 들어간다면, 그 땐 앞에서 요소를 바꿀 땐 UnsupportedOperationException가 발생했지만
그 땐 IllegalArgumentException이 발생한다.

```Java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut"); // IllegalArgumentException
```

### 8.1.3 맵 팩토리

맵 또한 키와 값을 번가아가며 입력하는 Map.of 메서드를 통해 맵을 만들 수 있다. 하지만 이는 10개 이하의 원소를 가지는
경우까지만 정의가 되어있다. 그 이후에서부터는 entry를 사용해서 Map.ofEntries 팩토리 메서드를 사용해야 한다.

```Java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
Map<String, Integer> ageOfFriends = Map.ofEntries(entry("Raphael", 30), entry("Olivia", 25), entry("Thibaut", 26));
```

---

## 8.2 리스트와 집합 처리
자바 8에선 List, Set 인터페이스에 아래와 같은 메서드가 추가되었다.
해당 메서드는 스트림과 달리 기존 컬렉션 자체를 바꾸는 역할을 한다.

- removeIf: 프레디케이트를 만족하는 요소를 제거한다. List나 Set을 구현하거나 상속받은 클래스에 사용가능
- replaceAll: 리스트에서 이용할 수 있는 기능으로 UnaryOperator 함수로 요소를 바꿈
- sort: List 인터페이스에서 제공하는 기능으로 리스트를 정렬

### 8.2.1 removeIf 메서드

아래에선 내부적 for-each 문이 Iterator 객체를 사용해서 어떻게 변환되는 지를 보여주고 있다.

```Java
for (Transaction transaction : transactions) {
    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
        transactions.remove(transaction);
    }
}

for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
    Transaction transaction = iterator.next();
    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
        transactions.remove(transaction);
    }
}
```

위의 코드는 반복자의 상태와 컬렉션의 상태가 서로 동기화가 되지 않기 때문에 ConcurrentModificationException을 일으킨다.
그러니까 요소를 삭제하면서 다음걸 순회한다는게 정상적인 행동이 아니기 때문이라는 것이다. 

위의 문제는 아래와 같이 해결할 수 있고 더 간단하게 removeIf를 써 처리할 수도 있다.

```Java
// Iterator를 사용한 방법
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
    Transaction transaction = iterator.next();
    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
        iterator.remove(); // 이렇게 사용하면 이 문제를 해결 가능함.
    }
}
// removeIf를 사용한 방법
transactions.removeIf(transaction ->
        Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

평소에 코테를 Java로 하는데 이런 상황에서 보통 이거 배열 복사한다고 문제가 안 생겨서 그냥 새로 할당해버려 시간이 x2가 된 경우가 많았는데
조금 이건 유용한 거 같다.

### 8.2.2 replaceAll메서드
위의 경우는 요소를 삭제해야 하는 경우에 대해서 이야기를 하는데 아래의 코드는 모든 요소에 대해서 변경하는 방법에 대해 정의한다.
iterator를 쓰면 코드가 복잡하지만 replaceAll을 통해 더 간단하게 구현할 수 있다.

```Java
for (ListIterator<String> iterator = referenceCodes.listIterator(); iterator.hasNext();) {
    String code = iterator.next();
    iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
}

referenceCodes.replaceAll(code ->
        Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

---

## 8.3 맵 처리

### 8.3.1 forEach 메서드
Map의 요소에 관해서 키와 밸류를 함께 가져와 반복문을 작성하는 건 entry를 받아와야 한다. 
그래서 아래처럼 코드를 짜야 하는데 아래의 코드는 키와 값을 인수로 받는 BiConsumer를 인수로 받는 forEach문으로 더 간단하게 처리할 수 있다.

```Java
for (Map.Entry<String, Integer> entry : ageOfFriends.entrySet()) {
    String friend = entry.getKey();
    Integer age = entry.getValue();
    System.out.println(friend + " is " + age + " years old");
}

ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```

### 8.3.2 정렬 메서드
Map의 요소를 정렬해서 꺼내는 방법에 대해서는 생각을 안해봤는데, 맵의 항목을 값이나 키를 통해서 정렬할 수 있는 메서드를 제공합니다.

- Entry.comparingByValue
- Entry.comparingByKey

```Java
Map<String, String> favouriteMovies
        = Map.ofEntries(entry("Raphael", "Star Wars"),
                        entry("Cristina", "Matrix"),
                        entry("Olivia", "James Bond"));
// 사람의 이름을 알파벳 순으로 스트림 요소를 처리해서 출력한다.
favouriteMovies
        .entrySet()
        .stream()
        .sorted(Map.Entry.comparingByKey())
        .forEachOrdered(System.out::println);
```

※ HashMap 성능

HashMap은 기존에 키로 생성한 해시코드로 접근할 수 있는 버켓에 저장했음. 많은 키가 같은 해시코드에 반환되면
링크드리스트로 버킷을 반환해야 하기에 성능이 저하됐는데 탐색 시간이 O(n)임.

이걸 개선하기 위해 트리 구조로 바꿨음. 대신 String, Number 같이 Comparable을 상속받은 애들만 정렬된 트리 지원됨.
TreeMap을 사용한 거 같고 어차피 넣는것도 O(logN)으로 바뀐건데 이게 꼭 성능 개선이라고 봐야되나 싶긴 합니다.

- LinkedList는 넣을 때가 O(1)이고 조회할 떄 최악이 O(N), 충돌이 없을 때 O(1)입니다.
- TreeMap은 넣을 때가 O(logN)이고 조회할 때 최악이 O(logN), 충돌이 없을 때 O(1)입니다.

### 8.3.2 getOrDefault 메서드
기존에는 찾으려는 키 값이 존재하지 않으면 null이 반환돼 NullPointException을 방지하기 위해 널인지 확인하는 부분이 추가됐어야 했다.
하지만 getOrDefault를 통해서 이 문제를 해결할 수 있다. 
대신, 키 값이 존재하지 않을 때를 방지해주는 것이지, 키는 있고 밸류가 null인 건 방지해주지 못한다.

```Java

Map<String, String> favouriteMovies
        = Map.ofEntries(entry("Raphael", "Star Wars"),
                        entry("Olivia", "James Bond"));
// 키 값이 없을 때 null대신 default 값 반환
System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix"));
System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix"));
```

### 8.3.4 계산 패턴
아래의 메서드들은 맵에 키가 존재하는지 여부에 따라 동작을 실행하거나 저장할 때 유용하다.

- computeIfAbsent : 제공된 키에 해당하는 값이 없거나 널이면 키를 이용해 새 값을 계산하고 맵에 추가한다.
- computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.
- compute : 제공된 키로 새 값을 계산하고 맵에 저장한다.

만약 키가 존재하지 않으면 캐싱을 하는동작의 경우에는 다음과 같이 처리를 할 수 있다.

```Java
lines.forEach(line ->dataToHash.computeIfAbsent(line, this::calculateDigest));

private byte[] calculateDigest(String key) {
    return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
}
```

중복된 요소에 관한 null체크도 간단하게 줄일 수 있게 도와준다. 기존에는 값을 가져와 null체크 후 반환해야 했다면,
현재는 computeIfAbsent 메서드를 사용하면 간단하게 사용할 수 있다.

```Java
String friend = "Raphael";
List<String> movies = firendsToMovies.get(friend);
if (movies == null) {
    movies = new ArrayList<>();
    friendsToMovies.put(friend, movies);
}
movies.add("Star Wars");

// computeIfAbsent 버전
friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>()).add("Star Wars");
```

### 8.3.5 삭제 패턴
자바 8 이전에 키 값에 특정 밸류 값을 가진 경우에 삭제를 하고 싶다면 아래와 같이 구현을 했어야 했다.

```Java
Map<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Raphael", "Star Wars");
favouriteMovies.put("Olivia", "James Bond");
favouriteMovies.put("Thibaut", "Matrix");

String key = "Raphael";
String value = "Jack Reacher 2";

if (favouriteMovies.containsKey(key) && 
        Objects.equals(favouriteMovies.get(key), value)) {
    favouriteMovies.remove(key);
}
```

자바 8에서는 아래와 같이 간단하게 remove 메서드를 통해 해결할 수 있다.
```Java
favouriteMovies.remove(key, value);
```

### 8.3.6 교체 패턴
맵의 항목을 바꿀 수 있는 메서드가 맵에 추가됐다

- replaceAll : BiFunction을 적용한 결과로 각 항목의 값을 교체한다.
- replace : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑됐을 때만 바꾸는 메서드도 오버로드돼있다.

```Java
Map<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Raphael", "Star Wars");
favouriteMovies.put("Olivia", "James Bond");

favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
System.out.println(favouriteMovies);
```

### 8.3.7 합침

두 그룹의 사람과 좋아하는 영화를 매핑한 맵을 합칠 때 putAll 메서드를 사용할 수 있다.

```Java
Map<String, String> family
        = Map.ofEntries(entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(entry("Raphael", "Star Wars"));

Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends);
System.out.println(everyone);
```

위의 경우는 중복이 없는 경우에만 정상적으로 동작을 한다. 만약에 중복이 있는 경우를 처리하고 싶다면 merge를 사용해야 하는데,
merge는 중복된 키를 어떻게 합칠 지 결정하는 BiFunction을 인수로 받는다. 

```Java
Map<String, String> family
        = Map.ofEntries(entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends 
        = Map.ofEntries(entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"));
        
Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k,v) -> everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
```

이걸 이용해서 초기화 여부를 검사할 수도 있다. 기존에는 초기화 여부를 검사할 때 다음과 같이 처리르 해줬지만 merge를 사용하면 더 쉽게 할 수 있다.

```Java
HashMap<String, Long> movieToCount = new HashMap<>();
String movieName = "JamesBond";
Long count = movieToCount.get(movieName);
if (count == null) {
    movieToCount.put(movieName, 1L);
} else {
    movieToCount.put(movieName, count + 1);
}

// merge로 개선
movieToCount.merge(movieName, 1L, (count, increment) -> count + 1L);
```

---

## 8.4 개선된 ConcurrentHashMap
ConcurrentHashMap은 동기화 방식으로 돌아가는 HashTable의 동시성이 떨어지는 이슈를 방지하기 위해 
스트라이프 락 방식으로 동작하여 동시 추가, 갱신 작업을 허용하여 성능이 빠르다.

### 8.4.1 리듀스와 검색
ConcurrentHashMap도 아래의 연산을 지원한다.

- forEach : 각 (키, 값) 쌍에 주어진 액션을 실행
- reduce : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search : 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용

또한 연산 방식에 따라 4가지 방법을 지원한다.
- 키, 값으로 연산(forEach, reduce, search)
- 키로 연산(forEachKey, reduceKeys, searchKeys)
- 값으로 연산(forEachValue, reduceValues, searchValues)
- Map.Entry 객체로 연산(forEachEntry, reduceEntries, searchEntries)

ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행한다는 점 때문에 이들 연산에 제공한 함수는
계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다는 특성을 가진다.

또한, 이들 연산에는 병렬성 기준 값을 지정할 수 있는데 맵의 크기가 주어진 기준 값보다 작으면 순차적으로 연산을 수행한다.
이 부분은 Fork/Join부분이랑 비슷한 것 같습니다.