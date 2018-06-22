# AOP @Pointcut

## 语法
```
@Aspect
@Component
public class ObjectAspectConfig {
    //@Pointcut("this(com.imooc.aop.log.Loggable)")
    @Pointcut("bean(logService)")
    public void matchCondition(){}

    @Before("matchCondition()")
    public void before(){
        System.out.println("before");
    }
}
```

## 包/类型匹配

* within()
```
@Pointcut("within(com.imooc.aop.service.ProductService+)")
```


## 对象匹配
* this()
```
// com.imooc.aop.log.Loggable is an interface
@Pointcut("this(com.imooc.aop.log.Loggable)")
```
* target()
```
// com.imooc.aop.log.Loggable is an interface
@Pointcut("target(com.imooc.aop.log.Loggable)")
```
* bean()
```
@Pointcut("bean(logService)")
```

```
package com.imooc.aop.log;

import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;

@Component
@Primary
public class LogService implements Loggable{
    @Override
    public void log(){
        System.out.println("log from Log service");
    }
}
```

## 方法匹配

* execution()

## 参数匹配
* args()
```
@Pointcut("args(String) && this(com.imooc.aop.log.Loggable)")
```


## 注解匹配
* @annotation() : 方法级别
```
// AdminOnly 注解是方法级别的
@Pointcut("@annotation(com.imooc.aop.secutiry.AdminOnly)")
```
```
package com.imooc.aop.secutiry;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface AdminOnly {
}
```
* @within(): 类级别
```
@Pointcut("@within(com.imooc.aop.secutiry.NeedSecured)")
```
```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface NeedSecured {
}
```
```
@NeedSecured
public class ProductService{
   ...
}
```

* @target(): 类级别
```
@Pointcut("@target(com.imooc.aop.secutiry.NeedSecured) && within(com.imooc.aop..*)")
```

* @args(): 参数级别
```
@Pointcut("@args(com.imooc.aop.secutiry.NeedSecured) && within(com.imooc.aop..*)")
```
