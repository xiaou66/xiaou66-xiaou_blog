---
title: 红黑树
date: 2021/08/08 19:15
math: true
categories:
  - [数据结构, hashTable]
tags:
  - [java]
---

# 哈希表

## 自定义对象实现 hash 函数

```java
@Override
public int hashCode() {
    Objects.hash(name, age);
    int hashCode = Integer.hashCode(age);
    hashCode = hashCode * 31 + Float.hashCode(height);
    hashCode = hashCode * 31 + (name != null ? 0 : name.hashCode());
    return hashCode;
}
```

也可以使用下面的写法

```java
@Override
public int hashCode() {
    return Objects.hash(name, age, height);
}
```

其 java 内部调用方法

```java
public static int hashCode(Object a[]) {
    if (a == null)
        return 0;

    int result = 1;

    for (Object element : a)
        result = 31 * result + (element == null ? 0 : element.hashCode());

    return result;
}
```

## 自定义对象实现 equals 函数

```java
@Override
public boolean equals(Object o) {
    // 内存地址的比较
    if (this == o) {
        return true;
    }
    // 如果 o == null 或者 类对象不一致的情况下直接返回 false
    if (o == null || getClass() != o.getClass()) {
        return false;
    }
    // 比较成员变量
    Person person = (Person) o;
    return age == person.age && Objects.equals(name, person.name) && height == person.height;
}
```
