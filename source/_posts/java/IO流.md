---
title: IO 流
date: 2021/11/21 18:40:40
math: true
categories:
  - [java]
tags:
  - [java]
---

# IO 流

## 流的分类

### 按操作数据单位

1. 字节流 8bit
2. 字符流 16bit

### 按数据流向

1. 输入流
2. 输出流

### 按流的角色

1. 节点流

   直接作用于文件的流。

2. 处理流

   在节点流再包一层，这个一层就叫做处理流

| 基类   | 字节流       | 字符流 |
| ------ | ------------ | ------ |
| 输入流 | InputSteam   | Reader |
| 输出流 | OutputStream | Writer |

## IO 流体系

| 分类       | 字节输入流              | 字节输出流              | 字符输入流            | 字符输出流             |
| :--------- | ----------------------- | ----------------------- | --------------------- | ---------------------- |
| 抽象基类   | **InputStream**         | **OutputStream**        | **Reader**            | **Writer**             |
| 访问文件   | **FileInputStream**     | **FileOutputStream**    | **FileReader**        | **FileWriter**         |
| 访问数组   | ByteArrayInputStream    | ByteArrayOutputSteam    | CharArrayReader       | CharArrayWriter        |
| 访问字符串 |                         |                         | StringReader          | StringWriter           |
| 缓冲流     | **BufferedInputStream** | **BufferedOutputSteam** | **BufferedReader**    | **BufferedWriter**     |
| 转换流     |                         |                         | **InputStreamReader** | **OutputStreamWriter** |
| 打印流     |                         | PrintStream             |                       | PrintWriter            |
| 推回输入流 | PushbackInputStream     |                         | PushbackReader        |                        |
| 特殊流     | DataInputStream         | DataOutputStream        |                       |                        |
| 访问管道   | PipedInputStream        | PipedOutputStream       | PipedReader           | PipedWriter            |
| 对象流     | **ObjectInputSteam**    | **ObjectOutputStream**  |                       |                        |

## 流的基本操作

### 输入流

1. 根据要读取的路径构建 File 对象

   ```java
   File file = new File(文件路径);
   ```

2. 按照需求构建对应流对象

   :::info

   1. 判断是字节流还是字符流
      - 对于文本文件(.txt,.java,.c,.cpp)，使用字符流处理
      - 对于非文本文件(.jpg,.mp3,.mp4,.avi,.doc,.ppt,...)，使用字节流处理
   2. 是否需要使用处理流包装

   :::

   ```java
   FileReader inputStream = new FileReader(file);
   ```

3. 操作流

   ```java
   char[] cBuf = new char[1024];
   int len;
   while ((len = inputStream.read(cBuf)) != -1) {
       System.out.print(new String(cBuf, 0, len));
   }
   ```

   

4. 关闭流

   ```java
   inputStream.close();
   ```

####  详细操作

```java
@Test
public void baseReadTextTest() {
    // 1. 根据要读取的路径构建 File 对象
    File file = new File("src/main/java/io/1.txt");
    FileReader fileReader = null;
    try {
        // 2. 按照需求构建对应流对象
        fileReader = new FileReader(file);
        // 3. 操作流
        char[] cBuf = new char[1024];
        int len;
        while ((len = fileReader.read(cBuf)) != -1) {
            System.out.print(new String(cBuf, 0, len));
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        // 4.关闭流
        if (fileReader != null) {
            try {
                fileReader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 输出流

1. 根据要写出的路径(保存路径)构建 File 对象

   ```java
   File file = new File(保存路径);
   ```

2. 按照需求构建对应流对象

   :::info

   1. 判断是字节流还是字符流
      - 对于文本文件(.txt,.java,.c,.cpp)，使用字符流处理
      - 对于非文本文件(.jpg,.mp3,.mp4,.avi,.doc,.ppt,...)，使用字节流处理
   2. 是否需要使用处理流包装

   :::

   ```java
   FileWriter fileWriter = new FileWriter(file);
   ```

3. 操作流

   ```java
   fileWriter.write("hello world");
   // 刷新流
   fileWriter.flush();
   fileWriter.write("lalalala");
   ```

4. 关闭流

   ```java
   fileWriter.close();
   ```

#### 详细操作

```java
@Test
public void baseWriteTextTest() {
    // 1. 根据要写出的路径(保存路径)构建 File 对象
    File file = new File("src/main/java/io/2.txt");
    // 2. 按照需求构建对应流对象
    FileWriter fileWriter = null;
    try {
        fileWriter = new FileWriter(file);
        // 3. 操作流
        fileWriter.write("hello world");
        // 刷新流
        fileWriter.flush();
        fileWriter.write("lalalala");
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        //4. 关闭流
        if (fileWriter != null) {
            try {
                fileWriter.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 缓冲流

:::info

缓冲流 是 处理流

为了提高数据读写的速度，Java API 提供了带缓冲功能的流类，在使用这些流类时，会创建一个内部缓冲区数组，缺省使用 8192 个字节 (8Kb) 的缓冲区。

:::

### 特点

1. 当读取数据时，数据按块读入缓冲区，其后的读操作则直接访问缓冲区
2. 当使用 BufferedInputStream 读取字节文件时，BufferedInputStream 会一次性从文件中读取 8192 个 (8Kb)，存在缓冲区中，直到缓冲区装满了，才重新从文件中读取下一个 8192 个字节数组
3. 向流中写入字节时，不会直接写到文件，先写到缓冲区中直到缓冲区写满，BufferedOutputStream 才会把缓冲区中的数据一次性写到文件里。使用方法 flush() 可以强制将缓冲区的内容全部写入输出流
4. 关闭流的顺序和打开流的顺序相反。只要关闭最外层流即可，关闭最外层流也会相应关闭内层节点流

### 使用缓冲流实现文件复制

```java
@Test
public void fileCopy()  {
    String targetPath = "src/main/java/io/2.txt";
    String destPath = "src/main/java/io/2_copy.txt";
    BufferedInputStream bis = null;
    BufferedOutputStream bos = null;
    try {
        // 使用缓冲流
        bis = new BufferedInputStream(new FileInputStream(targetPath));
        bos= new BufferedOutputStream(new FileOutputStream(destPath));
        byte[] buffer = new byte[1];
        int len;
        while ((len = bis.read(buffer)) != -1) {
            bos.write(buffer, 0, len);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        // 流关闭
        // 要求：先关闭外层的流，再关闭内层的流
        if (bos != null) {
            try {
                bos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (bis != null) {
            try {
                bis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 转换流

:::info

转换流提供了在字节流和字符流之间的转换

:::

### InputStreamReader

将 InputStream 转换为 Reader

实现将字节的输入流按指定字符集转换为字符的输入流。

#### 构造器签名

```java
public InputStreamReader(InputStream in);
public InputSreamReader(InputStream in,String charsetName);
public InputStreamReader(InputStream in, Charset cs);
```

#### 例子

```java
@Test
public void InputStreamReaderTest() throws IOException {
    String targetPath = "src/main/java/io/2.txt";
    FileInputStream fis = new FileInputStream(targetPath);
    InputStreamReader fsr = new InputStreamReader(fis, StandardCharsets.UTF_8);
    char[] cBuf = new char[1024];
    int len;
    while ((len = fsr.read(cBuf)) != -1) {
        String str = new String(cBuf, 0, len);
        System.out.println(str);
    }
    fsr.close();
}
```

### OutputStreamWriter

将 Writer 转换为 OutputStream

实现将字符的输出流按指定字符集转换为字节的输出流。

#### 构造器签名

```java
public OutputStreamWriter(OutputStream out);
public OutputStreamWriter(OutputStream out, String charsetName);
public OutputStreamWriter(OutputStream out, Charset cs);
```

### 将文件 UTF-8 转 GBK

```java
@Test
@SneakyThrows
public void UTF8ToGBK() {
    String targetPath = "src/main/java/io/2.txt";
    String destPath = "src/main/java/io/2_gbk.txt";
    FileInputStream fis = new FileInputStream(targetPath);
    InputStreamReader fsr = new InputStreamReader(fis, StandardCharsets.UTF_8);
    FileOutputStream fos = new FileOutputStream(destPath);
    OutputStreamWriter osw = new OutputStreamWriter(fos, "gbk");
    char[] cBuf = new char[1024];
    int len;
    while ((len = fsr.read(cBuf)) != -1) {
        osw.write(cBuf, 0, len);
    }
    osw.close();
    fsr.close();
}
```

## 随机存取文件流

:::info

1. 这个流是既可以写也可以读
   - 可以通过设置「访问模式」来确定「读/写/读写」

2. 支持 「随机访问」 的方式，程序可以直接跳到文件的任意地方来读、写文件

   - 支持只访问文件的部分内容

   - 可以向已存在的文件后追加内容

3. 

:::

### 构造器签名

```java
public RandomAccessFile(String name, String mode);
public RandomAccessFile(File file, String mode)
```

### 访问模式

1. `r` : 以只读方式打开
2. `rw` ：打开以便读取和写入
3. `rwd` : 打开以便读取和写入；同步文件内容的更新
4. `rws` : 打开以便读取和写入；同步文件内容和元数据的更新

> 如果模式为只读 `r`。则不会创建文件，而是会去读取一个已经存在的文件，如果读取的文件不存在则会出现异常。如果模式为 `rw` 读写。如果文件**不存在**则会去创建文件，如果**存在**则不会创建而是在**头部开始覆盖**要避免这个情况因为如果原来是中文的再用英文去覆盖会出现乱码

### 实现在 rw 模式下追加文本

```java
@Test
@SneakyThrows
public void RandomAccessFileAppend() {
    File file = new File("src/main/java/io/2.txt");
    RandomAccessFile raf = new RandomAccessFile(file, "rw");
    // 设置当前光标位置
    raf.seek(file.length());
    raf.write("我".getBytes());
    raf.close();
}
```

## 对象流

::: info

用于存储和读取基本数据类型数据或对象的处理流。它的强大之处就是可以把 Java 中的对象写入到数据源中，也能把对象从数据源中还原回来。

:::

### 特点

1. ObjectOutputStream 和 ObjectInputStream 不能序列化 static 和 transient 修饰的成员变量

2. 序列化的好处在于可将任何实现了 Serializable 接口的对象转化为字节数据，使其在保存和传输时可被还原
3. 如果需要让某个对象支持序列化机制，则必须让对象所属的类及其属性是可序列化的，为了让某个类是可序列化的，该类必须实现如下两个接口之一。
   - Serializable
   - Externalizable 

### 简单使用

```java
@Test @SneakyThrows
public void ObjectStreamTest() {
    File file = new File("src/main/java/io/object.dat");
    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
    oos.writeObject(new User(10, "xiaou", "123456"));
    oos.flush();
    oos.close();
    ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
    User user = (User) ois.readObject();
    System.out.println(user);
}
```

```java User.java
@AllArgsConstructor  @Data
@NoArgsConstructor @ToString
public class User implements Serializable {
    public static final long serialVersionUID = 475463534532L;
    private int id;
    private String username;
    // 密码不持久化
    private transient String password;
}
```

```java 输出
// 结果
User(id=10, username=xiaou, password=null)
```

### 深克隆

```java
public Object deepClone() throws IOException, ClassNotFoundException {
    ByteArrayOutputStream out = new ByteArrayOutputStream();
    ObjectOutputStream oout = new ObjectOutputStream(out);
    oout.writeObject(this);

    ByteArrayInputStream in = new ByteArrayInputStream(out.toByteArray());
    ObjectInputStream iin = new ObjectInputStream(in);
    return iin.readObject();
}
```

## 标准输入/输出流

### 描述

System.in 和 System.out 分别代表了系统标准的输入和输出设备

默认输入设备是：键盘，输出设备是：显示器

1. System.in 的类型是 InputStream
2. System.out 的类型是 PrintStream，其是 OutputStream 的子类 FilterOutputStream 的子类

重定向：通过 System 类的 setIn，setOut 方法对默认设备进行改变。

### 方法签名

```java
public static void setIn(InputStream in);
public static void setOut(PrintStream out);
```

### 输入后转成大写输出

```java
@Test @SneakyThrows
public void stdTest() {
    InputStreamReader isr = new InputStreamReader(System.in);
    BufferedReader br = new BufferedReader(isr);
    while (true) {
        System.out.println("请输入字符串");
        String data = br.readLine();
        if ("e".equalsIgnoreCase(data) || "exit".equalsIgnoreCase(data)) {
            System.out.println("程序结束");
            break;
        }
        String upperCase = data.toUpperCase();
        System.out.println(upperCase);
    }
}
```

## 打印流

:::info

实现将基本数据类型的数据格式转化为字符串输出

:::

### 描述

打印流：PrintStream 和 PrintWriter

1. 提供了一系列重载的 print() 和 println() 方法，用于多种数据类型的输出
2. PrintStream 和 PrintWriter 的输出不会抛出 IOException 异常
3. PrintStream 和 PrintWriter 有自动 flush 功能
4. PrintStream 打印的所有字符都使用平台的默认字符编码转换为字节。在需要写入字符而不是写入字节的情况下，应该使用 PrintWriter 类
5. System.out 返回的是 PrintStream 的实例

```java
@Test @SneakyThrows
public void PrintStreamTest() {
    String dict = "src/main/java/io/a.log";
    FileOutputStream fos = new FileOutputStream(dict);
    // 创建打印输出流,设置为自动刷新模式(写入换行符或字节 '\n' 时都会刷新输出缓冲区)
    PrintStream ps = new PrintStream(fos, true);
    // 把标准输出流(控制台输出)改成文件
    System.setOut(ps);
    for (int i = 'a'; i <= 'z'; i++) { // 输出ASCII字符
        System.out.print((char) i);
        System.out.println();
    }
    ps.close();
}
```

