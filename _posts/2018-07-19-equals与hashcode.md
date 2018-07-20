---
layout:     post
title:      equals与hashcode的稍深入理解
date:       2018-07-19
author:     yunzhs
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - 随笔
typora-root-url: ..
typora-copy-images-to: ..\img\posts
---

### **1、equals的作用及与==的区别。**

equals被用来判断两个对象是否相等。

equals通常用来比较两个对象的**内容**是否相等，==用来比较两个对象的**地址**是否相等。

equals方法**默认等同于“==”**

Object类中的equals方法定义为判断两个对象的地址是否相等（可以理解成是否是同一个对象），地址相等则认为是对象相等。这也就意味着，我们新建的所有类**如果没有复写equals方法**，那么判断两个对象是否相等时**就等同于“==**”，也就是两个对象的地址是否相等。

Object类中equals的方法实现如下：

```
public boolean equals(Object obj) {
        return (this == obj);
    }
```

但在我们的实际开发中，通常会认为两个对象的内容相等时，则两个对象相等，equals返回true。对象内容不同，则返回false。

所以可以总结为两种情况

1、类未复写equals方法，则使用equals方法比较两个对象时，相当于==比较，即两个对象的地址是否相等。地址相等，返回true，地址不相等，返回false。

2、类复写equals方法，比较两个对象时，则走复写之后的判断方式。通常，我们会将equals复写成：当两个对象内容相同时，则equals返回true，内容不同时，返回false。

举个例子：

```

public class EqualTest {
public static void main(String[] args) {
Person p1 = new Person(10,"张三");
Person p2 = new Person(10,"张三");
System.out.println(p1.equals(p2));
}
}
class Person{
int age;
String name;
public Person(int age, String name) {
super();
this.age = age;
this.name = name;
}
public int getAge() {
return age;
}
public void setAge(int age) {
this.age = age;
}
public String getName() {
return name;
}
public void setName(String name) {
this.name = name;
}
}
```



Person未复写equals方法，则默认使用了Object中的equals，即为两个对象（p1和p2）的内存地址判断，p1和p2很明显内存地址不同，所以输出结果很明显为false。

如果我们复写equals方法呢？我们认为名字和年龄一样的就是同一个人，那么p1和p2都表示10岁的张三，这两个对象应该是相等的。复写的equals方法如下：

```

@Override
public boolean equals(Object obj) {
if (this == obj)
return true;
if (obj == null)
return false;
if (getClass() != obj.getClass())
return false;
Person other = (Person) obj;
if (age != other.age)
return false;
if (name == null) {
if (other.name != null)
return false;
} else if (!name.equals(other.name))
return false;
return true;
}
```

同样的，执行上述用例，得到的结果是true。

BTW：如果equals方法返回true，那么==是否也是true？

不一定是true。equals返回true有两种可能，一种是两个对象地址相同，一种是两个对象内容相同。当内容相同时，地址可能不同，所以==比较的结果可能为false。

我们把main方法加上对==的判断，如下：

```

public static void main(String[] args) {
Person p1 = new Person(10,"张三");
Person p2 = new Person(10,"张三");
System.out.println(p1.equals(p2));
System.out.println(p1 == p2);
}
```

输出结果很明显 p1==p2的结果是false。

补充Java中对Equals的要求：

### **2、hashCode的作用及与equals的关系。**

hashCode的作用是用来获取哈希码，也可以称作散列码。实际返回值为一个int型数据。用于确定对象在哈希表中的位置。

Object中有hashcode方法，也就意味着所有的类都有hashCode方法。

但是，hashcode只有在创建某个类的散列表的时候才有用，需要根据hashcode值确认对象在散列表中的位置，但在其他情况下没用。

java中本质上是散列表的类常见的有HashMap，HashSet，HashTable

所以，如果一个对象一定不会在散列表中使用，那么是没有必要复写hashCode方法的。但一般情况下我们还是会复写hashCode方法，因为谁能保证这个对象不会出现再hashMap等中呢？

举个例子：

```

public class EqualTest {
public static void main(String[] args) {
Person p1 = new Person(10, "张三");
Person p2 = new Person(10, "张三");
System.out.println(
"p1.equals(p2)=" + p1.equals(p2) + ", p1.hashcode=" + p1.hashCode() + ", p2.hashcode=" +p2.hashCode());
}
}
class Person {
int age;
String name;
public Person(int age, String name) {
super();
this.age = age;
this.name = name;
}
public int getAge() {
return age;
}
public void setAge(int age) {
this.age = age;
}
public String getName() {
return name;
}
public void setName(String name) {
this.name = name;
}
@Override
public boolean equals(Object obj) {
if (this == obj)
return true;
if (obj == null)
return false;
if (getClass() != obj.getClass())
return false;
Person other = (Person) obj;
if (age != other.age)
return false;
if (name == null) {
if (other.name != null)
return false;
} else if (!name.equals(other.name))
return false;
return true;
}
}
```

两个对象equals相等的时候，hashcode并不一定相等。

Person没有复写hashCode方法，使用Object默认的hashCode实现，输出结果如下：

```
p1.equals(p2)=true, p1.hashcode=246688959, p2.hashcode=1457895203
```

从结果可以看出，equals虽然相同，但是p1和p2的hashcode并不相同。

如果Person用于散列表的类中呢，这里用HashSet来做测试。

```
public class EqualTest {
public static void main(String[] args) {
Person p1 = new Person(10, "张三");
Person p2 = new Person(10, "张三");
System.out.println(
"p1.equals(p2)=" + p1.equals(p2) + ", p1.hashcode=" + p1.hashCode() + ", p2.hashcode=" +p2.hashCode());
HashSet<Person> set = new HashSet<Person>();
set.add(p1);
set.add(p2);
System.out.println(set);
}
}
class Person {
int age;
String name;
public Person(int age, String name) {
super();
this.age = age;
this.name = name;
}
public int getAge() {
return age;
}
public void setAge(int age) {
this.age = age;
}
public String getName() {
return name;
}
public void setName(String name) {
this.name = name;
}
@Override
public boolean equals(Object obj) {
if (this == obj)
return true;
if (obj == null)
return false;
if (getClass() != obj.getClass())
return false;
Person other = (Person) obj;
if (age != other.age)
return false;
if (name == null) {
if (other.name != null)
return false;
} else if (!name.equals(other.name))
return false;
return true;
}
@Override
public String toString() {
return "Person [age=" + age + ", name=" + name + "]";
}
}
```

输出结果

```
p1.equals(p2)=true, p1.hashcode=246688959, p2.hashcode=1457895203
[Person [age=10, name=张三], Person [age=10, name=张三]]
```

p1和p2的equals相同，我们认为是两个对象相等，但是这两个对象竟然同时出现再hashSet中，hashSet中是不会出现两个相同的元素的。 

那问题在哪里？

hashset在添加一个元素的时候，会做如下判断：

1、如果添加元素的hashcode相等并且 对象equals为true或对象== 时，则认为是同一个元素，不添加到新元素中。

2、如果不符合上述条件，则认为是一个新元素，添加到set中。

所以，虽然p1和p2equals比较时相等，但是hashcode并不一样，所以在往set中添加的时候认为是两个不同的元素，所以才会出现了p1和p2同时在set中的情况。

我们改进下，复写一下hashcode方法，如下：

重新执行结果：

```
class Person {
int age;
String name;
public Person(int age, String name) {
super();
this.age = age;
this.name = name;
}
public int getAge() {
return age;
}
public void setAge(int age) {
this.age = age;
}
public String getName() {
return name;
}
public void setName(String name) {
this.name = name;
}
@Override
public int hashCode() {
final int prime = 31;
int result = 1;
result = prime * result + age;
result = prime * result + ((name == null) ? 0 : name.hashCode());
return result;
}
@Override
public boolean equals(Object obj) {
if (this == obj)
return true;
if (obj == null)
return false;
if (getClass() != obj.getClass())
return false;
Person other = (Person) obj;
if (age != other.age)
return false;
if (name == null) {
if (other.name != null)
return false;
} else if (!name.equals(other.name))
return false;
return true;
}
@Override
public String toString() {
return "Person [age=" + age + ", name=" + name + "]";
}
}
```

于是看到set中仅有一个Person值了。

```
p1.equals(p2)=true, p1.hashcode=776160, p2.hashcode=776160
[Person [age=10, name=张三]]
```

**补充几点：**

1、新建一个类，尤其是业务相关的对象类的时候，最好复写equals方法。

2、复写equals方法时，同时记着要复写hashCode方法，谁能保证说这个对象一定不会出现在hashMap中呢？如果你用的是eclipse的自动代码生成，你会发现eclipse中复写equals和hashCode是在一起的。

**引申出几个经常在面试中问到的问题：**

​     1、两个对象，如果a.equals(b)==true，那么a和b是否相等？

​          相等，但地址不一定相等。

​     2、两个对象，如果hashcode一样，那么两个对象是否相等？

​          不一定相等，可能存在不同对象也hashcode相等,判断两个对象是否相等，需要判断equals是否为true。