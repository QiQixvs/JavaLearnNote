# ajax练习

实现省市级联（xm,json）

## 通过返回xml来完成省市联动

* AjaxServlet返回xml数据

```java
response.setContentType("text/xml;charset=utf-8");//中文乱码问题
XStream xs = new XStream();
xs.autodetectAnnotations(true);//使注解生效
String xml = xs.toXML(ps);

response.getWriter().write(xml);
response.getWriter().close();
```

![responseXML](../.gitbook/assets/2020-03-04-11-15-42.png)

* 填充省份名称JS代码实现
* window.onload =function){}
* xml = xmlhttp.responseXML;
* 使用select元素的add(option)方法

```java
var xml;
window.onload = function() {
  var xmlhttp = getXmlHttpRequest();
  xmlhttp.onreadystatechange = function() {

    if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
        //得到xml是一个Document对象，它代表的是服务器端返回的xml.
        xml = xmlhttp.responseXML;
        //将xml中所有province的name子元素中的文本获取到.
        var ps = xml.getElementsByTagName("province");

        for ( var i = 0; i < ps.length; i++) {
          //得到每一个province元素下的第一个name子元素
          var nameElement = ps[i].getElementsByTagName ("name")[0];
          //得到name元素中的文本子节点中的信息,就是省份名称
          var pname = nameElement.firstChild.nodeValue;
          //创建option标签，将其添加到省份下拉框中.
          var option = document.createElement("option");
          option.text = pname;
          //将创建的option添加到省份下拉框中
          document.getElementById("province").add(option);
        }
    }
  };
  xmlhttp.open("GET", "${pageContext.request.contextPath}/ajax1");
  xmlhttp.send(null);
};
```

* 填充城市名称JS代码实现
* 获得已选的select province.options[province.selectedIndex]
* 添加option元素，可指定索引 city.add()

```java
function fillCity() {   //select标签onchange事件

  var province=document.getElementById("province");//省份下拉框
  var city=document.getElementById("city");//城市下拉框.

  //1.得到选中的省份名称.
  var pname=province.options[province.selectedIndex].text;

  var ps = xml.getElementsByTagName("province");

  for ( var i = 0; i < ps.length; i++) {
    //得到每一个province元素下的第一个name子元素中的文本信息,就是省份名称
    var name = ps[i].getElementsByTagName("name")[0].firstChild.nodeValue;
    if(pname == Name){
      //判断选择的省份名称与xml文件中的省份名称一致.
      var citys=ps[i].getElementsByTagName("city");
      //得到所有省份的城市
      for(var j=0;j<citys.length;j++){
        //得到所有的城市名称
        var cname=citys[j].getElementsByTagName("name")[0].firstChild.nodeValue;
        //创建option添加到城市下拉框中
        var option = document.createElement("option");
        option.text = cname;
        city.add(option，j+1);
    }
  }  
}
```

## 通过返回json来完成省市联动

如果服务器返回的是json数据，我们在浏览器端接收数据 eval()转换。 有些情况下，转换会出问题。 可使用

```java
var json=eval("("+xmlhttp.responseText+")");
```

* AjaxServlet返回json数据

```java
// 转换成json
String json = JSONArray.fromObject(ps).toString();

response.getWriter().write(json);
response.getWriter().close();
```

* 填充城市名称JS代码实现

```java
var jsonObj = eval("(" + xmlhttp.responseText + ")");

//得到省份名称
for ( var i = 0; i < jsonObj.length; i++) {
  var pname = jsonObj[i].name;

  var option = document.createElement("option");
  option.text = pname;

  province.add(option);
}

//得到已选选项
var pname = province.options[province.selectedIndex].text;
//填充城市
for ( var i = 0; i < jsonObj.length; i++) {
  if (jsonObj[i].name == pElementName) {
    var citys = jsonObj[i].citys;

    for ( var j = 0; j < citys.length; j++) {

      var cname = citys[j].name;

      var option = document.createElement("option");
      option.text = cname;

      city.add(option);

    }
  }
}
```

