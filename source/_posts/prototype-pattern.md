---
title: 原型模式
date: 2018-09-08 10:04:18
tags:
    - design pattern
categories: design pattern
description: 设计模式--原型模式
---
### 定义
> Specify the kinds of objects to create using a prototypical instance, and craete new objects by copying this prototype<br/>**用原型模式制定创建对象的种类, 并且通过拷贝这些原型创建新的对象**

原型模式的核心是clone方法, 通过该方法进行对象的拷贝, java提供了一个Cloneable接口来标示这个对象是可以拷贝的, Cloneable只是一个标识, Cloneable自身没有任何方法, 同样的接口还有Serializable, 这样的接口只是起到一个标记作用, 然后, 只需要重写Object的clone方法就可以了

### 原型模式的应用
#### 优点
##### 1. 性能优良
原型模式是在内存中二进制流的拷贝, 要比直接new一个对象性能好很多,特别是要在循环体内产生大量的对象时, 原型模式可以更好的体现其优点
##### 2. 逃避构造函数的约束
是优点也是缺点, 直接在内存中拷贝, 构造函数是不会执行的
#### 使用场景
##### 1. 资源优化场景
类的初始化需要消耗太多的资源
##### 2. 性能和安全要求的场景
new一个对象需要非常繁琐的数据准备, 使用原型模式避免new对象
##### 3. 一个对象多个修改者的场景
一个对象需要提供给多个调用者,并且都需要修改其值的时候,可以使用原型模式

**在真实的项目中, 原型模式很少单独出现, 一般是和工厂方法模式一起出现, 通过clone创建一个对象, 然后由工厂方法提供给调用者.**

### 原型模式的例子
批量发送广告邮件的例子: <br/>

广告信模板
```java
/**
 * 广告信模板
 * @author zonzie
 * @date 2018/9/8 10:12
 */
public class AdvTemplate {
    private String advSubject = "信用卡抽奖活动";

    private String advContent = "抽奖通知: 刷卡就送一百万!!!";

    public String getAdvSubject() {
        return advSubject;
    }

    public void setAdvSubject(String advSubject) {
        this.advSubject = advSubject;
    }

    public String getAdvContent() {
        return advContent;
    }

    public void setAdvContent(String advContent) {
        this.advContent = advContent;
    }
}
```

邮件类
```java
/**
 * 邮件类
 * @author zonzie
 * @date 2018/9/8 10:16
 */
public class Mail implements Cloneable, Serializable {

    // 收件人
    private String receiver;

    // 邮件名称
    private String subject;

    // 称谓
    private String application;

    // 内容
    private String context;

    // 邮件的尾部
    private String tail;

    public Mail(AdvTemplate advTemplate) {
        this.context = advTemplate.getAdvContent();
        this.subject = advTemplate.getAdvSubject();
    }

    /**
     * 复写的clone方法
     * @throws CloneNotSupportedException
     */
    @Override
    protected Mail clone() throws CloneNotSupportedException {
        return (Mail) super.clone();
    }

    public String getReceiver() {
        return receiver;
    }

    public void setReceiver(String receiver) {
        this.receiver = receiver;
    }

    public String getSubject() {
        return subject;
    }

    public void setSubject(String subject) {
        this.subject = subject;
    }

    public String getApplication() {
        return application;
    }

    public void setApplication(String application) {
        this.application = application;
    }

    public String getContext() {
        return context;
    }

    public void setContext(String context) {
        this.context = context;
    }

    public String getTail() {
        return tail;
    }

    public void setTail(String tail) {
        this.tail = tail;
    }

    @Override
    public String toString() {
        final StringBuilder sb = new StringBuilder("Mail{");
        sb.append("receiver='").append(receiver).append('\'');
        sb.append(", subject='").append(subject).append('\'');
        sb.append(", application='").append(application).append('\'');
        sb.append(", context='").append(context).append('\'');
        sb.append(", tail='").append(tail).append('\'');
        sb.append('}');
        return sb.toString();
    }
}
```

场景测试类
```java
import org.junit.Test;

import java.util.Random;

/**
 * 场景测试类
 * @author zonzie
 * @date 2018/9/8 10:30
 */
public class Client {

    /**
     * 发送的数量
     */
    private int maxCount = 10;

    @Test
    public void sendTest() throws CloneNotSupportedException {
        // 发送邮件
        int i = 0;
        // 定义发送模板
        Mail mail = new Mail(new AdvTemplate());
        mail.setTail("xx公司版权所有");
        while(i++ < maxCount) {
            // 邮件不同的地方
            Mail clone = mail.clone();
            clone.setApplication(randomString(5) + " Mr/Mrs");
            clone.setReceiver(randomString(5) + "@" + randomString(8) + ".com");
            sendMail(clone);
        }
    }

    /**
     * 发送邮件
     */
    private void sendMail(Mail mail) {
        System.out.println(mail);
    }


    /**
     * 生成随机的字符串
     */
    public String randomString(int i) {
        Random random = new Random();
        StringBuilder stringBuilder = new StringBuilder();
        for(int j = 0; j <= i; j++) {
            int randNum = random.nextInt(26) + 97;
            stringBuilder.append((char) randNum);
        }
        return stringBuilder.toString();
    }
}
```

### 注意事项
#### 考虑深拷贝和浅拷贝的区别
- Object对象提供的clone方法只是拷贝本对象, 对象内部的数组, 引用都不拷贝, 还是还是指向原生对象的内部元素地址,这种拷贝就叫做浅拷贝
- 使用原型模式, 引用类型的成员变量只有满足两个条件才不能拷贝: 
    1. 类的成员变量,而不是方法内部的变量 
    2. 必须是一个可变的引用对象, 而不是一个原始类型或者不可变对象

##### 浅拷贝的例子
```java
import org.junit.Test;

import java.util.ArrayList;

/**
 * 浅拷贝
 * @author zonzie
 * @date 2018/9/8 12:54
 */
public class ShallowCopy implements Cloneable {
    /**
     * 引用类型的变量
     */
    private ArrayList<String> list = new ArrayList<>();

    @Override
    protected ShallowCopy clone() throws CloneNotSupportedException {
        return (ShallowCopy) super.clone();
    }

    public ArrayList<String> getList() {
        return list;
    }

    public void setValue(String value) {
        this.list.add(value);
    }

    @Override
    public String toString() {
        final StringBuilder sb = new StringBuilder("ShallowCopy{");
        sb.append("list=").append(list);
        sb.append('}');
        return sb.toString();
    }

    /**
     * 测试方法
     * 运行结果: ShallowCopy{list=[jack, tom]}
     */
    @Test
    public void shallowTest() throws CloneNotSupportedException {
        ShallowCopy shallowCopy = new ShallowCopy();
        shallowCopy.setValue("jack");
        // 拷贝对象
        ShallowCopy clone = shallowCopy.clone();
        clone.setValue("tom");
        // 打印原始对象
        System.out.println(shallowCopy);
    }
}
```

##### 深拷贝的例子
```java
import org.junit.Test;

import java.util.ArrayList;

/**
 * 深拷贝
 * @author zonzie
 * @date 2018/9/8 12:54
 */
public class DeepCopy implements Cloneable {

    /**
     * 引用类型的成员变量
     */
    private ArrayList<String> list = new ArrayList<>();

    @Override
    protected DeepCopy clone() throws CloneNotSupportedException {
        DeepCopy clone = (DeepCopy) super.clone();
        clone.list =  (ArrayList<String>) this.list.clone();
        return clone;
    }

    public ArrayList<String> getList() {
        return list;
    }

    public void setValue(String value) {
        this.list.add(value);
    }

    @Override
    public String toString() {
        final StringBuilder sb = new StringBuilder("DeepCopy{");
        sb.append("list=").append(list);
        sb.append('}');
        return sb.toString();
    }

    /**
     * 测试方法
     * 运行结果: DeepCopy{list=[jack]}
     */
    @Test
    public void shallowTest() throws CloneNotSupportedException {
        DeepCopy deepCopy = new DeepCopy();
        deepCopy.setValue("jack");
        // 拷贝对象
        DeepCopy clone = deepCopy.clone();
        clone.setValue("tom");
        // 打印原始对象
        System.out.println(deepCopy);
    }
}
```

#### 对象的clone和final是有冲突的, 如果变量被final修饰, 是不可以被克隆的, 想要克隆对象, 需要去掉final关键字

#### 总之,就是先产生出一个包含大量的共有信息的类, 然后可以拷贝出副本, 再修正副本的细节

---
> **内容来自: &laquo;设计模式之禅&raquo; --- 秦小波**