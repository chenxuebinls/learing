# Java中System.getenv()和System.getProperties()的区别
原文链接：https://blog.csdn.net/DSLZTX/java/article/details/50188251

一 、System.getenv()   
返回系统环境变量值，示例如下：
```
LOCALAPPDATA=C:\Users\User_2\AppData\Local,
PROCESSOR_LEVEL=6,
FP_NO_HOST_CHECK=NO,
USERDOMAIN=ULIC-14,
LOGONSERVER=\\ULIC-14,
JAVA_HOME=D:\Tools\Java\jdk1.8.0_202,
...
```

常见的有以下几种途径可以设置系统环境变量：   
1.1、登录用户主目录下的".bashrc"文件   
在当前登录用户主目录下的".bashrc"文件中可以设置系统环境变量。   
1.2、在Shell中设置   
在Shell中依次执行如下命令，设置好"dslztx"这个环境变量：   

```
dslztx="This is a newbie."
export dslztx
```

现在有如下Java代码：
```
public class Main {
    public static void main(String[] args) {
        System.out.println(System.getenv("dslztx"));
    }
}
```
在相同Shell中依次执行"javac Main.java;java Main"可以得到"This is a newbie."的打印结果。  
1.3、在IDE中设置  
假如使用"Intellij Idea"，那么可以在"Run-->Edit Configurations"窗口的"Environment variables"选项中设置环境变量。  


二、System.getProperties()
返回Java进程变量值，示例如下：
```
{java.runtime.name=Java(TM) SE Runtime Environment,  os.name=Linux, sun.jnu.encoding=UTF-8, java.library.path=/home/dsl/programs/jdk1.6.0_45/jre/lib/i386/client:/home/dsl/programs/jdk1.6.0_45/jre/lib/i386:/home/dsl/programs/jdk1.6.0_45/jre/../lib/i386:/home/dsl/programs/jdk1.6.0_45/jre/lib/i386/server:/home/dsl/programs/jdk1.6.0_45/jre/lib/i386:/home/dsl/programs/jdk1.6.0_45/jre/../lib/i386:/mnt/bigdisk/programs/idea-IU-141.1532.4/bin::/usr/java/packages/lib/i386:/lib:/usr/lib, java.specification.name=Java Platform API Specification, java.class.version=50.0, sun.management.compiler=HotSpot Client Compiler, os.version=3.16.0-30-generic, user.home=/home/dsl, user.timezone=, java.awt.printerjob=sun.print.PSPrinterJob, idea.launcher.bin.path=/mnt/bigdisk/programs/idea-IU-141.1532.4/bin, file.encoding=UTF-8}
```
常见的有以下几种途径可以设置Java进程变量：  
2.1、通过命令行参数的"-D"选项  
现在有如下代码：  
```
public class Main {
    public static void main(String[] args) {
        System.out.println(System.getProperty("dslztx"));
    }
}
```
依次执行如下命令：  
```
javac Main.java
java -Ddslztx="This is a newbie!" Main
```
可以得到"This is a newbie!"的打印结果。  
2.2、进程运行中动态设置  
代码例子如下：  
```
public class Main {
    public static void main(String[] args) {
        System.setProperty("dslztx", "This is a newbie!");
        System.out.println(System.getProperty("dslztx"));
    }
}
```
运行可以得到"This is a newbie!"的打印结果。  
2.3、在IDE中设置  
假如使用"Intellij Idea"，那么可以在"Run-->Edit Configurations"窗口的"VM options"选项中设置Java进程变量。  



原文链接：https://blog.csdn.net/DSLZTX/java/article/details/50188251
