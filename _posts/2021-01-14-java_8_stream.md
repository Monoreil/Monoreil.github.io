---
title:  "java-8-stream"
excerpt: "java 8 stream 정리."

categories:
  - Blog
tags:
  - Blog
last_modified_at: 2021-01-14T15-08
---

스트림이란?
JDK 8에서 추가된 Stream API에 대해서 알아보자.(파일 I/O에서 사용되는 스트림과는 다르다.)스트림은 데이터소스를 추상화하고, 데이터를 다루는데 자주 사용되는 메서들을 정의해 놓았다. 데이터소스를 추상화하였다는 것은, 데이터 소스가 무엇이든 같은 방식으로 다룰 수 있게 되었다는 것과 코드의 재사용성이 높아진다는 것을 의미한다. 먼저, 코드를 보면서 차이를 느껴보자.
{% highlight ruby %}
//Stream 사용 전
String[] strArr = { "mash-up", "backend", "codingsquid" }
List<String> strList = Arrays.asList(strArr);

Arrays.sort(strArr);
Collections.sort(strList);

for(String str: strArr) {
  System.out.println(str);
}

for(String str : strList) {
  System.out.println(str);
}

//Stream 사용 후
Stream<String> listStream = strList.stream();
Stream<String> arrayStream = Arrays.stream(strArr);

listStream.sorted().forEach(System.out::println);
arrayStream.sorted().forEach(System.out::println);
{% endhighlight %}

위의 코드는 배열, 리스트를 정렬 후 출력하는 코드이다. 사용전과 사용후의 코드 간결성과 추상화가 잘 돼있다는 것을 느낄 수 있다.(스트림의 sort함수를 호출한다고 해서 List, Array가 정렬되는 것은 아님에 주의하자. 단지, 스트림의 요소가 정렬되어 출력되는 것이다.)

스트림의 특징
스트림은 데이터 소스를 변경하지 않는다.
스트림은 데이터 소스로 부터 데이터를 읽기만할 뿐, 데이터 소스를 변경하지 않는다. 필요하다면, 정렬된 결과를 컬렉션이나 배열에 담아서 반환할 수 있다.
{% highlight ruby %}
List<String> sortedList = listStream.sorted().collect(Collections.toList());
{% endhighlight %}
스트림은 일회용이다.
스트림은 Iterator처럼 일회용이다. Iterator로 컬렉션의 요소를 모두 읽고 나면 다시 사용할 수 없는 것처럼, 스트림도 한번 사용하면 닫혀서 다시 사용할 수 없다. 필요하다면 스트림을 다시 생성해야한다.
{% highlight ruby %}
listStream.sorted().forEach(System.out::print);
int numOfElement = listStream.count(); //에러. 스트림이 이미 닫힘
{% endhighlight %}
스트림은 작업을 내부 반복으로 처리한다.
스트림을 이용한 작업이 간결할 수 있는 비결중의 하나가 바로 '내부 반복'이다. 내부 반복이라는 것은 반복문을 메서드의 내부에 숨길 수 있다는 것을 의미한다.
스트림의 연산
스트림이 제공하는 다양한 연산을 이용해서 복잡한 작업들을 간단히 처리할 수 있다. 스트림에 정의된 메서드 중에서 데이터 소스를 다루는 작업을 수행하는 것을 연산(operation)이라고 한다.

스트림이 제공하는 연산은 중간 연산과 최종 연산으로 분류할 수 있다.

중간 연산: 연산결과를 스트림으로 반환하기 때문에 중간 연산을 연속해서 연결할 수 있다.
핵심 메서드: map(), flatMap()
최종 연산: 스트림의 요소를 소모하면서 연산을 수행하기 때문에 단 한번만 연산이 가능하다.
핵심 메서드: reduce(), collect()
{% highlight ruby %}
stream.distinct()
      .limit(5)
      .sorted()
      .forEach(System.out::print);
{% endhighlight %}
병렬 스트림
병렬 스트림은 내부적으로 fork & join 프레임웍을 이용해서 자동적으로 연산을 병렬로 수행한다. 우리가 할일이라고는 그저 스트름에 parallel()이라는 메서드를 호출해서 병렬로 연산을 수행하도록 지시하면 된다. 병렬로 처리되지 않게 하려면 sequential()을 호출하면 된다. 모든 스트림은 기본적으로 병렬 스트림이 아니므로 sequential()을 호출할 필요가 없다. 이 메서드는 parallel()을 호출한 것을 취소할 때만 사용한다.
{% highlight ruby %}
int sum = strStream.parallel()
                   .mapToInt(s -> s.length())
                   .sum();
{% endhighlight %}
스트림 생성
스트림으로 작업을 하려면, 스트림을 생성해야한다. 스트림의 소스가 될 수 있는 대상은 배열, 컬렉션, 임의의 수, 파일 등 다양하며, 이 다양한 소스들로부터 스트림을 생성하는 방법에 대해서 알아보자.

컬렉션
컬렉션의 최고 조상인 Collection에 stream()이 정의되어 있다. Collection의 자손인 List와 Set을 구현한 컬렉션 클래스들은 모두 이 메서드로 스트림을 생성할 수 있다.
stream()은 해당 컬렉션을 소스(source)로 하는 스트림을 반환한다.
{% highlight ruby %}
Stream<T> Collection.stream();

//List로부터 스트림을 생성하는 코드
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> listStream = list.stream();
{% endhighlight %}
forEach()는 지정된 작업을 스트림의 모든 요소에 대해 수행한다.
{% highlight ruby %}
//스트림의 모든 요소를 출력
listStream.forEach(System.out::print);
{% endhighlight %}
배열
배열을 소스로 하는 스트림을 생성하는 메서드는 다음과 같이 Stream과 Arrays에 static메서드로 정의되어 있다.
{% highlight ruby %}
//Stream의 static 메서드
Stream<T> Stream.of(T... values); //가변인자
Stream<T> Stream.of(T[]);
//Arrays의 static 메서드
Stream<T> Arrays.stream(T[]);
Stream<T> Arrays.stream(T[] array, int startInclusive, int endExclusive);
//문자열 스트림 생성 코드
Stream<String> strStream = Stream.of("a", "b", "c"); //가변인자
Stream<String> strStream = Stream.of(new String[] {"a", "b", "c"});
Stream<String> strStream = Arrays.stream(new String[] {"a", "b", "c"});
Stream<String> strStream = Arrays.stream(new String[] {"a", "b", "c"}, 0, 3); //end범위 포함 x
{% endhighlight %}
스트림의 핵심 메서드
스트림의 연산은 중간연산과 최종연산으로 나누어 진다고 하였다. 다양한 연산들이 이 있지만 우리는 map(), flatMap(), reduce(), collect() 네개의 핵심 연산만 알아보자. (다른 연산들은 stream api를 확인하자.)

중간연산
map(): 스트림의 요소에 저장된 값 중에서 원하는 필드로만 뽑아내거나 특정 형태로 변환해야 할 때 사용된다. 이 메서드의 선언부는 아래와 같으며, 매개변수로 T타입을 R타입으로 변환해서 반환하는 함수를 지정해야한다.
{% highlight ruby %}
Stream<R> map(Function<? super T, ? extends R> mapper);
{% endhighlight %}
예를 들어 File의 스트림에서 파일의 이름만 뽑아서 출력하고 싶을 때, 아래와 같이 map()을 이용하면 File객체에서 파일의 이름(String)만 간단히 뽑아낼 수 있다.
{% highlight ruby %}
Stream<File> fileStream = Stream.of(new File("Ex1.java"), new File("Ex1.txt"), new File("Ex2.java"));
//map()으로 Stream<File>을 Stream<String>으로 변환
Stream<String> fileNameStream = fileStream.map(File::getName);
fileNameStream.forEach(System.out:print);
{% endhighlight %}
flatMap(): 스트림의 요소가 배열이거나 map()의 연산결과가 배열인 경우, 즉 스트림의 타입이 Stream<T[]>인 경우, Stream로 다루는 것이 더 편리할 때가 있다. 그럴 때는 map()대신 flatMap()을 사용하면 된다.(Stream<T[]> -> Stream로 변환)
{% highlight ruby %}
Stream<String[]> strStream = Stream.of(
                                new String[] {"a", "b", "c"}),
                                new String[] {"d", "e", "f"});
{% endhighlight %}
위와 같이 요소가 문자열 배열(String[])인 스트림을 각 요소의 문자열들을 합쳐서 문자열이 요소인 스트림, 즉 Stream으로 만들어보자. 일단 위에서 배운 map()를 이용한다면
{% highlight ruby %}
Stream<Stream<String>> strStrStream = strArrStream.map(Arrays::stream);
{% endhighlight %}
예상한 것과 달리, Stream이 아닌 Stream<Stream>으로 변환 되었다. 각 요소의 문자열들이 합쳐지지 않고, 스트림의 스트림 형태로 되어버렸다. 이 때, 간단히 map()을 아래와 같이 flatMap()으로 바꾸기만 하면 우리가 원하는 결과를 얻을 수 있다.
{% highlight ruby %}
Stream<String> strStream = strArrStream.flatMap(Arrays:stream);
{% endhighlight %}
즉, flatMap()은 map()과 달리 스트림의 스트림이 아닌 스트림으로 만들어 준다.
최종연산: 스트림의 요소를 소모해서 결과를 만들어낸다. 따라서 최종 연산후에는 스트림이 닫히게 되고 더 이상 사용할 수 없다.
reduce(): 스트림의 요소를 줄여나가면서 연산을 수행하고 최종결과를 반환한다. 처음 두 요소를 가지고 연산한 결과를 가지고 그 다음 요소와 연산한다. 이 과정에서 스트림의 요소를 하나씩 소모하게 되며, 스트림의 모든 요소를 소모하게 되면 그 결과를 반환한다.
{% highlight ruby %}
Optional<T> reduce(BinaryOperator<T> accumulator);
List<Integer> intList = new ArrayList<>();
intList.add(1);
intList.add(12);
intList.add(15);
intList.add(7);
intList.add(8);
intList.add(9);
intList.add(10);
Stream<Integer> intStream = intList.stream();
System.out.println(intStream.reduce(Integer::max)
                              .get()
                              .toString());
{% endhighlight %}
이 외에도 연산결과의 초기값을 갖는 reduce()도 있는데, 이 메서드들은 초기값과 스트림의 첫 번째 요소로 연산을 시작한다. 이 부분은 여기서 다루지않겠다. api를 확인해보자.
collect(): 스트림의 요소를 수집하는 최종 연산으로 앞서 나온 reduce()와 유사하다. collect()가 스트림의 요소를 수집하려면, 어떻게 수집할 것인가에 대한 방법이 정의되어 있어야 하는데, 이방법을 정의한 것이 바로 컬렉터(collector)이다. 컬렉터는 Collector인터페이스를 구현한 것으로, 직접 구현할 수도 있고 미리 작성된 것을 사용할 수도 있다. Collectors클래스는 미리작성된 다양한 종류의 컬렉터를 반환하는 static메서드를 가지고 있다.
{% highlight ruby %}
collect() : 스트림의 최종연산, 매개변수로 컬렉터를 필요로 한다.
Collector : 인터페이스, 컬렉터는 이 인터페이스를 구현해야한다.
Collectors : 클래스, static메서드로 미리 작성된 컬렉터를 제공한다.
{% endhighlight %}
collect()는 매개변수의 타입이 Collector인데, 매개변수가 Collector를 구현한 클래스의 객체이어야 한다는 뜻이다. 이 객체에 구현된 방법대로 스트림의 요소를 수집한다.
{% highlight ruby %}
Object collect(Collector collector);
{% endhighlight %}
스트림을 컬렉션과 배열로 변환
{% highlight ruby %}
//Student 클래스가 정의, 스트림이 생성되어있다고 가정하자.
List<String> names = studentStream.map(Student:getName)
                                  .collect(Collectors.toList());
ArrayList<String> list = names.stream()
                              .collect(Collectors.toCollection(ArrayList:new));
{% endhighlight %}
이외에도 스트림을 컨트롤 할 수 있는 많은 메서드들이 있다. 여기서 소개하기는 너무 많으니 개념을 익힌 후 api를 확인해보자.