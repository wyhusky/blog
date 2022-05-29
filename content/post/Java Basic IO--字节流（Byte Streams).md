---
title: "Java Basic I/O--字节流（Byte Streams)"
date: 2019-03-29T03:59:16+08:00
lastmod: 2019-03-29T03:59:16+08:00
author: ["wyhusky"]
categories: 
- Java
tags: 
- java
- io
description: ""
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: false # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: ""
    caption: ""
    alt: ""
    relative: false
---

> 字节流用来处理最基本的8bit字节的的输入输出，所有字节流的类都继承自[`InputStream`](https://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html) and [`OutputStream`](https://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html)

## InputStream & OutputStream

InputStream和OutputStream从字面上就可以看出它们一个负责字节流的输入，一个负责字节流的输出，这里的输入输出都是相对与内存而言的。这两个类都是抽象类，意味着在使用字节流时，我们需要用它们的子类来对字节进行操作。

首先我们来看一下InputStream中定义的几个重要的方法。

* `` read()`` 方法读取输入流中的下一个字节，字节的值会被转化成一个int值（0～255）返回，读到流的末尾时即没有数据的时候会返回-1来表示读取结束了。这是一个阻塞方法，并且可能会抛出IOException。

```java

    /**
     * Reads the next byte of data from the input stream. The value byte is
     * returned as an <code>int</code> in the range <code>0</code> to
     * <code>255</code>. If no byte is available because the end of the stream
     * has been reached, the value <code>-1</code> is returned. This method
     * blocks until input data is available, the end of the stream is detected,
     * or an exception is thrown.
     *
     * <p> A subclass must provide an implementation of this method.
     *
     * @return     the next byte of data, or <code>-1</code> if the end of the
     *             stream is reached.
     * @exception  IOException  if an I/O error occurs.
     */
    public abstract int read() throws IOException;
```

* `` read(byte b[]) `` 方法将流中的数据读入到传入的byte数组中，实际是调用了`` read(byte[] b, int off, int len)`` 方法，我们可以直接看这个方法

```java
    public int read(byte b[]) throws IOException {
        return read(b, 0, b.length);
    }
```

* ``read(byte b[], int off, int len)`` 这个方法将流中的数据读入到b字节数组中，参数off指定从b字节数组的哪个位置开始写，len指定读取的最大字节数。这个方法的实现也是比较容易看懂的。     
  1. 检查入参是否有空指针异常和数组越界异常。字节数组b为null，抛出`` NullPointerException `` 异常，偏移量off小于0或字节长度len小于0，亦或是指定读取的最大字节数大于数组长度减去偏移量，抛出`` IndexOutOfBoundsException`` 异常
  2. 传入len为0时返回0
  3. 如果输入流中没有数据，返回-1
  4. 如果输入流中有数据，至少读取一个字节的数据
  5. 将读取的第一个字节放入b[off]位置，第二个数据放入b[off + 2]的位置，以此类推，直至读至流中没有数据或读了len长度个数据，返回读取的字节长度

```java
    public int read(byte b[], int off, int len) throws IOException {
        if (b == null) {
            throw new NullPointerException();
        } else if (off < 0 || len < 0 || len > b.length - off) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return 0;
        }

        int c = read();
        if (c == -1) {
            return -1;
        }
        b[off] = (byte)c;

        int i = 1;
        try {
            for (; i < len ; i++) {
                c = read();
                if (c == -1) {
                    break;
                }
                b[off + i] = (byte)c;
            }
        } catch (IOException ee) {
        }
        return i;
    }
```



## 使用字节流的例子

一下例子使用 `FileInputStream` and `FileOutputStream` 来拷贝字节，这两个类分别是InputSteam和OutputSteam的子类，其他字节流的子类的用法都差不多，差别只是在字节流的构造方法上。

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class CopyBytes {
    public static void main(String[] args) throws IOException {

        FileInputStream in = null;
        FileOutputStream out = null;

        try {
            in = new FileInputStream("xanadu.txt");
            out = new FileOutputStream("outagain.txt");
            int c;

            while ((c = in.read()) != -1) {
                out.write(c);
            }
        } finally {
            // 一定要在finally中调用close，防止发生异常时有资源泄漏
            if (in != null) {
                in.close();
            }
            if (out != null) {
                out.close();
            }
        }
    }
}
```

上面这个例子大部分时间花费在一个简单的循环里，从输入流中读取一个字节写到输出流中去，如下图所示，因此字节流的效率是比较低的，一次只能读一个字节。

![](assets/byteStream.gif)

## 总结

字节流是比较低层次的I/O方式，上面的例子中，txt文件都是含有字符的，用字符流会更加高效，另外还有更加复杂的数据流，字节流只用于最原始的I/O上。但字节流又是很重要的，因为其他所有的流类型都是在字节流的基础上构建的。

## reference

[Byte Streams](<https://docs.oracle.com/javase/tutorial/essential/io/bytestreams.html>)

