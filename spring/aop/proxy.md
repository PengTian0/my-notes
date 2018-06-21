# Spring AOP的proxy机制

## Types

* JDK dynamic proxy

如果要被代理的目标对象实现了任何接口，那么Spring AOP 默认会使用JDK dynamic proxy。
JDK dynamic proxy会创建目标对象的代理对象，该代理对象会实现目标对象实现的所有接口。

* CGLIB proxy

如果要被代理的目标对象没有实现任何接口，那么Spring AOP默认会使用CGLIB proxy。
CGLIB proxy会创建目标对象的代理对象，该代理对象继承自目标对象，会是想目标对象的所有非final方法。

## JDK dynamic proxy

```
package com.shanhy.demo.proxy; 
public interface Account { 
    public void queryAccount(); 
    public void updateAccount(); 
} 
```

```
package com.shanhy.demo.proxy; 
public class AccountImpl implements Account { 
    @Override 
    public void queryAccount() { 
        System.out.println("View the account"); 
    } 

    @Override 
    public void updateAccount() { 
        System.out.println("Modify the account"); 
    } 
} 
```

```
package com.shanhy.demo.proxy; 

import java.lang.reflect.InvocationHandler; 
import java.lang.reflect.Method; 
import java.lang.reflect.Proxy; 

public class AccountProxyFactory implements InvocationHandler { 

    private Object target; 

    public Object bind(Object target){ 
        //JDK dynamic proxy is used here, which must be bound with interfaces, and our businesses may be implemented without involving the interfaces. So this defect is remedied by CGLIB. 
        this.target = target; 
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), 
                target.getClass().getInterfaces(), this); 
    } 

    @Override 
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable { 
        //System.out.println(Proxy.isProxyClass(proxy.getClass())); 
        boolean objFlag = method.getDeclaringClass().getName().equals("java.lang.Object"); 

        Object result = null; 
        if(!objFlag) 
            System.out.println("Implement proxy before"); 
        result = method.invoke(this.target, args); 
        if(!objFlag) 
            System.out.println("Implement proxy after"); 
        return result; 
    } 
} 
```

```
package com.shanhy.demo.proxy; 

public class AccountProxyTest { 

    public static void main(String[] args) { 
        //For the JDK proxy classes used below, one proxy can manage many interfaces. 
        Account account1 = (Account)new AccountProxyFactory().bind(new AccountImpl()); 
        System.out.println(account1); 
        account1.queryAccount(); 
    }
} 
```

## CGLIB proxy

```
package com.shanhy.demo.proxy; 

import java.lang.reflect.Method; 

import net.sf.cglib.proxy.Enhancer; 
import net.sf.cglib.proxy.MethodInterceptor; 
import net.sf.cglib.proxy.MethodProxy; 

public class AccountCglibProxyFactory implements MethodInterceptor{ 

    private Object target; 

    public Object getInstance(Object target){ 
        this.target = target; 
        return Enhancer.create(this.target.getClass(), this); 

        //Enhancer enhancer = new Enhancer();//This class is used to generate a proxy object. 
        //enhancer.setSuperclass(this.target.getClass());//Set the parent class. 
        //enhancer.setCallback(this);//Set the callback object to itself. 

        //return enhancer.create(); 
    } 

    @Override 
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable { 
        //Exclude the methods such as toString in the Object class. 
        boolean objFlag = method.getDeclaringClass().getName().equals("java.lang.Object"); 
        if(!objFlag){ 
            System.out.println("before"); 
        } 
        Object result = null; 
        //We usually use the proxy.invokeSuper(obj,args) method. This is easy to understand - it is the method to implement the original class. There is a another method: proxy.invoke(obj,args), which is a method to implement the subclass generated. 
        //If the incoming object is a subclass, an out-of-memory exception will occur, because the subclass method enters the intercept method constantly, and this method again comes to invoke the subclass method, which leads to loop calls of the two methods. 
        result = methodProxy.invokeSuper(obj, args); 
        //result = methodProxy.invoke(obj, args); 
        if(!objFlag){ 
            System.out.println("after"); 
        } 
        return result; 
    } 
} 
```

```
package com.shanhy.demo.proxy; 

public class Person { 

    public void show(){ 
        System.out.println("showing"); 
    } 
} 
```
```
package com.shanhy.demo.proxy; 

public class AccountProxyTest { 

    public static void main(String[] args) { 
        //The following are the proxies using CGLIB. 
        // 1. Support the classes with interface implemented 
        Account account2 = (Account)new AccountCglibProxyFactory().getInstance(new AccountImpl()); 
        account2.updateAccount(); 

        // 2. Support the classes with interface unimplemented 
        Person person = (Person)new AccountCglibProxyFactory().getInstance(new Person()); 
        System.out.println(person); 
        person.show(); 
    } 
}
```


