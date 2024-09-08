# 9장. 리팩터링, 테스팅, 디버깅

# 9.1 가독성과 유연성을 개선하는 리팩터링

- 람다 표현식은 익명 클래스보다 코드를 좀 더 간결하게 만들어줌
- 람다와 동작 파라미터화의 형식을 지언해서 좀더 유연성을 갖출 수 있음

## 9.1.1 코드 가독성 개선

- 가독성이 좋다 = 어떤 코드를 다른 사람도 쉽게 이해할 수 있음
- 코드 가독성을 개선한다 = 우리가 구현한 코드를 다른 사람이 쉽게 이해하고 유지보수할 수 있게 만드는 것
    - 코드의 문서화를 잘하고, 표준 코딩 규칙을 준수하는 등의 노력을 기울여야함

## 9.1.2 익명 클래스를 람다 표현식으로 리팩터링 하기

- 람다 표현식을 이용해서 간결하고 가독성이 좋은 코드를 구현할 수 있음
- 익명 클래스를 람다 표현식으로 변환할 수 있는것은 아님
    1. 익명 클래스에서 사용한 `this` 와 `super` 는 람다 표현식에서 다른 의미를 가짐

       익명 클래스의 `this` 는 자신을, 람다에서는 람다를 감싸는 클래스를 가리킴

    2. 람다 표현식으로는 변수를 가릴 수 없음
    3. 익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있음

       익명 클래스는 인스턴스화 할때 명시적으로 형식이 정해지지만 람다는 콘텍스트에 따라 달라짐


## 9.1.3 람다 표현식을 메서드 참조로 리팩터링하기

- 메서드 참조를 이용하여 가독성을 높일 수 있음
- 메서드 참조의 메서드명으로 코드의 의도를 명확하게 알릴 수 있음

```java
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel1 = menu.stream()
                .collect(groupingBy(dish -> {
                    if (dish.getCalories() <= 400) {
                        return CaloricLevel.DIET;
                    } else if (dish.getCalories() <= 700) {
                        return CaloricLevel.NORMAL;
                    } else {
                        return CaloricLevel.FAT;
                    }
                }));

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel2 = menu.stream()
                .collect(groupingBy(Dish::getCaloricLevel));
```

- 정적 헬퍼 메서드를 활용하는 것도 좋음

```java
menu.sort((Dish dish1, Dish dish2) -> dish1.getName().compareTo(dish2.getName()));

menu.sort(Comparator.comparing(Dish::getCalories));
```

- 자주 사용하는 리듀싱 연산은 메서드 참조와 함께 사용할 수 있는 내장 헬퍼 메서드 제공

```java
Integer totalCalories1 = menu.stream().map(Dish::getCalories).reduce(0, (c1, c2) -> c1 + c2);
Integer totalCalories2 = menu.stream().collect(summingInt(Dish::getCalories));
```

## 9.1.4 명령형 데이터 처리를 스트림으로 리팩터링하기

- 스트림 API 는 데이터 처리 파이프라인의 의도를 더 명확하게 보여줌
- 쇼트 서킷과 강력한 최적화, 멀티코어 아키텍쳐를 활용하게 해줌
- 명령형 코드의 여러 패턴들을 직접적으로 기술할 수 있으며 쉽게 병렬화 할 수 있음

## 9.1.5 코드 유연성 개선

- 람다를 사용하여 동작 파라미터화를 쉽게 구현할 수 있음

### 함수 인터페이스 적용

- 람다 표현식을 이용하려면 함수형 인터페이스가 필요

### 조건부 연기 실행

- 람다를 사용하여 특정 조건에서만 작업을 실행할 수 있도록 할 수 있음

### 실행 어라운드

- 매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드를 람다로 변환할 수 있음

# 9.2 람다로 객체지향 디자인 패턴 리팩터링하기

- 다양한 패턴을 유형별로 정리한 것이 디자인 패턴
- 디자인 패턴은 공통적인 소트프웨어 문제를 설계할 때 재사용 할 수 있는 검증된 청사진을 제공
- 디자인 패턴에 람다를 이용하여 문제를 더 쉽고 간단하게 해결할 수 있음

## 9.2.1 전략

- 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법
- 3부분으로 구성
    - 알고리즘을 나타내는 인터페이스
    - 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현
    - 전략 객체를 사용하는 한 개 이상의 클라이언트

```java
public class Validator {

    private final ValidationStrategy strategy;

    public Validator(ValidationStrategy strategy) {
        this.strategy = strategy;
    }

    public boolean validate(String input) {
        return strategy.execute(input);
    }
    
    Validator numericValidator = new Validator(new IsNumeric());
    boolean b1 = numericValidator.validate("aaaa");
    Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
    boolean b2 = lowerCaseValidator.validate("bbbb");
    
    //lambda
    Validator numericValidatorLambda = new Validator((String s) -> s.matches("\\d+"));
    Validator lowerCaseValidatorLambda = new Validator((String s) -> s.matches("[a-z]"));
}

interface ValidationStrategy {

    boolean execute(String s);
}

class IsAllLowerCase implements ValidationStrategy {

    @Override
    public boolean execute(String s) {
        return s.matches("[a-z]+");
    }
}

class isNumeric implements ValidationStrategy {

    @Override
    public boolean execute(String s) {
        return s.matches("\\d+");
    }
}
```

## 9.2.2 템플릿 메서드

- 알고리즘의 개요를 제시한 다음 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때 사용
    - ‘이 알고리즘을 사용하고 싶은데 그대로가 아닌 약간의 수정이 필요할때’
- 수정이 필요한 메서드에 `Consumer` 를 파라미터로 넣고 람다로 동작을 정의하여 사용


<br>
<br>