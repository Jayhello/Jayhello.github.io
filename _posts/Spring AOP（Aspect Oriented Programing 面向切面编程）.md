# Spring AOP（Aspect Oriented Programing 面向切面编程）

## 1. 为什么要有AOP

考虑如下情景：

一个计算器接口：

```java
public interface ArithmeticCaculator {
	
	int add(int i, int j);
	
	int sub(int i, int j);
	
	int mul(int i, int j);
	
	int div(int i, int j);
}
```

一个该接口的实现类：

```java
public class ArithmeticCaculatorImpl implements ArithmeticCaculator {

	@Override
	public int add(int i, int j) {
		// TODO Auto-generated method stub
		int result = i + j;
		return result;
	}

	@Override
	public int sub(int i, int j) {
		// TODO Auto-generated method stub
		int result = i - j;
		return result;
	}

	@Override
	public int mul(int i, int j) {
		// TODO Auto-generated method stub
		int result = i * j;
		return result;	
	}

	@Override
	public int div(int i, int j) {
		// TODO Auto-generated method stub
		int result = i / j;
		return result;	
	}

}
```

使用该类：

```java
public class Main {

	public static void main(String[] args) {		
		ArithmeticCaculator caculator = new ArithmeticCaculatorImpl();
		
		int result = caculator.div(12, 4);
		System.out.println(result);
	}
}
```

输出：

> 3



这是正常的使用方法，现在有两个新需求：

1. 日志功能：每次进入方法时和退出方法时打印日志
2. 验证功能，要求方法只返回 >= 0的结果，如果结果 < 0 则返回0



那么就需要把实现类修改为如下：

```java
public class ArithmeticCaculatorImpl implements ArithmeticCaculator {

	@Override
	public int add(int i, int j) {
		// TODO Auto-generated method stub
		System.out.println("The method add begins with " + i + "," + j);

		int result = i + j;

		System.out.println("The method add ends");

		if (result < 0) {
			result = 0;
		}

		return result;
	}

	@Override
	public int sub(int i, int j) {
		// TODO Auto-generated method stub
		System.out.println("The method sub begins with " + i + "," + j);

		int result = i - j;

		System.out.println("The method sub ends");

		if (result < 0) {
			result = 0;
		}
		return result;
	}

	@Override
	public int mul(int i, int j) {
		// TODO Auto-generated method stub
		System.out.println("The method mul begins with " + i + "," + j);

		int result = i * j;

		System.out.println("The method mul ends");

		if (result < 0) {
			result = 0;
		}
		return result;
	}

	@Override
	public int div(int i, int j) {
		// TODO Auto-generated method stub
		System.out.println("The method div begins with " + i + "," + j);

		int result = i / j;

		System.out.println("The method div ends");

		if (result < 0) {
			result = 0;
		}
		return result;
	}

}

```

可以看到，代码的行数多了一倍，而且重复的代码写了好多次，这种写法至少有两个缺点：

1. 代码混乱：非业务需求日益增多，导致核心逻辑不清晰。
2. 代码分散：同样的代码在多个地方重复，如果需求发生变化，必须在多个地方同时修改代码。

## 2. 使用动态代理解决上述问题

代理设计模式的原理： 使用一个代理将对象包装起来,，然后用该代理对象取代原始对象。 任何对原始对象的调用都要通过代理.。代理对象决定是否以及何时将方法调用转到原始对象上。

使用代理模式的代码如下：

![](C:\Users\黄凯\AppData\Roaming\Typora\typora-user-images\1571455425391.png)

## 3. 使用Spring AOP进行代理

愚以为SpringAOP实际上就是动态代理模式的一个实现，只是它通过框架的方式让开发者使用起来更加方便。

### 3.1 使用注释方法使用SpringAOP

AspectJ：Java 社区里最完整最流行的 AOP 框架.
在 Spring2.0 以上版本中, 可以使用基于 AspectJ 注解或基于 XML 配置的 AOP

#### （1）导包

aopalliance.jar

aspectj.weaver.jar

spring-aspects.jar  

#### （2）编辑配置文件

```xml
	<!-- 使AspectJ注解起作用，自动为匹配的类生成代理对象 -->
	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

#### （3）编辑切面类

仍以上面的计算器需求为例，写一个日志代理类。

```java
@Aspect
@Order(2)// 可以使用order指定切面的优先级，值越小，优先级越高
@Component
public class LoggingAspect {
	/*
	 * 定义一个方法，用于声明切入点表达式
	 * 一般该方法都不需要添入其他代码
	 * 使用@Pointcut来声明切入点表达式
	 * 后面的其他方法引用当前的切入点表达式
	 */
	@Pointcut("execution(public int com.husthuangkai.spring.aop.impl.ArithmeticCaculator.*(int, int))")
	public void declareJointPointExpression() {
		
	}
	

	// 声明该方法是一个前置通知
	@Before("declareJointPointExpression()")
	public void beforeMethod(JoinPoint joinpoint) {
		String methodNameString = joinpoint.getSignature().getName();
		java.util.List<Object> args = Arrays.asList(joinpoint.getArgs());
		System.out.println("The method " + methodNameString + " begins with " + args);
	}

	// 后置通知：在目标方法执行后（无论是否发生异常），执行的通知
	// 在后置通知中还不能访问目标方法执行的结果
	@After("declareJointPointExpression()")
	public void afterMethod(JoinPoint joinPoint) {
		String methodNameString = joinPoint.getSignature().getName();

		System.out.println("The method  " + methodNameString + "ends");

	}

	/**
	 * 在方法正常结束后执行的代码 返回通知是可以访问方法的返回值的
	 * 
	 * @param joinPoint
	 */
	@AfterReturning(value = "declareJointPointExpression()"
			, returning = "result")
	public void afterReturning(JoinPoint joinPoint, Object result) {
		String methodNameString = joinPoint.getSignature().getName();
		System.out.println("The method " + methodNameString + "ends with " + result);
	}

	/**
	 * 在目标方法出现异常时执行的代码，而且可以访问到异常对象
	 * 
	 * @param joinPoint
	 * @param ex        可以指定在出现特定异常时执行
	 */
	@AfterThrowing(value = "execution(public int com.husthuangkai.spring.aop.impl.ArithmeticCaculator.*(int, int))", throwing = "ex")
	public void afterThrowing(JoinPoint joinPoint, Exception ex) {
		String methodNameString = joinPoint.getSignature().getName();
		System.out.println("The method " + methodNameString + " end with " + ex);
	}

	/*
	 * 环绕通知需要携带ProceedingJoinPoint类型的参数
	 * 环绕通知类似于动态代理的全过程：ProceedingJoinPoint类型的参数可以决定是否执行目标方法 环绕通知必须有返回值，且返回值为目标方法的返回值
	 */
//	@Around(value = "execution(public int com.husthuangkai.spring.aop.impl.ArithmeticCaculator.*(int, int))")
//	public Object aroundMethod(ProceedingJoinPoint proceedingJoinPoint) {
//		Object result = null;
//		String methodNameString = proceedingJoinPoint.getSignature().getName();
//		// 执行目标方法
//
//		try {
//			System.out.println("The method " + methodNameString + " begins with" + Arrays.asList(proceedingJoinPoint.getArgs()));
//
//			result = proceedingJoinPoint.proceed();
//			
//			System.out.println("The method " + methodNameString + " end with" + "result");
//
//		} catch (Throwable e) {
//			// TODO Auto-generated catch block
//			e.printStackTrace();
//		}
//
//		System.out.println("aroundMethod");
//		return 100;
//	}
}
```



#### （4）编辑切面类



```java
public class Main {

	public static void main(String[] args) {
		// 1.创建spring的IOC容器
		ApplicationContext ctx = new 			ClassPathXmlApplicationContext("applicationContext.xml");
		
		// 2.从IOC容器中获取bean的实例
		ArithmeticCaculator caculator = ctx.getBean(ArithmeticCaculator.class);
		
		// 3.使用bean
		int result = caculator.add(1, 1);
		System.out.println(result);
		
		result = caculator.div(12, 4);
		System.out.println(result);
	}

}
```

结果：

> Validate [1, 1]
> The method add begins with [1, 1]
> The method  addends
> The method addends with 2
> 2
> Validate [12, 4]
> The method div begins with [12, 4]
> The method  divends
> The method divends with 3
> 3

### 3.2 使用配置文件配置AOP

仍然使用以上的例子，去掉所有注解后，编辑如下的配置文件。

```xml
<!-- 配置bean -->
	<bean id="arithmeticCaculator"
		class="com.husthuangkai.spring.aop.impl.xml.ArithmeticCaculatorImpl"></bean>
	
	
	<!-- 配置切面的bean -->
	<bean id="loggingAspect"
		class="com.husthuangkai.spring.aop.impl.xml.LoggingAspect"></bean>
		
	<bean id="validateAspect"
		class="com.husthuangkai.spring.aop.impl.xml.ValidateAspect"></bean>
		
	<!-- 配置AOP -->
	<aop:config>
		<!-- 配置切点表达式 -->
		<aop:pointcut expression="execution(* com.husthuangkai.spring.aop.impl.xml.ArithmeticCaculator.*(..))" 
			id="pointcut"/>
		
		<!-- 配置切面及通知 -->
		<aop:aspect ref="loggingAspect" order="2">
			<aop:before method="beforeMethod" pointcut-ref="pointcut"/>
			<aop:after method="afterMethod" pointcut-ref="pointcut"/>
			<aop:after-returning method="afterReturning" pointcut-ref="pointcut" returning="result"/>
			<aop:after-throwing method="afterThrowing" pointcut-ref="pointcut" throwing="ex"/>
		</aop:aspect>
		
		<aop:aspect ref="validateAspect" order="1">
			<aop:before method="validateArgs" pointcut-ref="pointcut"/>	
		</aop:aspect>
	</aop:config>
```

结果：

> Validate [1, 1]
> The method add begins with [1, 1]
> The method  addends
> The method addends with 2
> 2
> Validate [12, 4]
> The method div begins with [12, 4]
> The method  divends
> The method divends with 3
> 3