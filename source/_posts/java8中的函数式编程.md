---
title: java8中的函数式编程
date: 2018-02-04 17:24:53
tags: java8
categories: java
description: java8中的一些新特性--函数式编程,stream API <br/> 其实不能叫新特性,14年距离现在也已经4年了,马上java10就来了
---
java8出现以来,lambda是最重要的特性之一,它可以让我们用简洁流畅的代码完成一个功能。
lambda表达式是一段可以传递的代码，他的核心思想是将面向对象中的传递数据变成传递行为。
##### 使用lambda表达式替换匿名内部类
使用lambda表达式创建一个线程
```java
    //java8以前
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("以前的写法");
        }
    }).start();

    //java8,lambda只需要一行代码
    new Thread(() -> System.out.println("lambda的写法")).start();
```
一般都会把lambda表达式的变量名起的短一些,这样能使代码更加的简洁。

##### 使用lambda表达式对列表进行迭代
```java
    List<String> list = Arrays.asList("a", "b", "c", "e");
    //8之前
    for (String s : list) {
        System.out.println(s);
    }

    //java8中列表有了一个foreach()的方法,可以迭代所有的对象,并将你的lambda代码应用在其中
    list.forEach(System.out::println); //使用方法引用更加的简短,C++里面的双冒号、范围解析操作符现在在Java 8中用来表示方法引用
    list.forEach(n -> System.out.println(n));
```
##### 使用lambda表达式和函数式接口
java8中添加了一个包,叫做java.util.function,它包含了很多的类,用来支持java的函数式编程
下面是java.util.function包中的接口示例
![函数式接口的类型](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/v2-502cb9d5e5368dbba2c8f22284694e23_r.jpg)
```java
	//消费型的接口
    private void consume(Integer money, Consumer<Integer> consumer) {
        consumer.accept(money);
    }
    void test11() {
        consume(1000,money -> System.out.println("this number is: "+money));
    }

    //供给型接口
    List<Integer> supply(Integer num, Supplier<Integer> supplier) {
        ArrayList<Integer> resultList = new ArrayList<>();
        for(int x = 0; x < num; x++) {
            resultList.add(supplier.get());
        }
        return resultList;
    }
    void test12() {
        List<Integer> supply = supply(10, () -> (int) (Math.random() * 100));
        supply.forEach(System.out::println);
    }

    //函数型接口
    Integer convert(String str, Function<String,Integer> function) {
        return function.apply(str);
    }

    void test13() {
        convert("28",Integer::parseInt);
    }

    //断言型接口
    List<String> isOrNot(List<String> fruit,Predicate<String> predicate) {
        ArrayList<String> list = new ArrayList<>();
        for(String s: fruit) {
            if(predicate.test(s)) {
                list.add(s);
            }
        }
        return list;
    }

    void test14() {
        List<String> fruits = Arrays.asList("香蕉", "哈密瓜", "榴莲", "火龙果", "水蜜桃");
        List<String> orNot = isOrNot(fruits, f -> f.length() == 2);
        System.out.println(orNot);
    }
```
- 使用lambda表达式和Predicate接口
```java
    /*
     * 函数式接口Predicate
     */
    void test3() {
        List<String> list = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");
        //以J开头的
        filter(list,str -> str.startsWith("J"));
        //以a结尾的
        filter(list,str -> str.endsWith("a"));
        //打印所有的
        filter(list,str -> true);
        //都不打印
        filter(list,str -> false);
        //长度大于4的
        betterFilter(list,str -> str.length() > 4);
    }

    private void filter(List<String> names, Predicate<String> condition) {
        for (String name : names) {
            if(condition.test(name)) {
                System.out.println(name+"");
            }
        }
    }

    //更好的办法
    private void betterFilter(List<String> names, Predicate<String> condition) {
//        names.stream().filter(name -> condition.test(name)).forEach(name -> System.out.println(name));
        //可以简化如下
        names.stream().filter(condition).forEach(System.out::println);
    }
```
- 在lambda表达式中加入Predicate
```java
    /*
     * 在表达式中加入Predicate
     * java.util.function.Predicate 允许将两个或更多的 Predicate 合成一个。它提供类似于逻辑操作符AND和OR的方法，名字叫做 and()、or() 和 xor()，用于将传入 filter() 方法的条件合并起来。
     * 例如，要得到所有以J开始，长度为四个字母的语言，可以定义两个独立的 Predicate 示例分别表示每一个条件，然后用 Predicate.and() 方法将它们合并起来
     */
    void test4() {
        List<String> list = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");
        Predicate<String> startWithJ = n -> n.startsWith("J"); //其实就是函数式接口Predicate接口的实现
        Predicate<String> fourLetterLong = n -> n.length() == 4;
        list.stream()
                .filter(startWithJ.and(fourLetterLong))
                .forEach(System.out::println);
    }
```
##### java8中使用lambda表达式的Map和Reduce示例
- map
```java
    /*
     * 使用lambda表达式的Map和reduce示例
     * map允许你将对象进行转换
     */
    void test5() {
        List<Integer> costBeforeTax = Arrays.asList(100,200,300,400,500);
        //不使用lambda表达式为每一个订单加上12%的税
        for (Integer cost : costBeforeTax) {
            double price = cost + .12*cost;
            System.out.println(price);
        }
        //使用lambda表达式
        costBeforeTax.stream().map(cost -> cost + .12*cost).forEach(System.out::println);
    }
```
- reduce
```java
    /*
     * 使用reduce()函数进行折叠操作,类似于sql中的聚合函数
     */
    void test6() {
        List<Integer> costBeforeTax = Arrays.asList(100,200,300,400,500);
        Double aDouble = costBeforeTax.stream().map(cost -> cost + .12 * cost).reduce((n, m) -> n + m).get();
        System.out.println("Total: "+aDouble);
    }
```
##### 通过过滤创建一个string的列表
```java
    /*
     * 通过过滤创建一个String列表
     * 过滤是Java开发者在大规模集合上的一个常用操作，而现在使用lambda表达式和流API过滤大规模数据集合非常的简单。
     * 流提供了一个 filter() 方法，接受一个 Predicate 对象，即可以传入一个lambda表达式作为过滤逻辑
     */
    private void test7() {
        List<String> strList = Arrays.asList("as","artfg","asdf","shat");
        List<String> collect = strList.stream().filter(x -> x.length() > 2).collect(Collectors.toList());
        System.out.printf("原始串:%s,过滤后:%s",strList,collect);
    }
```
##### 对列表的每个元素应用函数
```java
    /*
     * 对列表的每个元素应用函数
     */
    void test8() {
//        将字符串换成大写并用逗号链接起来
        List<String> g7 = Arrays.asList("USA", "Japan", "France", "Germany", "Italy", "U.K.","Canada");
        String collect = g7.stream().map(String::toUpperCase).collect(Collectors.joining(","));
        System.out.println(collect);
    }
```
##### 复制不同的值,创建一个子列表
```java
    // 数字去重的例子
    void test9() {
        List<Integer> numbers = Arrays.asList(9, 10, 3, 4, 7, 3, 4);
        List<Double> collect = numbers.stream().map(Math::sqrt).distinct().collect(Collectors.toList());
        System.out.printf("原来的: %s,现在的: %s",numbers,collect);
    }
```
##### 计算集合元素的最大值、最小值、总和以及平均值
```java
    /*
     * 计算集合元素的最大数值,最小值,总和以及平均值
     * IntStream、LongStream 和 DoubleStream 等流的类中，有个非常有用的方法叫做 summaryStatistics() 。可以返回 IntSummaryStatistics、LongSummaryStatistics 或者 DoubleSummaryStatistics，描述流中元素的各种摘要数据。
     * 在本例中，我们用这个方法来计算列表的最大值和最小值。它也有 getSum() 和 getAverage() 方法来获得列表的所有元素的总和及平均值。
     */
    void test10() {
        List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29);
        IntSummaryStatistics stat = primes.stream().mapToInt((x -> x)).summaryStatistics();
        System.out.println("最大值: "+stat.getMax());
        System.out.println("最小值: "+stat.getMin());
        System.out.println("求和: "+stat.getSum());
        System.out.println("平均值: "+stat.getAverage());
    }
```