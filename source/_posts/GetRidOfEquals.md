title: 摆脱 Equals, CompareTo and toString
date: 2016-03-09 16:29:51
tags: 
- JAVA
- LAMBDA

categories: JAVA
---
[译][https://dzone.com/articles/get-rid-of-equals-compareto-and-tostring](https://dzone.com/articles/get-rid-of-equals-compareto-and-tostring)

在我们平时编写POJOs常常要去覆盖方法.equals(), .compareTo() 以及 .toString() ,但是我们有另外一种方法可以使代码更加整洁并做到代码分离

## 整洁性-摆脱 `Equals`, `CompareTo`, `toString`

是否看过java Object类的Javadoc, 是否不得不是不是的浏览那颗继承树.你一定会注意到有几个方法是我们必须继承的.
`.toString()`, `.equals()` 和`.hashCode()`是我们最有可能自己实现,而不是用原本的方法(为什么要去实现可以参考[Minborg的博客](https://minborgsjavapot.blogspot.com/2014/10/new-java-8-object-support-mixin-pattern.html))

不过这些方法有可能还不够,很多情况要继承标准库里像是`Comparable`和`Serializable`这样的接口.但是这样真的很明智吗?
为何大家都愿意去自己实现那些方法?当然如果你要存储数据到`HashMap`或是控制哈希冲突的话,自己实现`.equal()`和`.hashCode()`就有意义了,那`.compareTo()`和`.toString()`又有必要吗?

这里介绍一种用在开源项目[Speedment](https://github.com/speedment/speedment)软件设计方法,这个设计的思想是对对象的运作是使用存贮在变量里的函数引用而不是去重写JAVA内购方法.这样做有几个好处.我们的POJOs会变得剪短和整洁,不用继承可以做到代码重用,并且可以灵活的在不同配置下切换
<!--more-->

# *普通代码*
下面举例展示.新建一个典型的java类 Person,新建一组Person于`Set`中,然后按照名和姓进行排序(按名排序优先,再按姓排,以防出现同名)

Person.java
``` java
public class Person implements Comparable<Person> {
    private final String firstname;
    private final String lastname;
    public Person(String firstname, String lastname) {
        this.firstname = firstname;
        this.lastname  = lastname;
    }
    public String getFirstname() {
        return firstname;
    }
    public String getLastname() {
        return lastname;
    }
    @Override
    public int hashCode() {
        int hash = 7;
        hash = 83 * hash + Objects.hashCode(this.firstname);
        hash = 83 * hash + Objects.hashCode(this.lastname);
        return hash;
    }
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null) return false;
        if (getClass() != obj.getClass()) return false;
        final Person other = (Person) obj;
        if (!Objects.equals(this.firstname, other.firstname)) {
            return false;
        }
        return Objects.equals(this.lastname, other.lastname);
    }
    @Override
    public int compareTo(Person that) {
        if (this == that) return 0;
        else if (that == null) return 1;
        int comparison = this.firstname.compareTo(that.firstname);
        if (comparison != 0) return comparison;
        comparison = this.lastname.compareTo(that.lastname);
        return comparison;
    }
    @Override
    public String toString() {
        return firstname + " " + lastname;
    }
}
```

Main.java
``` java
public class Main {
    public static void main(String... args) {
        final Set
      people = new HashSet<>();
        people.add(new Person("Adam", "Johnsson"));
        people.add(new Person("Adam", "Samuelsson"));
        people.add(new Person("Ben", "Carlsson"));
        people.add(new Person("Ben", "Carlsson"));
        people.add(new Person("Cecilia", "Adams"));
        people.stream()
            .sorted()
            .forEachOrdered(System.out::println);
    }
}
```

Output
``` bash
Adam Johnsson
Adam Samuelsson
Ben Carlsson
Cecilia Adams
```
`Person`类实现了一些方法得以控制流的输出.其中`.hashCode()`和`.equals()`方法去鹅毛重复的人不会添加到集合Set里.`.compareTo()`方法用来控制排序.`.toString()`用来控制打印输出的展现.这几乎在每一个java项目中看到他们的身影.


# *升级代码*

与其将所有的方法写到`Person`类中,我们更愿意使用函数的引用去处理这些过程,以便使我们的代码更加整洁.我们移除掉`.equals()`, `.hashCode()`, `.compareTo ()`和 `.toString()`,取而代之的是引入两个静态变量`COMPARATOR` 和 `TO_STRING.`

Person.java
``` java
public class Person {
    private final String firstname;
    private final String lastname;
    public Person(String firstname, String lastname) {
        this.firstname = firstname;
        this.lastname  = lastname;
    }
    public String getFirstname() {
        return firstname;
    }
    public String getLastname() {
        return lastname;
    }
    public final static Comparator<Person> COMPARATOR =
        Comparator.comparing(Person::getFirstname)
            .thenComparing(Person::getLastname);
    public final static Function<Person, String> TO_STRING =
        p -> p.getFirstname() + " " + p.getLastname();
}
```
Main.java
``` java
public class Main {
    public static void main(String... args) {
        final Set
      people = new TreeSet<>(Person.COMPARATOR);
        people.add(new Person("Adam", "Johnsson"));
        people.add(new Person("Adam", "Samuelsson"));
        people.add(new Person("Ben", "Carlsson"));
        people.add(new Person("Ben", "Carlsson"));
        people.add(new Person("Cecilia", "Adams"));
        people.stream()
            .map(Person.TO_STRING)
            .forEachOrdered(System.out::println);
    }
}
```

Output
``` bash
Adam Johnsson
Adam Samuelsson
Ben Carlsson
Cecilia Adams
```

通过这种方式我们实现了原先的功能而不用去改变`Person`类,这样我们的代码更具可维护性和可重用性,更不用说书写的方便了.