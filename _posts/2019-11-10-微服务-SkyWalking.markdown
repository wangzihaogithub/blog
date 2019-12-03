---
layout: post
title:  "微服务-SkyWalking"
tags: 微服务
---


### 微服务APM之 - SkyWalking
    
SkyWalking是一款java应用监控工具, 市面上大部分用于微服务领域. 它由以下3点组成。

1.客户端（skywalking-agent）

    使用java探针(Agent) 实现了用户应用的监控埋点。主要是根据方法名，类名对各种框架进行构造方法，静态方法，普通方法插入代码进行AOP拦截。

2.服务端（oap-server）
    
    使用(grpc)TCP服务器，接收客户端上报的监控数据，并使用elasticsearch或其他数据库存储监控数据。

3.可视化UI (skywalking-ui)
    
    使用（vue + typescript)实现了监控数据的可视化，并使用查询语言graphql查询监控数据。


---

### 微服务APM之 - 客户端（skywalking-agent）

### 1. 如何插入埋点的代码? 

编写下面类， 然后在jvm启动参数加 -javaagent:D:\wangzihao\plug\javaagent-1.0.jar 可以实现插入代码
    
该类的核心入口方法     1.premain(). 该方法在main方法前执行

该类的核心运行逻辑方法  2.java.lang.instrument.ClassFileTransformer的transform()方法

    
    public class Javaagent {
        //该方法在main方法前执行
        public static void premain(String agentArgs, java.lang.instrument.Instrumentation instrumentation){
            System.out.println("begin agentArgs="+agentArgs+", instrumentation="+instrumentation);
    
            long timeout = Long.parseLong(System.getProperty("rt.agent.timeout","1000"));
            String packages = System.getProperty("rt.agent.packages","");
            if(packages.isEmpty()){
                return;
            }
            java.lang.instrument.ClassFileTransformer transformer = new com.my.LogTransformerByInsertCode(packages.split(","),timeout);
            instrumentation.addTransformer(transformer);
        }
        
        public class LogTransformerByInsertCode implements java.lang.instrument.ClassFileTransformer {
            //这个方法要求返回java代码（字节码也就是.class文件的内容）。 
            //这个方法在执行Class.forname()时触发执行，你可以修改.class文件的内容，从而实现对java代码进行修改。
            @Override
            public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
                String fullClassName = className.replace("/", ".");
                for (CtMethod ctMethod : ctclass.getDeclaredMethods()) {
                    String fullMethodName = ctMethod.getMethodInfo2().toString();
                    ctMethod.insertBefore("startTime = System.currentTimeMillis();" );
                    ctMethod.insertAfter("long time = System.currentTimeMillis() - startTime;" +
                            "System.out.println(\"------------------\"+Thread.currentThread()+\"  "+fullMethodName+" runtime cost is \"+time+\"\");"+
                            "if(time > "+timeout+"){"+
                                "throw new RuntimeException(\"" + fullMethodName+") execute cost:\" +(time) +\"ms.\");"+
                            "}");
                    System.out.println("代理方法 - " + fullMethodName);
                }
                byte[] bytes = ctclass.toBytecode();
                return bytes;
            }
        }
    }
    
    
### 2. skywalking客户端的也是用这个方式实现的，不过skywalking是基于(bytebuddy框架)实现了java.lang.instrument.ClassFileTransformer的transform（）方法。

类org.apache.skywalking.apm.agent.SkyWalkingAgent 实现了入口方法premain()

类net.bytebuddy.agent.builder.AgentBuilder.Default.ExecutingTransformer 实现了运行逻辑 transform()

skywalking封装的基础框架类在 apm-agent-core包中, 

如果我们想写一个代码监控插件, 主要学习拦截器 org.apache.skywalking.apm.agent.core.plugin.interceptor.*

    InstMethodsInter.class (普通方法拦截)
    InstMethodsInterWithOverrideArgs.class (普通方法拦截,可以修改方法入参)
    
    StaticMethodsInter.class (静态方法拦截)
    StaticMethodsInterWithOverrideArgs.class (静态方法拦截,可以修改方法入参)
    
    ConstructorInter.class (构造方法拦截,可以修改方法入参)


接下来你就可以使用这些类中所使用的接口进行编程啦(方法执行前后时或异常时..等等).

写完plugin后,打包放到目录下,你些的方法就会被调用了(注意: 因为你的代码是阻塞执行的,会延长用户的业务代码执行时间). 


### 微服务APM之 - 服务端（oap-server） - 


### 微服务APM之 - 可视化UI (skywalking-ui) - 



### 客户端概括

类型

public enum org.apache.skywalking.apm.agent.core.context.trace.SpanLayer {
   DB(1),
   RPC_FRAMEWORK(2),
   HTTP(3),
   MQ(4),
   CACHE(5);
}