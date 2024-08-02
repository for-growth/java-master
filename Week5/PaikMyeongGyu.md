# Chapter. 5 - 스트림 활용 (5.1 ~ 5.5)

## 5.1 필터링

### 5.1.1 프레디케이트로 필터링

스트림 인터페이스는 filter 메서드를 지원한다. filter 메서드는 프레디케이트를 인수로 받아
일치하는 모든 요소를 포함하는 스트림을 반환한다. 

filter 내부의 프레디케이트는 and와 or 조건을 통해 체이닝이 가능하다.
이를 활용하여 다음과 같이 코드를 작성하면 and 조건이나 or 조건으로 필터링이 가능하다.

```Java
// 프레디케이트로 거름, 프리디케이트가 filter 내부에 조건으로 사용됨.
System.out.println("Filtering with a predicate");
List<Dish> vegetarianMenu 
        = Dish.menu.stream()
                .filter(Dish::isVegetarian)
                .collect(toList());

vegetarianMenu.forEach(System.out::println);

// 만약 and 조건으로 하고 싶다면?
System.out.println("Filtering with a predicate and condition");
List<Dish> lightVegetarian 
        = Dish.menu.stream()
                .filter(((Predicate<Dish>) Dish::isVegetarian)
                .and(d -> d.getCalories() < 450))
                .collect(toList());

lightVegetarian.forEach(System.out::println);
// 만약 or 조건으로 검색을 하고 싶다면?
System.out.println("Filtering with a predicate or condition");
List<Dish> lightOrVegetarian 
        = Dish.menu.stream()
                .filter(((Predicate<Dish>) Dish::isVegetarian)
                .or(d -> d.getCalories() <= 450))
                .collect(toList());

lightOrVegetarian.forEach(System.out::println);
```

### 5.1.2 고유 요소 필터링
스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct 메서드도 지원을 한다.
고유 여부는 스트림에서 만든 객체의 hashCode, equals로 결정된다.

기본 distinct 사용법은 아래와 같다. 아래의 코드는 리스트의 모든 짝수를 선택하고
중복을 필터링한다.

```Java
System.out.println("Filtering unique elements:");
List<Integer> numbers = Arrays.asList(1,2,1,3,3,2,4);
numbers.stream()
            .filter(i->i % 2 == 0)
            .distinct()
            .forEach(System.out::println);
```

그럼, 앞에서 말한 고유 여부를 지키지 않았을 때는 무슨 일이 벌어지는 지 확인해보자.
다음과 같이 임시로 Node라는 내부 클래스를 만들고 해당 값을 distinct로 거른 후, 출력을 해보았다.

```Java
public class WrongDistinct {

    public static void main(String[] args) {
        List<Node> nodes = new ArrayList<Node>();
        nodes.add(new Node(1,2));
        nodes.add(new Node(1,2));
        nodes.add(new Node(1,3));
        nodes.stream().distinct().forEach(System.out::println);
    }

    static class Node {
        int x;
        int y;

        public int getX() {
            return x;
        }

        public void setX(int x) {
            this.x = x;
        }

        public int getY() {
            return y;
        }

        public void setY(int y) {
            this.y = y;
        }

        public Node(int x, int y) {
            this.x = x;
            this.y = y;
        }

        @Override
        public String toString() {
            return "Node{" +
                    "x=" + x +
                    ", y=" + y +
                    '}';
        }
    }
}
```

결과는 당연히 stream() api에서 제공하는 equals와 hashCode는 Objects에 정의된 기본 값을 사용할 것이고
이는 객체 자체의 참조값이므로 값이 다를 수 밖에 없다. 따라서, 결과에 Node(1,2), Node(1,2)가 나올 걸 기대했겠지만,
Node(1,2)가 중복돼서 나온다.

이를 해결하기 위해 간단하게 인텔리제이에서 자동완성해주는 hashCode와 equals를 사용했다. 
객체 내부에 객체를 가질 수도 있는데 이 부분에 대해서도 추가적인 고민이 필요하다.

```Java
public class RightDistinct {
    public static void main(String[] args) {
        List<Node> nodes = new ArrayList<>();
        nodes.add(new Node(1,2));
        nodes.add(new Node(1,2));
        nodes.add(new Node(1,3));
        nodes.stream().distinct().forEach(System.out::println);
    }
    static class Node {
        int x;
        int y;

        public int getX() {
            return x;
        }

        public void setX(int x) {
            this.x = x;
        }

        public int getY() {
            return y;
        }

        public void setY(int y) {
            this.y = y;
        }

        public Node(int x, int y) {
            this.x = x;
            this.y = y;
        }

        @Override
        public String toString() {
            return "Node{" +
                    "x=" + x +
                    ", y=" + y +
                    '}';
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Node node = (Node) o;
            return x == node.x && y == node.y;
        }

        @Override
        public int hashCode() {
            return Objects.hash(x, y);
        }
    }
}
```
---

## 5.2 스트림 슬라이싱

### 5.2.1 프레디케이트를 이용한 슬라이싱

자바 9부터 스트림 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두 가지 메서드를 지원한다.

TAKEWHILE과 DROPWHILE:
여러 음식 메뉴 중 320 칼로리 이하인 요리를 선택하기 위해 filter를 선택하는 건, 모든 요소를 다 탐방해야 한다. 
하지만 값이 정렬되어 있다고 생각을 해보자. 값이 오름차순 정렬된 경우엔 320보다 높은 값이 나오는 경우, 해당 반복을 중단해도 상관이 없다.

이러한 상황에서 최적화를 할 수 있는 스트림 API가 TAKEWHILE과 DROPWHILE이다. 사용 방법은 다음과 같다.

```Java
System.out.println("Sorted menu sliced with takewhile():");
List<Dish> sliceMenu1 = specialMenu.stream()
        .takeWhile(dish -> dish.getCalories() < 320)
        .collect(toList());
sliceMenu1.forEach(System.out::println);

System.out.println("Sorted menu sliced with dropwhile():");
List<Dish> sliceMenu2 = specialMenu.stream()
                                .dropWhile(dish -> dish.getCalories() < 320)
                                .collect(toList());
sliceMenu2.forEach(System.out::println);
```

각각의 차이는 TAKEWHILE은 조건문 내의 범위 바깥의 것이 감지됐을 때 반복을 중단한다. 반대로 DROPWHILE은 조건문 내의 요소를 버리고
조건 범위 바깥의 남은 모든 요소를 반환한다.

그런데, 정렬이 보장되어 있지 않다면 어떻게 동작할까? 정렬이 되지 않은 경우엔 의도한 결과와 다를 수 있다. 
의도에 맞게 사용하기 위해 정렬되지 않은 요소를 정렬 후 사용하는 방법도 있다.

```Java
// 그러면 정렬이 되어 있지 않다면??
List<Integer> tmpNumbers = Arrays.asList(3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5);
List<Integer> result = tmpNumbers.stream()
                                .takeWhile(n -> n < 5)
                                .collect(Collectors.toList());

// 3, 1, 4, 1 값만 나옴.
System.out.println(result);

// 정렬을 한번 해주고 다시 하면? 원하는 값이 나옴.
List<Integer> tmpNumbers2 = Arrays.asList(3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5);
List<Integer> result2 = tmpNumbers.stream()
                                .sorted()
                                .takeWhile(n -> n < 5)
                                .collect(Collectors.toList());

System.out.println(result2);
```

### 5.2.2 스트림 축소
스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 limit(n) 메서드를 지원한다. 

```Java
List<Dish> dishes = specialMenu.stream()
                            .filter(dish -> dish.getCalories() > 300)
                            .limit(3)
                            .collect(toList());
```

스트림 축소에서 주의사항은 스트림 축소로 limit을 사용하면 입력이 무한인 경우에 limit에 선언된 개수만큼 요소를 채우려는 성질이 있다.
그래서 무한 스트림을 생성되는 경우 아래와 같은 코드를 조심해야 한다.

```Java
// 무한 스트림 문제 -> 10개가 채워질 때까지 계속 요소를 반복생산하는데,
// 스트림은 요소가 2개 밖에 없음을 모르기 때문에 무한 루프 발생
Stream.iterate(0, i -> (i + 1) % 2)
        .distinct()
        .limit(10)
        .forEach(System.out::println);

System.out.println("Complete!!");

// 순서를 바꿔서 해결하는 것을 추천
Stream.iterate(0, i -> (i + 1) % 2)
        .limit(10)
        .distinct()
        .forEach(System.out::println);

System.out.println("Complete!!");
```

### 5.2.3 요소 건너뛰기
스트림은 처음 n개 요소를 제외한 스트림을 반환하는 skip(n) 메서드를 지원한다. skip에서 주의할 건 모든 stream 중에서 2개를 건너 뛴다가 아닌
skip 전까지의 통과한 요소 중 n개를 제외한다. 따라서 원래 결과값이 4개이고 skip 값이 2이면 결과값이 무조건 2개가 나온다.

```Java
// 요소 생략
List<Dish> dishesSkip2 
        = Dish.menu.stream()
                    .filter(d -> d.getCalories() > 300)
                    .skip(2)
                    .collect(toList());
System.out.println("Skipping elements:");
dishesSkip2.forEach(System.out::println);
```

---

## 5.3 매핑

### 5.3.1 스트림의 각 요소에 함수 적용하기

스트림은 함수를 인수로 받는 map 메서드를 지원한다. 인수로 제공된 함수가 적용되면 기존 값을 수정하는 게 아닌 새로운 값으로 매핑된다.
이 말은 원본에는 영향을 주지 않음을 의미한다.

아래는 단어를 인수로 받아 길이를 반환하는 함수인데, 응답 결과와 기존 값을 출력해보자.

```Java
// 단순 map을 한번만 사용
List<String> words = Arrays.asList("Modern", "Java", "In", "Action");
List<Integer> wordLengths = words.stream()
                                 .map(String::length)
                                 .collect(toList());
System.out.println(wordLengths);
// 원본 결과에는 영향을 주지 않는다.
System.out.println(words.toString());
```

### 5.3.2 스트림 평면화
map을 이용해서 고유 문자로 리스트를 반환하고 싶은 경우엔 flatMap을 활용할 수 있다.


아래의 코드는 ["H", "e", "l", "o", "w", "r", "d"]를 기대하고 작성을 했을 것이다. 
결과를 그렇게 만들 수 없는 건 아니지만 응답 타입이 List<String[]>으로 나오게 된다.

```Java
// 스트림의 평면화 중 원치 않는 타입이 나옴.
List<String> words2 = List.of("Hello", "World");
List<String[]> collect = words2.stream()
                               .map(word -> word.split(""))
                               .distinct()
                               .collect(toList());

for (String[] w : collect) {
    for (String s : w) {
        System.out.println(s);
    }
}
```
flatMap을 활용하면 이 문제를 해결할 수 있다.

```Java
// split으로 나온 배열을 또 스트림화 시키면 List<Stream> 형태로 값이 나옴 이를 해결하기 위해선 따로 평면화가 필요함.
Stream<Stream<String>> streamStream 
        = words2.stream()
                .map(word -> word.split(""))
                .map(Arrays::stream);
        
List<Stream<String>> collect1 
        = streamStream.distinct().collect(toList());

// flatMap 사용
Stream<String> stringStream 
        = words2.stream()
                .map(word -> word.split(""))
                .flatMap(Arrays::stream);

List<String> collect2 
        = stringStream.distinct()
                      .collect(toList());
```

개인적으로 처음 학습할 때 굉장히 헷갈렸었는데 Stream을 사용한 작성과정에서
Stream<Stream<String>>의 형태로 된 경우 혹은 응답이 Stream<String[]>과 같은 중간 과정이 있는지를 확인해보는 걸 권장한다.

### 매핑 관련 예시

숫자 쌍과 관련해서 예시를 들어주는 부분은 꼭 다시 연습을 해볼 필요가 있다 생각함.

```Java
// 제곱 값 반환하기
List<Integer> numbers = Arrays.asList(1,2,3,4,5);

List<Integer> squares 
        = numbers.stream()
                 .map(n -> n * n)
                 .collect(Collectors.toList());
squares.forEach(n -> System.out.println(n));

// 주어진 모든 숫자 쌍 만들기
List<Integer> numbers1 = Arrays.asList(1,2,3);
List<Integer> numbers2 = Arrays.asList(3, 4);
List<int[]> collect 
        = numbers1.stream()
                  .flatMap(i -> numbers2.stream().map(j -> new int[]{i, j}))
                  .collect(Collectors.toList());
        
for (int[] arr : collect) {
    System.out.println(Arrays.toString(arr));
}

// 위의 예제 숫자 쌍 만들기에서 합이 3으로 나누어 떨어지는 쌍만 반환
List<int[]> pairs 
        = numbers1.stream()
                  .flatMap(i -> 
                          numbers2.stream()
                                  .filter(j -> (i + j) % 3 == 0).map(j -> new int[]{i, j}))
                  .collect(Collectors.toList());

for (int[] pair : pairs) {
    System.out.println(Arrays.toString(pair));
}
```

## 5.4 검색과 매칭
특정 속성이 데이터 집합에 있는지 확인하는 용도로 사용된다. 여러 요소가 있지만 아래의 요소의
이름만 보아도 될 것 같다 생각한다.

```Java
// 적어도 한 요소와 일치하는지 확인할 때 anyMatch 메서드를 이용한다.
if (menu.stream().anyMatch(Dish::isVegetarian)) {
    System.out.println("The menu is (somewhat) vegetarian friendly!!");
}

// 스트림의 모든 요소와 일치하는지는 allMatch를 통해서 가능하다.
boolean isHealthy 
        = menu.stream()
              .allMatch(dish -> dish.getCalories() < 1000);
System.out.println("isHealthy: " + isHealthy);

boolean isHealthy2 = menu.stream().noneMatch(d -> d.getCalories() >= 1000);
System.out.println("isHealthy2: " + isHealthy2);
```

※ 추가정보

anyMatch, allMatch, nonMatch는 쇼트 서킷 최적화가 적용되어 있다. 쇼트 서킷 최적화는
and나 or 검사에서 일부 조건을 만족하면 전체를 검사할 필요가 없는 아래의 특성을 활용한다.

1. and 조건의 경우, 하나라도 false인 경우 나머지 값에 상관없이 결과는 false이다.
2. or 조건의 경우, 하나라도 true인 경우 나머지 값에 상관없이 결과는 true이다.

### 5.4.4 첫 번째 요소 찾기
스트림에는 논리적인 순서가 존재한다. 이 때 조건에 만족하는 첫 번째 값을 찾아야 하는 경우, findFirst와 findAny함수를 고려할 수 있다.

이 때, 병렬 상황과 병렬 상황이 아닌 경우를 고려할 수 있다. 반드시 첫 번째 요소인 것이 중요하다면 비병렬 상황으로 findFirst를 사용하는 걸 추천한다.

만약, 병렬 상황이 아니라면 findFirst를 사용하면? findFirst가 첫 번째 요소가 아닌 본인 쓰레드에 분할된 결과값의 첫번째 값을 가져올 수도 있다.
하지만, 순서가 상관없이 아무거나 하나만 가져오는 경우엔 findAny를 통해 요소를 병렬처리를 하면 더 빨리 가져올 수 있다.

위의 내용에 대해서 정확하게 돌아가는 지는 모르겠습니다. 개인적으로 궁금해서 직접 돌려봤는데, FindAny랑 First랑 사실상 거의 차이가 없는 것 같아요.

```Java
long beforeTime = System.currentTimeMillis();
List<String> list = new ArrayList<>();

for (int i = 1; i < 50000000; i++) {
    list.add("Java " + i);
}

list.parallelStream().filter(s -> {
    String[] tmp = s.split(" ");
    if (Integer.valueOf(tmp[1]) % 7 == 0)
        return true;
    return false;
}).findFirst().ifPresent(System.out::println);

long afterTime = System.currentTimeMillis();

System.out.println(afterTime -beforeTime);
```

---

## 5.5 리듀싱

리듀스 연산을 통해 메뉴의 모든 칼로리의 합계를 구하시오, 메뉴에서 칼로리가 가장 높은 요리는? 같이 스트림 요소를 조합한 질의
응답을 하는 방법이다. 이런 질의를 리듀싱 연산, 폴드라고 부른다.

### 5.5.1 요소의 합

reduce를 통해서 반복된 패턴을 추상화가 가능하다. 스트림의 reduce 연산 과정에선 스트림이 하나의 값으로 줄어들 때까지
람다는 각 요소를 반복해서 조합한다. 

이런 과정을 최대한 간소화해서 미리 패턴화된 메서드가 존재하는데 아래와 같다.

```Java
// Integer 클래스에 두 숫자를 더하는 정적 메서드 sum
int sum = numbers.stream().reduce(0, Integer::sum);

// 최댓값과 최솟값을 구할 수 있는 메서드
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);

// 위를 보면 Optional이 있는데, 초깃값이 없는 경우엔 값이 없는 것과 구분을 해야 해서 Optional로 반환을 한다.
// 그 이유는 0을 넣어보면, 모든 값이 양수인 값인 경우 min이 오염된다. max도 비슷한 이유이다.
Optional<Integer> sum = numbers.stream().reduce((a,b) -> (a + b));
```

※ reduce 메서드의 장점과 병렬화

reduce 연산을 사용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 reduce를 실행할 수 있게 된다. 반복되는 합계에선
sum 변수를 공유해야 하므로 쉽게 병렬화가 어렵다. 이를 억지로 동기화 시키는 경우 병렬화로 얻어야 할 이득을 쓰레드 경쟁으로 
상쇄 시킬 수 있다.

parallelStream 스트림도 넘겨준 람다의 상태(인스턴스 변수)가 바뀌지 말아야 하고 연산이
어떤 순서로 실행되더라도 결과가 바뀌지 않는 구조여야 한다.

※ 스트림의 연산: 상태 없음과 상태 있음.

이 부분이 가장 이해가 안됐는데 개인적으로 3가지 분류의 의미에 대해 다시 생각해봤다.

1. map과 filter와 같은 메서드 : 이런 메서드는 본인만 알면 돼서 상태를 가지지 않는다.
2. reduce, sum, max와 같은 메서드 : 직전 상태에 대해서 순차적으로 처리를 해줘야 한다.
그래서 직전 상태만 알아도 되므로 크기가 한정(bound)되어있다고 언급한 것 같다.
3. sorted나 distinct와 같은 메서드 ; 해당 메서드는 요소 내 모든 원소들(언바운드)에 대해 정보를
알고 있어야지 된다. 그렇기 때문에 내부 상태를 갖는 연산이라 한다.

## 5.6 실전 연습

문제 1. 2011년에 일어난 모든 트랜재션을 찾아서 값을 오름차순으로 정렬하시오.

```Java
List<Transaction> answer1 
        = transactions.stream()
                      .filter(transaction -> transaction.getYear() == 2011)
                      .sorted(Comparator.comparing(Transaction::getValue))
                      .collect(Collectors.toList());
```

문제 2. 거래자가 근무하는 모든 도시를 중복 없이 나열하시오.

```Java
List<String> answer2 
        = transactions.stream()
                      .map(transaction -> transaction.getTrader().getCity())
                      .distinct()
                      .collect(Collectors.toList());
```

문제 3. 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오.

map이 처음에 배웠을 땐, 알아야 될 게 있나 싶었는데 직접 작성해보려고 하니까 map을 안쓰면
변수가 엄청 길어지는 게 확 느껴졌습니다.

```Java
List<Trader> answer3 
        = transactions.stream()
                      .map(Transaction::getTrader)
                      .filter(trader -> trader.getCity().equals("Cambridge"))
                      .distinct()
                      .sorted(Comparator.comparing(Trader::getName))
                      .collect(Collectors.toList());
```

문제 4. 모든 거래자의 이름을 알파벳 순으로 정렬해서 반환하시오.

Java에서 reduce로 값을 더하면 String을 복사해서 뒤에 추가한 뒤 이걸 반복함.
시간 복잡도가 매우 좋지 않으니 차라리 StringBuilder를 지원해주는 Joining()

```Java
String answer4
        = transactions.stream()
                      .map(transaction -> transaction.getTrader().getName())
                      .distinct()
                      .sorted()
                      .collect(joining());
```

문제 5. 밀라노에 거래자가 있는가?

```Java
boolean answer5 = 
        transactions.stream()
                    .anyMatch(transaction -> transaction.getTrader().getCity().equals("Milan"));
```

문제 6. 케임브리지에 거주하는 거래자의 모든 트랜잭션 값을 출력하시오.

```Java
transactions.stream()
            .filter(t -> t.getTrader().getCity().equals("Cambridge"))
            .map(Transaction::getValue)
            .forEach(System.out::println);
```

문제 7,8 전체 트랜잭션에서의 최댓값과 최솟 값은 무엇인가?

최솟값과 최대값을 구할 땐, 값이 없을 수도 있으니 Optional이 나오고 이후에 특화
스트림을 통해서도 가능하다.

```Java 
Optional<Integer> highestValue 
        = transactions.stream()
                      .map(Transaction::getValue)
                      .reduce(Integer::max);

Optional<Integer> lowestValue 
        = transactions.stream()
                      .map(Transaction::getValue)
                      .reduce(Integer::min);
```

---

## 5.7 숫자형 스트림


### 5.7.1 기본형 특화 스트림

일반 스트림은 전체 Object 타입을 대상으로 만들어졌기 때문에 sum, min, max 같은 자주
사용할 것 같지만 숫자에만 연관이 있는 메서드를 만들지 못함.

그래서 이런 것이 가능한 특화 스트림을 만들어져 있고 사용이 가능함.

```Java
int sum = menu.stream()
              .mapToInt(Dish::getCalories)
              .sum();
```

특화 스트림 타입을 원하지 않고 직접 정의한 타입으로 반환하고 싶을 수 있는데,
그렇다면 복원하기를 사용하면 된다.

```Java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
```

기본 값 : OptionalInt는 앞에서 언급한 것과 특화스트림 여부만 다릅니다.

### 5.7.2 숫자 범위

range와 rangeClosed는 생성하고자 하는 숫자의 범위를 작성이 가능하다.
- range: 시작값부터 종료값 범위 사이에서 시작값과 종료 값이 결과에 포함되지 않음.
- rangeClose: 시작값부터 종료값 모두 포함

아래는 1부터 100까지니, 2부터 100까지의 모든 짝수가 담긴다.

```Java 
IntStream evenNumbers 
        = IntStream.rangeClosed(1, 100)
                   .filter(i -> i % 2 == 0);
```

### 5.7.3 피타고라스의 수

개인적으로 flatMap의 사용과 변수가 두 개인 걸 알아볼 수 있었던 좋은 예제라 생각합니다.

```Java
IntStream.rangeClosed(1, 100)
         .filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
         .boxed()
         .map(b -> new int[]{a, b, (int) Math.sqrt(a * a + b * b)});
```

우선 위와 같이 작성하면 a가 없어서 오류가 뜹니다. 이를 위해선 rangeClosed를 한번 더 중첩해서 사용해야 합니다.

```Java
Stream<Stream<int[]>> streamStream 
        = IntStream.rangeClosed(1, 100).boxed()
                   .map(a ->
                        IntStream.rangeClosed(a, 100)
                                .filter(b -> Math.sqrt(a * a + b * b) % 1 == 0)
                                .mapToObj(b ->
                                        new int[]{a, b, (int) Math.sqrt(a * a + b * b)}));
```

중간 결과 값이 int[]는 원하는 값이 맞으니 상관없지만 Stream<Stream> 형태로 나오니 flatMap으로 Stream을 한차원 내려줘야 한다.

```Java
Stream<int[]> pythagoreanTriples 
        = IntStream.rangeClosed(1, 100).boxed()
                   .flatMap(a ->
                        IntStream.rangeClosed(a, 100)
                                .filter(b -> Math.sqrt(a * a + b * b) % 1 == 0)
                                .mapToObj(b ->
                                        new int[]{a, b, (int) Math.sqrt(a * a + b * b)}));
```

---

## 5.8 스트림 만들기

### 5.8.1 값으로 스트림 만들기

정적 팩토리 메서드 Stream.of 같은 걸 통해 특정 타입 스트림을 만들 수 있다.

```Java
Stream<String> streamOf = Stream.of("Modern", "Java", "In", "Action");
streamOf.map(String::toUpperCase).forEach(System.out::println);
```

empty 메서드를 통해서 빈 스트림도 만들거나 빈 스트림으로 만들 수도 있다.

```Java
Stream<Object> emptyStream = Stream.empty();
```

### 5.8.2 null이 될 수 있는 객체로 스트림 만들기

아래처럼 null이 될 수 있는 객체를 받아서 스트림으로 만들 수 있다.

```Java
String homeValue = System.getProperty("home");
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
```

이를 null과 문자열이 섞인 배열에 활용할 수 있다. 배열에서 각각의 배열로 된 객체로 나누어 사용하기 위해 flatMap을
사용해야 한다. flatMap을 활용하지 않으면 Stream<Stream> 형태가 된다.

```Java
Stream<String> values 
        = Stream.of("config", "home", "user")
                .flatMap(key -> Stream.ofNullable(System.getProperty(key)));

Stream<Stream<String>> nonFlatValues 
        = Stream.of("config", "home", "user")
                .map(key -> Stream.ofNullable(System.getProperty(key)));
```

### 5.8.3 배열로 스트림 만들기

배열을 인수로 받는 정적 메서드 Arrays.stream을 통해 스트림 만들 수 있음.

```Java
int[] numbers = {2,3,5,7,11,13};
int sumT = Arrays.stream(numbers).sum();
```

### 5.8.4 파일로 스트림 만들기 

파일을 처리하는 등의 I/O 연산에 사용하는 NIO API도 스트림을 활용할 수 있다. 파일에서 고유한 단어의 개수를 확인하는 연산

```Java
long uniqueWords = 0;
try(Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
    uniqueWords 
            = lines.flatMap(line -> Arrays.stream(line.split(" ")))
                   .distinct()
                   .count();
} catch (IOException e) {
}
```

### 5.8.5 함수로 무한 스트림 만들기

iterate와 generate를 통해 무한 스트림을 만들 수 있다. 고정되지 않은 크기의 스트림을 만들 수 있는데, iterate와 generate에서 만든 스트림은
요청할 때마다 주어진 함수를 이용해서 값을 만들고 무제한으로 만들 수 있다. 무한한 출력을 방지하기 위해 limit과 함께 사용한다.

```Java
Stream.iterate(0, n -> n + 2)
        .limit(10)
      .forEach(System.out::println);

// iterate 내부에는 Predicate도 들어감
IntStream.iterate(0, n -> n < 100, n -> n + 4)
         .forEach(System.out::println);
```

filter와 사용할 땐 무한한 스트림이 생성되지 않도록 조심하자. 아래의 예제는 filter는 전체 스트림에 대해서 해당 요소만을 걸러내는 역할을 한다.
그래서 iterate로 생성되는 내용이 무한이지만 정렬 순서가 아닌 전체 스트림에 대해 동작하는 filter는 그게 종료상황인지 알지 못한다.

하지만, takeWhile은 쇼트서킷을 지원하고 조건에 맞는 요소가 있는 경우 즉시 종료한다. 그래서 앞에서도 정렬이 되지 않은 경우 주의를 요했었다.
이 예제에 경우엔 옳게 사용된 경우이다.

```Java
IntStream.iterate(0, n -> n + 4)
         .filter(n -> n < 100)
         .forEach(System.out::println);

IntStream.iterate(0, n -> n + 4)
         .takeWhile(n -> n < 100)
         .forEach(System.out::println);
```

generate 메서드, generate는 Supplier<T>를 인수로 받아서 사용을 한다. 그런데, generate함수를 쓸 때 limit을 사용하지 않으면
무한한 상태로 크기가 한정되지 않는다. 

그리고 IntSupplier 같은 걸 사용할 때 익명 클래스를 사용할 때는 조심을 해야 한다. 특히 조심해야 하는 건, IntSupplier에서 익명 클래스를 구현하는 
과정에서 인스턴스 변수에 어떤 요소가 들어있는지 추적을 할 수 있게 만들 수 있는데 이는 매번 getAsInt를 호출할 때마다 새로운 값을 생산하고 저장한다.

이러한 구현이 일어나는 경우 병렬 처리에 있어서 올바른 값을 얻지 못할 수 있다.

```Java
IntSupplier fib = new IntSupplier() {
    private int previous = 0;
    private int current = 1;
    public int getAsInt() {
        int oldPrevious = this.previous;
        int nextValue = this.previous + this.current;
        this.previous = this.current;
        this.current = nextValue;
        return oldPrevious;
    }
};

IntStream.generate(fib).limit(10).forEach(System.out::println);
```