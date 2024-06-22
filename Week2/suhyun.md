### 기본

- 람다식을 사용하려면, 메소드를 하나(=추상메소드) 가진 인터페이스가 필요하다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/110001d7-b5e4-4360-903f-1a3250b75d15/5dee60e2-dfc7-4b4b-ac7f-3718eaa4877a/Untitled.png)

- A인터페이스를 구현한 클래스를 통해, A인터페이스 객체를 생성할 수 있다.
- A인터페이스를 구현한 클래스를 만들지 않고도, 익명내부클래스를 통해 A인터페이스 객체를 생성할 수 있다.
- 람다표현식으로도 A인터페이스 객체를 생성할 수 있다.
    
    ```java
    A a3 = () -> {
    	System.out.println("난 람다식이네!");
    }
    ```
    
- 아래 코드는 모두 동일한 결과를 갖는다.

```java
a1.m();
a2.m();
a3.m();
```

- 추상 메소드는 총 4가지의 종류를 갖는다. 즉, 람다식으로 사용할 수 있는 메소드의 형태는 총 4가지이다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/110001d7-b5e4-4360-903f-1a3250b75d15/87dade24-c710-420e-8b42-632acde4f9c7/Untitled.png)

- 람다식을 통해 위 4가지 인터페이스에 대한 객체를 모두 생성해보자.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/110001d7-b5e4-4360-903f-1a3250b75d15/938bb905-87af-4583-a90a-b940f5d5df55/Untitled.png)

- 첫번째 경우(파라미터가 없고, 구현 부분이 한줄인 경우), 아래처럼 간소화할 수 있다.

```java
A a = () -> System.out.println("A");

```

- 두번째 경우(파라미터가 한개이고, 구현 부분이 한줄인 경우), 아래처럼 간소화할 수 있다.

```java
B b = str -> System.out.println(str);
```

- 세번째 경우(파라미터가 없고, 구현 부분이 return으로 이루어진 한줄인 경우), 아래처럼 간소화할 수 있다.

```java
C c = () -> "C";
```

- 네번째 경우(파라미터가 있고, 구현 부분이 return으로 이루어진 한줄인 경우), 아래처럼 간소화할 수 있다. 파라미터가 2개인경우는 괄호 생략 불가

```java
D d = (x, y) -> x + y;
```

- 또한, 람다식을 사용한 인터페이스에는 아래처럼 `@FuntionalInterface` 를 붙여준다. 이렇게 하면, 이 인터페이스에는 추상메소드 외에 다른 메소드를 추가할 수 없다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/110001d7-b5e4-4360-903f-1a3250b75d15/577fc320-ce3d-40fe-8744-410d8cf687cb/Untitled.png)

---

### FunctionalInterface (java.util.function패키지 내 존재하는 FunctionalInterface)

- 간략하게 몇몇 FunctionalInterface를 살펴보도록 하겠다.

| Interface | 추상 메소드 형태 |
| --- | --- |
| Consumer | 매개변수 O, 리턴값 X |
| Supplier | 매개변수 X, 리턴값 O |
| Function | 매개변수 O, 리턴값 O |
| Operator | 매개변수 O, 리턴값 O |
| Predicate | 매개변수 O, 리턴값 O |
- 람다식을 사용할 메소드의 형태가 리턴값이 x, 매개변수가 하나면, Consumer라고 끝나는 인터페이스 중 하나를 선택하면 된다.

```java
Consumer<Integer> consumer = x -> System.out.println(x);
consumer.accept(5);

-- 결과
5출력
```

- 람다식을 사용할 메소드의 형태가 리턴값이 O, 매개변수가 하나면, Supplier라고 끝나는 인터페이스 중 하나를 선택하면 된다.

```java
Supplier<Integer> supplier = () -> new Random().nextInt(10);
System.out.println(supplier.get());

-- 결과
0부터 9사이의 숫자가 랜덤하게 출력
```

---

### FunctionalInterface의 추상메소드가 내가 구현한 람다식과 동일한 기능을 한다면?

 

```java
class A implements IntConsumer{

	static void mA(int x){
		System.out.println(x);
	}
}
```

```java
IntConsumer intConsumer = x -> System.out.println(x);
```

IntConsumer인터페이스에 x를 출력하는 추상메소드가 있다고 해보자.

또, 내가 구현한 람다식은 IntConsumer의 추상메소드와 동일한 기능이라고 가정하자.

만약 그렇다면, 아래처럼 간소화할 수 있다.

```java
IntCousumer intConsumer = x -> A.mA(x);
```

이를 한번 더 간소화할 수 있다.