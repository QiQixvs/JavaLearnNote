# Internationalization

## 概念

**i18n**：internationalization

对于程序中固定使用的文本元素，例如菜单栏、导航条等中使用的文本元素、或错误提示信息，状态信息等，需要根据来访者的地区和国家，选择不同语言的文本为之服务。

对于程序动态产生的数据，例如(日期，货币等)，软件应能根据当前所在的国家或地区的文化习惯进行显示。

## 配置文件

针对于不同的国家与地区要显示的信息，都配置到配置文件中，
根据当前访问者的国家或语言来从不同的配置文件中获取信息，
展示在页面上。

### 相关的概念

对于软件中的菜单栏、导航条、错误提示信息，状态信息等这些固定不变的文本信息，可以把它们写在一个properties文件中，
并根据不同的国家编写不同的properties文件。这一组properties文件称之为一个资源包。

* ResourceBundler，它是用于从资源包中获取数据的。

* 关于资源文件(properties)命名: 基名_语言_国家.properties

```markdown
message_zh_CN.properties
message_en_US.properteis
```

## 编码演示

properties文件操作以及通过ResourceBundler来获取资源包中信息.

1.资源包文件一般都放置在classpath下

2.关于ResourceBundle使用
创建:
ResourceBundle bundle = ResourceBundle.getBundle("message");		
ResourceBundle bundle = ResourceBundle.getBundle("message",Locale.US);
获取:
bundle.getString(String name);

扩展:关于properties文件中中文问题处理?
在jdk中有一个命令native2ascii.exe。

1.进行一次翻译
native2ascii 回车
中文  回车

2.批量翻译
native2ascii  源文件路径   目录文件路径
例如: native2ascii d:/a.txt  d:/a.properties

国际化的登录页面
1.创建登录页面
2.创建配置文件
3.在登录页面上根据不同的国家获取ResourceBundle
4.在页面上需要国际化的位置，通过ResourceBundle.getString()来获取信息.

问题:在页面上使用了jsp脚本.解决方案:使用标签 。
在jstl标签库中提供了国际化标签.


关于日期国际化
DateFormat类.
作用:
1.可以将一个Date对象格式化成指定效果的String     format方法
2.可以将一个String解析成Date对象     parse方法

1.DateFormat对象创建
DateFormat df1 = DateFormat.getDateInstance(); // 只有年月日
DateFormat df2 = DateFormat.getTimeInstance(); // 只有小时分钟秒
DateFormat df3 = DateFormat.getDateTimeInstance();// 两个都有

DateFormat df1 = DateFormat.getDateInstance(DateFormat.FULL); // 只有年月日
DateFormat df2 = DateFormat.getTimeInstance(DateFormat.MEDIUM); // 只有小时分钟秒
DateFormat df3 = DateFormat.getDateTimeInstance(DateFormat.LONG,DateFormat.SHORT);// 两个都有

DateFormat df1 = DateFormat.getDateInstance(DateFormat.FULL,Locale.US); // 只有年月日
DateFormat df2 = DateFormat.getTimeInstance(DateFormat.MEDIUM,Locale.US); // 只有小时分钟秒
DateFormat df3 = DateFormat.getDateTimeInstance(DateFormat.LONG,DateFormat.SHORT,Locale.US);// 两个都有

关于货币国际化
NumberFormat类
1.对数值进行格式化
NumberFormat nf = NumberFormat.getIntegerInstance();
2.对数值进行百分比
NumberFormat nf = NumberFormat.getPercentInstance(Locale.FRANCE);
3.对数值进行以货币显示
NumberFormat nf = NumberFormat.getCurrencyInstance(Locale.US);


MessageFormat
动态文件格式化.

MessageForamt可以对一个模板中的信息进行动态赋值.

1.MessageFormat使用
MessageForamt.format(String pattern,Object... params);

2.说明一下关于动态文本中的占位符?
例如:{0} is required 

1.注意占位符只能使用{0}---{9}之间的数值.
2.关于占们符的格式
{argumentIndex}: 0-9 之间的数字，表示要格式化对象数据在参数数组中的索引号
{argumentIndex,formatType}: 参数的格式化类型
{argumentIndex,formatType,FormatStyle}: 格式化的样式，它的值必须是与格式化类型相匹配的合法模式、或表示合法模式的字符串。

formatType可以取的值有:number date time
formatStyle可以取的值有
number类型可以取:integer currency  percent 
date类型可以取的:short medium  full long
time类型可以取的:short medium  full long