# chapter 5 스트림 활용

&nbsp;

## 5.1 필터링

> 스트림의 요소를 선택하는 것


- **filter(predicate)** : predicate(boolean 반환하는 함수)를 인수로 받아서 일치하는 모든 요소를 포함하는 스트림 반환
- **filter().distinct()** : 고유 요소(hashcode, equals 사용)로 이루어진 스트림을 반환

예제 - filter, distinct

```java
List<Integer> numbers = Arrays.asList(1,2,1,3,3,2,4);
numbers.stream()
				.filter(i -> i % 2 == 0) //짝수이면 줍줍
				.distinct() //고유 요소만 줍줍
				.forEach(System.out::println); 
```

&nbsp;

&nbsp;

## 5.2 스트림 슬라이싱

> 스트림의 요소를 효과적으로 선택하거나 스킵하는 것
> 

**predicate 이용방법**

- **takeWhile(predicate)** : 조건 false인 요소가 나오면 반복작업 멈추고 전까지의 요소 반환
- **dropWhile(predicate)** : 조건 false인 요소가 나오면 반복작업 멈추고 이후의 요소 반환

예제 - takeWhile

```java
//칼로리가 320 미만인 dish만 가지고 싶을 때
List<Dish> sliceMenu1 
= specialMenu.stream()
							.takeWhile(dish -> dish.getCalories() < 320)
							.collect(toList());
```

예제 - dropWhile

```java
//칼로리가 320 초과인 dish만 가지고 싶을 때 
List<Dish> sliceMenu2
= specialMenu.stream()
							.dropWhile(dish -> dish.getCalories() < 320)
							.collect(toList());
```

&nbsp;

**스트림 축소**

- **limit(n)** : n 이하의 크기를 갖는 새로운 스트림 반환

예제 - limit

```java
//칼로리가 300 초과인 dish 3개만 가지고 싶을 때
List<Dish> dishes
= specialMenu.stream()
							.dropWhile(dish -> dish.getCalories() > 300)
							.limit(3)
							.collect(toList());
```

&nbsp;

**요소 건너뛰기**

- skip(n) : 처음 n개 요소를 제외한 스트림을 반환

예제 - skip

```java
//칼로리가 300 초과인 dish에서 앞에 2요소는 원치 않을 때
List<Dish> dishes
= specialMenu.stream()
							.dropWhile(dish -> dish.getCalories() > 300)
							.skip(2)
							.collect(toList());
```
&nbsp;

&nbsp;

## 5.3 매핑

> 특정 데이터를 선택하는 것
> 

**스트림의 각 요소에 함수 적용하기**

- **map()** : 인수로 제공 된 함수를 각 요소에 적용되어 새로운 요소로 매핑(이 때 값을 고치기보다 새로운 버전을 만드므로 매핑이라는 개념 사용)

예제 - map

```java
//음식의 이름의 길이를 알고 싶을 때
List<Integer> dishNameLengths 
= menu.stream()
			.map(Dish::getName)
			.map(String::length)
			.collect(toList());
```

&nbsp;

**스트림 평면화**

- **flatmap()** : 스트림의 각 값을 다른 스트림으로 만든 다음, 모든 스트림을 하나의 스트림으로 연결

예제 - flatmap

```java
String[] words = {"Hello", "World"};
//우리가 원하는 결과 : ["H", "e", "l", "o", "W", "r", "d"]

List<String> uniqueChar
= words.stream()                 //Stream<String>
				.map(words -> split("")) //Stream<String[]> H e l l o / W o r l d
				.flatMap(Arrays::stream) //Stream<String> H e l l o W o r l d
				.distinct() //Stream<String> H e l o W r d
				.collect(toList());
```

&nbsp;

&nbsp;

## 5.4 검색과 매칭

> 특정 속성이 데이터 집합에 있는지의 여부를 검색하는 것


**predicate가 적어도 한 요소와 일치하는지**

- **anyMatch(predicate)** : 일치하는 것이 있으면 true 반환 (최종연산)

예제 - anyMatch

```java
//메뉴 중에 isVegetrian이 true인 것이 있다면 해당 문구를 출력하고 싶을 떄
if(menu.stream().anyMatch(Dish::isVegetarian)){
	System.out.pritnln("The menu is Vegetarian friendly!");
}
```
&nbsp;

**predicate가 모든 요소와 일치하는지**

- **allMatch(predicate)** : 모든 요소가 일치하면 true 반환 (최종연산)
- **noneMatch(predicate) :** allMatch와 반대 연산, 일치하는 요소가 없으면 true 반환 (최종연산)

예제 - allMatch, noneMatch

```java
//모든 menu가 1000칼로리 이하인지 알고 싶을 때
boolean isHealty = menu.stream()
												.allMatch(dish -> dish.getCalories() < 1000);

boolean isHealty = menu.stream()
												.noneMatch(dish -> dish.getCalories() >= 1000);
```

- anyMatch, allMatch, noneMatch 모두 스트림 쇼트서킷 기법(자바의 &&, || 연산 활용)

&nbsp;

**요소 검색**

- **findAny() :** 현재 스트림에서 임의의 요소를 반환(최종연산)

예제 - findAny

```java
// menu에서 isVegetrian이 true인 아무 dish나 원할 때
Optional<Dish> dish
= menu.stream()
			.filter(Dish::isVegetarian)
			.findAny();p
```

&nbsp;

**첫번째 요소 찾기**

- **findFirst()** : 논리적인 아이템 순서가 정해져있을 때 첫번째 요소를 반환

예제 - findFirst

```java
//숫자 리스트에서 3으로 나누어떨어지는 첫번째 제곱값을 알고 싶을 때
List<Integer> nums = Arrays.asList(1,2,3,4,5);
Optional<Integer> firstSqaureDivisionByThree
= nums.stream()
			.map(n -> n*n)
			.filter(n -> n%3 == 0)
			.findFirst();
			//9
```

findFirst vs findAny

- 병렬실행에서는 첫번째를 찾기 힘드므로 요소의 반환 순서가 상관이 없다면 findAny