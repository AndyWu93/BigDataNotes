- [文档1](https://blog.csdn.net/weixin_45225595/article/details/106203264)
- [文档2](https://blog.csdn.net/unique_perfect/article/details/110739222)
- [文档3](https://blog.csdn.net/weixin_45496190/article/details/106970254)
- [文档4](https://blog.csdn.net/weixin_45496190/article/details/106964719)

## Lambda表达式
- Lambda 表达式在Java 语言中引入了一个新的语法元素和操作符。这个操作符为 “->” ， 该操作符被称为 Lambda 操作符或剪头操作符。它将 Lambda 分为两个部分：
  - 左侧：指定了 Lambda 表达式需要的所有参数
  - 右侧：指定了 Lambda 体，即 Lambda 表达式要执行的功能。

### 语法格式：
- 无参数，无返回值：() -> sout
```java

```

- 有一个参数，无返回值
```java
@Test
public void test03(){
    Consumer<String> consumer = (a) -> System.out.println(a);
    consumer.accept("我觉得还行！");
}
```

- 有一个参数，无返回值 （小括号可以省略不写）
```java
@Test
public void test03(){
    Consumer<String> consumer = a -> System.out.println(a);
    consumer.accept("我觉得还行！");
}
```

- 有两个及以上的参数，有返回值，并且 Lambda 体中有多条语句
```java
@Test
public void test04(){
    Comparator<Integer> comparator = (a, b) -> {
        System.out.println("比较接口");
        return Integer.compare(a, b);
    };
}
```

## 函数式接口：
- 接口中只有一个抽象方法的接口 @FunctionalIterface
```java
// 定义一个函数式接口：
@FunctionalInterface
public interface MyFun {
    Integer count(Integer a, Integer b);
}
```

- Java内置四大核心函数式接口

|函数式接口 |	参数类型 |	返回类型 |	用途|
|-|-|-|-|
|Consumer       | 消费型接口	| T	 | void    |	对类型为T的对象应用操作：void accept(T t)                       |
|Supplier       | 提供型接口  |	无 |	T	     | 返回类型为T的对象：T get()                                       |
|Function<T, R> | 函数型接口  |	T  |	R      |	对类型为T的对象应用操作，并返回结果为R类型的对象：R apply(T t)       |
|Predicate      | 断言型接口	| T	 | boolean |	确定类型为T的对象是否满足某约束，并返回boolean值：boolean test(T t) |

- 消费型接口
```java
@Test
public void test01(){
    //Consumer
    Consumer<Integer> consumer = (x) -> System.out.println("消费型接口" + x);
    //test
    consumer.accept(100);
}
```

- 提供型接口
```java
@Test
public void test02(){
    List<Integer> list = new ArrayList<>();
    List<Integer> integers = Arrays.asList(1,2,3); 
    list.addAll(integers);
    //Supplier<T>
    Supplier<Integer> supplier = () -> (int)(Math.random() * 10);
    list.add(supplier.get());
    System.out.println(supplier);
    for (Integer integer : list) {
        System.out.println(integer);
    }
}
```

- 函数型接口
```java
@Test
public void test03(){
    //Function<T, R>
    String oldStr = "abc123456xyz";
    Function<String, String> function = (s) -> s.substring(1, s.length()-1);
    //test
    System.out.println(function.apply(oldStr));
}
```

- 断言型接口
```java
@Test
public void test04(){
    //Predicate<T>
    Integer age = 35;
    Predicate<Integer> predicate = (i) -> i >= 35;
    if (predicate.test(age)){
        System.out.println("你该退休了");
    } else {
        System.out.println("我觉得还OK啦");
    }
}
```

## 引用
### 1 方法引用
- 对象 :: 实例方法
```java
@Test
public void test01(){
    PrintStream ps = System.out;
    Consumer<String> con1 = (s) -> ps.println(s);
    con1.accept("aaa");

    Consumer<String> con2 = ps::println;
    con2.accept("bbb");
}
```
> Lambda 表达实体中调用方法的参数列表、返回类型必须和函数式接口中抽象方法保持一致

- 类 :: 静态方法
```java
@Test
public void test02(){
    Comparator<Integer> com1 = (x, y) -> Integer.compare(x, y);
    System.out.println(com1.compare(1, 2));

    Comparator<Integer> com2 = Integer::compare;
    System.out.println(com2.compare(2, 1));
}
```

- 类 :: 实例方法
```java
@Test
public void test03(){
    BiPredicate<String, String> bp1 = (x, y) -> x.equals(y);
    System.out.println(bp1.test("a","b"));

    BiPredicate<String, String> bp2 = String::equals;
    System.out.println(bp2.test("c","c"));
}
```
> Lambda 参数列表中的第一个参数是方法的调用者，第二个参数是方法的参数时，才能使用 ClassName :: Method

### 2 构造器引用
- ClassName :: new
```java
@Test
public void test04(){
    Supplier<List> sup1 = () -> new ArrayList();

    Supplier<List> sup2 = ArrayList::new;
}
```
> 需要调用的构造器的参数列表要与函数时接口中抽象方法的参数列表保持一致

### 3 数组引用
- Type :: new;

## Stream API
- Stream不会自己存储数据
- Stream不会改变源对象
- Stream是lazy

### 1 创建流方法
```java
/**
* 创建流
*/
@Test
public void test01(){
    /**
    * 集合流
    *  - Collection.stream() 穿行流
    *  - Collection.parallelStream() 并行流
    */
    List<String> list = new ArrayList<>();
    Stream<String> stream1 = list.stream();

    //数组流
    //Arrays.stream(array)
    String[] strings = new String[10];
    Stream<String> stream2 = Arrays.stream(strings);

    //Stream 静态方法
    //Stream.of(...)
    Stream<Integer> stream3 = Stream.of(1, 2, 3);

    //无限流
    //迭代
    Stream<Integer> stream4 = Stream.iterate(0, (i) -> ++i+i++);
    stream4.forEach(System.out::println);

    //生成
    Stream.generate(() -> Math.random())
        .limit(5)
        .forEach(System.out::println);
}
```

### 2 筛选 / 切片
- filter：接收 Lambda ，从流中排除某些元素
- limit：截断流，使其元素不超过给定数量
- skip(n)：跳过元素，返回一个舍弃了前n个元素的流；若流中元素不足n个，则返回一个空流；与 limit(n) 互补
- distinct：筛选，通过流所生成的 hashCode() 与 equals() 取除重复元素
```java
List<Employee> emps = Arrays.asList(
    new Employee(101, "Z3", 19, 9999.99),
    new Employee(102, "L4", 20, 7777.77),
    new Employee(103, "W5", 35, 6666.66),
    new Employee(104, "Tom", 44, 1111.11),
    new Employee(105, "Jerry", 60, 4444.44)
);

@Test
public void test01(){
    emps.stream()
        .filter((x) -> x.getAge() > 35)
        .limit(3) //短路？达到满足不再内部迭代
        .distinct()
        .skip(1)
        .forEach(System.out::println);
}
```

### 3 映射
- map：接收 Lambda ，将元素转换为其他形式或提取信息；接受一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素
```java
@Test
public void test02(){
    List<String> list = Arrays.asList("a", "b", "c");
    list.stream()
        .map((str) -> str.toUpperCase())
        .forEach(System.out::println);
}
```

- flatMap：接收一个函数作为参数，将流中每一个值都换成另一个流，然后把所有流重新连接成一个流
```java
public Stream<Character> filterCharacter(String str){
    List<Character> list = new ArrayList<>();
    for (char c : str.toCharArray()) {
        list.add(c);
    }
    return list.stream();
}

@Test
public void test03(){
    List<String> list = Arrays.asList("a", "b", "c");
    Test02 test02 = new Test02();
    list.stream()
        .flatMap(test02::filterCharacter)
        .forEach(System.out::println);
}
```

### 4 排序
- sorted()：自然排序(Comparable)
```java
@Test
public void test04(){
    List<Integer> list = Arrays.asList(1,2,3,4,5);
    list.stream()
        .sorted() //comparaTo()
        .forEach(System.out::println);
}
```

- sorted(Comparator c)：定制排序
```java
@Test
public void test05(){
    emps.stream()
        .sorted((e1, e2) -> { //compara()
            if (e1.getAge().equals(e2.getAge())){
                return e1.getName().compareTo(e2.getName());
            } else {
                return e1.getAge().compareTo(e2.getAge());
            }
        })
        .forEach(System.out::println);
}
```

### 5 查找 / 匹配 (终止操作)
- allMatch：检查是否匹配所有元素
- anyMatch：检查是否至少匹配一个元素
- noneMatch：检查是否没有匹配所有元素
- findFirst：返回第一个元素
- findAny：返回当前流中的任意元素
- count：返回流中元素的总个数
- max：返回流中最大值
- min：返回流中最小值
```java
public enum Status {
    FREE, BUSY, VOCATION;
}

@Test
public void test01(){
    List<Status> list = Arrays.asList(Status.FREE, Status.BUSY, Status.VOCATION);

    boolean flag1 = list.stream()
        .allMatch((s) -> s.equals(Status.BUSY));
    System.out.println(flag1);

    boolean flag2 = list.stream()
        .anyMatch((s) -> s.equals(Status.BUSY));
    System.out.println(flag2);

    boolean flag3 = list.stream()
        .noneMatch((s) -> s.equals(Status.BUSY));
    System.out.println(flag3);

    // 避免空指针异常
    Optional<Status> op1 = list.stream()
        .findFirst();
    // 如果Optional为空 找一个替代的对象
    Status s1 = op1.orElse(Status.BUSY);
    System.out.println(s1);

    Optional<Status> op2 = list.stream()
        .findAny();
    System.out.println(op2);

    long count = list.stream()
        .count();
    System.out.println(count);
}
```

### 6 归约 / 收集
- 归约：reduce(T identity, BinaryOperator) / reduce(BinaryOperator) 可以将流中的数据反复结合起来，得到一个值
```java
/**
* Java：
*  - reduce：需提供默认值（初始值）
* Kotlin：
*  - fold：不需要默认值（初始值）
*  - reduce：需提供默认值（初始值）
*/
@Test
public void test01(){
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
    Integer integer = list.stream()
        .reduce(0, (x, y) -> x + y);
    System.out.println(integer);
}
```

- 收集：collect 将流转换成其他形式；接收一个 Collector 接口的实现，用于给流中元素做汇总的方法
```java
List<Employee> emps = Arrays.asList(
    new Employee(101, "Z3", 19, 9999.99),
    new Employee(102, "L4", 20, 7777.77),
    new Employee(103, "W5", 35, 6666.66),
    new Employee(104, "Tom", 44, 1111.11),
    new Employee(105, "Jerry", 60, 4444.44)
);

@Test
public void test02(){
    //放入List
    List<String> list = emps.stream()
        .map(Employee::getName)
        .collect(Collectors.toList()); 
    list.forEach(System.out::println);
    
	//放入Set
    Set<String> set = emps.stream()
        .map(Employee::getName)
        .collect(Collectors.toSet());
    set.forEach(System.out::println);

    //放入LinkedHashSet
    LinkedHashSet<String> linkedHashSet = emps.stream()
        .map(Employee::getName)
        .collect(Collectors.toCollection(LinkedHashSet::new));
    linkedHashSet.forEach(System.out::println);
}

@Test
public void test03(){
    //总数
    Long count = emps.stream()
        .collect(Collectors.counting());
    System.out.println(count);

    //平均值
    Double avg = emps.stream()
        .collect(Collectors.averagingDouble(Employee::getSalary));
    System.out.println(avg);

    //总和
    Double sum = emps.stream()
        .collect(Collectors.summingDouble(Employee::getSalary));
    System.out.println(sum);

    //最大值
    Optional<Employee> max = emps.stream()
        .collect(Collectors.maxBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));
    System.out.println(max.get());

    //最小值
    Optional<Double> min = emps.stream()
        .map(Employee::getSalary)
        .collect(Collectors.minBy(Double::compare));
    System.out.println(min.get());
}

@Test
public void test04(){
    //分组
    Map<Integer, List<Employee>> map = emps.stream()
        .collect(Collectors.groupingBy(Employee::getId));
    System.out.println(map);

    //多级分组
    Map<Integer, Map<String, List<Employee>>> mapMap = emps.stream()
        .collect(Collectors.groupingBy(Employee::getId, Collectors.groupingBy((e) -> {
            if (e.getAge() > 35) {
                return "开除";
            } else {
                return "继续加班";
            }
        })));
    System.out.println(mapMap);
    
    //分区
    Map<Boolean, List<Employee>> listMap = emps.stream()
        .collect(Collectors.partitioningBy((e) -> e.getSalary() > 4321));
    System.out.println(listMap);
}

@Test
public void test05(){
    //总结
    DoubleSummaryStatistics dss = emps.stream()
        .collect(Collectors.summarizingDouble(Employee::getSalary));
    System.out.println(dss.getMax());
    System.out.println(dss.getMin());
    System.out.println(dss.getSum());
    System.out.println(dss.getCount());
    System.out.println(dss.getAverage());
    
    //连接
    String str = emps.stream()
        .map(Employee::getName)
        .collect(Collectors.joining("-")); //可传入分隔符
    System.out.println(str);
}
```


## Optional
- Optional 类 (java.util.Optional) 是一个容器类，代表一个值存在或不存在，原来用 null 表示一个值不存在，现在用 Optional 可以更好的表达这个概念；并且可以避免空指针异常

- 常用方法：
  - Optional.of(T t)：创建一个 Optional 实例
  - Optional.empty(T t)：创建一个空的 Optional 实例
  - Optional.ofNullable(T t)：若 t 不为 null，创建 Optional 实例，否则空实例
  - isPresent()：判断是否包含某值
  - orElse(T t)：如果调用对象包含值，返回该值，否则返回 t
  - orElseGet(Supplier s)：如果调用对象包含值，返回该值，否则返回 s 获取的值
  - map(Function f)：如果有值对其处理，并返回处理后的 Optional，否则返回 Optional.empty()
  - flatmap(Function mapper)：与 map 相似，要求返回值必须是 Optional

```java
@Test
public void test01(){
    Optional<Employee> op = Optional.of(new Employee());
    Employee employee = op.get();
}
```

```java
@Test
public void test02(){
    Optional<Employee> op = Optional.empty();
    Employee employee = op.get();
}
```

```java
@Test
public void test03(){
    Optional<Employee> op = Optional.ofNullable(new Employee());
    Employee employee = op.get();
}
```

```java
@Test
public void test03(){
    Optional<Employee> op = Optional.ofNullable(new Employee());
    if (op.isPresent()) {
        Employee employee = op.get();
    }
}
```


## Date / Time API
### LocalDateTime / LocalDate / LocalTime
```java
@Test
public void test01(){
    //获取当前时间日期 now
    LocalDateTime ldt1 = LocalDateTime.now();
    System.out.println(ldt1);

    //指定时间日期 of
    LocalDateTime ldt2 = LocalDateTime.of(2020, 05, 17, 16, 24, 33);
    System.out.println(ldt2);

    //加 plus
    LocalDateTime ldt3 = ldt2.plusYears(2);
    System.out.println(ldt3);

    //减 minus
    LocalDateTime ldt4 = ldt2.minusMonths(3);
    System.out.println(ldt4);

    //获取指定的你年月日时分秒... get
    System.out.println(ldt2.getDayOfYear());
    System.out.println(ldt2.getHour());
    System.out.println(ldt2.getSecond());
}
```

### 时间戳
- Instant：以 Unix 元年 1970-01-01 00:00:00 到某个时间之间的毫秒值
```java
@Test
public void test02(){
    // 默认获取 UTC 时区 (UTC：世界协调时间)
    Instant ins1 = Instant.now();
    System.out.println(ins1);

    //带偏移量的时间日期 (如：UTC + 8)
    OffsetDateTime odt1 = ins1.atOffset(ZoneOffset.ofHours(8));
    System.out.println(odt1);

    //转换成对应的毫秒值
    long milli1 = ins1.toEpochMilli();
    System.out.println(milli1);

    //构建时间戳
    Instant ins2 = Instant.ofEpochSecond(60);
    System.out.println(ins2);
}
```

### 时间 / 日期 差
- Duration：计算两个时间之间的间隔
- Period：计算两个日期之间的间隔
```java
@Test
public void test03(){
    //计算两个时间之间的间隔 between
    Instant ins1 = Instant.now();
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    Instant ins2 = Instant.now();
    Duration dura1 = Duration.between(ins1, ins2);
    System.out.println(dura1.getSeconds());
    System.out.println(dura1.toMillis());
}

@Test
public void test04(){
    LocalDate ld1 = LocalDate.of(2016, 9, 1);
    LocalDate ld2 = LocalDate.now();
    Period period = Period.between(ld1, ld2);  // ISO 标准
    System.out.println(period.getYears());
    System.out.println(period.toTotalMonths());
}
```

### 时间校正器
```java
@Test
public void test01(){
    //TemporalAdjusters：时间校正器
    LocalDateTime ldt1 = LocalDateTime.now();
    System.out.println(ldt1);

    //指定日期时间中的 年 月 日 ...
    LocalDateTime ldt2 = ldt1.withDayOfMonth(10);
    System.out.println(ldt2);

    //指定时间校正器
    LocalDateTime ldt3 = ldt1.with(TemporalAdjusters.next(DayOfWeek.SUNDAY));
    System.out.println(ldt3);

    //自定义时间校正器
    LocalDateTime ldt5 = ldt1.with((ta) -> {
        LocalDateTime ldt4 = (LocalDateTime) ta;
        DayOfWeek dow1 = ldt4.getDayOfWeek();
        if (dow1.equals(DayOfWeek.FRIDAY)) {
            return ldt4.plusDays(3);
        } else if (dow1.equals(DayOfWeek.SATURDAY)) {
            return ldt4.plusDays(2);
        } else {
            return ldt4.plusDays(1);
        }
    });
    System.out.println(ldt5);
}
```

### 格式化
- DateTimeFormatter：格式化时间 / 日期
```java
@Test
public void test01(){
    //默认格式化
    DateTimeFormatter dtf1 = DateTimeFormatter.ISO_DATE_TIME;
    LocalDateTime ldt1 = LocalDateTime.now();
    String str1 = ldt1.format(dtf1);
    System.out.println(str1);

    //自定义格式化 ofPattern
    DateTimeFormatter dtf2 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    LocalDateTime ldt2 = LocalDateTime.now();
    String str2 = ldt2.format(dtf2);
    System.out.println(str2);

    //解析
    LocalDateTime newDate = ldt1.parse(str1, dtf1);
    System.out.println(newDate);
}
```

### 时区
- ZonedDate
- ZonedTime
- ZonedDateTime
```java
@Test
public void test02(){
    //查看支持的时区
    Set<String> set = ZoneId.getAvailableZoneIds();
    set.forEach(System.out::println);

    //指定时区
    LocalDateTime ldt1 = LocalDateTime.now(ZoneId.of("Europe/Tallinn"));
    System.out.println(ldt1);

    //在已构建好的日期时间上指定时区
    LocalDateTime ldt2 = LocalDateTime.now(ZoneId.of("Europe/Tallinn"));
    ZonedDateTime zdt1 = ldt2.atZone(ZoneId.of("Europe/Tallinn"));
    System.out.println(zdt1);
}
```
