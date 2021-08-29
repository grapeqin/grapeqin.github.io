# 1、Java动态代理

在日常开发过程中，有很多场景会运用代理思想，为了对已有的行为进行拦截或增强，最高效的方式是通过代理对象来完成对目标对象的增强。这样既能保证目标对象的安全，同时也能根据自己的需要在目标对象行为前后植入自己的增强行为。

我们来看下面的例子:

有一个接口 `ClassicInterface`

```java
public interface ClassicInterface {
    String doSomething(int i);
}
```

它的默认实现 `DefaultClassicImplemention`

```java
public class DefaultClassicImplemention  implements ClassicInterface
{
    @Override
    public String doSomething(int i) {
        System.out.printf("method doSomething with param %d\n",i);
        return String.format("method doSomething with param %d",i);
    }
}
```

现在我们接到了一个需求，要记录下 `ClassicInteface` 接口所有方法的请求参数和响应结果，这个时候你会怎么来实现呢？

我们先看看使用静态代理如何实现：

## 1.1、静态代理

提供一个静态代理实现 `StaticProxyOfClassicInterface` 

```java
public class StaticProxyOfClassicInterface implements ClassicInterface{

    private ClassicInterface target;

    public StaticProxyOfClassicInterface(ClassicInterface target) {
        this.target = target;
    }

    @Override
    public String doSomething(int i) {
        System.out.printf(">>>static proxy method doSomething param %d\n",i);
        String retVal = target.doSomething(i);
        System.out.printf(">>>static proxy method doSomething return %s\n",retVal);
        return retVal;
    }
}
```
客户端使用方式如下

```java
public class StaticProxyClient {
    public static void main(String[] args) {
        //实例化目标对象
        ClassicInterface target = new DefaultClassicImplemention();
        //构造代理对象
        ClassicInterface proxy = new StaticProxyOfClassicInterface(target);
        //调用代理对象方法
        proxy.doSomething(100);
    }
}
```
它的执行结果:

```shell
>>>static proxy method doSomething param 100
method doSomething with param 100
>>>static proxy method doSomething return method doSomething with param 100
```

从结果来看，我们提供的这种实现很好的满足了需求，目标对象没有做任何修改，仅仅是在目标对象方法执行前后按照我们的需要记录了方法的请求参数和响应结果，我们来看看静态代理的代码组织方式，参考以下UML类图

![静态代理UML类图](_media\Clipboard_2021-08-14-17-47-40.png)
 
从上图可以看出：
1. 静态代理实现`StaticProxyOfClassicInterface`和目标实现`DefaultClassicImplemention`都必须要实现接口`ClassicInterface`,代理实现依赖目标实现
2. 如果接口`ClassicInterface`有多个方法时，还想实现类似的增强需求，所有的方法都会出现重复的增强行为，那有没有什么办法能解决即使有多个方法需要同样的增强行为，只需要一处实现就能满足需求呢？

一种可行的方法是选择动态代理。

## 1.2、Jdk动态代理

我们可以看下如何使用Jdk动态代理实现同样的需求。

首先，给出接口 `InvocationHandler`的一个实现`AuditClassicInterfaceInvocationHandler`

```java
public class AuditClassicInterfaceInvocationHandler implements InvocationHandler {

    /**
     * 目标对象
     */
    private ClassicInterface target;

    public AuditClassicInterfaceInvocationHandler(ClassicInterface target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object retVal;
        System.out.printf(">>>proxy method doSomething param %d\n",args[0]);
        retVal = method.invoke(this.target,args);
        System.out.printf(">>>proxy method doSomething return %s\n",retVal);
        return retVal;
    }
}
```
客户端使用方式如下:

```java
public class DynamicProxyClient {

    static Object getProxy(ClassicInterface target, InvocationHandler h){
        Class clazz = target.getClass();
        //创建目标对象target的代理对象
        return Proxy.newProxyInstance(clazz.getClassLoader(),clazz.getInterfaces(),h);
    }

    public static void main(String[] args) {
        ClassicInterface target = new DefaultClassicImplemention();

        ClassicInterface proxy = (ClassicInterface) getProxy(target,new AuditClassicInterfaceInvocationHandler(target));
        proxy.doSomething(100);
    }
}
```
它的执行结果:
```shell
>>>dynamic proxy method doSomething param 100
method doSomething with param 100
>>>dynamic proxy method doSomething return method doSomething with param 100
```

我们来看看动态代理的代码组织方式，参考以下UML类图:

![动态代理UML类图](_media\Clipboard_2021-08-14-17-30-45.png)

从结果来看,相比静态代理,它们有如下异同点

**相同点**:
1.由于都是代理模式的实现，无论静态代理还是动态代理，都依赖目标对象，毕竟在完成增强行为的同时，必须完成目标对象方法的调用

**不同点**:
1.静态代理的代理对象必须实现目标对象的接口，在重写接口方法的过程中完成增强行为，如果有多个方法需要增强，则会出现重复代码，增强行为代码的可复用性和可维护性下降
2.动态代理则将目标对象的增强行为委托给`InvocationHandler`的实现类，在生成代理对象的过程中完成目标对象与`InvocationHandler`增强类的绑定关系，由Java虚拟机保证调用代理对象方法时自动完成`InvocationHandler`接口的`invoke`方法调用，从而实现目标对象方法的行为增强。即使目标对象接口存在多个方法需要代理，也只需要编写一次行为增强代码。

## 参考资料

- [看懂UML类图](https://design-patterns.readthedocs.io/zh_CN/latest/read_uml.html)

- [platUML类图](https://plantuml.com/zh/class-diagram)

- [Dynamic Proxies in Java](https://www.baeldung.com/java-dynamic-proxies)

