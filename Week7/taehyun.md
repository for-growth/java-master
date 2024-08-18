## 6.4 분할

- 분할 함수라 불리는 프레디케이트를 분류 함수로 사용하는 기능
- 결과적으로 그룹화 맵은 `True` , `False` 의 두 그룹으로 분류

```java
Map<Boolean, List<Dish>> = menu.stream()
                               .collect(partitioningBy(Dish::isVegetarian));
        
//result
//{false=[pork, beef, chicken, prawns, salmon], 
//true=[french fies, rice, season fruit, pizza]}
```

### 6.4.1 분할의 장점

- `True` , `False` 의 두 가지 요소의 스트림 리스트를 모두 유지하는 것이 장점
- 두가지 키만 포함하므로 더 간결하고 효과적
- `partitioningBy()` 도 `groupingBy()` 컬렉터 처럼 다른 컬렉터와 조합하여 사용할 수 있음
    - ex) `{false=[true=[…], false=[…]],  true=[true=[…], false=[…]]}`

### 6.4.2  숫자를 소수화 비소수로 분할하기

```java
public Map<Boolean, List<Integer>> partitionPrimes(int n){
    return IntStream.rangeClosed(2, n).boxed(()
                    .collect(partitioningBy(candidate -> isPrime(candidate)));
}

private boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
                    .noneMatch(i -> candidate % i ==0);
}
```

<br>

## 6.5 Collector 인터페이스

- 리듀싱 연산을 어떻게 구현할지 제공하는 메서드 집합으로 구성

### 6.5.1 Collector 인터페이스의 메서드 살펴보기

- `Supplier` : 새로운 결과 컨테이너 만들기
    - 빈 결과로 이루어진 `Supplier` 를 반환해야함
    - 수집 과정애서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수

    
- `accumulator` : 결과 컨테이너에 요소 추가하기
    - 리듀싱 연산을 수행하는 함수 반환
    - 반환값은 void → 요소를 탐색하면서 적용하는 함수에 의해 누적자 내부상태가 바뀌므로 누적자가 어떤 값일지 단정할 수 없음
    - 스트림에서 n 번째 요소를 탐색할때 두 인수, 즉 누적자와 n 번째 요소를 함수에 적용

    ```java
    List<Integer> accumulatorList = new ArrayList<>();
    listCollector.accumulator().accept(accumulatorList, 1);
    ```


- `finisher` : 최종 변환값을 결과 컨테이너로 적용하기
    - 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼때 호출할 함수를 반환해야함
    - 누적자가 이미 최종 결과 상황이라면 항등 함수 반환


- `combiner` : 두 결과 컨테이너 병합
    - 스트림의 서로 다른 서브 파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의


- `Characteristics` : 컬렉터의 연산을 정의하는 `Characteristics` 형식의 불변 집합을 반환
    - 스트림을 병렬로 리듀스할 것인지 병렬로 리듀스 한다면 어떤 최적화를 선택할지 힌트를 제공
    - `Characteristics` 열거형
        - `UNORDERED` : 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않음
        - `CONCURRENT` : 병렬 리듀식을 수행할 수 있음
        - `IDENTIFY_FINISH` : 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있음


<br>


## 6.6 커스텀 컬렉터를 구현해서 성능 개선하기

```java
    public Map<Boolean, List<Integer>> partitionPrime(int n) {
        return IntStream.rangeClosed(2, n).boxed()
                .collect(Collectors.partitioningBy(candidate -> isPrime(candidate)));
    }

    private boolean isPrime(int candidate) {
        int candidateRoot = (int) Math.sqrt((double) candidate);
        return IntStream.rangeClosed(2, candidateRoot)
                .noneMatch(i -> candidate % i ==0);
    }
```

이 코드를 개선해보자

### 6.6.1 소수로만 나누기

**1단계 : Collector 클래스 시그니쳐 정의**

```java
/**
 * @param <T> 스트림 요소의 형식
 * @param <A> 중간 결과를 누적하는 객체의 형식
 * @param <R> 연산의 최종 결과 형식
 */
public interface Collector<T, A, R> {
}
```

**2단계 : 리듀싱 연산 구현**

```java
public class PrimeNumberCollector implements Collector<Integer, Map<Boolean, List<Integer>>, Map<Boolean, List<Integer>>>{

    public Supplier<Map<Boolean, List<Integer>>> supplier(){
        return () -> new HashMap<Boolean, List<Integer>>() {{
                put(true, new ArrayList<Integer>());
                put(false, new ArrayList<Integer>());
        }};
    }
}
```

- `Supplier` 는 누적자를 만드는 함수를 반환
- 수집 과정에서 빈 리스트에 각각 소수와 비소수를 추가

```java
public BiConsumer<Map<Boolean, List<Integer>>, Integer> accumulator() {
        return (Map<Boolean, List<Integer>> acc, Integer candidate) -> {
            acc.get(isPrime(acc.get(true), candidate))
            .add(candidate);
        };
}
```

- `accumulator` 가 스트림의 요소를 어떻게 수집할지 결정

**3단계 : 병렬 실행할 수 있는 컬렉터 만들기**

- 두 누적자를 합칠 수 있는 메서드

```java
public BinaryOperator<Map<Boolean, List<Integer>>> combiner() {
    return (Map<Boolean, List<Integer>> map1, Map<Boolean, List<Integer>> map2) -> {
            map1.get(true).addAll(map2.get(true));
            map1.get(false).addAll(map2.get(false));
            return map1;
        };
}
```

**4단계 finisher 메서드와 characteristics 메서드**

- 항등 함수 identity 를 반환하도록 finisher 메서드 구현

```java
public Function<Map<Boolean, List<Integer>>,Map<Boolean, List<Integer>> finisher() {
    return Function.identity();
} 
```

```java
public Set<Characteristics> characteristics() {
    return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH));
}
```

<br>

## 6.7 마치며

- collect 는 스트림의 요소를 요약 결과로 누적하는 다양한 방법(컬렉터라 불리는)을 인수로 갖는 최종 연산이다.
- 스트림의 요소를 하나의 값으로 리듀스하고 요약하는 컬렉터뿐 아니라 최솟값, 최댓값, 평균값을 계산나는 컬렉터등이 미리 정의되어 있다.
- 미리 정의된 컬렉터인 groupingBy 로 스트림의 요소를 그룹화하거나,  partioningBy 로 스트림의 요소를 분할할 수 있다.
- 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되어 있다.
- Collector 인터페이스에 정의된 메서드를 구현해서 커스텀 컬렉터를 개발할 수 있다.

<br>
<br>