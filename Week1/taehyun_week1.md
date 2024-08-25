# 2장. 동작 파라미터화 코드 전달하기

- 동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있음
  > 동작 파라미터 : 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록
- 코드 블록은 나중에 프로그램에서 호출하므로 실행은 나중으로 미뤄짐
- 메서드를 파라미터화 시킬 수 있음

<br>

## 2.1 변화하는 요구사항에 대응하기

### 2.1.1 첫번째 시도 : 녹색 사과 필터링

- 필터링의 조건이 달라질 경우 대처하기 어려움
```java
  public List<Apple> filteringGreenApple(List<Apple> inventory) {
      List<Apple> sortingResult = new ArrayList<>();
      for (Apple apple : inventory) {
          if (apple.getColor().equals(GREEN.getColor())) {
              sortingResult.add(apple);
          }
      }
      return sortingResult;
  }
```

<br>

### 2.1.2 두번째 시도 : 색을 파라미터화

- 필터링할 색을 파라미터로 넘기면 좀더 상황에 대처할 수 있음
```java
public List<Apple> filterAppleByColor(List<Apple> inventory, Color color) {
    List<Apple> sortingResult = new ArrayList<>();
        for (Apple apple : inventory) {
            if (apple.getColor().equals(color.getColor())) {
                sortingResult.add(apple);
        }
    }
    return sortingResult;
  }
  ```
- 하지만 중복되는 코드가 많아서 좋다고 할 수 없음

<br>

### 2.1.3 세번째 시도 : 가능한 모든 속성으로 필터링

- 각 파라미터의 의미들이 모호하고 유연하게 대처할 수도 없으므로 좋지않은 방법
```java
public List<Apple> filterApples(List<Apple> inventory, Color color, int weight, Boolean flag) {
    List<Apple> sortingResult = new ArrayList<>();
        for (Apple apple : inventory) {
            if (flag && apple.getColor().equals(color.getColor()) && apple.getWeight() > weight) {
                sortingResult.add(apple);
            }
        }
    return sortingResult;
}
```

<br>

## 2.2 동작 파라미터화

```java
public interface ApplePredicate {

    boolean test(Apple apple);

}

//Predicate 함수를 재정의 -> 전략
class AppleHeavyWeightPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

//Predicate 함수를 재정의 -> 전략
class AppleGreenColorPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
        return Color.GREEN.getColor().equals(apple.getColor());
    }
}
```

- 조건에 따라 filter 메서드가 다르게 동작함 → 전략 디자인 패턴

  > 전략 디자인 패턴은 각 알고리즘을 캡슐화하여 알고리즘 패밀리를 정의해둔 다음 런타임에 알고리즘을 선택하는 기법
  알고리즘 패밀리 : `AppliPredicate`
  전략 : `AppleHeavyWeightPredicate` , `AppleGreenColorPredicate`

<br>

### 2.2.1 네번째 시도 : 추상적 조건으로 필터링

**코드 동작 전달하기**

- 메서드의 파라미터를 동작으로 정의함으로써 더 유연한 코드를 만들 수 있음
```java
static class AppleRedAndHeavyPredicate implements ApplePredicate {
  
      @Override
    public boolean test(Apple apple) {
        return GREEN.getColor().equals(apple.getColor())
            && apple.getWeight() > 150;
    }
}
  
  //...
  
    public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
        List<Apple> result = new ArrayList<>();
        for (Apple apple : inventory) {
            if (p.test(apple)) {
                result.add(apple);
            }
        }
        return result;
    }
  
  //...
  
  List<Apple> appleList = filterApples(apples, new AppleRedAndHeavyPredicate());
```
- `test` 메서드의 구현이 가장 중요함

<br>

**한개의 파라미터, 다양한 동작**

- 컬렉션 탐색 로직과 각 항목에 적용할 동작을 분리할 수 있는 것이 동작 파라미터의 강점

<br>

## 2.3 복잡한 과정 간소화

- 파라미터로 전달할 동작을 매번 정의하는 것은 번거롭고 시간이 많이 드는 작업이라 개선이 필요
- 익명 클래스와 람다 표현식으로 가독성있는 코드의 구현이 가능

<br>

### 2.3.1 익명 클래스

- 이름이 없는 클래스. 선언과 인스턴스화를 동시에 할 수 있음

<br>

### 2.3.2 다섯 번째 시도 : 익명 클래스 사용

```java
List<Apple> appleList2 = filterApples(apples, new ApplePredicate() {
    @Override
    public boolean test(Apple apple) {
        return apple.getColor().equals(GREEN.getColor());
    }
});
```

- 장황한 코드는 구현하고 유지보수하는데 시간이 오래 걸리고 가독성도 좋지 않음
- 동작 파라미터를 사용하여 요구사항 변화에 더 유연하게 대응하고 간결한 코드 전달 기법이 필요

<br>

### 2.3.3 여섯번째 시도 : 람다 표현식 사용

```java
List<Apple> appleList3 =
        filterApples(apples, apple -> apple.getColor().equals(GREEN.getColor()));
```

- 람다 표현식으로 더 간단하게 작성 가능

<br>

### 2.3.4 일곱번째 시도 : 리스트 형식으로 추상화

```java
public interface Predicate<T> {

boolean test(T t);
}

//...

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for (T e : list) {
        if (p.test(e)) {
            result.add(e);
        }
    }
    return result;
}

//...

ComparingParameter.filter(inventory.getInventory()
        ,(Apple apple)->RED.getColor().equals(apple.getColor()));
```

- 추상화를 통해 유연성과 간결함을 모두 얻을 수 있음

<br>

## 2.4 실전 예제

### 2.4.1 Comparator 로 정렬하기

- 변화하는 요구사항에 따른 다양한 역할을 수행할 수 있는 코드가 필요
- 여러가지로 정렬을 해야할 경우 Comparator 를 구체화해서 다양한 정렬을 할 수 있음

```java
inventory.getInventory().sort(new Comparator<>() {
    @Override
    public int compare (Apple o1, Apple o2){
        return o1.getWeight() - o2.getWeight();
}});

//...
        
inventory.getInventory().sort((o1, o2) ->o1.getWeight() -o2.getWeight());

```

<br>

### 2.4.2 Runnable 로 코드 블록 실행하기

- 스레드를 사용하여 병렬로 코드블록을 실행할 수 있음
- 어떤 코드를 실행할지는 Runnable 인터페이스를 이용해서 실행할 코드 블록을 지정할 수 있음

```java
public interface Runnable() {

    void run();
}

//...

Thread t = new Thread(new Runnable() {
    public void run() {
        //...
    }
})

//...

Thread t = new Thread(() -> System.out.println("Hello world"));
```

<br>

### 2.4.3 Callable 을 결과로 반환하기

- ExecutorService 인터페이스는 테스크 제출과 실행 과정의 연관성을 끊어줌
- 테스크를 쓰레트 풀로 보내고 결과를 Future 로 저장할 수 있음

```java
public interface Callable<V> {

    V call();
}

ExecutorService executorService = Executors.newCachedThreadPool();
Future<String> threadName = excutorService.submit(new Callable<String>() {
    @Override
    public String call() throws Exception {
        return Thread.currentThread().getName();
    }
})

//...

Future<String> threadName =
        excutorService.submit(() -> Thread.currentThread().getName());
```

<br>

### 2.4.4 GUI 이벤트 처리하기

- 사용자의 다양한 반응들을 처리할 수 있는 유연한 코드가 필요
- 자바 FX 의 setOnAction 메서드에 EventHandler 를 전달함으로써 이벤트에 어떻게 반응할지 설정할 수 있음

```java
Button button = new Button("Send");

button.setOnAction(new EventHandler<ActionEvent>() {
    public void handle (ActionEvent event){
        label.setText("Sent!!");
    }
})

//...

button.setOnAction((ActionEvent event) ->label.setText("Sent!!"));
```

<br>

## 2.5 마치며

- 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있으며 나중에 엔지니어링 비용을 줄일 수 있다.
- 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있다. 하지만 자바 8 이전에는 코드를 지저분하게 구현해야 했다. 익명 클래스로도 어느정도 코드를 깔끔하게 만들 수 있지만 자바 8 에서는 인터페이스를
  상속받아 여러 클래스를 구현해야하는 수고를 없앨 수 있는 방법을 제공한다.
- 자마 API 의 많은 메서드는 정렬, 스레드, GUI 처리등을 포함한 다양한 동작으로 파라미터화할 수 있다.

<br>

### 사용 예시

```java
    public List<Car> judgeWinner() {
        List<Car> participantsCars = participants.getCars();
        int maxMovedDistance = participantsCars.stream().mapToInt(Car::getMovedDistance).max().orElse(0);
        return participantsCars.stream()
                .filter(car -> car.getMovedDistance() == maxMovedDistance)      //predicate 타입 파라미터 필요
                .toList();
    }

```


<br>
<br>

