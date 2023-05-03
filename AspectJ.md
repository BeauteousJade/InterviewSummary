# 文章连接
&emsp;&emsp;[Android使用AspectJ](https://juejin.cn/post/7024344347504017444)

# 原理（ajc编译器）
&emsp;&emsp;执行AspectJ的时候，我们需要使用ajc编译器，对Aspect和需要织入的Java Source Code进行编译，得到字节码后，可以使用java命令来执行。

&emsp;&emsp;ajc编译器会首先调用javac将Java源代码编译成字节码，然后根据我们在Aspect中定义的pointcut找到相对应的Java Byte Code部分，使用对应的advice动作修改源代码的字节码。最后得到了经过Aspect织入的Java字节码，然后就可以正常使用这个字节码了。


# 1. 定义Plugin
&emsp;&emsp;可以自定义Plugin，也可以使用第三方的Plugin。Plugin的目的运用AspectJ工具，在代码编译过程中去生成配置的插桩代码。

&emsp;&emsp;定义Plugin需要`javaCompile`的任务后面.

# 2. 组成部分
&emsp;&emsp;AspectJ组成分别如下:
### (1). 定义切面
```
@Aspect
public class AspectJTest {

}
```
&emsp;&emsp;也就是使用Aspect注解一个类，表明这个类就是一个切面类。

### (2). 定义切点
&emsp;&emsp;有了切点表达式，接下来就要把这个表达式定义到切点中。
```
@Pointcut(executionTest)
public void executionMethod(){
}
```
### (3). 定义Advice，也就是插入的时机
&emsp;&emsp;可以定义如：Before（前置通知），AfterReturning（后置通知），AfterThrowing（异常通知），After（最终通知），Around（环绕通知）等类型的通知。
```
    @Around("executionMethod()")
    public void aroundMethod(ProceedingJoinPoint point){
    
        //doSomeThing
        try {
            point.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        //doSomeThing
    }
    
    
    @Before("executionMethod()")
    public void aroundMethod(JoinPoint point){
        //doSomething
    }
    
    @After("executionMethod()")
    public void afterMethod(){
        //doSomething
    }
```
