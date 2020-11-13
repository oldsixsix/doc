# Steam概述
Stream是 Java 8新增加的类，用来补充集合类。
Stream代表数据流，流中的数据元素的数量可能是有限的，也可能是无限的。
Java Stream提供了提供了串行和并行两种类型的流，保持一致的接口，提供函数式编程方式，`以管道方式提供中间操作和最终执行操作`，为Java语言的集合提供了现代语言提供的类似的高阶函数操作，`简化和提高了Java集合的功能`。
# 介绍
1. 不存储数据：流是基于数据源的对象，它本身不存储数据元素，而是通过管道将数据源的元素传递给操作。
2. 函数式编程：流的操作不会修改数据源，例如filter不会将数据源中的数据删除
3. 延迟操作： 惰性求值，流在中间处理过程中，只是对操作进行了记录，并不会立即执行，需要等到执行终止操作的时候才会进行实际的计算。
4. 可以解绑：对于无限数量的流，有些操作是可以在有限的时间完成的，比如limit(n) 或 findFirst()，这些操作可是实现"短路"(Short-circuiting)，访问到有限的元素后就可以返回。
5. 纯消费：流的元素只能访问一次，类似iterator，操作没有回头路，如果你想从头访问一遍流的元素，那必须重新生成一个流。
流的操作是以管道方式串起来的，流管道包含一个数据源，接着包含0-n个中间操作，最后包含一个终点操作结束。
# 流的常用创建方法
1.1 使用Collection下的 stream() 和 parallelStream() 方法

```java
List<String> list = new ArrayList<>();
Stream<String> stream = list.stream(); //获取一个顺序流
Stream<String> parallelStream = list.parallelStream(); //获取一个并行流
```
1.2 使用Arrays 中的 stream() 方法，将数组转成流

```java
Integer[] nums = new Integer[10];
Stream<Integer> stream = Arrays.stream(nums);
```
1.3 使用Stream中的静态方法：of()、iterate()、generate()

```java
Stream<Integer> stream = Stream.of(1,2,3,4,5,6);
 
Stream<Integer> stream2 = Stream.iterate(0, (x) -> x + 2).limit(6);
stream2.forEach(System.out::println); // 0 2 4 6 8 10
 
Stream<Double> stream3 = Stream.generate(Math::random).limit(2);
stream3.forEach(System.out::println);
```
1.4 使用 BufferedReader.lines() 方法，将每行内容转成流

```java
BufferedReader reader = new BufferedReader(new FileReader("F:\\test_stream.txt"));
Stream<String> lineStream = reader.lines();
lineStream.forEach(System.out::println);
```
1.5 使用 Pattern.splitAsStream() 方法，将字符串分隔成流

```java
Pattern pattern = Pattern.compile(",");
Stream<String> stringStream = pattern.splitAsStream("a,b,c,d");
stringStream.forEach(System.out::println);
```
# 流的中间操作
## 筛选与切片
 filter：过滤流中的某些元素
 limit skip distinct sorted 都是有状态操作，这些操作只有拿到前面处理后的所有元素之后才能继续下去。
 limit(n)：获取前n个元素
 skip(n)：跳过前n元素，配合limit(n)可实现分页
distinct：通过流中元素的 `hashCode() 和 equals()` 去除重复元素

```java
Stream<Integer> stream = Stream.of(6, 4, 6, 7, 3, 9, 8, 10, 12, 14, 14);
 
Stream<Integer> newStream = stream.filter(s -> s > 5) //6 6 7 9 8 10 12 14 14
        .distinct() //6 7 9 8 10 12 14
        .skip(2) //9 8 10 12 14
        .limit(2); //9 8
newStream.forEach(System.out::println);
```
## 映射
map：接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
flatMap：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。

```java
   List<String> list = Arrays.asList("a,b,c", "1,2,3");
//      去掉字符串中所有的,
        List<String> collect = list.stream().map(s -> s.replaceAll(",", "")).collect(Collectors.toList());
        System.out.println(collect);

//        flatMap 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。

        Stream<String> stringStream = list.stream().flatMap(s -> {
//            将字符串以,分割后得到一个字符串数组
            String[] split = s.split(",");
//            然后将每个字符串数组对应流返回，flatMap会自动把返回的所有流连接成一个流
            Stream<String> stream = Arrays.stream(split);
            return stream;
        });
        System.out.println(stringStream.collect(Collectors.toList()));
```
## 排序
 sorted()：自然排序，流中元素需实现Comparable接口
 sorted(Comparator com)：定制排序，自定义Comparator排序器  
 

```java
自然排序
//   按照color的字符串大小进行排序
   @Override
  public int compareTo(Apple o) {
     return color.compareToIgnoreCase(o.color);
  }
自定义排序
   List<Apple> compare = appleList.stream().sorted((o1, o2) -> {
            if (!o1.getColor().equals(o2.getColor())) {
//                compareToIgnoreCase 忽略大小写排序，大于返回ascll的差值，正值，小于返回负值
                return o1.getColor().compareToIgnoreCase(o2.getColor());
            } else {
//                逆序排列
                return o1.getWeight() - o2.getWeight() ;
            }
        }).collect(Collectors.toList());
        System.out.println(compare);
```
## 消费
如同于map，能得到流中的每一个元素。但map接收的是一个Function表达式，有返回值；而peek接收的是Consumer表达式，`没有返回值`。
peek接收一个没有返回值的λ表达式，可以做一些输出，外部处理等
 
# 流的终止操作
## 匹配和聚合
allmatch，noneMatch，anyMatch用于对集合中对象的某一个属性值进行判断，
allMatch全部符合该条件返回true，
noneMatch全部不符合该断言返回true
anyMatch 任意一个元素符合该断言返回true
```java

List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
boolean allMatch = list.stream().allMatch(e -> e > 10); //false
boolean noneMatch = list.stream().noneMatch(e -> e > 10); //true
boolean anyMatch = list.stream().anyMatch(e -> e > 4);  //true
```
findFirst：返回流中第一个元素
findAny：返回流中的任意元素
count：返回流中元素的总个数
max：返回流中元素最大值
min：返回流中元素最小值



