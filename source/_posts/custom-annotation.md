title: custom annotation
author: Joe Tong
tags:
  - ANNOTATION
  - JAVAEE
  - SPRINGTBOOT
categories:
  - IT
date: 2019-07-15 23:37:00
---
自定义注解详细介绍  

1 注解的概念 &nbsp;
1.1 注解的官方定义 

首先看看官方对注解的描述： 

An annotation is a form of metadata, that can be added to Java source code. Classes, methods, variables, parameters and packages may be annotated. Annotations have no direct effect on the operation of the code they annotate.

翻译：  

注解是一种能被添加到java代码中的元数据，类、方法、变量、参数和包都可以用注解来修饰。注解对于它所修饰的代码并没有直接的影响。

通过官方描述得出以下结论：

注解是一种元数据形式。即注解是属于java的一种数据类型，和类、接口、数组、枚举类似。
注解用来修饰，类、方法、变量、参数、包。
注解不会对所修饰的代码产生直接的影响。

1.2 注解的使用范围
继续看看官方对它的使用范围的描述：

Annotations have a number of uses, among them:Information for the complier - Annotations can be used by the compiler to detect errors or suppress warnings.Compiler-time and deployment-time processing - Software tools can process annotation information to generate code, XML files, and so forth.Runtime processing - Some annotations are available to be examined at runtime.

翻译：

注解又许多用法，其中有：为编译器提供信息 - 注解能被编译器检测到错误或抑制警告。编译时和部署时的处理 - 软件工具能处理注解信息从而生成代码，XML文件等等。运行时的处理 - 有些注解在运行时能被检测到。

2 如何自定义注解
基于上一节，已对注解有了一个基本的认识：注解其实就是一种标记，可以在程序代码中的关键节点（类、方法、变量、参数、包）上打上这些标记，然后程序在编译时或运行时可以检测到这些标记从而执行一些特殊操作。因此可以得出自定义注解使用的基本流程：

第一步，定义注解——相当于定义标记；  
第二步，配置注解——把标记打在需要用到的程序代码中；  
第三步，解析注解——在编译期或运行时检测到标记，并进行特殊操作。  


2.1 基本语法  
注解类型的声明部分：  
注解在Java中，与类、接口、枚举类似，因此其声明语法基本一致，只是所使用的关键字有所不同@interface。在底层实现上，所有定义的注解都会自动继承  
```
java.lang.annotation.Annotation接口。
public @interface CherryAnnotation {
}
```
注解类型的实现部分：
根据我们在自定义类的经验，在类的实现部分无非就是书写构造、属性或方法。但是，在自定义注解中，其实现部分只能定义一个东西：注解类型元素（annotation type element）。咱们来看看其语法：
```
public @interface CherryAnnotation {
	public String name();
	int age();
	int[] array();
}
```
也许你会认为这不就是接口中定义抽象方法的语法嘛？别着急，咱们看看下面这个：
```
public @interface CherryAnnotation {
	public String name();
	int age() default 18;
	int[] array();
}
```
看到关键字default了吗？还觉得是抽象方法吗？
注解里面定义的是：注解类型元素！
定义注解类型元素时需要注意如下几点：

访问修饰符必须为public，不写默认为public；  
该元素的类型只能是基本数据类型、String、Class、枚举类型、注解类型（体现了注解的嵌套效果）以及上述类型的一位数组；  
该元素的名称一般定义为名词，如果注解中只有一个元素，请把名字起为value（后面使用会带来便利操作）；  
()不是定义方法参数的地方，也不能在括号中定义任何参数，仅仅只是一个特殊的语法；
default代表默认值，值必须和第2点定义的类型一致；  
如果没有默认值，代表后续使用注解时必须给该类型元素赋值。  

可以看出，注解类型元素的语法非常奇怪，即又有属性的特征（可以赋值）,又有方法的特征（打上了一对括号）。但是这么设计是有道理的，我们在后面的章节中可以看到：注解在定义好了以后，使用的时候操作元素类型像在操作属性，解析的时候操作元素类型像在操作方法。  
2.2 常用的元注解  
一个最最基本的注解定义就只包括了上面的两部分内容：1、注解的名字；2、注解包含的类型元素。但是，我们在使用JDK自带注解的时候发现，有些注解只能写在方法上面（比如@Override）；有些却可以写在类的上面（比如@Deprecated）。当然除此以外还有很多细节性的定义，那么这些定义该如何做呢？接下来就该元注解出场了！  
元注解：专门修饰注解的注解。它们都是为了更好的设计自定义注解的细节而专门设计的。我们为大家一个个来做介绍。  
2.2.1 @Target  
@Target注解，是专门用来限定某个自定义注解能够被应用在哪些Java元素上面的。它使用一个枚举类型定义如下：  
```
public enum ElementType {
    /** 类，接口（包括注解类型）或枚举的声明 */
    TYPE,

    /** 属性的声明 */
    FIELD,

    /** 方法的声明 */
    METHOD,

    /** 方法形式参数声明 */
    PARAMETER,

    /** 构造方法的声明 */
    CONSTRUCTOR,

    /** 局部变量声明 */
    LOCAL_VARIABLE,

    /** 注解类型声明 */
    ANNOTATION_TYPE,

    /** 包的声明 */
    PACKAGE
}
```
```
//@CherryAnnotation被限定只能使用在类、接口或方法上面
@Target(value = {ElementType.TYPE,ElementType.METHOD})
public @interface CherryAnnotation {
    String name();
    int age() default 18;
    int[] array();
}
```

2.2.2 @Retention  
@Retention注解，翻译为持久力、保持力。即用来修饰自定义注解的生命力。  
注解的生命周期有三个阶段：1、Java源文件阶段；2、编译到class文件阶段；3、运行期阶段。同样使用了RetentionPolicy枚举类型定义了三个阶段：  
```
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     * （注解将被编译器忽略掉）
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     * （注解将被编译器记录在class文件中，但在运行时不会被虚拟机保留，这是一个默认的行为）
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     * （注解将被编译器记录在class文件中，而且在运行时会被虚拟机保留，因此它们能通过反射被读取到）
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

我们再详解一下：  

如果一个注解被定义为RetentionPolicy.SOURCE，则它将被限定在Java源文件中，那么这个注解即不会参与编译也不会在运行期起任何作用，这个注解就和一个注释是一样的效果，只能被阅读Java文件的人看到；  
如果一个注解被定义为RetentionPolicy.CLASS，则它将被编译到Class文件中，那么编译器可以在编译时根据注解做一些处理动作，但是运行时JVM（Java虚拟机）会忽略它，我们在运行期也不能读取到；  
如果一个注解被定义为RetentionPolicy.RUNTIME，那么这个注解可以在运行期的加载阶段被加载到Class对象中。那么在程序运行阶段，我们可以通过反射得到这个注解，并通过判断是否有这个注解或这个注解中属性的值，从而执行不同的程序代码段。我们实际开发中的自定义注解几乎都是使用的RetentionPolicy.RUNTIME；  
在默认的情况下，自定义注解是使用的RetentionPolicy.CLASS。  

2.2.3 @Documented  
@Documented注解，是被用来指定自定义注解是否能随着被定义的java文件生成到JavaDoc文档当中。   
2.2.4 @Inherited  
@Inherited注解，是指定某个自定义注解如果写在了父类的声明部分，那么子类的声明部分也能自动拥有该注解。@Inherited注解只对那些@Target被定义为ElementType.TYPE的自定义注解起作用。  
3 自定义注解的配置使用
回顾一下注解的使用流程：

第一步，定义注解——相当于定义标记；    
第二步，配置注解——把标记打在需要用到的程序代码中；    
第三步，解析注解——在编译期或运行时检测到标记，并进行特殊操作。    

到目前为止我们只是完成了第一步，接下来我们就来学习第二步，配置注解，如何在另一个类当中配置它。  
3.1 在具体的Java类上使用注解  
首先，定义一个注解、和一个供注解修饰的简单Java类   
```
@Retention(RetentionPolicy.RUNTIME)
@Target(value = {ElementType.METHOD})
@Documented
public @interface CherryAnnotation {
    String name();
    int age() default 18;
    int[] score();
}
12345678
public class Student{
    public void study(int times){
        for(int i = 0; i &lt; times; i++){
            System.out.println("Good Good Study, Day Day Up!");
        }
    }
}
```

简单分析下：  

CherryAnnotation的@Target定义为ElementType.METHOD，那么它书写的位置应该在方法定义的上方，即：public void study(int times)之上；  
由于我们在CherryAnnotation中定义的有注解类型元素，而且有些元素是没有默认值的，这要求我们在使用的时候必须在标记名后面打上()，并且在()内以“元素名=元素值“的形式挨个填上所有没有默认值的注解类型元素（有默认值的也可以填上重新赋值），中间用“,”号分割；  

所以最终书写形式如下：  
```
public class Student {
    @CherryAnnotation(name = "cherry-peng",age = 23,score = {99,66,77})
    public void study(int times){
        for(int i = 0; i &lt; times; i++){
            System.out.println("Good Good Study, Day Day Up!");
        }
    }
}
```
3.2 特殊语法  
特殊语法一：  
如果注解本身没有注解类型元素，那么在使用注解的时候可以省略()，直接写为：@注解名，它和标准语法@注解名()等效！  
```
@Retention(RetentionPolicy.RUNTIME)
@Target(value = {ElementType.TYPE})
@Documented
public @interface FirstAnnotation {
}
12345
//等效于@FirstAnnotation()
@FirstAnnotation
public class JavaBean{
	//省略实现部分
}
```
特殊语法二：  
如果注解本本身只有一个注解类型元素，而且命名为value，那么在使用注解的时候可以直接使用：@注解名(注解值)，其等效于：@注解名(value = 注解值)  
```
@Retention(RetentionPolicy.RUNTIME)
@Target(value = {ElementType.TYPE})
@Documented
public @interface SecondAnnotation {
	String value();
}
```
```
//等效于@ SecondAnnotation(value = "this is second annotation")
@SecondAnnotation("this is annotation")
public class JavaBean{
	//省略实现部分
}
```

特殊用法三：  
如果注解中的某个注解类型元素是一个数组类型，在使用时又出现只需要填入一个值的情况，那么在使用注解时可以直接写为：@注解名(类型名 = 类型值)，它和标准写法：@注解名(类型名 = {类型值})等效！  
```
@Retention(RetentionPolicy.RUNTIME)
@Target(value = {ElementType.TYPE})
@Documented
public @interface ThirdAnnotation {
	String[] name();
}
```
```
//等效于@ ThirdAnnotation(name = {"this is third annotation"})
@ ThirdAnnotation(name = "this is third annotation")
public class JavaBean{
	//省略实现部分
}
```
特殊用法四：  
如果一个注解的@Target是定义为Element.PACKAGE，那么这个注解是配置在package-info.java中的，而不能直接在某个类的package代码上面配置。  
4 自定义注解的运行时解析  
这一章是使用注解的核心，读完此章即可明白，如何在程序运行时检测到注解，并进行一系列特殊操作！  
4.1 回顾注解的保持力  
首先回顾一下，之前自定义的注解@CherryAnnotation，并把它配置在了类Student上，代码如下：  
```
@Retention(RetentionPolicy.RUNTIME)
@Target(value = {ElementType.METHOD})
@Documented
public @interface CherryAnnotation {
    String name();
    int age() default 18;
    int[] score();
}
```
```
package pojos;
public class Student {
    @CherryAnnotation(name = "cherry-peng",age = 23,score = {99,66,77})
    public void study(int times){
        for(int i = 0; i &lt; times; i++){
            System.out.println("Good Good Study, Day Day Up!");
        }
    }
}
```
注解保持力的三个阶段：  

Java源文件阶段；  
编译到class文件阶段；  
运行期阶段。  

只有当注解的保持力处于运行阶段，即使用@Retention(RetentionPolicy.RUNTIME)修饰注解时，才能在JVM运行时，检测到注解，并进行一系列特殊操作。  
4.2 反射操作获取注解  
因此，明确我们的目标：在运行期探究和使用编译期的内容（编译期配置的注解），要用到Java中的灵魂技术——反射！  
```
public class TestAnnotation {
    public static void main(String[] args){
        try {
            //获取Student的Class对象
            Class stuClass = Class.forName("pojos.Student");

            //说明一下，这里形参不能写成Integer.class，应写为int.class
            Method stuMethod = stuClass.getMethod("study",int.class);

            if(stuMethod.isAnnotationPresent(CherryAnnotation.class)){
                System.out.println("Student类上配置了CherryAnnotation注解！");
                //获取该元素上指定类型的注解
                CherryAnnotation cherryAnnotation = stuMethod.getAnnotation(CherryAnnotation.class);
                System.out.println("name: " + cherryAnnotation.name() + ", age: " + cherryAnnotation.age()
                    + ", score: " + cherryAnnotation.score()[0]);
            }else{
                System.out.println("Student类上没有配置CherryAnnotation注解！");
            }
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
    }
}
```
解释一下：  

如果我们要获得的注解是配置在方法上的，那么我们要从Method对象上获取；如果是配置在属性上，就需要从该属性对应的Field对象上去获取，如果是配置在类型上，需要从Class对象上去获取。总之在谁身上，就从谁身上去获取！  
isAnnotationPresent(Class&lt;? extends Annotation&gt; annotationClass)方法是专门判断该元素上是否配置有某个指定的注解；  
getAnnotation(Class&lt;A&gt; annotationClass)方法是获取该元素上指定的注解。之后再调用该注解的注解类型元素方法就可以获得配置时的值数据；  
反射对象上还有一个方法getAnnotations()，该方法可以获得该对象身上配置的所有的注解。它会返回给我们一个注解数组，需要注意的是该数组的类型是Annotation类型，这个Annotation是一个来自于java.lang.annotation包的接口。  

