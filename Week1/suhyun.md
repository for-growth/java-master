
# Ch01. 동적 파라미터화 코드 전달

### 배경
- 소프트웨어 개발에서 요구사항은 항상 변한다.
- 이러한 요구사항을 반영하면서도 엔지니어링적인 비용이 가장 최소화될 수 있으면 좋다.
- 그뿐 아니라 새로 추가한 기능은 쉽게 구현할 수 있어야하며 장기적인 관점에서 유지보수가 쉬워야한다.

---

### 변화하는 요구사항에 대응하기
- 첫번째 시도 - 녹색사과만 필터링
    - 기존의 농장 재고목록 애플리케이션 리스트에서 녹색 사과만 필터링하는 기능을 추가한다고 가정하자.

```java
enum Color { RED, GREEN }
```

```java
public List<Apple> filterGreenApples(List<Apple> inventory) {
    final var result = new ArrayList<Apple>();
    for (final var apple : inventory) {
        if (GREEN.equals(apple.color())) {
            result.add(apple);
        }
    }
    return result;
}
```
- 현재의 요구사항은 녹색사과만 필터링하는 것이지만 빨간색 사과만 필터링하는 요구사항이 올 수도 있다.


- 두번째 시도 - 색을 파라미터화
```java
public List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
	final var result = new ArrayList<Apple>();
	for (final var apple : inventory) {
		if (color.equals(apple.color())) {
			result.add(apple);
		}
	}
	return result;
}
```
- 여기서 색 이외에 무게를 이용해서 사과를 필터링하고 싶다는 요구사항이 생기면 어떻게 해야할까?


- 세번째 시도 - 가능한 모든 속성을 필터링
```java
public List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    final var result = new ArrayList<Apple>();
    for (final var apple : inventory) {
        if (apple.weight() > weight) {
            result.add(apple);
        }
    }
    return result;
}
```
- 위의 코드에서는 **색**과 **무게** 중 어떤 기준에 따라 필터링할지를 결정하기 위해 boolean 타입의 **flag**라는 파라미터를 사용했는데, 이 메서드를 사용하는 사용자 입장에서 어떤 것이 **true**이고 어떤 것이 **false**인지 불명확하다.
- 게다가 **색**과 **무게** 이외에 `Apple` 클래스에 또 다른 속성(크기, 모양, 출하지 등)이 부여된다면 현재의 코드로는 유연하게 대처할 수 없다.
- 심지어 **빨간색 사과 중에 무거운 사과**를 필터링하고 싶다면? 결국 중복된 메서드를 계속해서 만들 수 밖에 없다.

---

### 동작 파라미터화(Behavior Parameterization)
- 우리는 선택 조건을 다음처럼 결정할 수 있다.
- 사과의 어떤 속성에 기초해서 불리언값을 반환 (예를 들어 사과가 녹색인가? 200그램 이상인가?)하는 방법이 있다.
- 참 또는 거짓을 반환하는 함수를 **프레디케이트**라고 한다.
- **선택 조건을 결정하는 인터페이스**를 정의하자.

```java
public interface ApplePredicate {
	boolean test(Apple apple);
}
```
- 다음과 같이 다양한 선택 조건을 대표하는 ApplePredicate를 정의할 수 있다.

```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
      @Override
      public boolean test(Apple apple) {
          return apple.weight() > 200;
      }
  }
```

```java
public class AppleGreenColorPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return GREEN.equals(apple.color());
    }
}
```
- `filterApples`에서 `ApplePredicate` 객체를 받아 애플의 조건을 검사하도록 메서드를 고쳐야 한다.
    - 이렇게 **동작 파라미터화**, 즉 메서드가 **다양한 동작(또는 전략)을 받아서** 내부적으로 다양한 **동작**을 수행할 수 있다.
    - 이렇게 하면 `filterApples` 메서드 내부에서 컬렉션을 반복하는 로직과 컬렉션의 각 요소에 적용할 동작(우리 예제에서는 프레디케이트)을 분리할 수 있다는 점에서 소프트웨어 엔지니어링적으로 큰 이득을 얻는다.
- 이제 세 번째 시도 마지막 부분에서 언급했던 **빨간색 사과 중에 무거운 사과**를 다음과 같이 만들 수 있다.

```java
public class AppleRedAndHeavyPredicate implements ApplePredicate {
    @Override
    public boolean test(Apple apple) {
        return RED.equals(apple.color()) && apple.weight() > 180;
    }
}
```
- 네 번째 시도 : 추상적 조건으로 필터링
```java
public List<Apple> filterApples(List<Apple> inventory, ApplePredicate applePredicate) {
    final var result = new ArrayList<Apple>();
    for (final var apple : inventory) {
        if (applePredicate.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}
```
<img src="/Week1/suhyun-img/Behavior%20Parameterization.png">

---
### 복잡한 과정 간소화 → 1. 익명클래스 , 2. 람다
- 지금까지의 작업에서 `filterApples` 메서드로 새로운 동작을 전달하려면 `ApplePredicate` 인터페이스를 구현하는 여러 클래스를 정의한 다음에 인스턴스화해야 한다.
- 자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 **익명 클래스(anonymous class)** 라는 기법을 제공한다.
  - **익명 클래스**는 자바의 지역 클래스(local class, 블럭 내부에 선언된 클래스)와 비슷한 개념이다.
  - 익명 클래스는 말 그대로 이름이 없는 클래스다.
  - 익명 클래스를 이용하면 클래스 선언과 인스턴스화를 동시에 할 수 있다.
  - 즉, 즉석에서 필요한 구현을 만들어서 사용할 수 있다.



- 다섯 번째 시도 : 익명 클래스 사용
```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate(){
  public boolean test(Apple apple){
    return RED.equals(apple.getColor());
  }
});
```

- 여섯번째 시도 - 람다
```java
  List<Apple> result = filterApples(inventory, (Apple apple) ->
  RED.equals(apple.getColor()));
```