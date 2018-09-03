---
title: 建造者模式
date: 2018-09-03 21:51:40
tags:
	- design pattern
categories: design pattern
description: 设计模式--建造者模式
---
### 建造者模式(生成器模式)

> separate the construction of a complex object from its representation so that the same construction process can create diffrent representations<br/>**将一个复杂对象的构建和表示分离,使得同样的构建过程可以创建不同的表示**

#### 建造者中的角色
##### Product产品类
要创建的复杂对象,通常实现了模板方法模式,也就是有模板方法和普通方法
##### Builder抽象建造者
规范产品的构建, 一般由子类实现
##### ConcreteBuilder具体建造者
实现具体的构建方法, 返回一个组建好的对象
##### Director导演类
负责安排已有模块的建造顺序, 然后告诉builder开始建造

#### 使用场景
- 相同的方法,不同的执行顺序, 产生的事件的结果不同,可以使用建造者模式
- 多个部件和零件, 都可以装配到一个对象中, 但是产生的运行结果又不相同时, 可以使用该模式
- 产品类字段很多, 不同的调用顺序产生不同的效能时,使用建造者模式非常合适

#### 一个建造者模式的demo
##### 首先提供产品类
```java
/**
 * 组装一台电脑,假设它有三个部件
 * @author zonzie
 * @date 2018/9/3 20:44
 */
public class Computer {
    private String cpu;
    private String mainBoard;
    private String ram;

    public String getCpu() {
        return cpu;
    }

    public void setCpu(String cpu) {
        this.cpu = cpu;
    }

    public String getMainBoard() {
        return mainBoard;
    }

    public void setMainBoard(String mainBoard) {
        this.mainBoard = mainBoard;
    }

    public String getRam() {
        return ram;
    }

    public void setRam(String ram) {
        this.ram = ram;
    }

    @Override
    public String toString() {
        final StringBuilder sb = new StringBuilder("Computer{");
        sb.append("cpu='").append(cpu).append('\'');
        sb.append(", mainBoard='").append(mainBoard).append('\'');
        sb.append(", ram='").append(ram).append('\'');
        sb.append('}');
        return sb.toString();
    }
}
```

##### 然后是抽象的builder
```java
/**
 * 一个抽象的建造者
 * @author zonzie
 * @date 2018/9/3 20:49
 */
public abstract class AbstractBuilder {
    public abstract void buildCpu(String cpu);
    public abstract void buildMainBoard(String mainBoard);
    public abstract void buildRam(String ram);
    public abstract Computer create();
}

```

##### 建造者的具体实现
```java
/**
 * 建造者的实现类
 * @author zonzie
 * @date 2018/9/3 20:52
 */
public class ComputerBuilder extends AbstractBuilder{

    // 创建一个computer对象
    private Computer computer = new Computer();

    @Override
    public void buildCpu(String cpu) {
        computer.setCpu(cpu);
    }

    @Override
    public void buildMainBoard(String mainBoard) {
        computer.setMainBoard(mainBoard);
    }

    @Override
    public void buildRam(String ram) {
        computer.setRam(ram);
    }

    @Override
    public Computer create() {
        return this.computer;
    }
}
```

##### 导演类,指挥建造者建造的流程
```java
/**
 * 指挥者类,指挥建造流程
 * @author zonzie
 * @date 2018/9/3 20:57
 */
public class Director {
    private ComputerBuilder build = null;

    public Director(ComputerBuilder build) {
        this.build = build;
    }

    /**
     * 返回建造好的实体
     * @param cpu
     * @param mainBoard
     * @param ram
     * @return
     */
    public Computer createComputer(String cpu, String mainBoard, String ram) {
        // 规范建造流程
        this.build.buildMainBoard(mainBoard);
        this.build.buildCpu(cpu);
        this.build.buildRam(ram);
        return build.create();
    }
}
```

##### 测试
```java
import org.junit.Test;

/**
 * @author zonzie
 * @date 2018/9/3 21:03
 */
public class BuilderTest {

    @Test
    public void buildTest() {
        ComputerBuilder computerBuilder = new ComputerBuilder();
        Director director = new Director(computerBuilder);
        Computer computer = director.createComputer("i7", "intel主板", "8G");
        System.out.println(computer);
    }
}
```

#### 建造者模式的优点
- 封装性: 使用者不需要关注产品内部的实现细节
- 建造者独立,容易扩展
- 便于控制细节风险, 具体的建造者是独立的, 可以对建造过程逐步细化, 不对其他模块产生影响

#### 其他
在实际开发中,往往会省略Director角色, 直接使用builder对象直接进行组装