java8 in action 5장

스트림 활용

### 이 장의 내용

> 필터링, 슬라이싱, 매칭
>
> 검색, 매칭, 리듀싱
>
> 특정 범위의 숫자와 같은 숫자 스트림 사용하기
>
> 다중 소스로부터 스트림 만들기
>
> 무한 스트림



```java
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
  public String toString() {
    return super.toString();
  }

  public enum Type {
    MEAT, FISH, OTHER
  }
}


List<Dish> menu = Arrays.asList(
  new Dish("pork", false, 800, Dish.Type.MEAT),
  new Dish("beef", false, 700, Dish.Type.MEAT),
  new Dish("chicken", false, 400, Dish.Type.MEAT),
  new Dish("french fries", true, 530, Dish.Type.OTHER),
  new Dish("rice", true, 350, Dish.Type.OTHER),
  new Dish("season fruit", true, 120, Dish.Type.OTHER),
  new Dish("pizza", true, 550, Dish.Type.OTHER),
  new Dish("prawns", false, 300, Dish.Type.FISH),
  new Dish("salmon", false, 450, Dish.Type.FISH)
);
```

[Stream interface](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)



## 5.1 필터링과 슬라이싱

### 5.1.1 Predicate(boolean을 반환하는 함수)로 필터링

`filter` 메서드를 지원한다. 

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| Stream<T>         | **filter**(Predicate<? super T> predicate) <br>Returns a stream consisting of the elements of this stream that match the given predicate. |

```java
List<Dish> vegetarianMenu = menu.stream()
                .filter(Dish::isVegetarian)     // dish -> dish.isVegetarian()
                .collect(toList());
```



### 5.1.2 고유 요소 필터링

고유 요소로 이루어진 스트림을 반환하는 `distinct` 메서드를 지원한다.  (고유의 여부는 객체의 hashcode, equals로 결정)

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| Stream<T>         | **distinct()**<br>Returns a stream consisting of the distinct elements (according to[`Object.equals(Object)`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#equals-java.lang.Object-)) of this stream. |

~~~java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
       .filter(i -> i % 2 == 0)
       .distinct()
       .forEach(System.out::println);
~~~



### 5.1.3 스트림 축소

`limit` 메서드를 지원한다.

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| Stream<T>         | **limit**(long maxSize)<br>Returns a stream consisting of the elements of this stream, truncated to be no longer than `maxSize` in length. |

```java
// NUMBERS 1~45 캐싱
public Lotto makeLotto() {
	Collections.shuffle(NUMBERS);
    return new Lotto(NUMBERS.stream()
            				.limit(LOTTO_SIZE)	// LOTTO_SIZE == 6
            				.collect(Collectors.toSet()));
}
```



### 5.1.4 요소 건너뛰기

`skip` 메서드를 지원한다.

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| Stream<T>         | **skip**(long n)<br>Returns a stream consisting of the remaining elements of this stream after discarding the first `n` elements of the stream. |

```java
List<Dish> dishes = menu.stream()
                        .filter(dish -> dish.getCalories() > 300)
                        .skip(2)
                        .collect(toList());
```



## 5.2 매핑

특정 객체에서 특정 데이터를 선택하는 작업은 데이터 처리 과정에서 자주 수행되는 연산. 예를 들어 SQL의 테이블에서 특정 열만 선택할 수 있다. 스트림 API의 `map`과 `flatMap` 메서드는 특정 데이터를 선택하는 기능을 제공한다.

### 5.2.1 스트림의 각 요소에 함수 적용하기

`map` 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑.

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| <R> Stream<R>     | **map**(**Function**<? super **T**,? extends R> mapper)<br>Returns a stream consisting of the results of applying the given function to the elements of this stream. |

```java
List<String> dishNames = menu.stream()
                              .map(Dish::getName)	//dish -> dish.getName()
                              .collect(toList());

List<Integer> dishNameLengths = menu.stream()
                                    .map(Dish::getName)
                                    .map(String::length)
                                    .collect(toList());
```



### 5.2.2 스트림 평면화

`flatMap` 스트림이 아닌 스트림의 콘텐츠로 매핑.

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| <R> Stream<R>     | **flatMap**([Function](https://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html)<? super [T](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html),? extends [Stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)<? extends R>> mapper)<br>Returns a stream consisting of the results of replacing each element of this stream with the contents of a mapped stream produced by applying the provided mapping function to each element. |

```java
String[] arrayOfWords = {"Goodbye", "World"};
List<String> uniqueCharacters = Arrays.stream(arrayOfWords) 		//Stream<String>
                                        .map(s -> s.split("")) 		//Stream<String[]>
                                        .flatMap(Arrays::stream) 	//Stream<String>
                                        .distinct() 				//Stream<String>
                                        .collect(toList());
```



## 5.3 검색과 매칭

### 5.3.1 프레디케이트가 적어도 한 요소와 일치하는지 확인

`anyMatch` 프레디케이트가 주어진 스트림에서 적어도 한 요소와 일치하는지 확인할 때 사용

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| boolean           | **anyMatch**([Predicate](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html)<? super [T](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)> predicate)<br>Returns whether any elements of this stream match the provided predicate. |

```java
if (menu.stream().anyMatch(Dish::isVegetarian)) {
    
}    
```



### 5.3.2 프레디케이트가 모든 요소와 일치하는지 검사

`allMatch` 스트림에서 요소가 모두 일치하는지 확인할 때 사용

 `noneMacth` 스트림에서 요소가 모두 일치하는 요소가 없는지 확인할 때 사용

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| boolean           | **allMatch**([Predicate](https://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html)<? super [T](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)> predicate)<br>Returns whether all elements of this stream match the provided predicate. |
| boolean           | **noneMatch**(Predicate<? super T> predicate)<br>Returns whether no elements of this stream match the provided predicate. |

```java
if (menu.stream().allMatch(dish -> dish.getCalories() < 1000)) {
            
}

if (menu.stream().noneMatch(Dish::isVegetarian)) {
            
}
```

### 5.3.3 요소 검사

`findAny` 현재 스트림에서 임의의 요소를 반환

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| Optional<T>       | **findAny**()<br>Returns an [`Optional`](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) describing some element of the stream, or an empty **Optional** if the stream is empty. |

```java
Optional<Dish> dish = menu.stream()
                            .filter(Dish::isVegetarian)
                            .findAny();
```

#### Optional이란?

값이 존재하는지 확인하고 값이 없을 때 어떻게 처리할 것인지 강제하는 기능을 제공한다.

### 5.3.4 첫 번째 요소 찾기

`findFirst` 스트림에는 논리적인 아이템 순서가 정해져 있을수 있다. 스트림에서 첫번째 요소를 반환

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| Optional<T>       | **findFirst**()<br/>Returns an [`Optional`](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) describing the first element of this stream, or an empty **Optional** if the stream is empty. |

```java
List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
                                                            .map(i -> i * i)
                                                            .filter(i -> i % 3 == 0)
                                                            .findFirst();
```



#### findFirst vs findAny

병렬 실행에서는 첫 번째 요소를 찾기가 힘들다. 요소의 반환 순서가 상관없다면 병렬 스트림에서는 제약이 적은 `findAny` 를 사용한다.



## 5.4 리듀싱

스트림 요소를 조합해서 더 복잡한 질의를 표현하는 방법. 결과가 나올 대까지 스트림의 모든 요소를 반복적으로 처리. 이러한 연산을 **리듀싱 연산**이라고 한다.



### 5.4.1 요소의 합

| Modifier and Type | Method and Description                                       |
| ----------------- | ------------------------------------------------------------ |
| Optional<T>       | **reduce**([**BinaryOperator**](https://docs.oracle.com/javase/8/docs/api/java/util/function/BinaryOperator.html) <[**T**]> accumulator) |
| T                 | **reduce**(**T** identity, **BinaryOperator**<T> accumulator) |

```java
int sum = 0;
for (int x : numbers) {
    sum += x;
}

int sum = numbers.stream().reduce(0, (a, b) -> a + b);
// 0 초기값

Optional<Integer> sum = numbers.stream().reduce((a, b) -> a + b);
// 초기값 없음
// 스트림에 아무 요소도 없는 상황에서는 초기값이 없으므로 결과 반환이 불가능
// 합계가 없음을 가리킬 수 있도록 Optional 객체로 감싼 결과를 반환
```

### 5.4.2 최댓값과 최솟값

```java
int max = numbers.stream().reduce(0, Integer::max);
int min = numbers.stream().reduce(0, Integer::min);

Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

#### 

## 5.5 실전 연습

1. 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하시오.
2. 거래자가 근무하는 모든 도시를 중복 없이 나열하시오.
3. 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오.
4. 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오.
5. 밀라노에 거래자가 있는가?
6. 케임브리지에 거주하는 거래자의 모든 트랜잭션값을 출력하시오.
7. 전체 트랜잭션 중 최댓값은 얼마인가?
8. 전체 트랜잭션 중 최솟값은 얼마인가?

### 5.5.1 거래자와 트랜잭션

```java
public class Trader {
  private final String name;
  private final String city;

  public Trader(String name, String city) {
    this.name = name;
    this.city = city;
  }

  public String getName() {
    return this.name;
  }

  public String getCity() {
    return this.city;
  }

  public String toString() {
    return "Trader:" + this.name + " in " + this.city;
  }
}

public class Transaction {
  private final Trader trader;
  private final int year;
  private final int value;

  public Transaction(Trader trader, int year, int value) {
    this.trader = trader;
    this.year = year;
    this.value = value;
  }

  public Trader getTrader() {
    return this.trader;
  }

  public int getYear() {
    return this.year;
  }

  public int getValue() {
    return this.value;
  }

  public String toString() {
    return "{" + this.trader + ", " +
            "year: " + this.year + ", " +
            "value: " + this.value + "}";
  }
}

Trader raoul = new Trader("Raoul", "Cambridge");
Trader mario = new Trader("Mario","Milan");
Trader alan = new Trader("Alan","Cambridge");
Trader brian = new Trader("Brian","Cambridge");
```

### 5.5.2 실전 연습 정답

1. 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하시오.

    ```java
    
    ```

2. 거래자가 근무하는 모든 도시를 중복 없이 나열하시오.

    ```java
    
    ```

3. 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬하시오.

    ```java
    
    ```

4. 모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오.

    ```java
    
    ```

5. 밀라노에 거래자가 있는가?

    ```java
    
    ```

6. 케임브리지에 거주하는 거래자의 모든 트랜잭션값을 출력하시오.

    ```java
    
    ```

7. 전체 트랜잭션 중 최댓값은 얼마인가?

    ```java
    
    ```

8. 전체 트랜잭션 중 최솟값은 얼마인가?

    ```java
    
    ```

## 5.6 숫자형 스트림

### 5.6.1 기본형 특화 스트림

#### 특정 스트림으로 매핑

`IntStream` int 요소에 특화

`DoubleStream` double 요소에 특화

`LongStream` long 요소에 특화

```java
// 불가능
int calories = menu.stream()
                    .map(Dish::getCalories)	//Stream<String>
                    .sum();	// sum() 메서드 존재 X

int calories = menu.stream()
                    .mapToInt(Dish::getCalories)	// IntStream
                    .sum();
```



#### 객체 스트림으로 복원

`boxed` 특화 스트림을 일반 스트림으로 반환

```java
Stream<Integer> calories = menu.stream()
                .mapToInt(Dish::getCalories)
                .collect(toList());	//불가능

Stream<Integer> calories = menu.stream()
                                .mapToInt(Dish::getCalories)
                                .boxed();
```



### 5.6.2 숫자 범위

`IntStream`

`LongStream`

```java
IntStream.rangeClosed(1, 100).forEachOrdered(System.out::println);
LongStream.rangeClosed(1, 100).forEachOrdered(System.out::println);

IntStream.rangeClosed(MINIMUM_LOTTO_NUMBER, MAXIMUM_LOTTO_NUMBER)	// min = 1, max = 45
                .forEachOrdered(value -> LOTTO_NUMBERS.put(value, new LottoNumber(value)));
```



## 5.7 스트림 만들기

### 5.7.1 값으로 스트림 만들기

`Stream.of`

```java
Stream<String> stream = Stream.of("java8", "Lambdas", "in", "Action");
```



### 5.7.2 배열로 스트림 만들기

`Arrays.stream`

```java
int[] numbers = {1, 2, 3, 4, 5};
int sum = Arrays.stream(numbers).sum;
```



### 5.7.3 파일로 스트림 만들기

```java
long uniqueWords = 0;
        
try (Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())){
    uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
                                                .distinct()
                                                .count();
} catch (IOException e) {
    e.printStackTrace();
}
```



### 5.7.4 함수로 무한 스트림 만들기

#### iterate

`iterate` 기존 결과에 의존해서 순차적으로 연산을 수행. 끝이 없으므로 무한 스트림을 생성

| Modifier and Type        | Method and Description                                       |
| ------------------------ | ------------------------------------------------------------ |
| static <T> **Stream**<T> | **iterate**(T seed, [UnaryOperator](https://docs.oracle.com/javase/8/docs/api/java/util/function/UnaryOperator.html)<T> f) |



```java
Stream.iterate(0, n -> n + 2)
    .limit(10)
    .forEach(System.out::println);
// 0 -> 2 -> 4 -> ...
```



#### generate

`generate` iterate와 달리 값을 연속적으로 계산하지 않는다.

| Modifier and Type        | Method and Description                                       |
| ------------------------ | ------------------------------------------------------------ |
| static <T> **Stream**<T> | **generate**([Supplier](https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html)<T> s) |



```java
Stream.generate(Math::random)
       .limit(5)
       .forEach(System.out::println);
```

