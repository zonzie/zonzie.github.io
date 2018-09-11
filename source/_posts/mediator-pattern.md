---
title: mediator_pattern
date: 2018-09-11 18:14:01
tags:
    - design pattern
categories: design pattern 
description: 设计模式-中介者模式
---

### 定义
> Define an object that encapsulates how a set of objects interact. Mediator Promotes loose coupling by keeping objects from referring to each other explicitly, and it lets you vary their interation independently.<br/>**用一个中介对象封装一系列的对象交互,中介者使各个对象不需要显式的相互作用, 从而使其耦合松散, 而且可以独立的改变他们的交互**

### 一个不使用中介者模式的进销存的例子
这个例子中, 我们简单创建几个部分: 采购管理, 销售管理, 库存管理部分,实现货物的基本的采购,销售,库存管理的功能
采购管理部分:
```java
/**
 * 采购管理
 * @author zonzie
 * @date 2018/9/10 18:58
 */
public class Purchase {

    /** 采购IBM电脑 */
    public void buyIBMComputer(int number) {
        // 访问库存
        Stock stock = new Stock();
        // 访问销售
        Sale sale = new Sale();
        // 电脑的销售情况
        int saleStatus = sale.getSaleStatus();
        if(saleStatus > 80) {
            System.out.println("采购电脑: " + number + "台");
        } else { // 销售情况不好
            int buyNumber = number / 2;
            System.out.println("采购IBM电脑: " + buyNumber + "台");
        }
    }

    /** 不再采购 */
    public void refuseBuyIBM() {
        System.out.println("不再采购IBM电脑");
    }
}
```

销售管理:
```java
/**
 * 销售管理
 * @author zonzie
 * @date 2018/9/10 19:06
 */
public class Sale {

    /** 销售IBM电脑 */
    public void sellIBMComputer(int number) {
        // 访问库存
        Stock stock = new Stock();
        // 访问采购
        Purchase purchase = new Purchase();
        if(stock.getStockNumber() < number) {
            purchase.buyIBMComputer(number);
        }
        System.out.println("销售IBM电脑" + number + "台");
        stock.decrease(number);
    }

    /** 反馈销售情况 */
    public int getSaleStatus() {
        Random random = new Random(System.currentTimeMillis());
        int saleStatus = random.nextInt(100);
        System.out.println("IBM电脑的销售情况是: " + saleStatus);
        return saleStatus;
    }

    /** 折价处理 */
    public void offSale() {
        Stock stock = new Stock();
        System.out.println("电脑折价销售: " + stock.getStockNumber() + "台");
    }
}
```

库存管理:
```java
/**
 * 库存管理
 * @author zonzie
 * @date 2018/9/10 19:00
 */
public class Stock {
    /** 100台电脑 */
    private static int COMPUTER_NUMBER = 100;

    /** 库存增加 */
    public void increase(int number) {
        COMPUTER_NUMBER += number;
        System.out.println("库存数量为: " + COMPUTER_NUMBER);
    }

    /** 库存减少 */
    public void decrease(int number) {
        COMPUTER_NUMBER -= number;
        System.out.println("库存数量为: " + COMPUTER_NUMBER);
    }

    /** 获得库存数量 */
    public int getStockNumber() {
        return COMPUTER_NUMBER;
    }

    /** 存货压力增大, 清库存 */
    public void clearStock() {
        Purchase purchase = new Purchase();
        Sale sale = new Sale();
        System.out.println("清理库存数量: " + COMPUTER_NUMBER);
        // 要求折价销售
        sale.offSale();
        // 要求采购人员不要采购
        purchase.refuseBuyIBM();
    }
}
```

场景类:
```java
/**
 * 场景类
 * @author zonzie
 * @date 2018/9/10 19:36
 */
public class Client {

    public static void main(String[] args) {
        // 采购人员采购电脑
        System.out.println("采购人员采购电脑");
        Purchase purchase = new Purchase();
        purchase.buyIBMComputer(100);
        // 销售电脑
        System.out.println("销售电脑");
        Sale sale = new Sale();
        sale.sellIBMComputer(1);
        // 库存扣减
        System.out.println("清理库存");
        Stock stock = new Stock();
        stock.clearStock();
    }
}
```
在这个例子中, 三个对象之间互相调用, 每个部分要实现功能都必须依赖其他的部分完成,如果再加入物流,资产等管理模块, 代码整体会更加的复杂, 维护成本将越来越大
这里, 引入中介者的概念, 不同模块之间, 只处理自己的逻辑, 其余部分交给中介者去管理
各个部分之间的复杂关系:
![复杂关系](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/%E5%A4%8D%E6%9D%82%E5%85%B3%E7%B3%BB.png)

### 中介者模式(也叫调停者模式)中的组成部分
1. Mediator 抽象中介者角色, 定义统一的接口, 用于各同事角色之间的通信
2. Concrete Mediator 具体中介者角色, 协调各同事角色实现协作行为, 必须依赖各个同事角色
3. Colleague 同事角色, 每个同事角色,都知道中介者的角色, 而且, 在需要与其他同事角色交互时, 必须通过中介者与其他角色交互,同事类的行为分为两种:
    1. 自发行为: 同事自身的行为, 比如, 改变对象自身的状态, 处理自身的行为
    2. 依赖方法: 依赖中介者才能完成的行为

### 使用中介者模式的进销存的例子

抽象的中介者: 
```java
/**
 * 抽象的中介者
 * @author zonzie
 * @date 2018/9/10 19:47
 */
public abstract class AbstractMediator {
    protected Purchase purchase;
    protected Sale sale;
    protected Stock stock;

    // 使用构造函数注入同事类, 这里如果不是必须的同事类, 可以使用setter/getter注入
    public AbstractMediator() {
        this.purchase = new Purchase(this);
        this.sale = new Sale(this);
        this.stock = new Stock(this);
    }

    // 中介者最重要的方法, 处理多个对象之间的关系
    public abstract void execute(String str, Object... objects);
}
```

在抽象类Mediator中只定义了同事类的注入, 为什么使用同事类注入, 而不使用同事类的实现类注入呢? 那是因为同事类虽然有抽象, 但是没有每个同事类必须要完成的业务方法, 当然, 如果每个同事类都有相同的方法, 比如execute, handler, 当然要注入抽象类, 做到依赖倒置

具体的中介者,一般只有一个:
```java
/**
 * 中介者
 * @author zonzie
 * @date 2018/9/10 20:08
 */
public class Mediator extends AbstractMediator {

    @Override
    public void execute(String str, Object... objects) {
        // 采购电脑
        if("purchase.buy".equals(str)) {
            this.buyComputer((Integer) objects[0]);
            // 销售电脑
        } else if("sale.sell".equals(str)) {
            this.sellComputer((Integer) objects[0]);
            // 折价销售
        } else if("sale.offset".equals(str)) {
            this.offSail();
            // 出清
        } else if("stock.clear".equals(str)) {
            this.clearStock();
        }
    }

    /** 采购电脑 */
    private void buyComputer(int number) {
        int saleStatus = super.sale.getSaleStatus();
        if(saleStatus > 80) {
            System.out.println("采购IBM电脑" + number + "台");
            super.stock.increase(number);
        } else {
            int buyNumber = number / 2; // 折半采购
            System.out.println("采购IBM电脑" + buyNumber + "台");
        }
    }

    /** 销售电脑 */
    public void sellComputer(int number) {
        if(super.stock.getStockNumber() < number) { // 库存数量不够销售
            super.purchase.buyIBMComputer(number);
        }
        super.stock.decrease(number);
    }

    /** 折价销售电脑 */
    public void offSail() {
        System.out.println("折价销售IBM电脑" + stock.getStockNumber() + "台");
    }

    /** 清仓处理 */
    private void clearStock() {
        // 要求清仓销售
        super.sale.offSale();
        // 要求采购人员不要采购
        super.purchase.refuseBuyIBM();
    }
}
```

抽象的同事类:
```java
/**
 * 抽象的同事类
 * @author zonzie
 * @date 2018/9/10 19:58
 */
public abstract class AbstractColleague {
    protected AbstractMediator mediator;

    public AbstractColleague(AbstractMediator mediator) {
        this.mediator = mediator;
    }
}
```

采购管理:
```java
/**
 * 修改后的采购管理
 * @author zonzie
 * @date 2018/9/10 19:56
 */
public class Purchase extends AbstractColleague {

    public Purchase(AbstractMediator mediator) {
        super(mediator);
    }

    // 采购电脑
    public void buyIBMComputer(int number) {
        super.mediator.execute("purchase.buy", number);
    }

    // 不再采购电脑
    public void refuseBuyIBM() {
        System.out.println("不再采购电脑");
    }
}
```

销售管理:
```java
import java.util.Random;

/**
 * 销售管理
 * @author zonzie
 * @date 2018/9/10 20:03
 */
public class Sale extends AbstractColleague {

    public Sale(AbstractMediator mediator) {
        super(mediator);
    }

    /** 销售电脑 */
    public void sellIBMComputer(int number) {
        this.mediator.execute("sale.sell", number);
        System.out.println("销售电脑" + number + "台");
    }

    /** 销售情况 */
    public int getSaleStatus() {
        Random random = new Random(System.currentTimeMillis());
        int saleStatus = random.nextInt(100);
        this.sellIBMComputer(saleStatus);
        System.out.println("电脑的销售情况: " + saleStatus);
        return saleStatus;
    }

    /** 出清 */
    public void offSale() {
        this.mediator.execute("sale.offsale");
    }
}
```

库存管理:
```java
/**
 * 库存管理
 * @author zonzie
 * @date 2018/9/10 20:03
 */
public class Stock extends AbstractColleague{

    public Stock(AbstractMediator mediator) {
        super(mediator);
    }

    // 100台电脑
    private static int COMPUTER_NUMBER = 100;

    // 库存增加
    public void increase(int number) {
        COMPUTER_NUMBER += number;
        System.out.println("库存数量为" + COMPUTER_NUMBER + "台");
    }

    // 库存降低
    public void decrease(int number) {
        COMPUTER_NUMBER -= number;
        System.out.println("库存数量为" + COMPUTER_NUMBER + "台");
    }

    // 获得库存数量
    public int getStockNumber() {
        return COMPUTER_NUMBER;
    }

    // 存货压力大, 通知采购人员不要采购, 销售人员尽快销售
    public void clearStock() {
        System.out.println("清理存货数量为: " + COMPUTER_NUMBER);
        super.mediator.execute("stock.clear");
    }
}
```

同事类必须有中介者, 所以用构造函数注入, 而中介者可以只有部分同事类, 因此,可以使用setter/getter注入
使用中介者模式后, 形成的星型结构:
![星型结构](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/%E6%98%9F%E5%9E%8B%E7%BB%93%E6%9E%84.png)

### 中介者模式的优点
减少类间的依赖,把原有的一对多的依赖变成了一对一的依赖, 同事类只依赖中介者, 减少了依赖,当然同时也降低了类间的耦合

### 中介者模式的缺点
中介者会膨胀的很大, 而且逻辑复杂, 原本多个对象的逻辑会全部放在中介者中,同事类越多, 中介者的逻辑就越复杂

### 中介者模式的使用场景
中介者模式容易被误用, 一个对象与多个对象存在依赖关系是必然的情况, 但是并不代表就要使用中介者模式, 中介者模式适用于多个对象之间紧密耦合的情况, 紧密耦合的标准是,在类图中出现了蜘蛛网状结构,这种情况下一定要考虑使用中介者模式,这有利于把蜘蛛网状的结构梳理为星型结构,使原本复杂混乱的关系变得清晰简单

#### 实际应用
- 机场调度中心
- MVC框架
    - 其中的C(Controller) 就是一个中介者,叫做前端控制器, 作用就是把M(Model)和V(view)隔离开, 并且把M运行的结果和V代表的视图糅合成前端可以展示的页面,减少M和V的依赖关系
- 媒体网关
- 中介服务

---
> 内容来自: &laquo;设计模式之禅&raquo;