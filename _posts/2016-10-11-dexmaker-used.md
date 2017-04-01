---
layout: post
title: Android动态类生成预加载dexmaker使用
date: 2016-10-11 09:34:12
---
### 一、dexmaker简单介绍
dexmaker是运行在Android Dalvik VM上，利用Java编写，来动态生成DEX字节码的API。如果读者了解AOP编程的话，应该听说过cglib or ASM，但这两个工具生成都是Java字节码，而Dalvik加载的必须是DEX字节码。所以，想要在Android上进行AOP编程，dexmaker可以说是一个很好的选择。项目地址：https://github.com/crittercism/dexmaker。
### 二、简单使用
下面这个例子非常典型，可以说入门非常好了。过程很简单，生成一个包含一个函数的类，在主程序里面动态加载（使用ClassLoader），然后执行类里面的函数。这是在Java平台的例子，我直接在Android上进行编程的，后面或说明相应的问题以及解决办法，下来看看这个例子吧。
```java
public final class HelloWorldMaker {
    public static void main(String[] args) throws Exception {
        DexMaker dexMaker = new DexMaker();

        // Generate a HelloWorld class.
        TypeId<?> helloWorld = TypeId.get("LHelloWorld;");
        dexMaker.declare(helloWorld, "HelloWorld.generated", Modifier.PUBLIC, TypeId.OBJECT);
        generateHelloMethod(dexMaker, helloWorld);

        // Create the dex file and load it.
        File outputDir = new File(".");
        ClassLoader loader = dexMaker.generateAndLoad(HelloWorldMaker.class.getClassLoader(),
                outputDir, outputDir);
        Class<?> helloWorldClass = loader.loadClass("HelloWorld");

        // Execute our newly-generated code in-process.
        helloWorldClass.getMethod("hello").invoke(null);
    }

    private static void generateHelloMethod(DexMaker dexMaker, TypeId<?> declaringType) {
        // Lookup some types we‘ll need along the way.
        TypeId<System> systemType = TypeId.get(System.class);
        TypeId<PrintStream> printStreamType = TypeId.get(PrintStream.class);

        // Identify the ‘hello()‘ method on declaringType.
        MethodId hello = declaringType.getMethod(TypeId.VOID, "hello");

        // Declare that method on the dexMaker. Use the returned Code instance
        // as a builder that we can append instructions to.
        Code code = dexMaker.declare(hello, Modifier.STATIC | Modifier.PUBLIC);

        // Declare all the locals we‘ll need up front. The API requires this.
        Local<Integer> a = code.newLocal(TypeId.INT);
        Local<Integer> b = code.newLocal(TypeId.INT);
        Local<Integer> c = code.newLocal(TypeId.INT);
        Local<String> s = code.newLocal(TypeId.STRING);
        Local<PrintStream> localSystemOut = code.newLocal(printStreamType);

        // int a = 0xabcd;
        code.loadConstant(a, 0xabcd);

        // int b = 0xaaaa;
        code.loadConstant(b, 0xaaaa);

        // int c = a - b;
        code.op(BinaryOp.SUBTRACT, c, a, b);

        // String s = Integer.toHexString(c);
        MethodId<Integer, String> toHexString
                = TypeId.get(Integer.class).getMethod(TypeId.STRING, "toHexString", TypeId.INT);
        code.invokeStatic(toHexString, s, c);

        // System.out.println(s);
        FieldId<System, PrintStream> systemOutField = systemType.getField(printStreamType, "out");
        code.sget(systemOutField, localSystemOut);
        MethodId<PrintStream, Void> printlnMethod = printStreamType.getMethod(
                TypeId.VOID, "println", TypeId.STRING);
        code.invokeVirtual(printlnMethod, null, localSystemOut, s);

        // return;
        code.returnVoid();
    }
}


```
generateHelloMethod函数生成的函数是：
```java
public static void hello() {
             int a = 0xabcd;
             int b = 0xaaaa;
             int c = a - b;
            String s = Integer.toHexString(c);
            System.out.println(s);
            return;
        }
```
这里很关键的是变量的声明与赋值和函数的声明与调用。例如int变量声明：
```java
Local<Integer> a = code.newLocal(TypeId.INT);
```
变量赋值：
```java
code.loadConstant(a, 0xabcd);
```

函数声明：
```java
MethodId<Integer, String> toHexString = TypeId.get(Integer.class).getMethod(TypeId.STRING, "toHexString", TypeId.INT);
```

函数调用：
```java
code.invokeStatic(toHexString, s, c);
```
过程非常简单，要生成一个完整的简单的类按照此步骤能很快完成。好的，现在进入正片部分了，即在Android平台使用dexmaker的情况。
