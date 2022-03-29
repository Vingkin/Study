## 00x01. 接口和抽象类有什么共同点和区别

共同点：

* 都不能被实例化
* 都可以包含抽象方法
* 都可以有默认的实现方法（Java 8可以用default关键字在接口中定义默认方法）

区别：

* 接口主要用于对类的行为进行约束，实现了某个接口就有了相应的行为。抽象类主要用于代码复用，强调的是所属关系
* 一个类可以继承一个抽象类，但是能实现多个接口
* 接口中的成员变量只能是public static final类型的，不能被修改且必须有初始值，而抽象类的成员变量默认default，可在子类中被重新定义，也可被重新赋值。

## 0x02. equals和hashCode

>  equals和hashCode都是Object类中的方法。hashCode是为了获取对象的哈希码，这个哈希码的作用是确定对象在哈希表中的索引位置

举个HashSet插入对象的例子，HashSet首先会根据对象的hashcode值来判断对象加入的位置，同时也会与其他已经加入的对象的hashcode值进行比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。我们这样可以大大减少了equals的次数，相应就大大提高了执行速度。

## 0x02.1 ==与equals的区别

* 对于基本类型来说，==比较的是值是否相等
* 对于引用类型来说，==比较的是两个引用是否指向同一个对象地址
* 对于引用类型来说，equals方法如果没有被重写，equals与==作用一样，如果重写了，比如String则比较的具体内容是否相等。

## 0x03. 包装类型的常量池技术

Byte,Short,Integer,Long这四种包装类默认创建了数值[-128,127]的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean直接返回True/False

## 0x04. 自动装箱与自动拆箱

**装箱：**将基本类型用他们的引用类型包装起来，调用包装类的`valueOf()`方法

**拆箱：**将包装类型转换为基本数据类型，调用了`xxxValue()`方法

```java
Integer i = 10; // 自动装箱
int n = i; // 自动拆箱
```

## 0x05. ArrayList和LinkedList的区别

1. 是否保证线程安全：ArrayList和LinkedList都是不同步的，也就是**都不保证线程安全**
2. ArrayList底层采用的是**Object数组**，LinkedList采用的是**双向链表**（JDK 1.6之前采用的是双向循环链表，JDK 1.7取消了循环）
3. 数组和链表在插入删除方面的区别（注意两种结构都需要考虑是在中间操作还是在两端操作）
4. 数组支持随机访问，链表不支持随机访问

## 0x06. ArrayList的扩容机制

1. new的时候可以给ArrayList设置数组的长度值，也可以不设置，不设置的情况下，在第一次add时会默认赋值长度为10
2. 每次add操作都会对比add后的长度值与数组原有的长度值，判断是否要扩容
3. 如果需要扩容，默认1.5倍长度进行扩容，先会去创造一个新的长度的数组，再将原来数组赋值过去，完成扩容操作
4. 扩容的时候还是会将扩容后的数据长度与`Integer.MAX_VALUE`进行对比，以防越界
5. 工作时，我们尽可能给ArrayList一个初始长度，避免扩容操作

## 0x07. Comparable和Comparator的区别

> 两者都是用于自定义排序的接口

* Comparable接口出自java.lang，用compareTo(Object obj)方法来排序
* Comparator接口出自java.util，通过compare(Object obj1, Object obj2)来进行排序

## 0x08. 无序性和不可重复性的含义

* 无序性：无序性不等于随机性，指的是元素存储在数组底层并不是按照数组索引进行存错，而是根据数据的hashcode决定的
* 不可重复性：指的是添加元素是，按照equals进行判断时，返回false。需要同时重写equals方法和hashCode方法

## 0x09. HashSet、LinkedHashSet和TreeSet三者的异同

1. HashSet、LinkedHashSet和TreeSet都是Set接口的实现类，都能保证数据的唯一性，并且都不是线程安全的
2. 三者的主要区别在于底层数据结构不同，HashSet底层采用的时哈希表（基于HashMap实现），LinkedHashSet底层采用的是链表和哈希表，TreeSet底层采用红黑树
3. 底层数据结构的不同导致应用场景也有所不同。HashSet用于不需要保证元素插入和取出顺序的场景，LinkedHashSet用于保证元素的插入和取出满足FIFO场景，TreeSet用于支持元素的自定义排序场景。

## 0x0A. HashMap的底层实现

> JDK 1.8之前链表采用头插法，1.8采用了尾插法

JDK 1.8之前HashMap底层采用的是**数组和链表**。HashMap通过key的hashCode经过**扰动函数**后得到hash值，然后通过`(n-1)&hash`判断当前元素的存放位置（这里n指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的hash以及key是否相同，如果相同的话直接覆盖，不相同就通过拉链表解决冲突。

所谓扰动函数指的就是HashMap中的hash方法。目的是为了防止一些实现比较差的hashCode方法，换句话说使用扰动函数之后可以减少碰撞

JDK 1.8之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）（将链表转换成红黑树之前会判断，如果当前数组的长度小于64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

> (n-1)&hash的规则决定在扩容后需要对已有元素进行重新hash

## 0x0B. HashMap的扩容机制

> 针对JDK 1.8

* 容量（capacity）：hash表数组的大小，默认为16
* 初始化容量（initial capacity）：创建hash表时指定的初始容量
* 尺寸（size）：当前hash表中的元素数量
* 负载（load）：`load = size / capacity`。负载为0时，表示空的hash表。轻负载的hash表具有冲突少、适宜插入和查询的特点
* 负载因子（load factor）：决定hash表的最大填满程度（范围是0-1，默认为0.75）

当hash表的负载达到了指定的“负载因子”值时，hash表就会加倍扩容，将原有的对象**重新分配**，放入新的hash表中，这成为rehashing。rehashing过程很复杂，而且非常消耗性能，所以指定一个合适的“负载因子”值很重要。

## 0x0C. HashMap在JDK 1.7的死链问题

> [jdk1.7中 hashmap为什么会发生死链?](https://www.zhihu.com/question/394039290)
>
> 发生在多线程数组扩容的的情况下

## 0x0D. BIO，NIO，AIO

