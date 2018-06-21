# Autowired

## 匹配规则

* By Type

* By Name


## By Type
* 如果有两个类实现了同一个接口，Autowired 该接口，那么会报如下错误：
```
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'com.imooc.aop.log.Loggable' available: expected single matching bean but found 2: logService,logService2
```
code:
```
package com.imooc.aop.log;

public interface Loggable {
    public void log();
}
```

```
package com.imooc.aop.log;

import org.springframework.stereotype.Component;

@Component
public class LogService implements Loggable{
    @Override
    public void log(){
        System.out.println("log from Log service");
    }
}
```

```
package com.imooc.aop.log;

import org.springframework.stereotype.Component;

@Component
public class LogService2 implements Loggable{
    @Override
    public void log(){
        System.out.println("################################### log from LogService2");
    }
}
```

```
package com.imooc.aop;


import com.imooc.aop.log.Loggable;
import com.imooc.aop.secutiry.CurrentUserHolder;
import com.imooc.aop.service.ProductService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;


@RunWith(SpringRunner.class)
@SpringBootTest
public class AopGuideApplicationTest {

    @Autowired
    Loggable logService3;

    @Test
    public void adminInsert(){
        logService3.log();
        System.out.println(logService3.getClass().getName());
    }
}
```

* Fix：
1. 将```@Autowired Loggable``` --> ```@Autowired LogService```
2. 将```@Autowired Loggable logService3``` --> ```@Autowired Loggable logService``` or ```@Autowired Loggable logService2```
3. 使用```@Primary``` 
```
package com.imooc.aop.log;

import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Component;

@Component
@Primary
public class LogService implements Loggable, Loggable2{
    @Override
    public void log(){
        System.out.println("log from Log service");
    }

    @Override
    public void log(String content){
        System.out.println(content);
    }
}

```

## By Name

默认情况下，先按照By Type的方式查找bean，如果出现多个相同类型的bean，再By Name的方式查找，如果仍然有多个，报错。
使用```@Qualifier```显示使用By Name的方式。
ps: ```@Qualifier```优先级高于```@Primary```

```
package com.imooc.aop;


import com.imooc.aop.log.Loggable;
import com.imooc.aop.secutiry.CurrentUserHolder;
import com.imooc.aop.service.ProductService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;


@RunWith(SpringRunner.class)
@SpringBootTest
public class AopGuideApplicationTest {

    @Autowired
    @Qualifier("logService2")
    Loggable logService3;

    @Test
    public void adminInsert(){
        logService3.log();
        System.out.println(logService3.getClass().getName());
    }
}

```
bean的名称默认是类名，首字母小写。
也可以显示指定，如：```@Component("myName")```
