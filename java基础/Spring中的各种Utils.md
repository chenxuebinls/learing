在Spring中，有非常多Utils工具类，这些工具类有的是为了开发者使用的，有的只是提供给Spring框架使用的。  
了解这些工具类，在适当的时候使用这些工具类，对我们平时的开发还是很有帮助的，能极大方便我们的开发。

## org.springframework.core.io.support.PropertiesLoaderUtils   
这个工具类主要是针对Properties文件的加载操作，在Spring对.properties文件和.factories文件的操作都有使用到。
其实Properties对象不仅仅支持.properties文件，还支持XML格式的资源配置文件。

先来简单看看这个类提供的有用方法：

*  Properties loadProperties(Resource resource) throws IOException：从一个资源文件加载Properties；
* Properties loadProperties(EncodedResource resource) throws IOException：加载资源文件，传入的是提供了编码的资源类（EncodedResource）；和上面方法基本一致；
* void fillProperties(Properties props, Resource resource) throws IOException：从一个资源类中加载资源，并填充到指定的Properties对象中；
* void fillProperties(Properties props, EncodedResource resource) throws IOException：从一个编码资源类中加载资源，并填充到指定的Properties对象中；和上面方法基本一致；
* Properties loadAllProperties(String resourceName) throws IOException：根据资源文件名称，加载并合并classpath中的所有资源文件；
* Properties loadAllProperties(String resourceName, ClassLoader classLoader) throws IOException：从指定的ClassLoader中，根据资源文件名称，加载并合并classpath中的所有资源文件；

方法不是很多，而且共性较大，我们就从最简单的Properties loadProperties(Resource resource)开始。

**loadProperties**   
测试方法很简单，我们首先准备一个test.properties文件，放到resources下面：

```
key=value
key2=\u4E2D\u6587
```

```
@Test
public void testLoadPropertiesResource() throws Exception {
    Properties ret = PropertiesLoaderUtils
            .loadProperties(new ClassPathResource("test.properties"));
    assertEquals("value", ret.getProperty("key"));
    assertEquals("中文", ret.getProperty("key2"));
}
```
```
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>一些自定义说明</comment>
    <entry key="key">value</entry>
    <entry key="key2">中文</entry>
</properties>
```
```
@Test
public void testLoadPropertiesResourceXml() throws Exception {
    Properties ret = PropertiesLoaderUtils
            .loadProperties(new ClassPathResource("test.xml"));
    assertEquals("value", ret.getProperty("key"));
    assertEquals("中文", ret.getProperty("key2"));
}
```
参考链接：https://www.jianshu.com/p/52f8b7de1d76

## org.springframework.util.CollectionUtils
+ 实现对象转集合序列List(arrayToList)
+ 合并数组到集合中（mergeArrayIntoCollection）
+ 合并属性配置文件到Map(mergePropertiesIntoMap) 
+ 集合遍历Iterator集/实例集/是否包含对象（contains）
+ 两集合包含关系校验（containsAny）
+ 获取两集合第一个相当匹配对象（findFirstMatch）
+ 查询集合指定Class类型的对象（findValueOfType） 
+ 判断集合对象不重复校验（hasUniqueObject）等。


## CopyOnWriteArrayList

**引入 CopyOnWrite 思想解决问题**
这个时候就要引入CopyOnWrite思想来解决问题了。  
他的思想就是，不用加什么读写锁，锁统统给我去掉，有锁就有问题，有锁就有互斥，有锁就可能导致性能低下，你阻塞我的请求，导致我的请求都卡着不能执行。  
**那么他怎么保证多线程并发的安全性呢？**

很简单，顾名思义，利用“CopyOnWrite”的方式，这个英语翻译成中文，大概就是“写数据的时候利用拷贝的副本来执行”。  
你在读数据的时候，其实不加锁也没关系，大家左右都是一个读罢了，互相没影响。  
问题主要是在写的时候，写的时候你既然不能加锁了，那么就得采用一个策略。  
假如说你的ArrayList底层是一个数组来存放你的列表数据，那么这时比如你要修改这个数组里的数据，你就必须先拷贝这个数组的一个副本。  
然后你可以在这个数组的副本里写入你要修改的数据，但是在这个过程中实际上你都是在操作一个副本而已。因为是通过副本来进行更新的，万一要是多个线程都要同时更新呢？那搞出来多个副本会不会有问题？  
当然不能多个线程同时更新了，这个时候就是看上面源码里，加入了lock锁的机制，也就是同一时间只有一个线程可以更新。  
这样的话，读操作是不是可以同时正常的执行？这个写操作对读操作是没有任何的影响的吧！  
关键问题来了，那那个写线程现在把副本数组给修改完了，现在怎么才能让读线程感知到这个变化呢？  
**关键点来了，划重点！这里要配合上volatile关键字的使用。**  
volatile关键字的使用，核心就是让一个变量被写线程给修改之后，立马让其他线程可以读到这个变量引用的最近的值，这就是volatile最核心的作用。所以一旦写线程搞定了副本数组的修改之后，那么就可以用volatile写的方式，把这个副本数组赋值给volatile修饰的那个数组的引用变量了。  
只要一赋值给那个volatile修饰的变量，立马就会对读线程可见，大家都能看到最新的数组了。

下面是JDK里的 CopyOnWriteArrayList 的源码。  
大家看看写数据的时候，他是怎么拷贝一个数组副本，然后修改副本，接着通过volatile变量赋值的方式，把修改好的数组副本给更新回去，立马让其他线程可见的。

```
 // 这个数组是核心的，因为用volatile修饰了
 // 只要把最新的数组对他赋值，其他线程立马可以看到最新的数组

    private transient volatile Object[] array;
 
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            // 对数组拷贝一个副本出来
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            // 对副本数组进行修改，比如在里面加入一个元素
            newElements[len] = e;
            // 然后把副本数组赋值给volatile修饰的变量
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

那么更新的时候，会对读操作有任何的影响吗？  
绝对不会，因为读操作就是非常简单的对那个数组进行读而已，不涉及任何的锁。而且只要他更新完毕对volatile修饰的变量赋值，那么读线程立马可以看到最新修改后的数组，这是volatile保证的。

```
  private E get(Object[] a, int index) {
        // 最简单的对数组进行读取
        return (E) a[index];
    } 
 ```

这样就完美解决了我们之前说的读多写少的问题。如果用读写锁互斥的话，会导致写锁阻塞大量读操作，影响并发性能。  
但是如果用了CopyOnWriteArrayList，就是用空间换时间，更新的时候基于副本更新，避免锁，然后最后用volatile变量来赋值保证可见性，更新的时候对读线程没有任何的影响！

参考链接：https://blog.csdn.net/u013452337/java/article/details/90238052

