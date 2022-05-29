---
title: "Serializable & Parcelable 序列化接口"
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

所谓序列化，就是将内存中的对象持久化(转换成字节序列）存储到各种不同的媒介中，当我们需要时还能够反序列化得到原来的对象。        
一般序列化有以下用途：      
* 将对象存储在硬盘上做持久化保存
* 通过网络传输对象字节序列交换数据          

在Android中想要将对象序列化有两种方式，一种是实现Serializable接口，另一种是实现Parcelable接口。

## 实现Serializable接口

```java
public class User implements Serializable {
    /*
    *serialVersionUID 用于表示序列化对象的版本，也可以缺省，缺省时JVM会自动分配
    *当反序列化时，系统会把当前类的serialVersionUID和文件中的serialVersionUID比较
    *如果两者相同才能成功反序列化
    *如果两者不同，反序列化将失败
    *有时，我们改变了原来序列化的类（如增加一个域变量），但仍指定了相同的serialVersionUID
    *只要类没有结构性的损坏，仍然可以反序列成功
    */
    private static final long serialVersionUID = 1L;

    private int userId;
    private String userNmae;

    public User(int userId, String userName) {
        this.userId = userId;
        this.userName = userName;
    }
}
```

```java
//序列化过程
User user = new User(1, "wyhusky");
ObjectOutputStream out = new ObjectOutputStream(
                                new FileOutputStream("user.txt"));
out.writeObject(user);
out.close();

//反序列化过程
ObjectInputStream in = new ObjectInputStream(
                                new FileInputStream("user.txt"));
//反序列化并非会得到原来的对象，而是一个信息相同的新对象
User newUser = (User)in.readObject();
in.close();
```

## 实现Parcelable接口

```java
public class User implements Parcelable {
    public int userId;
    public String userName;
    public Book book;

    public User(int userId, String userName, Book book) {
        this.userId = userId;
        this.userName = userName;
        this.book = book;
    }

    /*
    *Parcelable 方法
    *返回当前对象的内容描述，若含有文件描述符返回1
    *否则返回0，几乎所有情况都返回0
    */
    public int describeContents() {
        return 0;
    }

    /*
    *Parcelable 方法
    *将当前对象写入序列化结构中
    *@param flags
    *为1时标示当前对象需要作为返回值返回，不能立即释放资源
    *几乎所有情况都为0
    */
    public void writeToParcel(Parcel out, int flags) {
        out.writeInt(userId);
        out.writeString(userName);
        out.writeParcelable(book, 0);
    }

    public static final Parcelable.Creator<User> CREATOR = new Parcelable.Creator<User>() {
        //从序列化对象中创建原始对象
        public User createFromParcel(Parcel in) {
            return new User(in);
        }

        //创建指定长度的原始对象数组
        public User[] newArray(int size) {
            return new User[size];
        }
    };
    
    private User(Parcel in) {
        this.userId = in.readInt();
        this.userName = in.readString();
        //book是Parcelable对象，反序列化过程需要当前线程的上下文类加载器
        this.book = in.readParcelable(Thread.currentThread().getContextClassLoader)
    }
}
```

## 总结

Parcelable和Serializable都能够实现序列化，并且都可以用于Intent间数据传输，但各有优劣势。    
Serializable是java中的序列化方式，实现简单，但IO开销大。      
Parcelable是Android中序列方式，实现复杂，但效率高，主要用于内存序列化上（binder）。

##参考资料
* Android开发艺术探索
