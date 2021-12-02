# Stream学习笔记

## 创建流：

### 通过几个简单的例子

```java
// 1.通过数组创建
String[] arr = {"不","高","兴","就","喝","水"};
Stream<String> stream1 = Arrays.stream(arr);
stream1.forEach(System.out::println);

// 2.通过集合创建
List<String> list = Arrays.asList("不","高","兴","就","喝","水");
// 2.1创建一个顺序流(串行执行)
Stream<String> stream2 = list.stream();
stream2.forEach(System.out::println);

//2.2创建并行流(以多线程方式对流进行操作，底层用的线程池)
// 通过parallel方法把顺序流转成并行流
Stream<String> parallelStream1 = list.stream().parallel();
parallelStream1.forEach(System.out::println);

// 通过集合创建一个并行流
Stream<String> parallelStream2 = list.parallelStream();
parallelStream2.forEach(System.out::println);

// 3.通过Stream的静态方法创建
// 3.1 iterate方法：
Stream<Integer> stream3 = Stream.iterate(1, (x) -> x + 1).limit(6);
stream3.forEach(System.out::println);

// 3.2 of方法：
Stream<String> stream4 = Stream.of("不","高","兴","就","喝","水");
stream4.forEach(System.out::println);

// 3.3 generate方法：
Stream<String> stream5 = Stream.generate(() ->"不高兴就喝水!").limit(3);
stream5.forEach(System.out::println);

// 4.通过数字Stream创建
IntStream intStream1 = IntStream.of(1, 2, 3);
intStream1.forEach(System.out::println);
IntStream intStream2 = IntStream.rangeClosed(1,3);
intStream2.forEach(System.out::println);

// 5.通过random创建无限流
IntStream randomStream = new Random().ints(1,3).limit(3);
randomStream.forEach(System.out::println);
```

## Stream的中间操作：

该操作会返回一个新的流，目的主要是对流做数据映射或者过滤之类的操作，之后会返回一个新的流，交给下一个操作使用。这类操作都是惰性化的，并没有真正对流进行遍历。一个流可以后面跟随 0 个或者多个中间操作。

![图片](https://mmbiz.qpic.cn/mmbiz_png/paRXy9ic0MZOiaz3RaZvKpqQFVT1Gg3pf7lIFJWtuvKiaicble7KI5OH9lm1sF0SW8o4H9pR01AZEmMhW3W1QFiaN5g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 无状态：元素的操作不依赖之前元素，不受之前元素的影响，这些操作没有内部状态，称为无状态操作
- 有状态：只有拿到所有的元素之后才能继续操作，这些操作都需要利用到之前元素的内部状态，所以称为有状态操作

### 举例

```java
String str = "bu gao xing jiu he shui";
// 转成list集合
// 输出：bu gao xing jiu he shui
Stream.of(str.split(" ")).collect(Collectors.toList())
        .forEach(System.out::println);
// 拿到长度大于2的单词
// 输出：gao xing jiu shui
Stream.of(str.split(" ")).filter(s -> s.length() > 2)
        .forEach(System.out::println);

// flatMap的使用，可以拿到str中的所有的字母
Stream.of(str.split(" ")).flatMap(s -> s.chars().boxed())
        .forEach(i -> System.out.println((char) i.intValue()));

// filter用来过滤符合条件的元素，peek可以输出流操作中的中间值，主要用来调试代码用
Stream.of(str.split(" ")).filter(e -> e.length() > 3)
        .peek(e -> System.out.println("过滤出来的元素: " + e))
        .map(String::toUpperCase)
        .peek(e -> System.out.println("映射过后的元素: " + e))
        .collect(Collectors.toList());
```

## Stream的终止操作：

该操作才会返回最终的执行结果。当这个操作执行完成后，就无法继续操作了。随着终止操作的执行，才会真正开始流的遍历，生成执行结果，一个流只能有一个终止操作。



![图片](https://mmbiz.qpic.cn/mmbiz_png/paRXy9ic0MZOiaz3RaZvKpqQFVT1Gg3pf7loNGqZytaad4PbvgPUxSUQSFvvQqJx2uBIC9UPrVtLh8HdZDQ0ZB4A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 非短路操作：必须所有元素都处理完才能得到最终的结果
- 短路操作：一旦得到符合条件的元素就可以中断流得到最终的结果

### 举例

```java
String str = "bu gao xing jiu he shui";

// 使用foreach得到乱序结果(因为用了并行流)
// 输出：n go xu sgbuhiaui  hiej
str.chars().parallel().forEach(i -> System.out.print((char) i));
// 使用forEachOrdered得到顺序结果
// 输出bu gao xing jiu he shui
str.chars().parallel().forEachOrdered(i -> System.out.print((char) i));

// 收集到list集合中
// 输出：[bu, gao, xing, jiu, he, shui]
List<String> list = Stream.of(str.split(" "))
        .collect(Collectors.toList());
System.out.println(list);

// 使用reduce对字符串进行拼接
// 输出bu_gao_xing_jiu_he_shui
Optional<String> stringOptional = Stream.of(str.split(" "))
        .reduce((s1, s2) -> s1 + "_" + s2);
System.out.println(stringOptional.get());

// 计算字符串总长度
// 输出：18
Integer length = Stream.of(str.split(" ")).map(s -> s.length())
        .reduce(0, (s1, s2) -> s1 + s2);
System.out.println(length);

// 取长度最大的单词
// 输出：xing
Optional<String> max = Stream.of(str.split(" "))
        .max((s1, s2) -> s1.length() - s2.length());
System.out.println(max.get());

// 使用findFirst取第一个元素，取到则中断操作
// 输出：bu
Optional<String> findFirst = list.stream().findFirst();
System.out.println(findFirst.get());
```

## Stream操作实战

### 先准备一个学生类

//get，set，toString 方法,自己生成一下哈

```java
static class  Student {
        /**
         * id
         */
        private Integer id;
        /**
         * 学生姓名
         */
        private String name;
        /**
         * 性别
         */
        private String sex;
        /**
         * 年龄
         */
        private Integer age;
        /**
         * 分数
         */
        private Integer score;

        /**
         * 籍贯
         */
        private String area;

        public Student(Integer id, String name, String sex, Integer age, Integer score, String area) {
            this.id = id;
            this.name = name;
            this.sex = sex;
            this.age = age;
            this.score = score;
            this.area = area;
        }
}
```

### 初始化几条数据，用来测试

```java
List<Student> studentList = new ArrayList<Student>();
studentList.add(new Student(1,"小明", "男", 18, 100, "江西"));
studentList.add(new Student(2,"小花", "女", 17, 90, "安徽"));
studentList.add(new Student(3,"小虎", "男", 21, 80, "黑龙江"));
studentList.add(new Student(4,"小军", "男", 20, 85, "黑龙江"));
```

### 1.toList()/toSet()/toMap() (重组)

因为 Stream 流不储存数据，所以当流中的数据完成处理之后，需要将流中的数据重组到新的集合里

```java
// 1.获取studentList中所有的籍贯
List<String> areaList1 = studentList.stream().map(Student :: getArea).collect(Collectors.toList());
System.out.println("areaList1：" + areaList1.toString());
// 2.获取studentList中所有的籍贯,并去重
Set<String> areaList2 = studentList.stream().map(Student :: getArea).collect(Collectors.toSet());
System.out.println("areaList2：" + areaList2.toString());
// 3.将studentList转成map
Map<Integer, Student> studentMap = studentList.stream().collect(Collectors.toMap(Student::getId, o -> o));
System.out.println("studentMap：" + studentMap);
```

### 2.filter()(过滤)

过滤出我们想要的数据

```java
// 1.获取小明的信息
List<Student> xmStudent = studentList.stream().filter(o -> o.getName().equals("小明")).collect(Collectors.toList());
System.out.println("获取小明的信息：" + xmStudent.toString());
// 2.获取年龄大于18的学生名称
List<String> filterList = studentList.stream().filter(o -> o.getAge() > 18).map(Student::getName)
        .collect(Collectors.toList());
System.out.println("获取年龄大于18的学生名称：" + filterList);
```

### 3.groupingBy()（分组）和 partitioningBy()(分区)

```java
// 1.按地区分组，得到学生id的Set集合
Map<String, Set<Integer>> areaIdSetMap = studentList.stream().collect(Collectors.groupingBy(Student::getArea, Collectors.mapping(Student::getId, Collectors.toSet())));
System.out.println("按地区分组，得到学生id的Set集合：" + areaIdSetMap);

// 2.按性别分组
Map<String, List<Student>> group1 = studentList.stream().collect(Collectors.groupingBy(Student::getSex));
System.out.println("按性别分组：" + group1);

// 3.将员工先按性别分组,再按籍贯分组
Map<String, Map<String, List<Student>>> group2 = studentList.stream().collect(Collectors.groupingBy(Student::getSex, Collectors.groupingBy(Student::getArea)));
System.out.println("将员工先按性别分组,再按籍贯分组：" + group2);

// 4.将学生按是否大于80分，分成两个map
Map<Boolean, List<Student>> partMap = studentList.stream().collect(Collectors.partitioningBy(x -> x.getScore() > 80));
System.out.println("将学生按是否大于80分，分成两个map：" + partMap);
```

### 4.count()/averagingXxx()/maxBy()(统计操作)

```java
// 1.求学生集合总条数
Long count = studentList.stream().collect(Collectors.counting());
System.out.println("求学生集合总条数：" + count);
// 2.求学生的平均分数
Double averageScore = studentList.stream().collect(Collectors.averagingDouble(Student::getScore));
System.out.println("求学生的平均分数：" + averageScore);
// 3.求学生中的最高分
Optional<Integer> maxScore = studentList.stream().map(Student::getScore).collect(Collectors.maxBy(Integer::compare));
System.out.println("求学生中的最高分：" + maxScore.get());
// 4.一次性统计关于分数的所有信息
DoubleSummaryStatistics collect = studentList.stream().collect(Collectors.summarizingDouble(Student::getScore));
System.out.println("一次性统计关于分数的所有信息：" + collect);
// 5.统计总分前3的学生姓名并打印出来
Map<String, Integer> map = new HashMap();
studentList.stream().collect(Collectors.groupingBy(Student::getName))
        .forEach((k, v) -> map.put(k, v.stream().collect(Collectors.summingInt(Student::getScore))));
map.keySet().stream().sorted(Comparator.comparing(item->map.get(item)).reversed()).limit(3).collect(Collectors.toList()).forEach(item-> System.out.println(item));
```

### 5.join() (连接)和reduce() (浓缩)

join():将元素用连接符连接起来 

reduce():将流浓缩成一个值，用来求和、求最大值等操作

```java
// 1.获取所有学生的姓名,用逗号分割
String joinNames = studentList.stream().map(o -> o.getName()).collect(Collectors.joining(","));
System.out.println("获取所有学生的姓名,用逗号分割：" + joinNames);

// 2.获取学生的年龄总和
Optional<Integer> sum = studentList.stream().map(Student::getAge).reduce(Integer::sum);
System.out.println("获取学生的年龄总和：" + sum.get());
```

### 6.sorted()（排序）

```java
// 1.按分数升序排序
List<String> sortList1 = studentList.stream().sorted(Comparator.comparing(Student::getScore)).map(Student::getName)
        .collect(Collectors.toList());
System.out.println("按分数升序排序：" + sortList1);

// 2.按分数降序排序
List<String> sortList2 = studentList.stream().sorted(Comparator.comparing(Student::getScore).reversed())
        .map(Student::getName).collect(Collectors.toList());
System.out.println("按分数降序排序：" + sortList2);
// 3.先按分数再按年龄升序排序
List<String> sortList3 = studentList.stream()
        .sorted(Comparator.comparing(Student::getScore).thenComparing(Student::getAge)).map(Student::getName)
        .collect(Collectors.toList());
System.out.println("先按分数再按年龄升序排序：" + sortList3);
```