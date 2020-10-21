---
title: "Java集合"
date: 2020-10-19T16:45:28+08:00
draft: false
tags: [ "Java" ]
categories: [ "技术文档" ]
---

# Java集合

## 概述

**对象的容器，实现了对对象常用的操作**。

**集合与数组的区别**

- 数组长度固定，集合长度不固定。
- 数组可以存储基本类型和引用类型，集合只能存储引用类型。

**包**

`java.util.*`

## Collection接口

### Collection体系

![img](/img/Java集合001.png)

### Collection父接口

特点：代表一组任意类型的对象，无序、无下标、元素不可重复。

创建集合：`Collection<Object> collection = new ArrayList<>();`

### 常用方法

**添加元素**
- `collection.add();`

**删除元素**
- `collection.remove();`

**清空集合**
- `collection.clear();`

**判断元素**
- `collection.isEmpty();`
- `collection.contains();`

**遍历元素**
- 无下标所以不能使用普通for循环，只能使用增强for循环：`for(Object object : collection){ }`
- 使用迭代器
    - 判断下个元素是否存在：`it.hasNext();`
    - 获取下个元素：`it.next();`
    - 删除当前元素：`it.remove();`
    - **遍历迭代器时不允许使用`collection.remove();`删除元素，会发生并发修改异常**。
        ```java
        Iterator it = collection.iterator();
        while(it.hasNext()) {
            Object object = it.next();
            // 遍历迭代器时不允许使用`collection.remove();`删除元素，会发生并发修改异常
            // it.remove();
        }
        ```

## List接口

特点：继承自Collection接口，有序、有下标、元素可重复。

创建集合：`List list = new ArrayList();`

### 常用方法

**添加元素**
- `list.add();`
- 会对基本数据类型进行自动封装。

**获取元素**
- `list.get();`

**删除元素**
- `list.remove();`
- 可以使用索引或删除对象，若删除对象也为数字与索引矛盾时，需要对对象进行封装`list.remove(new Integer(0));`。

**清空集合**
- `list.clear();`

**判断元素**
- `list.isEmpty();`
- `list.contains();`

**获取元素索引**
- `list.indexOf();`

**获取元素数量**
- `list.size();`

**遍历元素**
- 普通for循环
    ```java
    for(int i = 0; i < lise.size(); i++) {
        list.get(i); 
    }
    ```
- 增强for循环：`for(Object object : collection){ }`
- 使用迭代器
    - 判断下个元素是否存在：`it.hasNext();`
    - 获取下个元素：`it.next();`
    - 删除当前元素：`it.remove();`
        - **遍历迭代器时不允许使用`collection.remove();`删除元素，会发生并发修改异常**。
            ```java
            Iterator it = lise.iterator();
            while(it.hasNext()) {
                Object object = it.next();
                // 遍历迭代器时不允许使用`lise.remove();`删除元素，会发生并发修改异常
                // it.remove();
            }
            ```
- 使用列表迭代器
    - **功能比普通迭代器强大，不仅能从前往后遍历，还可以从后往前遍历**。
        ```java
        ListIterator lit = lise.listIterator();

        // 顺序遍历
        while(lit.hasNext()) {
            int index = li.nextIndex();
            Object object = lit.next();
        }

        // 倒序遍历
        while(lit.hasPrevious()) {
            int index = li.previousIndex();
            Object object = lit.previous();
        }
        ```

**返回集合(x, y)左闭右开的子集合**：如(1, 3)返回索引为(1、2)的子集合。
- `list.sublist(x, y);`

### List实现类

**ArrayList**
- 数组结构实现，必须要连续空间，查询快、增删慢。
- jdk1.2版本，运行效率快、线程不安全。

**Vector**
- 数组结构实现，查询快、增删慢。
- jdk1.0版本，运行效率慢、线程安全。

**LinkedList**
- 双向链表结构实现，无需连续空间，增删快，查询慢。

## ArrayList类

特点：数组结构实现，必须要连续空间，查询快、增删慢；运行效率快、线程不安全。

创建集合：`ArrayList arrayList = new ArrayList();`

### 源码分析

**存放元素的数组**：`Object[] elementData;`

**元素计数器**：`int size`

**扩容**
- 默认容量：`DEFAULT_CAPACITY = 10;`
- 没有向集合中添加任何元素时，容量为0；添加一个元素后，容量扩容为10。
- 当放入的元素超过集合容量时，扩容到原来的1.5倍：`newCapacity = oldCapacity + (oldCapacity >> 1)`

## Vector类

特点：数组结构实现，必须要连续空间，查询快、增删慢；运行效率慢、线程安全。

创建集合：`Vector vector = new Vector();`

### 常用方法

**遍历元素**
- 使用枚举器遍历
    ```java
    Enumeration en = vector.elements();
    while(en.hasMoreElements()){
        en.nextElement();
    }
    ```

## LinkedList类

特点：双向链表结构实现，无需连续空间，增删快，查询慢。

创建集合：`LinkedList li = new LinkedList();`

常用方法同ArrayList一致。

### 源码分析

**节点**
- 元素：`E item;`
- 后一节点：`Node<E> next;`
- 前一节点：`Node<E> prev;`

**属性**
- 元素个数：`int size;`
- 头节点：`Node<E> first;`
- 尾节点：`Node<E> last;`

## 泛型

- JDK1.5引入，本质是参数化类型，把类型作为参数传递。
- 常见形式有泛型类、泛型接口、泛型方法。
- 语法：`<T, ...>` T称为类型占位符，表示一种引用类型，可以写多个逗号隔开。
- 好处：提高代码重用性；防止类型转换异常，提高代码安全性。

### 泛型类

语法：`类名<T>`

**创建泛型类**
```java
public class MyClass<T> {

    // 创建变量
    T t;

    // 泛型作为方法的参数或者返回值
    public T test(T t) {

        return t;
    }
}
```

**使用泛型类**
- 泛型只能使用引用类型。
- 不用泛型类型对象之间不能相互赋值。
```java
public class TestGeneric {

    public static void main(String[] args) {

        MyClass<String> myClass = new MyClass<>();
        myClass.t = "hello";
        String string = myClass.test("hello world!");
    }
}
```

### 泛型接口

语法：`接口名<T>`

**创建泛型接口**
- 不能使用泛型创建静态常量，不能直接创建对象。
```java
public interface MyInterface<T> {

    T test (T t);
}
```

**实现泛型接口**
- 确定参数类型
    ```java
    public class MyImpl implements MyInterface<String> {

        @Override
        public String test (String t) {
            
            return t;
        }

        public static void main(String[] args) {
            
            MyImpl myImpl = new MyImpl();
            String string = myImpl.test("hello world!");
        }
    }
    ```
- 不确定参数类型
    ```java
    public class MyImpl<T> implements MyInterface<T> {

        @Override
        public T test (T t) {
            
            return t;
        }

        public static void main(String[] args) {
            
            MyImpl<String> myImpl = new MyImpl<>();
            String string = myImpl.test("hello world!");
        }
    }
    ```

### 泛型方法

语法：`<T> 返回值类型 方法名`

**创建泛型方法**
```java
public class MyClass {

    // 泛型作为方法的参数或者返回值
    public <T> T test(T t) {

        return t;
    }
        
    public static void main(String[] args) {

        MyClass myClass = new MyClass();
        String string = myClass.test("hello world!");
    }
}
```

### 泛型集合

概念：参数化类型、类型安全的集合，强制集合元素的类型必须一致。

语法：`List<String> list = new ArrayList<>();`

特点：
- 编译时即可检查，而非运行时抛出异常。
- 访问时，不必类型转换（拆箱）。
- 不同泛型之间应用不能相互赋值，泛型不存在多态。

## Set接口

特点：继承自Collection接口，无序、无下标、元素不可重复。

创建集合：`Set set = new HashSet();`

### 常用方法

全部继承自Collection接口中的方法，与collection一致。

## HashSet类

概念：
- 基于HashCode实现元素不重复。
- 当存入元素的HashCode相同时，会调用equals进行确认，如结果为true，则拒绝后者存入。

存储结构：哈希表(数组 + 链表 + 红黑树(jdk1.8))

存储过程(重复依据)：
- 根据hashCode计算保存在数组中的位置，如果位置为空，直接保存，若不为空，执行第二步。
- 再执行equals方法，如果equals为true，则认为是重复，否则形成链表。

特点：
- 基于HashCode计算元素存放位置。
    - 利用31这个质数，减少散列冲突。
        - `31 * i`提高执行效率`31 * i = (i << 5) - i`转为移位操作。
    - 当存入元素的哈希码相同时，会调用equals进行确认，如果结果为true，则拒绝后者存入。

### 重写重复规则

重写对象的`hashCode()`方法与`equals()`方法，实现对对象值的比较，默认是比较对象地址。

- 重写`hashCode()`方法
    ```java
    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + id;
        result = prime * result + ((name == null) ? 0 : name.hashCode);
        return result;
    }
    ```

- 重写`equals()`方法
    ```java
    @Override
    public boolean equals(Objedt obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() !== obj.getClass())
            return false;
        Test other = (Test) obj;
        if (id != other.id)
            return false;
        if (name == null) {
            if (other.name != null)
                return false;
        } else if (!name.equals(other.name))
            return false;
        return true;
    }
    ```

## TreeSet类

概念：
- 基于排列顺序实现元素不重复。

存储结构：红黑树

特点:
- 基于排列顺序实现元素不重复。
- 实现SortedSet接口，对集合元素自动排序。
- 元素对象的类型必须实现Comparable接口，指定排序规则。
- 通过`compareTo()`方法确定是否为重复元素。

### 实现排序规则

TreeSet需要对对象进行排序，所以必须指定排序规则。

- 对象实现Comparable接口`compareTo()`方法。
    ```java
    public class Test implements Comparable<Test> {

        @Override
        public int compareTo(Test t) {

            int n1 = this.id = t.id;
            int n2 = this.getName().compareTo(t.name);
            return n1 == 0 ? n2 : n1;
        }
    }
    ```
- 创建TreeSet时传入Comparator比较器。
    ```java
    TreeSet<Test> treeSet = new TreeSet<>(new Comparator<Test>() {
        @Override
        public int compare(Test o1, Test o2) {

            int n1 = this.id = t.id;
            int n2 = this.getName().compareTo(t.name);
            return n1 == 0 ? n2 : n1;
        }
    });
    ```

## Map接口

特点：
- 用于存储任意键值对(key - value)。
- 键：无序、无下标、不允许重复(唯一)。
- 值：无序、无下标、允许重复。

创建集合：`Map<String, Object> map = new HashMap();`

### 常用方法

**添加元素**
- `map.put("key", value);`

**获取元素**
- `map.get("key");`

**删除元素**
- `map.remove("key");`

**清空集合**
- `map.clear();`

**判断元素**
- `map.isEmpty();`
- `map.containsKey();`
- `map.containsValue();`

**获取元素数量**
- `map.size();`

**遍历元素**
- 使用keySet()，获取所有key的Set集合
    ```java 
    Set<String> keyset = map.keySet();
    for(String key : map.keyset){
        key;
        map.get(key);
    }
    ```
- 使用entrySet()，获取所有Entry的Set集合
    ```java
    Set<Entry<String, Object>> entries = map.entrySet();
    for(Entry<String, Object> entry : map.entries){
        entry.getKey();
        entry.getValue();
    }
    ```

## HashMap类

特点：JDK1.2引入，线程不安全，运行效率快；允许使用null作为key或者value。

存储结构：哈希表(数组 + 链表 + 红黑树(jdk1.8))

创建集合：`HashMap<String, Object> map = new HashMap();`

### 重写重复规则

重写Key对象的`hashCode()`方法与`equals()`方法，实现对对象值的比较，默认是比较对象地址。

### 源码分析

**常量**
- 默认初始容量：`int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16`
- 默认加载因子：`float DEFAULT_LOAD_FACTOR = 0.75f;`
- 树化/树退化阈值：`int TREEIFY_THRESHOLD = 8;` `int UNTREEIFY_THRESHOLD = 6;`
- 最小树形化容量阈值：`int MIN_TREEIFY_CAPACITY = 64;`
    - 当哈希表中的容量 > 该值时，才允许树形化链表。

**变量**
- 哈希表数组：`Node<K, V>[] table;`
- 元素个数：`int size;`

**扩容**
- 没有向集合中添加任何元素时，容量为0；添加一个元素后，容量扩容为16。
- 当放入的元素超过集合容量*加载因子时，扩容到原来的2倍：`newCap = oldCap << 1`。

**新增**
- JDK1.8之前使用头插法。
- JDK1.8之后使用尾插法。

> HashSet底层实际是使用HashMap实现的，只使用了Key存储数据。

## Hashtable类

特点：JDK1.0引入，线程安全，运行效率慢；不允许使用null作为key或者value。

### Properties类

特点：Hashtable的子类，要求key和value都是String。通常用于配置文件的读取。可保存在流中或从流中加载。

## TreeMap类

特点：实现了SortedMap接口(Map子接口)，可以对key自动排序。

创建集合：`TreeMap<String, Object> map = new TreeMap();`

### 实现排序规则

TreeMap需要对对象进行排序，所以必须指定排序规则。

- 对象实现Comparable接口`compareTo()`方法。
- 创建TreeMap时传入Comparator比较器。

> TreeSet底层实际是使用TreeMap实现的，只使用了Key存储数据。

## Collections工具类

概念：集合工具类，定义了除了存取以外的集合常用方法。

**二分查找**
- `int i = Collections.binarySearch(list, x);`成功返回索引。

**排序**
- `Collections.sort(list);`

**复制**
- `Collections.copy(newList, oldList);`
    - 需两个数组长度相等。

**反转**
- `Collections.reverse(list);`

**打乱**
- `Collections.shuffle(list);`

**扩展**
- 集合转数组
    - `Integer[] arr = list.toArray(new Integer[]{});`
- 数组转集合
    - `List<Integer> list = Arrays.asList(arr);`
    - 把基本类型数组转为集合时，需要使用包装类包装类。