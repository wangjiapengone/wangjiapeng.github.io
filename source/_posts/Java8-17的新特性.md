---
title: Java8-17的新特性
id: java-new-features
date: 2023-07-21 11:06:35
categories:
  - 编程语言
tags:
  - Java
---

## Lambda表达式（Java 8）

#### 1. Lambda表达式的本质

- 就是一个对象，是接口实现类的对象
- 也是一个匿名函数

#### 2. 使用场景

- 应用于函数式接口（接口只声明一个抽象方法）
- 只有给函数式接口提供实现类的对象时，我们才可以使用lambda表达式

#### 3. 语法

![image-20230219160849057](./image/Java8-17的新特性/image-20230219160849057.png)

```
-> : lambda操作符或箭头操作符
-> 的左边：lambda形参列表，参数的类型都可以省略。如果形参只有一个，则一对()也可以省略。
-> 的右边：lambda体，对应着重写的方法的方法体。如果方法体中只有一行执行语句，则一对{}可以省略。如果有return关键字，则必须一并省略。
```

<!-- more -->


## 方法引用、构造器引用（Java 8）

#### 1. 方法引用的本质

- 和Lambda一样，就是一个对象，是接口实现类的对象
- 可以认为是lambda表达式进一步简化，当lambda表达式满足一定条件时，可以使用方法引用

#### 2. 使用场景

- 当以下情形满足时，可以简化lambda表达式的代码

  | 情形           | 条件                                                         |
  | -------------- | ------------------------------------------------------------ |
  | 对象::实例方法 | 1. 函数式接口的抽象方法a与实现方法b的形参列表和返回值类型相同<br />2. 注意：b为非静态方法，需要对象调用 |
  | 类::静态方法   | 1. 函数式接口的抽象方法a与实现方法中类的静态方法b的形参列表和返回值类型相同<br />2. 注意：b为静态方法，需要类调用 |
  | 类::实例方法   | 1. 函数式接口的抽象方法a有n个参数，实现方法中，第一个参数用于调用方法，方法中的参数为后面的n-1个参数<br />2. 注意：b为非静态方法，需要类调用 |

#### 3. 语法

![image-20230219171944787](./image/Java8-17的新特性/image-20230219171944787.png)

```
1. 方法引用：类（对象）::方法名
2. 构造器引用：类::new，不需要提供形参列表，java会根据使用时传入的形参列表去判断
3. 数组引用：类[]::new
```



## 3. Stream API（Java 8）

#### 1. Stream API 和集合框架对比

- Stream API 关注的是数据的计算，面向CPU
- 集合关注数据存储，面向内存
- Steam API 和集合的关系，类似于SQL语句和数据表的关系

#### 2. Stream API特性

- `Stream`自己不会存储元素
- `Stream`不会改变源对象。相反，他们会返回一个持有结果的新`Stream`
- `Stream`操作是惰性的。这意味着他们会等到需要结果的时候才执行。即一旦执行终止操作，就执行中间操作链，并产生结果
- `Stream`一旦执行了终止操作，就不能再调用其它中间操作或终止操作了



## 4. 其他新语法和API

#### 1. jShell

```
/help: 获取有关使用 jshell 工具的信息
/help intro : jshell 工具的简介
/list : 列出当前 session 里所有有效的代码片段
/vars : 查看当前 session 下所有创建过的变量
/methods : 查看当前 session 下所有创建过的方法
/imports : 列出导入的包
/history : 键入的内容的历史记录
/edit : 使用外部代码编辑器来编写 Java 代码
/exit : 退出 jshell 工具
```

#### 2. try-catch资源关闭（是java 7的）

- `try()`中可以放需要关闭的资源，不再需要在`finally`中手动关闭

```java
try (FileWriter fw = new FileWriter("test.txt");
     BufferedWriter bw = new BufferedWriter(fw)
) {
    bw.write("hello, world");
} catch (IOException e) {
    e.printStackTrace();
}
```

#### 3. 局部变量类型推断

#### 4. instanceof模式匹配

- `obj instanceof String str`,`str`是变量名称，如果模式匹配成功，`str`会被自动创建，不再需要强转类型

#### 5. switch语句

- 可以省略`break`：使用符号`->`，多行语句使用大括号

  ```java
  public void test2(Week week) {
      switch (week) {
          case MONDAY -> System.out.println(1);
          case TUESDAY, WEDNESDAY, THURSDAY -> System.out.println(2);
          case FRIDAY -> {
              System.out.println(3);
              System.out.println(4);
          }
          case SATURDAY, SUNDAY -> System.out.println(5);
          default -> throw new RuntimeException("Invalid week: " + week);
      }
  }
  ```

- 可以带返回值，多行语句要用`yield`返回

  ```java
  public void test4(Week week) {
      int res = switch (week) {
          case MONDAY -> 1;
          case TUESDAY, WEDNESDAY, THURSDAY -> {
              System.out.println(2);
              yield 2;
          }
          case FRIDAY -> {
              System.out.println(3);
              yield 3;
          }
          case SATURDAY, SUNDAY -> 4;
      };
      System.out.println(res);
  }
  ```

#### 6. 多行文本块

- 类似于Python的多行文本块

```java
String s = """
    <html>
      <body>
          <p>Hello, world</p>
      </body>
    </html>
    """;
```

#### 7. record

- 类似于`Scala`中的`case class`，可以不用定义类的结构

  ```java
  public record Person(String name, int age) {
  }
  ```

- 传入的参数即为成员变量，`final`修饰，不可更改

- 只有`getter`方法，没有`setter`方法

- `record`类的父类是`Record`，因而不可继承

- `record`类被`final`修饰，不可被继承

#### 8. sealed密封类

- 密封类可以指定只能被某些类继承，其他的类不能继承
- 密封类的子类必须是以下三者之一：`final`、`sealed`、`non-sealed`

```java
// 密封类只能被Student, Teacher继承
public sealed class SealedTest permits Student, Teacher {
}
// Student不能再被继承
final class Student extends SealedTest {
}
// 子类还是密封类
sealed class Teacher extends SealedTest permits JuniorTeacher {
}
// non-sealed为非密封类，不再限制子类
non-sealed class JuniorTeacher extends Teacher {
}
// 可以正常继承
class SeniorTeacher extends JuniorTeacher {
}
```

#### 9. Optional

- 类似于`Scala`中的`Option`
- 为了避免代码中的空指针异常

```java
String s = "Hello, world";
// s = null;
// 实例化
Optional<String> optional = Optional.ofNullable(s);
// get元素
System.out.println(optional.orElse("default"));
```

