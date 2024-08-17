# Chapter 4 스트림 소개
대부분의 자바 애플리케이션은 **컬렉션**을 만들고 처리하는 과정을 포함한다

*컬렉션 : 데이터를 그룹화 & 처리*

## 스트림이란?

- 스트림을 이용하면 **선언형**(데이터를 처리하는 임시구현 코드 대신 질의로 표현)으로 컬렉션 데이터를 처리할 수 있다
- 멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다

**장점**

1. 선언형으로 코드를 구현할 수 있다 (if, for 문 같은것이 필요 X)
2. 여러 빌딩 블록 연산으로 복잡한 데이터 처리 파이프라인 만들수 있다
    - stream API에서 병렬화를 할 때 thread safty를 신경쓸 필요 X 알아서 처리해준다
    - 예제 코드
        
        ```java
        public class ParallelStreamExample {
            public static void main(String[] args) {
                List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
                // 병렬 스트림을 사용하여 각 숫자를 제곱
                List<Integer> squaredNumbers = numbers.parallelStream()
                        .map(n -> {
                            System.out.println("Thread: " + Thread.currentThread().getName() + " - Processing number: " + n);
                            return n * n;
                        })
                        .toList();
        
                System.out.println("Squared Numbers: " + squaredNumbers);
            }
        }
        ```
        
        ```bash
        Thread: ForkJoinPool.commonPool-worker-4 - Processing number: 8
        Thread: ForkJoinPool.commonPool-worker-7 - Processing number: 3
        Thread: ForkJoinPool.commonPool-worker-3 - Processing number: 2
        Thread: ForkJoinPool.commonPool-worker-4 - Processing number: 9
        Thread: ForkJoinPool.commonPool-worker-7 - Processing number: 4
        Thread: ForkJoinPool.commonPool-worker-3 - Processing number: 1
        Thread: ForkJoinPool.commonPool-worker-4 - Processing number: 6
        Thread: main - Processing number: 10
        Thread: ForkJoinPool.commonPool-worker-2 - Processing number: 7
        Thread: ForkJoinPool.commonPool-worker-1 - Processing number: 5
        Squared Numbers: [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
        ```
        
        - map을 실행할 때 thread들이 동시에 할당되어 독립적으로 각자 숫자들을 처리한다
    

**스트림의 특징**

1. 선언형 : 더 간결하고 가독성이 좋아진다
2. 조립할 수 있음 : 유연성이 좋아진다
3. 병렬화 : 성능이 좋아진다

**예제**

Stream 사용하지 않은 예제

```java
 static void nonStream(){
        List<Dish> lowCaloriesDishes = new ArrayList<>();   //컨테이너 역할만 하는 가비지 변수

        //1. 칼로리가 400보다 낮은 애들 필터링
        for(Dish dish : menu){
            if(dish.getCalories() < 400){
                lowCaloriesDishes.add(dish);
            }
        }

        //2. 칼로리 기준 정렬
        Collections.sort(lowCaloriesDishes, new Comparator<Dish>() {
            public int compare(Dish dish1, Dish dish2){
                return Integer.compare(dish1.getCalories(), dish2.getCalories());
            }
        });

        //3. 이름 할당
        List<String> lowCaloriesDishesName = new ArrayList<>(); //최종적으로 필요한 변수
        for(Dish dish : lowCaloriesDishes){
            lowCaloriesDishesName.add(dish.getName());
        }
    }
```

Stream 예제

```java
 static void stream(){
        List<String> lowCaloricDishesName =
                menu.stream()
                        .filter(dish -> dish.getCalories() < 400)    //1. 칼로리가 400보다 낮은 애들 필터링
                        .sorted(Comparator.comparing(Dish::getCalories)) //2. 칼로리 기준 정렬
                        .map(Dish::getName) //3. 이름 할당
                        .toList();

    }
```

## 스트림 시작하기

> 📃 **데이터 처리 연산**을 지원하도록 **소스**에서 추출된 **연속된 요소**

- **연속된 요소** : 특정 요소 형식으로 이루어진 연속된 값 집합
- **소스** : 데이터 제공 소스로 부터 데이터를 **소비**
- **데이터 처리 연산** : filter, map, reduce, find, match ..

**특징**

- 파이프라이닝
    - 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환
- 내부반복
    - 반복문을 이용하는  컬렉션과 달리 스트림은 내부반복을 지원

**예제**

```java
    static void stream2(){
        List<String> threeHighCaloricDishesName =
                menu.stream()   //메뉴에서 스트림을 얻는다
                        .filter(dish -> dish.getCalories() > 300)   //고칼로리 요리를 필터링
                        .map(Dish::getName) //요리명 추출
                        .limit(3)   //선착순 세개만 선택
                        .collect(toList());  //결과를 리스트로 저장

        System.out.println(threeHighCaloricDishesName);
    }
```

- 데이터 소스 : menu
- 연속된 요소를 stream에 제공
- 데이터 처리 연산 : filter, map, limit, collect
    - collect 제외한 모든 연산은 파이프라인을 형성할 수 있도록 스트림 반환
- 연산
    - filter : 람다를 인수로 받아 스트림에서 특정 요소 제외
    - map : 람다를 이용해서 한 요소를 다른 요소로 변환 or 정보 추출
    - limit : 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기 축소
    - collect : 스트림을 다른 형식으로 변환

### 스트림과 컬렉션

1. **데이터를 언제 계산하느냐**

**스트림**

- 요청할 때만 요소를  계산하는 고정된 자료구조
- 사용자가 요청하는 값만 스트림에서 추출
- 생산자 - 소비자 관계
- 게으르게 만들어지는 컬렉션

**컬렉션**

- 현재 자료구조가 포함하는 모든 값을 메모리에 저장
- 적극적으로 생성
- 생산자 중심

스트림은 반복문과 동일하게 한번만 탐색할 수 있음

1. **데이터 반복을 어떻게 처리하느냐**

**스트림 :** 내부반복

- 알아서 처리하고 결과를 어딘가 저장

**컬렉션 :** 외부반복 

- 사용자가 직접 요소를 반복(for - each)
- 병렬성을 개발자가 관리해야함

내부반복 시 : 작업을 투명하게 병렬로 처리, 더 최적화된 다양한 순서로 처리할 수 있음 

⇒ 우리가 신경쓰지 않아도 된다

### 스트림 연산

```java
        List<String> threeHighCaloricDishesName =
                menu.stream()   //메뉴에서 스트림을 얻는다
                        .filter(dish -> dish.getCalories() > 300)   //고칼로리 요리를 필터링
                        .map(Dish::getName) //요리명 추출
                        .limit(3)   //선착순 세개만 선택
                        .toList();  //결과를 리스트로 저장
```

**중간연산** 

- filter, map, limit
- 연결할 수 있는 스트림 연산
- 중간 연산을 다른 스트림을 반환, 얘만 있으면 어떠한 결과도 얻을 수 없음

**최종연산** 

- collect
- 스트림을 닫는 연산, 결과를 반환하는 연산

**이렇게 구분하는 이유?**

- 중간 연산의 특징 : **lazy**
- 중간 연산을 합친 다음, 합쳐진 중간 연산을 최종연산으로 한번에 계산한다
    - 이런 특성 덕분에 최적화가능
    - filter → map → limit가 아니라 1번째 fiter → map 2번째 fiter → map 3번째 filter map 하고 최종 연산 수행 (쇼트 서킷 + 루프 퓨전)

**스트림 이용과정**

1. 질의를 수행할 데이터 소스
2. 스트림 파이프라인을 구성할 중간 연산 연결
3. 스트림 파이프라인을 실행하고 결과를 만들 최종연산