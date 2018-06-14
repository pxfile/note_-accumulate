MS_java
===

## 执行代码输出结果：
**（先加载父类的静态代码块，接着是子类的静态代码块，先执行父类的构造函数，接着是子类的构造函数）**
static Base
static Sub
Base: new Sub
Sub: new Sub

## [Java中private、protected、public和default的区别](http://www.cnblogs.com/jingmengxintang/p/5898900.html)

public：

具有最大的访问权限，可以访问任何一个在classpath下的类、接口、异常等。它往往用于对外的情况，也就是对象或类对外的一种接口的形式。

protected：

主要的作用就是用来保护子类的。它的含义在于子类可以用它修饰的成员，其他的不可以，它相当于传递给子类的一种继承的东西

default：

有时候也称为friendly，它是针对本包访问而设计的，任何处于本包下的类、接口、异常等，都可以相互访问，即使是父类没有用protected修饰的成员也可以。

private：

访问权限仅限于类的内部，是一种封装的体现，例如，大多数成员变量都是修饰符为private的，它们不希望被其他任何外部的类访问。

![](https://images2015.cnblogs.com/blog/690292/201609/690292-20160923095944481-1758567758.png)

注意：java的访问控制是停留在编译层的，也就是它不会在.class文件中留下任何的痕迹，只在编译的时候进行访问控制的检查。其实，通过反射的手段，是可以访问任何包下任何类中的成员，例如，访问类的私有成员也是可能的。

区别：

public：可以被所有其他类所访问

private：只能被自己访问和修改

protected：自身、子类及同一个包中类可以访问

default：同一包中的类可以访问，声明时没有加修饰符，认为是friendly

# java泛型类型擦除发生在什么时候，通配符有什么需要注意的。

1.发生在编译的时候
2.PECS，extends善于提供精确的对象 A是B的子集，Super善于插入精确的对象 A是B的超集
3.博客推荐：**Effective Java笔记（不含反序列化、并发、注解和枚举）**、**android阿里面试java基础锦集**

# java 多态

## 多态的存在有**三个前提**:
**1.要有继承关系**
**2.子类要重写父类的方法**
**3.父类引用指向子类对象**

## 多态成员访问的特点：
**成员变量**：编译看左边（父类），运行看左边（父类）
**成员方法**：编译看左边（父类），运行看右边（父类）
**静态方法**：编译看左边（父类），运行看左边（父类）
**只有非静态成员方法，编译看左边，运行看右边**

## 多态有什么弊端呢
即多态后不能使用子类特有的属性和方法

## 为何使用内部类

*   内部类提供了更好的封装，只有外部类能访问内部类
*   内部类可以独立继承一个接口，不受外部类是否继承接口影响
*   内部类中的属性和方法即使是外部类也不能直接访问，相反内部类可以直接访问外部类的属性和方法，即使private
*   利于回调函数的编写，而内部类有效地实现了“多重继承”。

## 抽象类的意义

为其子类提供一个公共的类型 封装子类中得重复内容 定义抽象方法，子类虽然有不同的实现 但是定义是一致的

## 抽象类与接口的应用场景

### 语义上的区别：

首先 类描述的是 这个东西是什么（强调所属）？包含了静态属性，静态行为 ，属性和行为。

而接口 描述的它能做什么事儿（强调行为）？     只是 静态常量属性 和 行为

### 使用场景

**在设计类时，如果有些方法我们能确定，而有些方法不能确定，这时候我们就可以把该类声明成抽象类**。

抽象类的应用场景非常多，例如**模板方法模式**就是抽象类的一个应用，

#### interface的应用场合 

* A. 类与类之前需要特定的接口进行协调，而不在乎其如何实现。       
* B. 作为能够实现特定功能的标识存在，也可以是什么接口方法都没有的纯粹标识。       
* C. 需要将一组类视为单一的类，而调用者只通过接口来与这组类发生联系。       
* D. 需要实现特定的多项功能，而这些功能之间可能完全没有任何联系。

#### abstract class的应用场合
一句话，在既需要统一的接口，又需要实例变量或缺省的方法的情况下，就可以使用它。

* A. 定义了一组接口，但又不想强迫每个实现类都必须实现所有的接口。可以用abstract class定义一组方法体，甚至可以是空方法体，然后由子类选择自己所感兴趣的方法来覆盖。        

* B. 某些场合下，只靠纯粹的接口不能满足类与类之间的协调，还必需类中表示状态的变量来区别不同的关系。abstract的中介作用可以很好地满足这一点。

* C. 规范了一组相互协调的方法，其中一些方法是共同的，与状态无关的，可以共享的，无需子类分别实现；而另一些方法却需要各个子类根据自己特定的状态来实现特定的功能

## [java泛型中的通配符 extends与super](https://blog.csdn.net/bitcarmanlee/article/details/78313983)

java泛型中有如下关键字： 

1\. ? 表示通配符类型 

2\. <? extends T> 既然是extends，就是表示泛型参数类型的上界，说明参数的类型应该是T或者T的子类。 

3\. <? super T> 既然是super，表示的则是类型的下界，说明参数的类型应该是T类型的父类，一直到object。

extends 可用于的返回类型限定，不能用于参数类型限定。 
super 可用于参数类型限定，不能用于返回类型限定。 
带有super超类型限定的通配符可以向泛型对易用写入，带有extends子类型限定的通配符可以向泛型对象读取

## Java中静态变量与静态方法的继承
 
* 1.静态变量与静态方法说继承并不确切，静态方法与变量是属于类的方法与变量。而子类也属于超类，比如说Manage extends Employee，则Manage也是一个Employee，所以子类能够调用属于超类的静态变量和方法。注意，子类调用的其实就是超类的静态方法和变量，而不是继承自超类的静态方法与变量。但是如果子类中有同名的静态方法与变量，这时候调用的就是子类本身的，因为子类的静态变量与静态方法会隐藏父类的静态方法和变量。

* 2.如果子类中没有定义同名的变量和方法，那么调用 "子类名.静态方法/变量"调用的是父类的方法及变量

* 3,.如果子类中只定义了同名静态变量，而没有定义与父类同名静态方法，则调用”子类名.静态方法"时，调   用的是父类的静态方法，静态方法中的静态变量也是父类的 (如程序中注[1])

* 4.如果子类中既定义了与父类同名的静态变量，也定义了与父类同名的静态方法，这时候调用”子类名.静态方法"时，完全与父类无关，里面的静态变量也是子类的(如程序中注[2])

## java中静态内部类的设计意图

**非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围内，但是静态内部类却没有。**

没有这个引用就意味着：

*   它的创建是不需要依赖于外围类的。

*   它不能使用任何外围类的非static成员变量和方法。

*   使用静态内部类，多个外部类的对象可以共享同一个内部类的对象。

使用普通内部类，每个外部类的对象都有自己的内部类对象，外部对象之间不能共享内部类的对象

## utf-8编码中的中文占几个字节；int型几个字节

一个utf8数字占1个字节

一个utf8英文字母占1个字节

少数是汉字每个占用3个字节，多数占用4个字节。

## 静态代理和动态代理的区别，什么场景使用

**静态**也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。
应用：一些第三方框架的代理，便后后期替换或者定制化变更。

**动态代理类**的源码是在程序运行期间由JVM根据反射等机制动态的生成，所以不存在代理类的字节码文件。代理类和委托类的关系是在程序运行时确定。
应用：被代理类庞大时，需要在某些方法执行前后处理一些事情时，亦或接口类与实现类经常变动时(因为使用反射所以方法的增删改并不需要修改`invoke`方法)。

## 使用Java序列化把对象存储到文件中，再从文件中读出来
使用Java序列化把对象存储到文件中去，再从文件中读取出来。

此时，我们使用ObjectOutputStream和ObjectInputStream来进行对象的读取。

使用ObjectOutputStream对象的writeObject()方法来进行对象的写入。

使用ObjectInputStream对象的readObject()方法来读取对象。
```
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import junit.framework.TestCase;
/**
 * 使用Java序列化把对象存储到文件中，再从文件中读出来 注意读取的时候，读取数据的顺序一定要和存放数据的顺序保持一致
 * 
 * @author Champion Wong
 * 
 */
public class Test08 extends TestCase {
	public void test() {
		// 创建一个User对象
		User user = new User();
		user.setId(1);
		user.setName("Mr XP.Wang");
		// 创建一个List对象
		List<String> list = new ArrayList<String>();
		list.add("My name");
		list.add(" is");
		list.add(" Mr XP.Wang");
		try {
			ObjectOutputStream os = new ObjectOutputStream(
					new FileOutputStream("C:/wxp.txt"));
			os.writeObject(user);// 将User对象写进文件
			os.writeObject(list);// 将List列表写进文件
			os.close();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		try {
			ObjectInputStream is = new ObjectInputStream(new FileInputStream(
					"C:/wxp.txt"));
			User temp = (User) is.readObject();// 从流中读取User的数据
			System.out.println(temp.getId());
			System.out.println(temp.getName());
			List tempList = (List) is.readObject();// 从流中读取List的数据
			for (Iterator iterator = tempList.iterator(); iterator.hasNext();) {
				System.out.print(iterator.next());
			}
			is.close();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}
	}
}
class User implements Serializable {
	private int id;
	private String name;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
}

```
## 一般Java对象在虚拟机上有7个运行阶段：
创建阶段->应用阶段->不可见阶段->不可达阶段->收集阶段->终结阶段->对象空间重新分配阶段

## 静态变量存储在_A_区 
A 全局区 
B 堆 
C 栈 
D 常量区

**知识点**

内存到底分几个区？

1、栈区（stack）— 由编译器自动分配释放 ，存放**函数的参数值**，**局部变量的值**等。

2、堆区（heap） — 一般由程序员分配释放， 若程序员不释放，程序结束时可能由os回收 。注意它与数据结构中的堆是两回事，分配方式倒是类似于链表。

3、全局区（静态区）（static）—**全局变量和静态变量**的存储是放在一块的，初始化的全局变量和静态变量在一块区域， 未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。程序结束后有系统释放。

4、文字常量区 —常量**字符串**就是放在这里的。 程序结束后由系统释放。

5、程序代码区—存放函数体的二进制代码。

# 重写(Overriding)和重载(Overloading)
方法的重写(Overriding)和重载(Overloading)是java多态性的不同表现，重写是父类与子类之间多态性的一种表现，重载可以理解成多态的具体表现形式。

*   (1)方法重载是一个类中定义了多个方法名相同,而他们的参数的数量不同或数量相同而类型和次序不同,则称为方法的重载(Overloading)。
*   (2)方法重写是在子类存在方法与父类的方法的名字相同,而且参数的个数与类型一样,返回值也一样的方法,就称为重写(Overriding)。
*   (3)方法重载是一个类的多态性表现,而方法重写是子类与父类的一种多态性表现。
