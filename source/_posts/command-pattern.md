---
title: command_pattern
date: 2018-11-19 14:50:55
tags: 
    - design pattern
categories: design pattern
description: 设计模式--命令模式
---
### 命令模式

> Ebcapsulate a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.<br>**将一个请求封装成一个对象, 从而让你使用不同的请求把客户端参数化, 对请求排队或者记录请求日志, 可以提供命令的撤销和恢复功能**

#### 通用的命令模式类图
![command pattern](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/raw_command_pattern.jpg)
<center>通用类图</center>

#### 命令模式中的角色
##### Receive接收者角色
实际执行操作的角色
##### Command命令角色
需要执行的命令都在这里
##### Invoker调用者角色
接受命令, 执行命令

##### 通用代码
```java
// 通用Receiver
public abstract class Receiver {
    // 抽象接受者, 每个接收者都必须完成的任务
    public abstract void doSomething();
}

// 具体的Receiver
public class ConcreteReceiver1 extends Receiver {
    // 每个接收者必须处理一定的业务逻辑
    public void doSomething() {
    }
}

// 具体的Receiver
public class ConcreteReceiver2 extends Receiver {
    // 每个接收者必须处理一定的业务逻辑
    public void doSomething() {
    }
}

// 抽象的Command
public abstract class Command {
    // 每一个命令类都必须有一个执行命令的方法
    public abstract void execute();
}

// 具体的Command类, 可以有多个
public class ConcretCommand1 extends Command {
    // 对哪个Receiver进行命令处理
    private Receiver receiver;

    // 构造函数传递接收者
    public ConcretCommand1(Receiver receiver) {
        this.receiver = receiver;
    }

    // 实现命令
    public void execute() {
        // 业务处理
        this.receiver.doSomething();
    }
}

// 调用者Invoker
public class Invoker {
    private Command command;

    // 传入命令
    public void setCommand(Command command) {
        this.command = command;
    }

    // 执行命令
    public void action() {
        this.command.execute();
    }
}

// 场景类
public class Client {
    public static void main(String... args) {
        // 声明invoker
        Invoker invoker = new Invoker();
        // 定义接收者
        Receiver receiver = new ConcreteReceiver();
        // 定义发送给接收者的命令
        Command command = new ConcreteCommand1(receiver);
        // 把命令交给调用者去执行
        invoker.setCommand(command);
        invoker.action();
    }
}
```

##### 开发项目的例子
- 类图
![demo_command_pattern](https://hexoblog-1255784309.cos.ap-beijing.myqcloud.com/demo_command_pattern.jpg)
<center>demo类图</center>

```java
/**
 * 抽象组
 * @author zonzie
 * @date 2018/11/19 12:43 PM
 */
public abstract class Group {

    //找到group
    public abstract void find();

    // 增加功能
    abstract void add();

    // 删除功能
    abstract void delete();

    // 修改功能
    abstract void change();

    // 给出所有的变更计划
    abstract void plan();
}

/**
 * 代码组
 * @author zonzie
 * @date 2018/11/19 12:54 PM
 */
public class CodeGroup extends Group {

    @Override
    public void find() {
        System.out.println("找到代码组");
    }

    @Override
    void add() {
        System.out.println("添加功能");
    }

    @Override
    void delete() {
        System.out.println("删除功能");
    }

    @Override
    void change() {
        System.out.println("修改功能");
    }

    @Override
    void plan() {
        System.out.println("代码变更计划");
    }
}

/**
 * 需求组
 * @author zonzie
 * @date 2018/11/19 12:49 PM
 */
public class RequirementGroup extends Group {

    /**
     * 找到需求组
     */
    @Override
    public void find() {
        System.out.println("找到需求组");
    }

    @Override
    void add() {
        System.out.println("增加一项需求");
    }

    @Override
    void delete() {
        System.out.println("修改需求");
    }

    @Override
    void change() {
        System.out.println("修改需求");
    }

    @Override
    void plan() {
        System.out.println("需求变更计划");
    }
}

/**
 * 美工组
 * @author zonzie
 * @date 2018/11/19 12:52 PM
 */
public class PageGroup extends Group {


    @Override
    public void find() {
        System.out.println("找到美工组");
    }

    @Override
    void add() {
        System.out.println("增加一个页面");
    }

    @Override
    void delete() {
        System.out.println("删除页面");
    }

    @Override
    void change() {
        System.out.println("修改页面");
    }

    @Override
    void plan() {
        System.out.println("页面变更计划");
    }
}

/**
 * 抽象命令类
 * @author zonzie
 * @date 2018/11/19 1:39 PM
 */
public abstract class Command {

    // 定义好, 子类可以直接使用

    /**
     * 需求
     */
    protected RequirementGroup rg = new RequirementGroup();

    /**
     * 美工
     */
    protected PageGroup pg = new PageGroup();

    /**
     * 代码
     */
    protected CodeGroup cg = new CodeGroup();

    /**
     * 执行命令的方法
     */
    public abstract void execute();
}

/**
 * 添加需求的命令
 * @author zonzie
 * @date 2018/11/19 1:44 PM
 */
public class AddRequirementCommond extends Command {

    @Override
    public void execute() {
        super.pg.find();
        super.pg.add();
        super.pg.plan();
    }
}

/**
 * 删除页面的命令
 * @author zonzie
 * @date 2018/11/19 1:47 PM
 */
public class DeletePageCommand extends Command {

    @Override
    public void execute() {
        pg.find();
        pg.delete();
        pg.plan();
    }
}

/**
 * 负责人类
 * @author zonzie
 * @date 2018/11/19 1:48 PM
 */
public class Invoker {

    /**
     * 要执行的命令
     */
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    /**
     * 执行客户的命令
     */
    public void action() {
        this.command.execute();
    }
}

/**
 * 场景类
 * @author zonzie
 * @date 2018/11/19 1:51 PM
 */
public class Client {

    public static void main(String[] args) {
        // 定义负责人
        Invoker invoker = new Invoker();
        // 增加一项需求
        System.out.println("增加一项需求....");
        // 命令
        AddRequirementCommond addRequirementCommond = new AddRequirementCommond();
        // 负责人收到命令
        invoker.setCommand(addRequirementCommond);
        // 执行命令
        invoker.action();
    }
}
```

##### 命令模式的优点
- 类间解耦
调用者和接收者之间没有任何依赖关系, 调用者实现功能只需要调用Command抽象类的execute方法, 不需要了解是哪个接收者
- 可扩展性
Command的子类可以非常容易的扩展, 调用者Invoker和高层次的模块Client不产生严重的代码耦合
- 结合其他模式
可以结合责任链模式, 实现命令族的解析任务, 结合模板方法模式, 可以减少Command子类的膨胀问题

##### 缺点
命令越多, Command子类就会越来越多

##### 使用场景
是命令的地方就可以使用命令模式