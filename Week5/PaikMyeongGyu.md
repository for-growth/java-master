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