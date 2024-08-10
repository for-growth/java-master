## 6.1 컬렉터란 무엇인가?

함수형 프로그래밍의 이점 : 무엇을 원하는지 직접 명시할 수 있다는 점, 높은 수준의 조합성과 재사용성

- collect 호출 → 스트림의 요소에 리듀싱 연산 수행(요소를 방문하면서 컬렉터가 작업을 처리)
- Collectors 인터페이스의 메서드 구현하여 어떤 리듀싱 연산을 수행할지 결정

예제 - 스트림의 모든 요소를 리스트로 수집하는 정적 메서드 toList

```java
List<Transaction> transactions = 
	transactionStream.collect(Collectors.toList());
```


### 미리 정의된 컬렉터

> Collectors 클래스는 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 static factory method (굳이 new로 생성하지 않아도 됨) 제공
> 

분류

1. 스트림 요소를 하나의 값으로 리듀스하고 요약
    
    → 요소들로 계산을 수행할 때 유용
    
2. 요소 그룹화
    
    → 요소를 다수준으로 그룹화 or 그룹화 한뒤에 리듀싱 연산
    
3. 요소 분할
    
    → predicate를 그룹화 함수로 사용하여 그룹화
    

## 6.2 리듀싱과 요약


개수를 계산하여 반환하는 메서드 : **`counting`**

스트림에서 최댓값과 최솟값 검색 : **`maxBy`, `minBy`**

- 인수 : Comparator<class>
- 반환 값 : Optional<class>

```java
Comparator<Dish> dishCaloriesComparator =
	Comparator.compareingInt(Dish::getCalories);
	
Optional<Dish> mostCalorieDish = 
	menu.stream().collect(maxBy(dishCaloriesComparator));
```

특정 자료형으로 요약(합) : **`summingInt` `summingLong` `summingDouble`** 

- 인수 : 객체를 int, long, double로 매핑하는 함수
- 반환 : int로 매핑한 컬렉터를 반환 → collect메서드로 전달되며 합

```java
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```

평균 값으로 요약 : **`averagingInt` `averagingLong` `averagingDouble`**

```java
double avgCalories = 
	menu.stream.collect(averagingDouble(Dish::getCalories));
```

count, sum, min, max, average 한번에 계산 : **`summarizingInt` `summarizingLong` `summarizingDouble`**

```java
IntSummaryStatistics menuStatistics = 
	menu.stream().collect(summarizingInt);
//출력 
//IntSummaryStatistics{count = 0, sum = 00, min = 00, average = 00.000, max = 00};
```

toString으로 문자열을 연결 : **`joining`**

- 인수
    - void : 그냥 연결하여 반환
    - String : 연결 된 두 요소 사이에 string을 넣어서 반환
- 각 요소들을 toString으로 호출하여 합쳐서 반환
- 내부적으로 stringBuilder를 이용해서 문자열 생성
- 만약 toString 메서드가 있다면 추출과정 생략 가능

```java
String shortMenu = 
	menu.stream().map(Dish::getName).collect(joining());

String shortMenu = 
	menu.stream().map(Dish::getName).collect(joining(", "));
```

- 위에 컬렉터들은 reducing 메서드로도 정의할수있음 그러나 위의 방식은 프로그래밍적 편의성 + 가독성으로 이점이 있음

**collect와 reduce의 차이점**

- collect : 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드
- reduce : 두 값을 하나로 도출하는 불변형 연산

**최적의 해법**

- 하나의 연산을 다양한 방법으로 해결 가능함
- 컬렉터를 이용하는 코드가 스트림에서 직접 제공하는 메서드를 이용하는 것 보다 복잡
- 대신 재사용성과 커스터마이즈 가능성 제공
- 각 상황마다 문제를 파악하고 해당 문제에 특화된 해결책을 고르는 것이 바람직

## 6.3 그룹화

 ****그룹화 **: `groupingBy(분류함수)`**

- 인수 : 분류할 기준(분류함수라고 부름)
    - 키를 반환 → 키와, 각 키에 대응하는 리스트를 값으로 갖는 맵 반환
    - 만약 더 복잡한 분류 기준이 필요한 경우 : 람다 표현식 사용 → 예제 2

예제 1

```java
Map<Dish.Type, List<Dish>> dishesByType =
	menu.stream().collect(groupingBy(Dish::getType));

//출력
//{Fish=[prawns, salmon], Meat=[pork, beef]}
```

예제 2

```java
public enum CaloricLevel {DIET, NORMAL, FAT}

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel =
	menu.stream().collect(
		groupingBy(dish -> {
			if(dish.getCalories) <= 400) return CaloricLevel.DIET;
			else if(dish.getCalories) <= 700) return CaloricLevel.NORMAL;
			else return CaloricLevel.FAT;
		})
	)
```

그룹화 요소 조작 1 : `groupingBy(분류함수, Collector)`

- 만약 filter.groupingBy를 한다면 filter에서 걸리면 결과에서 해당 키가 사라지는 문제가 발생
    
    ex) key : meat, fish, other 인데 filter 함수에서 fish요소가 다 걸러지면 map결과에서 meat, other만 남음
    
- 두번째 인수로 predicate함수를 넣어주면 해당 문제를 해결할 수 있음

```java
//문제의 코드
Map<Dish.Type, List<Dish>> caloricDishesByType = 
	menu.stream().filter(dish -> dish.getCalories()>500).collect(groupingBy(Dish::getType))

//해결
Map<Dish.Type, List<Dish>> caloricDishesByType = 
	menu.stream().collect(groupingBy(Dish::getType, 
	filtering(dish -> dish.getCalories()>500, toList())));
```

- mapping을 이용해서도 요소 조작가능

```java
Map<Dish.Type, List<String>> dishNamesByType =
	menu.stream().collect(groupingBy(Dish::getType), mapping(Dish::getName), toList()));
```

- flatMap 변환도 가능

```java
Map<Dish.Type, List<String>> dishNamesByType =
	menu.stream().collect(groupingBy(Dish::getType), 
	flatMapping(dish->dishTags.get(dish.getName()).stream(), toSet())));
```

- 다수준 그룹화도 가능

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = 
	menu.stream().collect(
		groupingBy(Dish::getType.//첫번째 분류 함수
			if(dish.getCalories <= 400)//두번째 분류함수
				return CaloricLevel.DIET;
			else if(dish.getCalories) <= 700) 
				return CaloricLevel.NORMAL;
			else return CaloricLevel.FAT;
		)
	)
```

```java
// 요리 종류를 분류하는데 이때 각 요리에서 가장 높은 칼로리를 가진 요리를 찾는 방법
Map<Dish.Type, Optional<Dish>> mostCaloriesByType=
	menu.stream().collect(groupingBy(Dish::getType)
	, maxBy(comparingInt(Dish::getCalories)));
```

Optional에서 처리 방법 : **`collectingAndThen`** 

```java
Map<Dish.Type, Optional<Dish>> mostCaloriesByType=
	menu.stream().collect(groupingBy(Dish::getType), //분류함수
		collectingAndThen(
			maxBy(comparingInt(Dish::getCalories)),  //감싸인 컬렉터
		Optional::get)));  //변환 함수
```

- 작동방법
    1. 요리의 종류에 따라 3가지 서브스트림으로 그룹화
    2. groupingBy컬렉터 안에 collectingAndThen 컬렉터가 있음 즉 collectingAndThen 컬렉터가 3개 
    3. collectingAndThen 컬렉터는 maxBy, Optional::get 함수를 감싼다