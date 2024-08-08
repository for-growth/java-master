# 스트림으로 데이터 수집

해당 Chapter의 목표는 4장, 5장에서는 최종 연산 collect만을 사용햇는데, 이를 다양한 방식으로
사용할 수 있도록 돕는 것을 목표로 합니다.


명령형 버전의 경우엔 문제를 해결하기 위해 다중 루프와 조건문을 추가하기 때문에 가독성이 떨어지지만
선언 방식을 통해 가독성을 높여 유지보수성을 늘릴 수 있다.

해당 장의 학습을 통해 통화별로 트랜잭션 리스트를 그룹화하라는 식으로 코드를 명령형이 아닌
선언형으로 짤 수 있다. 아래는 그 예시이다.

```Java
Map<Currency, List<Transaction>> transactionsByCurrencies
                = transactions.stream()
                              .collect(groupingBy(Transaction::getCurrency));
```

### 6.1.2 미리 정의된 컬렉터

Collectors에서 제공하는 메서드의 기능은 크게 세 가지로 구분이 가능하다.
- 스트림 요소를 하나의 값으로 리듀스하고 요약
- 요소 그룹화
- 요소 분할

※ 주의할 사항<br>
Collection, Collector, Collect를 혼동하지 않고 살펴 볼 것!

collect: 스트림 연산의 최종 연산 메서드 중 하나
collector: collect에서 필요한 메서드들을 정의해놓은 인터페이스
collectors: collector를 구현한 클래스들을 제공

Collector 내의 인터페이스 종류

```Java
import java.util.function.BinaryOperator;
import java.util.function.Supplier;

public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    BinaryOperator<A> combiner();
    Function<A,R> finisher();
    Set<Characteristics> characteristics();
}
```

Collectors 클래스는 미리 Collector를 구현한 클래스 5가지 종류
- 변환 : mapping(), toList(), toMap(), toCollection(), ...
- 통계 : counting(), summingInt(), averagingInt(), maxBy(), minBy(), summarizingInt(),
- 문자열 결합 : joining()
- 리듀싱 : reducing()
- 그룹화와 분할 : groupingBy(), partitioningBy(), collectingAndThen()

---

## 6.2 리듀싱과 요약

리듀싱 관련 요소를 직접적으로 보여주는 것보다는 연습용 자료로 쓰는 게 좋을 것 같다 생각
해서 질의로 바꿔봤습니다.

Dish, menu, type 구성
```Java
public class Dish {

    private final String name;
    private final boolean vegetarian;
    private final int calories;
    private final Type type;

    public Dish(String name, boolean vegetarian, int calories, Type type) {
        this.name = name;
        this.vegetarian = vegetarian;
        this.calories = calories;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public boolean isVegetarian() {
        return vegetarian;
    }

    public int getCalories() {
        return calories;
    }

    public Type getType() {
        return type;
    }

    @Override
    public String toString(){
        return name;
    }


    public enum Type {
        MEAT,
        FISH,
        OTHER
    }

    public static final List<Dish> menu = asList(
            new Dish("pork", false, 800, Type.MEAT),
            new Dish("beef", false, 700, Type.MEAT),
            new Dish("chicken", false, 400, Type.MEAT),
            new Dish("french fries", true, 530, Type.OTHER),
            new Dish("rice", true, 350, Type.OTHER),
            new Dish("season fruit", true, 120, Type.OTHER),
            new Dish("pizza", true, 550, Type.OTHER),
            new Dish("prawns", false, 400, Type.FISH),
            new Dish("salmon", false, 450, Type.FISH)
    );

    public static final Map<String, List<String>> dishTags = new HashMap<>();

    static {
        dishTags.put("pork", asList("greasy", "salty"));
        dishTags.put("beef", asList("salty", "roasted"));
        dishTags.put("chicken", asList("fried", "crisp"));
        dishTags.put("french fries", asList("greasy", "fried"));
        dishTags.put("rice", asList("light", "natural"));
        dishTags.put("season fruit", asList("fresh", "natural"));
        dishTags.put("pizza", asList("tasty", "salty"));
        dishTags.put("prawns", asList("tasty", "roasted"));
        dishTags.put("salmon", asList("delicious", "fresh"));
    }
}
```

1. 요리의 개수를 구하시오.(counting)


2. 요리 중 가장 높은 칼로리를 가진 요리를 구하시오.(maxBy, comparingInt)


3. menu에 있는 요리의 총 칼로리를 구하시오.(summingInt)


4. menu에 있는 요리의 카롤리 평균을 구하시오.(averagingInt)


5. menu에 있는 요소 수, 요리의 칼로리 합계, 평균, 최댓값, 최솟값을 보여주는 IntSummaryStatistics로 반환하시오.


6. 요리의 이름을 스페이스를 구분자로 합쳐 문자열로 반환하시오.


7. 요리의 이름을 ,를 구분자로 합쳐 문자열로 반환하시오.

### 6.2.4 범용 리듀싱 요약 연산

앞의 예제들은 모두 reducing 팩토리 메서드를 통해서 구현을 할 수 있는데, 특화된 컬렉터를 앞처럼
사용하면 더 코드를 간단하게 구현할 수 있다.

reducing은 인수를 3개를 받으며 세 개의 인수는 아래와 같다.
- 첫 번째 인수: 리듀싱 연산의 시작 값 혹은 인수가 없을 땐 반환 값
- 두 번째 인수: 요소 변환 함수
- 세 번째 인수: 같은 종류!!!의 두 항목을 하나의 값으로 더하는 BinaryOperator

만약에 한 개의 인수를 갖는 경우엔 스트림의 첫 번째 요소를 시작 요소로, 두 번째 요소는
자기 자신을 반환하는 항등 함수, 세번째 요소는 받아온다.

1. 가장 칼로리가 높은 요리(reducing 버전)
```Java
Optional<Dish> mostCalorieDishReducing
        = menu.stream().collect(reducing(
                (d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

2. menu에 있는 요리의 총 칼로리 합계를 구하시오(reducing 버전)
```Java
Integer totalCalories = menu.stream().collect(reducing(0, // 초깃값
                                Dish::getCalories, // 변환함수
                                Integer::sum)); // 합계 함수
```

3. menu에 있는 음식의 개수를 구하시오.(reducing 버전)
```Java
menu.stream().collect(reducing(0L, e -> 1L , Long::sum));
```

※ 주의사항!! 

reducing 메서드는 BinaryOperator를 인수로 받는다. 아래의 코드가 잘못된 이유는 입력 타입과 출력 타입이
불일치하기 때문이다.

```Java
String shortMenu = menu.stream()
        .collect(reducing( (d1, d2) -> d1.getName() + d2.getName())).get();
```

---

## 6.3 그룹화

데이터 집합을 하나의 특성으로 그룹화하는 연산은 명령어로 구하기 정말 어렵지만 함수형 연산(Collectors.groupingBy)을 통해
쉽게 그룹화를 할 수 있다.

groupingBy는 첫 번째 인자로 분류 함수를 받고 두 번째 인자로 컬렉터를 받는다.
collectingAndThen은 첫 번쨰 인자로 컬렉터를 받고 두 번째로 Optional을 꺼낼 수 있는 변환함수를 받는다.

1. 요리 타입 별로 요리를 List로 그룹화하시오.(Map<Dish.Type, List<Dish> 형태로 출력)

출력 결과 값:
```Java
{MEAT=[pork, beef, chicken], FISH=[prawns, salmon], OTHER=[french fries, rice, season fruit, pizza]}
```

2. 요리를 칼로리 별로 분류하여 List로 그룹화하시오.(Map<CaloricLevel, List<Dish>> 형태로 출력)
```Java
// 400 칼로리 이하 DIET, 700 칼로리 이하 NORMAL, 700 칼로리 초과 FAT
public enum CaloricLevel { DIET, NORMAL, FAT }
```

출력 결과 값:
```Java
{NORMAL=[beef, french fries, pizza, salmon], DIET=[chicken, rice, season fruit, prawns], FAT=[pork]}
```

3. 요리 타입 별 태그를 추출하시오.(Map<Dish.Type, Set<String>> 형태로 출력, flatMapping활용)

출력 결과 값:
```Java
{MEAT=[salty, greasy, roasted, fried, crisp], FISH=[roasted, tasty, fresh, delicious], OTHER=[salty, greasy, natural, light, tasty, fresh, fried]}
```

4. 요리를 요리 타입 별로 분류 후, 칼로리 별로 구분하여 그룹화하시오.(Map<Dish.Type, Map<CaloricLevel, List<Dish>>>)

출력 결과 값:
```Java
{MEAT={NORMAL=[beef], DIET=[chicken], FAT=[pork]}, FISH={NORMAL=[salmon], DIET=[prawns]}, OTHER={NORMAL=[french fries, pizza], DIET=[rice, season fruit]}}
```

5. 요리 타입 별 가장 높은 칼로리를 찾으시오.(Map<Dish.Type, Dish> 형채로 출력, collectingAndThen 활용)

※ 내부에 요소가 비어있지 않아서 Optional.empty가 없어 마음대로 가져올 수 있음을 활용한다.

출력 결과 값:
```Java
{MEAT=pork, FISH=salmon, OTHER=pizza}
```

6. 요리 타입 별 메뉴에 있는 모든 요리의 칼로리 합계를 구하시오.(Map<Dish.Type, Integer> 형태로 출력)

출력 결과 값:
```Java
{MEAT=1900, FISH=850, OTHER=1550}
```

7. 요리 타입 별 존재하는 모든 CaloricLevel을 구하시오.(Map<Dish.Type, Set<CaloricLevel>> 형태로 출력)

출력 결과 값:
```Java
{MEAT=[NORMAL, DIET, FAT], FISH=[NORMAL, DIET], OTHER:=[NORMAL, DIET]}
```

