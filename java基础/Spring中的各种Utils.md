在Spring中，有非常多Utils工具类，这些工具类有的是为了开发者使用的，有的只是提供给Spring框架使用的。  
了解这些工具类，在适当的时候使用这些工具类，对我们平时的开发还是很有帮助的，能极大方便我们的开发。

### org.springframework.core.io.support.PropertiesLoaderUtils   
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
