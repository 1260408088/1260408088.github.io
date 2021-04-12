---
title: 手写SpringAop
date: 2020-01-07 22:06:30
categories: spring
tags: [spring,深入了解系列]
---

# Spring的核心技术

​	spring的核心技术有AOP与IOC，控制反转与面向切面编程，首先对于AOP有一些概念的解释，AOP的作用很多，日志打印、权限管理等等。

``` java
关注点：重复代码就叫做关注点；
切面：
关注点形成的类，就叫切面(类)！
面向切面编程，就是指 对很多功能都有的重复的代码抽取，再在运行的时候网业务方法上动态植入“切面类代码”。
切入点：
执行目标对象方法，动态植入切面代码。可以通过切入点表达式，指定拦截哪些类的哪些方法； 给指定的类在运行的时候植入切面类代码。
```

``` java
@Aspect							   指定一个类为切面类		
@Pointcut("execution(* com.itmayiedu.service.UserService.add(..))")  指定切入点表达式
@Before("pointCut_()")				前置通知: 目标方法之前执行
@After("pointCut_()")				后置通知：目标方法之后执行（始终执行）
@AfterReturning("pointCut_()")		 返回后通知： 执行方法结束前执行(异常不执行)
@AfterThrowing("pointCut_()")		 异常通知:  出现异常时候执行
@Around("pointCut_()")				环绕通知： 环绕目标方法执行

```

SpringAop的demo

``` java
@Component
@Aspect
public class AopLog {

	// 前置通知
	@Before("execution(* com.itmayiedu.service.UserService.add(..))")
	public void begin() {
		System.out.println("前置通知");
	}

	//
	// 后置通知
	@After("execution(* com.itmayiedu.service.UserService.add(..))")
	public void commit() {
		System.out.println("后置通知");
	}

	// 运行通知
	@AfterReturning("execution(* com.itmayiedu.service.UserService.add(..))")
	public void returning() {
		System.out.println("运行通知");
	}

	// 异常通知
	@AfterThrowing("execution(* com.itmayiedu.service.UserService.add(..))")
	public void afterThrowing() {
		System.out.println("异常通知");
	}

	// 环绕通知
	@Around("execution(* com.itmayiedu.service.UserService.add(..))")
	public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
		System.out.println("环绕通知开始");
		proceedingJoinPoint.proceed();
		System.out.println("环绕通知结束");
	}
}

```

## Spring的声明式事务管理

### Spring声明式事务管理器类：

1. Jdbc技术：DataSourceTransactionManager

2. Hibernate技术：HibernateTransactionManager

### 编程式事务实现：

``` java
@Component
public class TransactionUtils {

	@Autowired
	private DataSourceTransactionManager dataSourceTransactionManager;

	// 开启事务
	public TransactionStatus begin() {
		TransactionStatus transaction = dataSourceTransactionManager.getTransaction(new DefaultTransactionAttribute());
		return transaction;
	}

	// 提交事务
	public void commit(TransactionStatus transactionStatus) {
		dataSourceTransactionManager.commit(transactionStatus);
	}

	// 回滚事务
	public void rollback(TransactionStatus transactionStatus) {
		dataSourceTransactionManager.rollback(transactionStatus);
	}
}

@Service
public class UserService {
	@Autowired
	private UserDao userDao;
	@Autowired
	private TransactionUtils transactionUtils;

	public void add() {
		TransactionStatus transactionStatus = null;
		try {
			transactionStatus = transactionUtils.begin();
			userDao.add("wangmazi", 27);
			int i = 1 / 0;
			System.out.println("我是add方法");
			userDao.add("zhangsan", 16);
			transactionUtils.commit(transactionStatus);
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (transactionStatus != null) {
				transactionStatus.rollbackToSavepoint(transactionStatus);
			}
		}

	}

}

```

### AOP技术封装手动事务

``` java
@Component
@Aspect
public class AopTransaction {
	@Autowired
	private TransactionUtils transactionUtils;

	// // 异常通知
	@AfterThrowing("execution(* com.itmayiedu.service.UserService.add(..))")
	public void afterThrowing() {
		System.out.println("程序已经回滚");
		// 获取程序当前事务 进行回滚
		TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
	}

	// 环绕通知
	@Around("execution(* com.itmayiedu.service.UserService.add(..))")
	public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
		System.out.println("开启事务");
		TransactionStatus begin = transactionUtils.begin();
		proceedingJoinPoint.proceed();
		transactionUtils.commit(begin);
		System.out.println("提交事务");
	}

}
```

