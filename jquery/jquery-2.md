# jQuery-1

### description: Ajax

## jQuery-2

### ajax

在做调试代码的时候我们一般使用alert，但是alert 它只能打印字符串，或int 类型的数据，不能打印对象里面比较详细的信息， 浏览器给我们提供了一个对象叫 console.info 可以打印对象里面更加详细的信息..

#### ajax方法的调用

```java
jQuery(function(){
    $("#ajaxbutton").click(function () {
        var options={
            url:"../../ajaxServlet",
            type:"POST",

            //data:"username=卢雨",//提交数据的第一个格式..  字符串
            //请求数据的第二种格式... 一般我们使用第二种
            data:{
                username:"高亚"
            },
            success:function(data){
                alert(data);   
            },
        //    dateType:"",//服务端预期的数据返回类型...
            timeout:"3000",//请求超时的时间
            error:function(){//请求失败时的一个回调函数..//可以给用户号的提示...
            }
        };
        $.ajax(options)
    })
})
```

#### POST方法的调用

```java
jQuery(function(){
    $("#postbutton").bind("click",function(){
    //一般如果我们有请求参数，就使用post 方法..
        $.post("../../postServlet",{},function(data){//Servlet访问路径，参数，回调函数(返回的参数)
            //alert(data);    
            //<xml><province><city id="1">邯郸</city><city>邯郸</city></province></xml>
            alert($(data).find('city').text());
        })    
    })
})
```

#### load方法的调用

```java
jQuery(function(){
    $("#loadbutton").bind("click",function(){
        //$.load()
        /**
        * 1:如果有参数的时候，则使用post 方式提交，
        * 2：如果没有请求参数，则默认使用get 方式提交..
        * load 方法它一般都是用于去载入一个静态的页面..
        */

        $("div").load("ajax.html"); 
        //$("div").load("../../loadServlet",{username:"高压"},function(){});        
    })

})
```

#### get方法的调用

```java
jQuery(function(){
    $("#getbutton").bind("click",function(){
        $.get("../../getServlet",{},function(data){
            alert(data);

        //    {total:118,rows:[{id:1,title:'左娜'},{id:2,title:'梅斌'},{id:3,title:'酸梅汤'}]}
        //传统的解析方法，用substring

        //json 的字符串它是符合一个javascript 的对象.

            var obj=eval("("+data+")");
            var rows=obj.rows;
            for(var i=0;i<rows.length;i++){
                alert(rows[i].id);
                alert(rows[i].title);
            }
        })
    })

})
```

#### getJSON

getJSON放问服务端，服务端返回的数据格式必须是json 的数据格式

json数据格式\[{"name":"haha","desc":"xixi"},{"name":"shshs",desc":"hahahah"}\]

```java
$(function(){

    $("#getJSON").click(function(){

        $.getJSON("data.json",function(data){
                //alert(data);
                //alert(data);
                //jQuery 在解析json 的时候要求key 上面需要有双引号...
                var length=data.length;
                alert(length);
        });

    })

})
```

#### getScript

异步去加载服务端的一段脚本文件

```java
$(function(){

    $("#getScript").click(function(){
        $.getScript("../../js/test.js")
    })
})
```

### 异步提交form表单

```java
$(function(){
    $("#formbutton").click(function(){//这里不用submit提交
        //发送异步的ajax 请求        
        //将表单里面的选项序列化成一个字符串...     

        //第一种方式...
        //var data=$("#form1").serialize();


        //序列化成一个数组...
        var data=$("#form1").serializeArray();

        $.ajax({
            url:"../../formServlet",


//                         data:{
//                             username:$("#username").val(),
//                            password:$("#password").val(),
//                            email:$("#email").val(),
//                         },

        //通过ajax 方法提交的时候有两种数据格式，一种字符串
        // data:data,

            //一个是json

            data:data,
            type:"POST",
            success:function(data){

            }
        })
    })
})
```

#### 滚动异步加载

```java
//js中
//window.onscroll=function(){}

//jQuery

$(window).scroll(function(){

        var t = document.documentElement.scrollTop;

        if(t>0  && t<800){          
            loadImage("1");
        }

        if(t>800  && t<1600){      
            loadImage("2");
        }

        function loadImage(imageType){
            $.ajax({
                url:"../../imageServlet",
                type:"POST",
                data:{
                    imageType:imageType
                },
                success:function(data){
                    var area="#area_"+imageType;
                    var image="<img src='../../"+data+"'>";
                    $(area).html(image);
                }
            })

        }
})
```

### 插件

jQuery对外提供了一些接口，可以扩展jQuery定义自己的功能

#### 全局方法的插件

```java
<script type="text/javascript">
        $.extend({
            Constants:{
                baseURL:"http://localhost"
            },
            validateTelephone:function (tel) {
                alert(tel);
            }
        })
</script>

<script type="text/javascript">
        //使用自己定义的功能
        $.validateTelephone("123234");
</script>
```

#### 局部方法的插件

```java
 <script type="text/javascript">

 //局部方法插件扩展的固定写法
    $fn.extend({
        datagrid:function (obj) {

        }
    })
</script>
 <script type="text/javascript">
    $(function(){
        $("#datagrid").datagrid({

        });
    })
</script>
```

#### 选择器插件

#### jplayer

