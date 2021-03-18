---
title: java设计模式初探
date: 2019-04-24 10:52:23
categories:
- java
tags:
- java
---

# java设计模式初探

----

## java反射机制

> 我们平常都是通过new一个类来实例化一个对象，但是我们同时可以通过反射机制来构建，并且动态获取类里面的信息 比如说构造函数，方法和属性。

常用的代码像是这个：

<!-- more -->

| 方法                             | 功能                                           |
| -------------------------------- | ---------------------------------------------- |
| Class<?> Class.forName("全类名") | 加载该类对象，并且通过这个方法可以获得类的信息 |
| xxx.newInstance() | 实例化一个对象，这个我一般和上面的代码一起用来实例化对象 |
|Class.forName("xx").getDeclaredxxx|获得该类的信息|



## 单例模式

* 单利模式的共同点就是将构造函数私有化。

### 饿汉式

```java
public class UserEntity {

    private String username;

    private static final  UserEntity user = new UserEntity();

    static {
        System.out.println("静态代码块");
    }

    private UserEntity() {
        System.out.println("默认构造函数");
    }


    {
        System.out.println("非静态代码块");
    }


    public static UserEntity getInstance(){
        return user;
    }


    public void setUsername(String username){
        this.username = username;
    }

    public String getUsername(){
        return username;
    }

}

```

* 通过对该类进行java反射可以知道，static代码块在Class.forName装载的时候开始执行，而实例化的时候非静态代码块和构造函数开始执行

### 懒汉式

* 实体类在需要的时候才创建。

```java
public class UserEntity {

    private String username;

    private static UserEntity user;

    static {
        System.out.println("静态代码块");
    }

    private UserEntity() {
        System.out.println("默认构造函数");
    }


    {
        System.out.println("非静态代码块");
    }

	/*这里加锁保证线程安全*/
    public synchronized static UserEntity getInstance(){
        if(user == null){
            user = new UserEntity();
        }
        return user;
    }


    public void setUsername(String username){
        this.username = username;
    }

    public String getUsername(){
        return username;
    }

}
```

### 静态内部类

```java
public class UserEntity2 {


    private UserEntity2() {
    }

    public static class SingleInstance{
        private static final UserEntity2 user = new UserEntity2();
    }


    public static UserEntity2 getInstance(){
       return SingleInstance.user;
    }
}
```



### 枚举

```java
//使用枚举实现单例模式 优点:实现简单、枚举本身就是单例，由jvm从根本上提供保障!避免通过反射和反序列化的漏洞 缺点没有延迟加载
public class User {
	public static User getInstance() {
		return SingletonDemo04.INSTANCE.getInstance();
	}

	private static enum SingletonDemo04 {
		INSTANCE;
		// 枚举元素为单例
		private User user;

		private SingletonDemo04() {
			System.out.println("SingletonDemo04");
			user = new User();
		}

		public User getInstance() {
			return user;
		}
	}

	public static void main(String[] args) {
		User u1 = User.getInstance();
		User u2 = User.getInstance();
		System.out.println(u1 == u2);
	}
}


```

### 双重检验锁

可以看作是对懒汉式的一个改版 

```java
public class SingletonDemo04 {
	private SingletonDemo04 singletonDemo04;

	private SingletonDemo04() {

	}

	public SingletonDemo04 getInstance() {
		if (singletonDemo04 == null) {
			synchronized (this) {
				if (singletonDemo04 == null) {
					singletonDemo04 = new SingletonDemo04();
				}
			}
		}
		return singletonDemo04;
	}

}
```



### 单例模式的选择

如果不需要延迟加载单例，可以使用**枚举**或者**饿汉式**，相对来说枚举性好于饿汉式。

如果需要延迟加载，可以使用静态内部类或者懒汉式，相对来说静态内部类好于懒韩式。

最好使用饿汉式  

## 工厂模式

* 工厂模式将实例化的工作从客户手里夺了回来，Spring的依赖反转也是这个道理

### 简单工厂模式

* 简单工厂一般代码不会更改，拓展性差

![](https://s2.ax1x.com/2019/04/28/EQ4BAU.png)

### 工厂方法模式

* 工厂方法一般是如果产品非常多的情况下，派生出不同的工厂去实例化不同的产品。

![](https://s2.ax1x.com/2019/04/28/EQoDN8.png)

### 抽象工厂

> 这里分清楚产品族和产品树的关联

在下面的图中，两个产品有关联的实体形成了产品族，一个工厂只生产一个产品族。

产品树可以理解为由一个接口派生出来的类。

![](https://s2.ax1x.com/2019/04/28/EQo4EV.png)

## 模板方法模式

* 这个方法是平时用的蛮多的一个
* 其实模板方法就是将子类都利用的方法在父类中写好。

![](https://s2.ax1x.com/2019/04/28/EQTdxJ.png)

## 建造者模式

* Builder 可以根据客户提交的参数自定义产品的零件来产生不同的产品，这也是建造者模式和工厂模式的一个很大的区别。

![](https://s2.ax1x.com/2019/04/28/EQ7JOA.png)

### 建造者的应用场景
* 去肯德基，汉堡、可乐、薯条、炸鸡翅等是不变的，而其组合是经常变化的，生成出所谓的"套餐"。
* JAVA 中的 StringBuilder。 

## 代理模式

通过代理控制对象的访问,可以详细访问某个对象的方法，在这个方法调用处理，或调用后处理。SpringAOP就是代理模式

![](https://s2.ax1x.com/2019/04/28/EQ7DSg.png)

### 静态代理
静态代理一般都是将代码写死了，在项目中用的也很少，理解即可。

![](https://s2.ax1x.com/2019/04/28/EQ7fYT.png)

###  动态代理
> 静态代理有很大的缺陷，就是如果像让很多类都就行切面，不可能都去写进Proxy里面，这样动态代理就可以解决这个问题。
#### JDK动态代理

```java
// 每次生成动态代理类对象时,实现了InvocationHandler接口的调用处理器对象 
public class InvocationHandlerImpl implements InvocationHandler {
	private Object target;// 这其实业务实现类对象，用来调用具体的业务方法
	// 通过构造函数传入目标对象
	public InvocationHandlerImpl(Object target) {
		this.target = target;
	}

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		Object result = null;
		System.out.println("调用开始处理");
		result = method.invoke(target, args);
		System.out.println("调用结束处理");
		return result;
	}

	public static void main(String[] args) throws NoSuchMethodException, SecurityException, InstantiationException,
			IllegalAccessException, IllegalArgumentException, InvocationTargetException {
		// 被代理对象
		IUserDao userDao = new UserDao();
		InvocationHandlerImpl invocationHandlerImpl = new InvocationHandlerImpl(userDao);
		ClassLoader loader = userDao.getClass().getClassLoader();
		Class<?>[] interfaces = userDao.getClass().getInterfaces();
		// 主要装载器、一组接口及调用处理动态代理实例
		IUserDao newProxyInstance = (IUserDao) Proxy.newProxyInstance(loader, interfaces, invocationHandlerImpl);
		newProxyInstance.save();
	}

}
```

#### CGLib动态代理

需要asm-all和cglib的jar包

```java
public class CglibProxy implements MethodInterceptor {
	private Object targetObject;
	// 这里的目标类型为Object，则可以接受任意一种参数作为被代理类，实现了动态代理
	public Object getInstance(Object target) {
		// 设置需要创建子类的类
		this.targetObject = target;
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(target.getClass());
		enhancer.setCallback(this);
		return enhancer.create();
	}

	public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
		System.out.println("开启事物");
		Object result = proxy.invoke(targetObject, args);
		System.out.println("关闭事物");
		// 返回代理对象
		return result;
	}
	public static void main(String[] args) {
		CglibProxy cglibProxy = new CglibProxy();
		UserDao userDao = (UserDao) cglibProxy.getInstance(new UserDao());
		userDao.save();
	}
}

```

## 门面(外观)模式

外观模式，其实就是将后面很复杂的方法封装成一个类里面，让客户直接调用封装类里面的方法即可。

![](https://s2.ax1x.com/2019/04/28/EQblKH.png)

## 适配器模式

将一个类的接口转换成客户希望的另一个接口。适配器模式让那些接口不兼容的类可以一起工作。

![](https://s2.ax1x.com/2019/04/28/EQqPFP.jpg)



## 策略模式

现实开发中如果我们由一个根据不同情况创建不同实体的情况下，无论是用if-else或者是swith实现，代码是不符合开闭原则的，也就是他的拓展性并不高。

那么，策略模式，解决了这个问题。其实，他的代码是比之前多了不少，但是这样也同时隐藏后台的核心代码，只将Strategy接口交给了用户，保证了一定的安全性。

![](https://s2.ax1x.com/2019/04/28/EQqb0s.png)