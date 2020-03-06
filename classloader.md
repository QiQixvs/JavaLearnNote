# 类加载器

## 1. 概念

类加载器的作用就是将java中的字节码文件(.class文件)转换成java.lang.Class对象。

当 JVM 启动时，会形成由三个类加载器组成的初始类加载器层次结构：

1. 引导类加载器 **BootStrap classloader**    jre/lib/rt.jar
2. 扩展类加载器  **ExtClassLoader**   JRE/lib/ext/*.jar
3. 应用类加载器(系统类加载器) **AppClassLoader SystemClassLoader**   CLASSPATH指定的所有jar或目录

在java中 ClassLoader 代表类加载器，所有的类加载器都是 ClassLoader 的子类.

## 2. 演示类加载器

问题:类加载器如果获取?

在Class类中有一个方法 getClassLoader()它返回的就是一个类加载器.

### 2.1 获取引导类加载器

```java
ClassLoader cl = String.class.getClassLoader();
System.out.println(cl);
```

结果是null.

原因: 引导类加载器特殊，它不是java实现的，负责加载Java的核心类, 所以在得到引导类加载器时结果就是null.

### 2.2 扩展类加载器

```java
ClassLoader cl = AccessBridge.class.getClassLoader();
System.out.println(cl); //sun.misc.Launcher$ExtClassLoader@9cb0f4
```

### 2.3 应用类加载器

```java
ClassLoader cl = this.getClass().getClassLoader();
System.out.println(cl); //sun.misc.Launcher$AppClassLoader@164dbd5
```

## 3. 全盘负责委托机制

* **全盘负责**：即是当一个classloader加载一个Class的时候，这个Class所依赖的和引用的其它Class通常也由这个classloader负责载入。

* **委托机制**：先让parent（父）类加载器 寻找，只有在parent找不到的时候才从自己的类路径中去寻找。

* 类加载还采用了cache机制：如果 cache中保存了这个Class就直接返回它，如果没有才从文件中读取和转换成Class，并存入cache，这就是为什么修改了Class但是必须重新启动JVM才能生效,并且类只加载一次的原因。

## 4. 自定义类加载器

问题：创建了一个类  javax.activation.MimeType，在这个类中有一个方法show();
当jvm加载这个类时，因为在rt.jar包下也存在一个MimeType类，并且包名都一样，这时jvm就会使用引导类加载器加载这个类，而我们想得到的其实是应该由应用类加载器加载的Class。

解决方案:

自定义类加载器

1. 创建一个类，去继承自ClassLoader
2. 重写findClass方法，在这个方法中通过defineClass将一个.class文件转换成Class对象

```java
public class MyClassLoader extends ClassLoader {
    private String rootDir;//当前工程的classes目录

    public MyClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    @Override
    public Class<?> findClass(String name) throws ClassNotFoundException {

        String extname = name.replace(".", "\\");//包名.类名变成包名\\类名
        String filename = rootDir + "\\" + extname + ".class";//得到需要加载的类的绝对磁盘路径

        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try {
            InputStream is = new FileInputStream(filename);
            int len = -1;
            byte[] b = new byte[1024];

            while ((len = is.read(b)) != -1) {
                baos.write(b, 0, len);
            }
            baos.flush();
            baos.close();
            is.close();

            byte[] data = baos.toByteArray();//class文件中的内容

            return defineClass(name, data, 0, data.length);//想要加载的Class

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    return null;
}
```

使用自定义类加载器

```java
String rootDir = ClassLoaderTest.class.getResource("/").getPath();

MyClassLoader mcl = new MyClassLoader(rootDir);

Class clazz = mcl.findClass("javax.activation.MimeType");

clazz.getDeclaredMethod("show").invoke(clazz.newInstance());
```
