---
title: Lambda 表达式
date: 2021/12/27 10:35:11
math: true
categories:
  - [java]
tags:
  - [java]
---
# Lambda 表达式

## Lambda 表达式

### 举例

```java
(o1, o2) -> Integer.compare(o1, o2);
```

### 格式

1. `->` : lambda 操作符 或 箭头操作符
2. 左边: lambda 行参列表 = 抽象方法的行参列表
3. 右边 lambda 体 = 抽象方法的方法体

### lambda 表达式的本质

> 作为函数式接口的实例

## 快速入门

> 利用 Runnable 接口 使用 Lambda 表达式


```java
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("hello word");
    }
};
r1.run();
Runnable r2 = () -> System.out.println("hello word");
r2.run();
```

> 利用  Comparator 接口 使用方法引用

```java
Comparator<Integer> c1 = new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return Integer.compare(o1, o2);
    }
};
System.out.println(c1.compare(1, 2));
Comparator<Integer> c2 = Integer::compare;
System.out.println(c2.compare(1, 2));
```

## 使用

### 无参无返回值

```java
Runnable r2 = () -> System.out.println("hello word");
r2.run();
```

### 需要一个参数无返回值

```java
Consumer<String> c1 = (s) -> System.out.println(s);
Consumer<String> c2 = System.out::println;
c1.accept("1");
c1.accept("2");
```

### 两个或以上参数，多条执行语句有返回值

```java
Comparator<Integer> com1 = (a, b) -> {
    System.out.println(a);
    System.out.println(b);
    return a.compareTo(b);
};
System.out.println(com1.compare(1, 2));
```

## 函数式接口

:::info

如果一个接口中只声明了一个抽象方法，则此接口就称为函数式接口可用 `@FunctionalInterface`  注解，可用通过 lambda 表达式使用函数式接口

:::

### 函数式接口

```java
@FunctionalInterface
public interface MyInterface {
    void method1();
}
```

### java 内置四大核心的函数式接口

| 函数式接口           | 参数 | 返回    | 用途                                                         |
| -------------------- | ---- | ------- | ------------------------------------------------------------ |
| Consumer\<T> 消费型  | T    | void    | 对类型为T对象应用操作包含方法: `void accept(T t)`            |
| Supplier\<T> 供给型  | 无   | T       | 返回类型为T对象，包含方法 `T get()`                          |
| Function<T,R> 函数型 | T    | R       | 对象类型为T对象应用操作并返回返回结果<br />结果是R类型的对象包含方法 `R apply (T t)`; |
| Predicate\<T> 断定型 | T    | boolean | 启动类型为T的对象是否满足某约束，并返回Boolean值<br />包含方法 `boolean test(T t)` |

#### 扩展的函数接口

| 函数式接口        | 参数 | 返回 | 用途 |
| ------------------ | ---- | ---- | ---- |
| BiFunction<T,U,R>  | T,U  | R    | 对类型为 T,U 参数应用操作,返回 R 类型的结果包含方法为 `R apply(T t, U u)`; |
| UnaryOperator\<T>  | T    | T    | 对类型为T的对象进行一元运算, 并返回T类型的结果包含方法为: `T apply(T t)`; |
| BinaryOperator\<T> | T,T  | T    | 对类型为 T 的对象进行二元运算，并返回T类型的结果。包含方法为 `T apply(T t1, T t2)` |
| BiConsumer<T, U>   | T, U | void | 对类型为 T,U  参数进行应用操作。包含方法为 `void accept(T t, U u)` |
| BiPredicate<T,U> | T,U | boolean | 包含方法为 `boolean test(T t, U u)`; |
| ToIntFunction\<T><br />LongFunction\<T><br />ToDoubleFunction\<T> | T | int<br />long<br />double | 分别计算 int、long、double  值的函数 |
| IntFunction\<R><br />LongFunction\<R><br />DoubleFunction\<R> | int<br />long <br />double | R | 参数分别为 int、long、double类型的函数 |

### 部分使用

#### Consumer 使用

```java
public static <T> void print(T s, Consumer<T> consumer) {
    consumer.accept(s);
}
@Test
public void test3() {
    // hello
    Demo2.print("hello", a ->
                System.out.println(a.length() >= 5 ? a : "字数不足"));

}
```

#### Sepplier 使用

```java
public int calcSum (Supplier<int[]> supplier) {
    final int[] ints = supplier.get();
    return Arrays.stream(ints).sum();
}
@Test
public void test4() {
    // 6
    calcSum(() -> new int[]{1, 2, 3});
}
```

#### Function 使用

```java
public String [] handlerString(String[] s, Function<String, String> handler) {
    return Arrays.stream(s).map(handler).toArray(String[]::new);
}
@Test
public void test5() {
    String [] a = new String[]{"1", "2", "3"};
    // a1, a2, a3
    Arrays.toString(handlerString(a, s -> "a" + s));
}

```

#### Predicate 使用

```java
public String isPrint(String surname, String name, Predicate<String> predicate) {
    if (predicate.test(name)) {
        return surname + name;
    } else {
        return surname;
    }
}
@Test
public void test6() {
    // xiaou
    isPrint("xiaou", "", name -> !name.isEmpty());
    // xiaou66
    isPrint("xiaou", "66", name -> !name.isEmpty());
}
```

## 方法引用

:::info

当要传递给 lambda 体的操作，已经有实现的方法就可用使用方法引用

:::

### 对象 :: 非静态方法

在 Supplier 中 get 方法 `T get();`

在 Users 中 getName 方法 `String getName()`

因为函数的方法参数一致返回值一致

```java
@Test
public void test7 () {
    Users user = new Users("xiaou");
    Supplier<String> getName = user::getName;
    // xiaou
    getName.get();
}
```

### 类 :: 静态方法

Comparator 方法 `int compare(T o1, T o2)`

Integer compare 方法 `int compare(int x, int y)`

因为函数的方法参数一致返回值一致

```java
@Test
public void test8() {
    Comparator<Integer> s = Integer::compare;
    // -1
    s.compare(1,2);
}
```

### 类 :: 非静态方法

#### 第一种情况

在 Users 中 getName 方法 `String getName()`

在 Function 函数方法 `R apply(T t);`

```java
@Test
public void test9() {
    // Users :: getName = 传递进来的 Users 对象调用 getName
    // 也就是将第一个参数作为调用对象
    Function<Users, String> getName = Users::getName;
    getName.apply(new Users("xiaou"));
}
```

#### 第二种情况

Comparator 方法 `int compare(T o1, T o2)`

String compareTo 方法  `int compareTo(String anotherString)`

```java
@Test
public void test10() {
    // 将第一个参数作为调用对象 "1".compareTo("2")
    Comparator<String> s = String::compareTo;
    // -1
    s.compare("1", "2");
}
```

## 实际应用

1. 在枚举类中使用 

```java
enum OrderTransform {
        ORDER_SN("订单号", "$.orderSn", null),
        AFTER_SALES_STATUS("订单状态", "$.afterSalesStatus", (csvRow) -> {
            String status = csvRow.getByName(OrderTransform.AFTER_SALES_STATUS);
            return status;
        }),
        ;
        private String title;
        private String jsonPath;
        private Function<CsvRow, String> getValue;
}
```