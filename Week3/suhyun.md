# 스트림 소개
거의 모든 자바 애플리케이션은 컬렉션을 만들고 처리하는 과정을 포함합니다. 대부분의 프로그래밍 작업에 사용되며, 컬렉션으로 데이터를 그룹화하고 처리할 수 있습니다. 비즈니스 로직상 컬렉션에 대해 특정 카테고리로 그룹화 하던가, 특정 키워드를 사용하여 원하는 결과를 찾는 연산을 요구하는 작업이 있을 수 있습니다. 대부분 데이터베이스에서는 선언형으로 이와 같은 연산을 표현할 수 있습니다. 예를 들어 ‘SELECT name FROM cars WHERE price < 2500’이라는 문장같이 2500 이하인 차량을 선택하라는 SQL질의를 만들 수 있습니다.

이처럼 자동차의 속성을 이용하여 어떻게 필터링 할 것인지는 구현할 필요가 없습니다. 어떻게 구현해야 할지 명시할 필요가 없고 구현은 자동으로 제공됩니다.



컬렉션으로도 이와 비슷한 기능을 만들 수 있지 않을까?  많은 요소를 포함하는 커다란 컬렉션은 어떻게 처리해야 할까요?

→ 이런 문제를 해결하기 위해, 프로그래머가 더 편하게 개발을 하기 위해 만들어 진 것이 스트림입니다.







## 스트림이란 무엇인가?
자바 8 API에 추가된 기능으로 스트림을 이용하면 선언형으로 컬렉션 데이터를 처리할 수 있습니다. 또한 스트림을 이용하면 멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있습니다. 우선은 병렬 처리는 7장에서 설명하고 우선 스트림이 어떤 유용한 기능을 제공하는지 알아봅시다.


```java
package streamExample;

import java.util.Arrays;
import java.util.List;

import static streamExample.Car.Type.*;

public class Car {

    private final String name;
    private final int price;
    private final Type type;

    public Car(String name, int price, Type type) {
        this.name = name;
        this.price = price;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public int getPrice() {
        return price;
    }

    public Type getType() {
        return type;
    }

    @Override
    public String toString() {
        return name;
    }

    public enum Type{
        GASOLINE,
        DIESEL,
        ELECTRIC,
        HYBRID
        }

    public static final List<Car> carList = Arrays.asList(
        new Car("k3", 1800, GASOLINE),
        new Car("sonata", 2700, HYBRID),
        new Car("gv70", 6000, GASOLINE),
        new Car("g80", 8000, GASOLINE),
        new Car("benzC", 6400, DIESEL),
        new Car("benzE", 8700, GASOLINE),
        new Car("avante", 2700, GASOLINE),
        new Car("k5", 2800, HYBRID),
        new Car("TUCSON", 3000, DIESEL)
    );
}
    List<Car> lowPriceCars = new ArrayList<>();
        for (Car car : Car.carList) {
            if (car.getPrice() < 3000) {
                lowPriceCars.add(car);
        }
    }

    Collections.sort(lowPriceCars, new Comparator<Car>() {
        @Override
        public int compare(Car o1, Car o2) {
            return o1.getPrice() - o2.getPrice();
        }
    });

    List<String> lowPriceCarsName = new ArrayList<>();
    for (Car car : lowPriceCars) {
        lowPriceCarsName.add(car.getName());
    }
```



위 코드에서는 lowPirceCars라는 ‘가비지 변수’ 즉 중간 컨테이너 역할을 하는 중간 변수이다. 자바 8에서 이러한 세부 구현은 라이브러리 내에서 모두 처리한다.

List<String> lowPriceCarsName = Car.carList.stream()
.filter(c -> c.getPrice() < 3000)   //3000미만 자동차 선택
.sorted(Comparator.comparing(Car::getPrice)) //가격순으로 정렬
.map(Car::getName)    //자동차 이름 추출
.collect(Collectors.toList()); //모든 자동차 이름을 리스트에 저장


스트림의 새로운 기능이 소프트웨어공학적으로 다음의 다양한 이득을 줄 수 있습니다.

선언형으로 코드를 구현할 수 있다. 어떻게 동작을 구현할지 지정할 필요 없이 ‘특정 가격이하의 자동차를 선택해라’ 동작의 수행을 지정할 수 있다. 선언형 코드와 동작 파라미터화를 활용하면 변화하는 요구에도 쉽게 대응이 가능하다.
filter, sorted, map, collect 같은 여러 빌딩 블록 연산을 연결해서 복잡한 데이터 처리 파이프라인을 만들 수 있습니다. 여러 연산을 파이프라인으로 연결해도 여전히 가독성과 명확성을 유지할 수 있습니다.
filter (sorted, map, collect) 같은 연산은 고수준 빌딩 블록 으로 이루어져 있으므로 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서든 사용할 수 있습니다.

또한 이들은 내부적으로 단일 스레드 모델에 사용할 수 있지만 멀티코어 아키텍처를 최대한 투명하게 활용할 수 있게 구현되어 있습니다. 결과적으로 우리는 스트림 API를 통해 데이터 처리 과정을 병렬화화면서 스레드와 락을 걱정할 필요할 없습니다.



자바 8 스트림의 API 특징을 다음처럼 요약할 수 있습니다.

- 선언형 : 더 간결하고 가독성이 좋아진다.
- 조립가능 : 유연성이 좋아진다.
- 병렬화 : 성능이 좋아진다.


## 스트림 시작하기
간단한 스트림 작업인 컬렉션 스트림 부터 살펴봅시다. 자바 8 컬렉션에는 스트림을 반환하는 stream 메서드가 추가됐습니다.

*스트림이란 정확이 뭘까요?

→ 스트림이란 ‘데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소' 로 정의할 수 있습니다.

이 정의를 하나씩 살펴 보겠습니다.

연속된 요소
컬렉션과 마찬가지로 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공합니다. 컬렉션은 자료구조이므로 컬렉션에서는 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룹니다. 

반면 스트림은 filter, sorted, map 처럼 표현 계산식이 주를 이룹니다. 즉, 컬렉션의 주제는 데이터이고 스트림의 주제는 계산입니다.
소스
스트림은 컬렉션, 배열, IO 자원 등의 데이터 제공 소스로부터 데이터를 소비합니다. 정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지된다. 즉, 리스트로 스트림을 만들면 스트림의 요소는 리스트의 요소와 같은 순서를 유지합니다.
데이터 처리 연산
스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원합니다. ex) filter, map, reduce, find, match . . . 등 으로 데이터를 조작할 수 있습니다. 스트림 연산은 순차적 또는 병렬로 실행 가능합니다.
또한 스트림에는 두 가지 중요 특성이 있습니다.

파이프라이닝: 대부분 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환합니다. 그 덕분에 게으름(laziness), 쇼트서킷(short-circuiting) 같은 최적화도 얻을 수 있습니다. 연산 파이프라인은 데이터 소스에 적용하는 데이터베이스 질의와 비슷합니다.
내부 반복: 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복을 지원합니다.

```java
List<String> twoHighPriceCarNames =
Car.carList.stream()
.filter(car -> car.getPrice() > 5000)
.map(Car::getName)
.limit(2)
.collect(Collectors.toList());

System.out.println(twoHighPriceCarNames);
```

데이터 소스는 차량 리스트 이고, 데이터 소스는 연속된 요소를 스트림에 제공합니다. 다음으로 스트림의 filter, map, limiit, collect 로 이어지는 일련의 데이터 처리 연산을 적용하고 collect를 제외한 모든 연산은 파이프라인을 형성할 수 있도록 스트림을 반환합니다. 마지막으로 collect 연산으로 파이프라인을 처리해서 결과를 반환합니다.

마지막 collect를 호출하기 전까지는 carList에서 무엇도 선택되지 않으며 출력 결과도 없습니다. 즉 collect가 호출되기 전까지 메서드 호출이 저장되는 효과가 있습니다.

- filter : 람다를 인수로 받아 스트림에서 특정 요소를 제외시킨다.

- map : 람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보를 추출합니다.

- limit : 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기를 축소합니다.

- collect : 스트림을 다른 형식으로 변환합니다.




스트림과 컬렉션
자바의 기존 컬렉션과, 스트림 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공합니다.

여기서 연속된 이라는 표현은 순서와 상관없이 아무 값에나 접근하는 것이 아니라 순차적으로 값에 접근한다는 것을 의미합니다.





시각적으로 차이를 한번 알아봅시다.




데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이입니다. 컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조입니다. 그래서 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 합니다.



스트림은 이론적으로 요청할 때만 요소를 계산 하는 고정된 자료구조 입니다.(스트림에 요소를 추가/제거 불가) 이러한 스트림의 특성은 프로그래밍에 큰 도움을 줍니다. 요청할 때만 요소를 계산하기 때문에 사용자가 요청하는 값만 스트림에서 추출한다는 것이 핵심입니다.

즉, 스트림은 게으르게 만들어지는 컬렉션과 같습니다.









딱 한 번만 탐색할 수 있다.
반복자와 마찬가지로 스트림도 한 번만 탐색이 가능합니다. 탐색된 스트림의 요소는 소비되며 반복자와 마찬가지로 한 번 탐색한 요소를 다시 탐색하려면 초기 데이터 소스에서 새로운 스트림을 만들어야합니다.

*만약 데이터 소스가 I/O 채널이라면 소스를 반복사용 할 수 없어 새로운 스트림을 만들 수 없다.



## 외부 반복과 내부 반복
컬렉션 인터페이스를 사용하려면 사용자가 명시적으로 ‘직접 요소를 반복해야 합니다.’ 이를 외부 반복 이라고 합니다.

반면에 스트림 라이브러리는 반복을 알아서 처리하고 결과 스트림값을 어딘가 저장해주는 내부 반복을 사용합니다. 함수에 어떤 작업을 수행할지만 지정하면 모든 것이 알아서 처리가 됩니다.

//외부 반복
List<String> names = new ArrayList<>();
for (Car car : Car.carList) { //메뉴 리스트를 명시적으로 순차 반복
names.add(car.getName());

}

//내부 반복
List<String> name = Car.carList.stream()
.map(Car::getName)  //map 메서드를 getNAme 메서드로 파라미터화하여 추출
.collect(Collectors.toList());



스트림은 내부 반복을 사용하므로 반복 과정을 신경 쓰지 않아도 된다. 하지만 이와 같은 이점을 누리려면 filter, map 같은 반복을 숨겨주는 연산 리스트가 미리 정의되어 있어야 한다. 반복을 숨겨주는 대부분의 연산은 람다 표현식을 인수로 받으므로 동작 파라미터화를 활용할 수 있다.









## 스트림 연산
List<String> twoHighPriceCarNames =
Car.carList.stream()  //요리 리스트에서 스트림 얻기
.filter(car -> car.getPrice() > 5000) //중간 연산
.map(Car::getName) //중간 연산
.limit(2) //중간 연산
.collect(Collectors.toList()); //스트림을 리스트로 변환
스트림 인터페이스의 연산을 크게 두가지로 구분할 수 있습니다.

filter, map, limit는 서로 연결되어 파이프라인을 형성한다.
collect로 파이프라인을 실행한 다음에 닫는다.
연결할 수 있는 스트림 연산을 중간 연산이라고 하며, 스트림을 닫는 연산을 최종 연산이라고 한다. 왜 스트림의 연산을 두 가지로 구분할까요?











중간 연산
filter나 sorted 같은 중간 연산은 다른 스트림을 반환한다. 따라서 여러 중간 연산을 연결해서 질의를 만들 수 있다. 중간 연산의 중요한 특징은 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다는 것, 즉 lazy 하다는 것입니다. 중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한번에 처리하기 때문입니다.



*중간 연산만으로는 결과를 생성할 수 없다.









최종 연산
최종 연산은 스트림 파이프라인에서 결과를 도출한다. 보통 최종 연산에 의해 List, Integer, void 등 스트림 이외의 결과가 반환된다. 예를 들어 다음 파이프라인에서 forEach는 소스의 각 요리에 람다를 적용한 다음에 void를 반환하는 최종 연산입니다.

ex) Car.carList.stream().forEach(System.out::println);











## 스트림 이용하기
스트림 이용 과정은 다음과 같이 세 가지로 요약할 수 있다.

질의를 수행할 (컬렉션 같은) 데이터 소스
스트림 파이프라인을 구성할 중간 연산 연결
스트림 파이프라인을 실행하고 결과를 만들 최종 연산


중간 연산

연산	형식	반환 형식	연산의 인수	함수 디스크립터
filter	중간 연산	Stream<T>	Predicate<T>	T → boolean
map	중간 연산	Stream<R>	Function<T, R>	T → R
limit	중간 연산	Stream<T>	 	 
sorted	중간 연산	Stream<T>	Comparator<T>	(T, T) → int
distinct	중간 연산	Stream<T>




최종 연산

연산	형식	반환 형식	목적
forEach	최종 연산	void	스트림의 각 요소를 소비하면서 람다를 적용한다.
count	최종 연산	long (generic)	스트림의 요소 개수를 반홚나다.
collect	최종 연산	 	스트림을 리듀스해서 리스트, 맵, 정수 형식의 컬렉션을 만든다.






## 마무리
- 스트림은 소스에서 추출된 연속 요소로, 데이터 처리 연산을 지원한다.
- 스트림은 내부 반복을 지원한다. 내부 반복은 filter, map, sorted 등의 연산으로 반복을 추상화한다.
- 스트림에는 중간 연산과 최종 연산이 있다.
- 중간 연산을 이용해서 파이프라인을 구성할 수 있지만 어떤 결과도 생성할 수 없다.
- 스트림이 아닌 결과를 반환하는 연산을 최종 연산이라고 한다.
- 스트림의 요소는 요청할 때 lazily 하게 계산된다