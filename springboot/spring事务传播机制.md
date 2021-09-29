### 1.什么是事务？

> 事务是数据库操作的最小工作单元，是作为单个逻辑工作单元执行的一系列操作;这些操作作为一个整体一起向系统提交,要么都执行、要么都不执行;事务是一组不可再分割的操作集合(工作逻辑单元)

### 2.什么是事务的传播

  事务的传播是针对事务嵌套而言的，即需要事务的业务方法A调用了需要事务的方法B

### 3.开启事务的方法

  Spring 事务管理分为编码式和声明式两种。编码式事务指的是通过编码方式实现事务；声明式事务基于 AOP，将具体业务逻辑与事务处理解耦。这里只说声明式事务的使用。在需要开启事务的方法上面添加@Transactional注解，只能应用在public方法上面（因为声明式事务是基于AOP，而AOP的实现是基于动态代理的，private不能被重写，也就不能被代理，所以事务不生效）。

  @Transactional也可以应用在类上面，表示该类下的所有的public方法都开启事务。当类级别配置了@Transactional，方法级别也配置了@Transactional，应用程序会以方法级别的事务属性信息来管理事务。方法级别的事务属性信息会覆盖类级别的相关配置信息。

  @Transactional的属性描述

| 属性名称               | 属性描述                                                     |
| :--------------------- | ------------------------------------------------------------ |
| propagation            | 该属性用于设置事务的传播行为，例如@Transactional(propagation=Propagation.NOT_SUPPORTED) |
| isolation              | 该属性用于设置底层数据库的事务隔离级别，事务隔离级别用于处理多事务并发的情况，通常使用数据库的默认隔离级别即可，基本不需要进行设置 |
| timeout                | 该属性用于设置事务的超时秒数，默认值为-1表示永不超时         |
| readOnly               | 该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，默认值为false。 |
| rollbackFor            | 该属性用于设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。例如：<br />指定单一异常类：@Transactional(rollbackFor=RuntimeException.class)<br />指定多个异常类：@Transactional(rollbackFor={RuntimeException.class, Exception.class}) |
| rollbackForClassName   | 该属性用于设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚。例如：<br />指定单一异常类名称：@Transactional(rollbackForClassName="RuntimeException")<br />指定多个异常类名称：@Transactional(rollbackForClassName={"RuntimeException","Exception"}) |
| noRollbackFor          | 该属性用于设置不需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，不进行事务回滚。例如：<br />指定单一异常类：@Transactional(noRollbackFor=RuntimeException.class)<br />指定多个异常类：@Transactional(noRollbackForClassName={"RuntimeException","Exception"}) |
| noRollbackForClassName | 该属性用于设置不需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，不进行事务回滚。例如：<br />指定单一异常类名称：@Transactional(noRollbackForClassName="RuntimeException")<br />指定多个异常类名称：@Transactional(noRollbackForClassName={"RuntimeException","Exception"}) |

### 4.spring事务传播类型

  spring提供了7种事务传播行为，枚举类型在org.springframework.transaction.annotation.Propagation。Springboot中默认的传播行为Propagation.REQUIRED

| 类型                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Propagation.REQUIRED      | 如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。如果多个方法都声明了 **REQUIRED**，并且他们嵌套调用，那么他们会共享同一个物理事务。就是 inner 产生了回滚，那么 outer 会跟着回滚。 |
| Propagation.SUPPORTS      | 如果当前存在事务，则加入该事务；如果当前不存在事务，则以非事务的方式继续运行。 |
| Propagation.MANDATORY     | 如果当前存在事务，则加入该事务；如果当前不存在事务，则抛出异常。 |
| Propagation.REQUIRES_NEW  | 重新创建一个新的事务，如果当前存在事务，暂停当前的事务。内层事务的提交回滚都是独立于外层事务的。外层事务不受内层事务结果的影响，他们运行于独立的物理事务。 |
| Propagation.NOT_SUPPORTED | 以非事务的方式运行，如果当前存在事务，暂停当前的事务。       |
| Propagation.NEVER         | 以非事务的方式运行，如果当前存在事务，则抛出异常。           |
| Propagation.NESTED        | NESTED 使嵌套的事务使用相同的物理事务，但是对嵌套调用设置了保存点，所以 inner 事务可以独立于 outer 事务回滚。 |

### 6.事务回滚/不会滚场景

#### 6.1什么情况下事务会回滚

  spring 的默认事务机制，当出现unchecked异常时候回滚，checked异常的时候不会回滚；  

  unchecked异常包括error和runtime异常，需要try catch或向上抛出的异常为checked异常，比如IOException，也就是说程序抛出runtime异常的时候才会进行回滚，其他异常不回滚。可以配置设置所有异常回滚： @Transactional(rollbackFor = { Exception.class }) 

  当有try catch后捕获了异常，事务不会回滚。*如果不得不在service层写try catch，需要catch后 throw new RuntimeException 让事务回滚；* 

#### 6.2代码示例

*case1*.抛出Exception异常，为checked的异常，此时事务不会回滚，数据插入成功

```java
    @Transactional
    public void insertA() throws Exception {
        User user = new User();
        user.setName("张三");
        userMapper.insert(user);
        // checked异常
        throw new Exception();
    }
```

*case2*.抛出RuntimeException，为unchecked异常，事务会回滚

```java
    @Transactional
    public void insertA() throws Exception {
        User user = new User();
        user.setName("张三");
        userMapper.insert(user);
        // unchecked异常
        throw new RuntimeException();
    }
```

*case3*.默认只在unchecked异常回滚。A和B都有事务，A方法调用B方法，B抛出unchecked异常

事务可以回滚，B的异常抛给A，A又往上抛，被事务切面感知，事务回滚。

如果B方法没有@Transactional注解，事务一样回滚。Springboot的默认事务传播机制为REQUIRED，B会加到A的事务中去。

如果B方法抛出checked异常，事务不会回滚，原理和*case1*一样。

```java
    @Transactional
    public void insertA() {
        User user = new User();
        user.setName("张三");
        userMapper.insert(user);
        insertB();
    }

    @Transactional
    public void insertB() {
        User user = new User();
        user.setName("李四");
        userMapper.insert(user);
        // unchecked异常
        throw new RuntimeException();
    }
```

case4.默认只在unchecked异常回滚。A和B都有事务，A方法调用B方法，B抛出unchecked异常，A方法try-catch

事务不会回滚。B的异常，在A中被捕获，不会被AOP感知，事务不会回滚。

```java
    @Transactional
    public void insertA() {
        User user = new User();
        user.setName("张三");
        userMapper.insert(user);
        try {
            insertB();
        } catch (Exception e){
            e.printStackTrace();
        }
    }

    @Transactional
    public void insertB() {
        User user = new User();
        user.setName("李四");
        userMapper.insert(user);
        // unchecked异常
        throw new RuntimeException();
    }
```

*case5*.默认只在unchecked异常回滚。A没有事务，B有事务，A方法调用B方法，B抛出unchecked异常

这种场景没有事务，Spring开启事务是根据调用的方法上面是否有@Transactional注解。外界调用A方法，A没有注解，所以不开启事务，也就不存在回滚。

```java
    public void insertA() {
        User user = new User();
        user.setName("张三");
        userMapper.insert(user);
        insertB();
    }

    @Transactional
    public void insertB() {
        User user = new User();
        user.setName("李四");
        userMapper.insert(user);
        throw new RuntimeException();
    }
```

