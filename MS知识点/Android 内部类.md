Android 内部类
===

## 一、为何使用内部类

*   内部类提供了更好的封装，只有外部类能访问内部类
*   内部类可以独立继承一个接口，不受外部类是否继承接口影响
*   内部类中的属性和方法即使是外部类也不能直接访问，相反内部类可以直接访问外部类的属性和方法，即使private
*   利于回调函数的编写

**一个内部类的例子**：

```
public class OuterClass {
    private String outerName;
    private int outerAge;
    public class InnerClass{
        private String innerName;
        private int innerAge;
    }
}

```

## 二、内部类与外部类的联系

### 2.1 内部类是一个相对独立的实体，与外部类不是is-a关系

内部类是一个编译时概念，编译后外部类及其内部类会生成两个独立的class文件： `OuterClass.class`和`OuterClass$InnerClass.class`，我用`javac`编译器对上面的`OuterClass`进行编译：

```
 D:\>javac OuterClass.class

```

编译后的结果：

![这里写图片描述](http://img.blog.csdn.net/20160630170618001)

### 2.2 内部类可以直接访问外部类的元素，但是外部类不可以直接访问内部类的元素

```
public class OuterClass {

    private String outerName;
    private int outerAge;

    public class InnerClass{
        private int innerName;
        InnerClass(){
            //内部类可以访问外部类的元素
            outerName="I am outer class";
            outerAge=23;
        }
        public void display(){
            System.out.println(outerName+" and my age is "+outerAge);
        }
    }
    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        OuterClass.InnerClass innerClass = outerClass.new InnerClass();
        innerClass.display();
    }
}

```

在上面例子中我们可以看到，内部类可以直接访问外部类属性，尽管外部类属性是用private修饰的。这是因为在创建外部类时，内部类会自动捕获一个外部类的引用，所以内部类访问外部类元素，实际上是通过他所持有外部类引用访问的。在java中，我们可以通过`OuterClass.this`来获得外部类的引用，请看下面例子：

```
public class OuterClass {
    public void display(){
        System.out.println("this is OuterClass...");
    }
    public class InnerClass{
        //获取外部类的引用
        public OuterClass getOuterClass(){
            return OuterClass.this;
        }
        public void innerDisplay(){
            //内部类也可以通过外部类的引用访问外部元素
            getOuterClass().display();
        }
    }
    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        OuterClass.InnerClass innerClass = outerClass.new InnerClass();
        innerClass.innerDisplay();
    }
}

```

### 2.3 外部类可以通过内部类引用**间接**访问内部类元素

```
public class OuterClass {
    public void display(){
        //外部类访问内部类元素，需要通过内部类引用访问
        InnerClass innerClass=new InnerClass();
        innerClass.innerDisplay();
    }
    public class InnerClass{
        public void innerDisplay(){
            System.out.println("I am inner class");
        }
    }
    public static void main(String[] args) {
        OuterClass outerClass=new OuterClass();
        outerClass.display();
    }
}

```

## 三、创建内部类

### 3.1 在外部类外面（或外部类main方法）创建内部了对象

其实上面`2.2`例子中我们已经看到了如何创建内部类。如果要创建一个内部类对象，必须利用`outerClass.new`来创建：

```
OuterClass outerClass = new OuterClass();
OuterClass.InnerClass innerClass = outerClass.new InnerClass();

```

其实我们还可以一步到位：

```
OuterClass.InnerClass innerClass=new OuterClass().new InnerClass();

```

内部类创建方法示例：

```
public static void main(String[] args) {
    //先创建外部类对象，再创建内部类对象
    OuterClass outerClass = new OuterClass();
    OuterClass.InnerClass innerClass1 = outerClass.new InnerClass();
    innerClass1.innerDisplay();
    //一步到位创建
    OuterClass.InnerClass innerClass2=new OuterClass().new InnerClass();
    innerClass2.innerDisplay();
}

```

### 3.2 在外部类里面创建内部类

正如`2.3`代码中`display()`方法那样，在外部类里面创建内部类，就像创建普通对象一样直接创建：

```
InnerClass innerClass=new InnerClass()

```

## 四、内部类的种类：

在Java中内部类主要分为成员内部类、方法内部类、匿名内部类、静态内部类。

### 4.1 成员内部类

成员内部类也是最普通的内部类，它是外部类的一个成员，所以他是可以无限制的访问外围类的所有成员属性和方法，尽管是private的，但是外围类要访问内部类的成员属性和方法则需要通过内部类实例来访问。

在成员内部类中要注意两点：

1.  成员内部类中不能存在任何static的变量和方法
2.  成员内部类是依附于外围类的，所以只有先创建了外围类才能够创建内部类

![](https://t11.baidu.com/it/u=3374719981,1384879290&fm=173&s=A8C2B74ABEE0B76C18D9CC070000A0C3&w=640&h=333&img.JPEG)

### 4.2 方法内部类

方法内部类定义在外部类的方法中，局部内部类和成员内部类基本一致，只是它们的作用域不同，方法内部类只能在该方法中被使用，出了该方法就会失效。 对于这个类的使用主要是应用与解决比较复杂的问题，想创建一个类来辅助我们的解决方案，到那时又不希望这个类是公共可用的，所以就产生了局部内部类。

![](https://t10.baidu.com/it/u=3534311715,1105954985&fm=173&s=A8C2834FCAA5F3745E71C40F0000B0C3&w=640&h=201&img.JPEG)
### 4.3 匿名内部类(OnClick事件)

匿名内部类其实就是一个没有名字的方法内部类，所以它符合方法内部类的所有约束，初次之外，还有一些地方需要注意：

1.  匿名内部类是没有访问修饰符的。
2.  匿名内部类必须继承一个抽象类或者实现一个接口
3.  匿名内部类中不能存在任何静态成员或方法
4.  匿名内部类是没有构造方法的，因为它没有类名。

一般使用匿名内部类的场景是，要继承或实现的接口只有一个抽象方法，比如添加一个监听器：

```
public class Button {
    public void click(){
        //匿名内部类，实现的是ActionListener接口
        new ActionListener(){
            public void onAction(){
                System.out.println("click action...");
            }
        }.onAction();
    }
    //匿名内部类必须继承或实现一个已有的接口
    public interface ActionListener{
        public void onAction();
    }

    public static void main(String[] args) {
        Button button=new Button();
        button.click();
    }
}

```
![](https://t10.baidu.com/it/u=2720160774,1172617468&fm=173&s=A8C2874E9AE6A36C5E7DE40F0000E0C3&w=640&h=436&img.JPEG)

### 4.4 静态内部类

关键字`static`可以修饰成员变量、方法、代码块，其实它还可以修饰内部类，使用`static`修饰的内部类我们称之为静态内部类。静态内部类与非静态内部类之间存在一个最大的区别，我们知道非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围内，但是静态内部类却没有。没有这个引用就意味着：

1.  静态内部类的创建是不需要依赖于外围类，可以直接创建
2.  静态内部类不可以使用任何外围类的非static成员变量和方法，而内部类则都可以

    ```
    public class OuterClass {
        private static String outerName;
        public  int age;
        static class InnerClass1{
            /* 在静态内部类中可以存在静态成员 */
            public static String _innerName = "static variable";
            public void display(){
                /*
                 * 静态内部类只能访问外部类的静态成员变量和方法
                 * 不能访问外部类的非静态成员变量和方法
                 */
                System.out.println("OutClass name :" + outerName);
            }
        }
        class InnerClass2{
            /* 非静态内部类中不能存在静态成员 */
            public String _innerName = "no static variable";
            /* 非静态内部类中可以调用外部类的任何成员,不管是静态的还是非静态的 */
            public void display(){
                System.out.println("OuterClass name：" + outerName);
                System.out.println("OuterClass age：" + age);
            }
        }
        public void display(){
            /* 外部类能直接访问静态内部类静态元素 */
            System.out.println(InnerClass1._innerName);
            /* 静态内部类可以直接创建实例不需要依赖于外部类 */
            new InnerClass1().display();
            /* 非静态内部的创建需要依赖于外部类 */
            OuterClass.InnerClass2 inner2 = new OuterClass().new InnerClass2();
            /* 非静态内部类的成员需要使用非静态内部类的实例访问 */
            System.out.println(inner2._innerName);
            inner2.display();
        }
        public static void main(String[] args) {
            OuterClass outer = new OuterClass();
            outer.display();
        }
    }
    ```
![](https://t12.baidu.com/it/u=2540737979,3017205369&fm=173&s=A8C2A74A1AE0A7740C4D840F0000E0C3&w=640&h=268&img.JPEG)