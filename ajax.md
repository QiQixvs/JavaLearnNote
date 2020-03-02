# ajax
	问题:ajax是什么，它有什么用?
		AJAX即“Asynchronous Javascript And XML”（异步JavaScript和XML），是指一种创建交互式网页应用的网页开发技术。
		AJAX = 异步 JavaScript和XML（标准通用标记语言的子集）。
		AJAX 是一种用于创建快速动态网页的技术
		
		使用ajax目的是为了提高用户的感受。
		
	问题:异步是什么?
		查看图		
		异步操作的核心 XMLHttpRequest对象.
		
		传统web交互模型，浏览器直接将请求发送给服务器，服务器回送响应，直接发给浏览器， 
		Ajax交互模型，浏览器首先将请求 发送 Ajax引擎（以XMLHttpRequest为核心），AJax引擎再将请求发送给 服务器，服务器回送响应先发给Ajax引擎，再由引擎传给浏览器显示 

		1、同步交互模式，客户端提交请求，等待，在响应回到客户端前，客户端无法进行其他操作 
		2、异步交互模型，客户端将请求提交给Ajax引擎，客户端可以继续操作，由Ajax引擎来完成与服务武器端通信，
		    当响应回来后，Ajax引擎会更新客户页面，在客户端提交请求后，用户可以继续操作，而无需等待 。 

		Google ： suggest建议、邮件定时保存、map地图
	----------------------------------------------------------	
	ajax开发步骤:
		ajax核心就是XMLHttpRequest对象.
		
		1.得到XMLHttpRequest对象.(js对象)
			在w3school文档中的 xmldom文档中就可以查找到  dom XMLHttpRequest对象.
			var xmlhttp=null;
			if (window.XMLHttpRequest)
			 {// code for all new browsers
			  xmlhttp=new XMLHttpRequest();
			 }
			else if (window.ActiveXObject)
			 {// code for IE5 and IE6
				xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
			 }
		2.注册回调函数
			xmlhttp.onreadystatechange=function(){		
		
			};
		3.open
			只是用于设置请求方式 以及url,它不发送请求.
			
		4.send
			它是用于发送请求的。
			send(null);null代表没有参数  如果有参数可以写成:"username=tom&password=123"
			
			
		5.在回调函数中处理数据
			
			1.XMLHttpRequest对象有一个属性  readyState
				它代表的是XMLHttpRequest对象的状态。
				
				0.代表XMLHttpRequest对象创建
				1.open操作
				2.send操作
				3.接收到了响应数据，但是只有响应头，正文还没有接收。
				4.所有http响应接收完成。		

			2.status
				由服务器返回的 HTTP 状态代码，如 200 表示成功

			3.在回调函数中可以通过以下方式获取服务器返回的数据
				1.responseText
				2.responseXML
				
	--------------------------------------------------------------------------
	关于ajax操作中请求参数的设置问题:
		
		1.对于get请求方式，参数设置
			直接在url后面拼接
			例如:"${pageContext.request.contextPath}/ajax2?name=tom"
			
		2.对于post请求方式，参数设置
			
			xmlhttp.open("POST","${pageContext.request.contextPath}/ajax2");
			xmlhttp.send("name=tom");
			
			注意:如果是post请求方式，还需要设置一个http请求头。
			xmlhttp.setRequestHeader("","");
			
			例如:
			xmlhttp.open("POST","${pageContext.request.contextPath}/ajax2");	
			xmlhttp.setRequestHeader("content-type","application/x-www-form-urlencoded");	
			xmlhttp.send("name=tom");
			
	------------------------------------------------------------------------
	ajax案例1--验证用户名是否可以使用
	
	-------------------------------------------------------------------------
	ajax案例2--显示商品信息
		第一个版本:
			1.创建一个Product类
				private int id;
				private String name;
				private double price;
				
			2.创建ajax4.jsp
				<a href="javascript:void(0)" id="p">显示商品信息</a>
				<div id="d"></div>
				
				在回调函数中得到服务器返回的信息innerHTML到div中.
				
			3.在Ajax4Servlet中
				将List<Product>中的数据，手动拼接成了html代码，写回到浏览器端.
			
				builder.append("<table border='1'><tr><td>商品编号</td><td>商品名称</td><td>商品价格</td></tr>");
				for (Product p : ps) {
					builder.append("<tr><td>" + p.getId() + "</td><td>" + p.getName()
							+ "</td><td>" + p.getPrice() + "</td></tr>");
				}
				builder.append("</table>");
			
		------------------------
		第二个版本
			创建一个product.jsp页面，在页面上去组装table,直接将数据返回了.
			
			步骤
				1.在Ajax4Servlet中
					request.setAttribute("ps", ps);
					request.getRequestDispatcher("/product.jsp").forward(request, response);
				2.在product.jsp页面上
					<table border='1'>
						<tr>
							<td>商品编号</td>
							<td>商品名称</td>
							<td>商品价格</td>
						</tr>
						<c:forEach items="${ps}" var="p">
							<tr>
								<td>${p.id }</td>
								<td>${p.name }</td>
								<td>${p.price }</td>
							</tr>
						</c:forEach>
					</table>
		------------------------------------------
		第三个版本	
			在服务器端得到数据，只将要显示的内容返回，而不返回html代码
			而html代码的拼接，在浏览器端完成。
			
			问题:服务器返回什么样的数据格式?
				json:它是一种轻量级的数据交换格式。
				
				[{'id':'1','name':'洗衣机','price':'1800'},{'id':'2','name':'电视机','price':'3800'}]
				在js中{name:value,name1:valu1}这就是一个js对象.
				[{},{}]这代表有两个对象装入到了一个数组中。
				
	-----------------------------------------------------------------------
	关于json插件使用:
		在java中，可以通过jsonlib插件，在java对象与json之间做转换。
		
		关于jsonlib插件使用:
			1.导包(6个包)
			
			2.将java对象转换成json
				
				1.对于数组，List集合，要想转换成json
					JSONArray.fromObject(java对象); ["value1","value2"]
				
				2.对于javaBean，Map
					JSONObject.fromObject(javaBean对象); {name1:value1,name2:value2}
					
			对于json数据，它只有两种格式
				1.[值1,值2,...]  ------>这就是javascript中的数组
				2.{name:value,....} ---->就是javascript中的对象。
				但是这两种格式可以嵌套.
				[{},{},{}]
				{name:[],name:[]}
				
			3.如果javaBean中有一个属性，不想生成在json中，怎样处理?
				JsonConfig config = new JsonConfig();
				config.setExcludes(new String[] { "type" });
				JSONArray.fromObject(ps, config).toString();
				上述代码就是在生成json时，不将type属性包含.
				
	-------------------------------------------------------------------------------------------------
	ajax操作中服务器端返回xml处理
		XMHttpRequest.resposneXML;----->得到的是一个Document对象.
		
		操作：可以自己将xml文件中的内容取出来，写回到浏览器端。也可以请求转发到一个xml文件，将这个文件信息写回到
		      浏览器端.注意   response.setContextType("text/xml;charset=utf-8");
			  
		问题:如果没有xml文件，我们的数据是从数据库中查找到了，想要将其以xml格式返回怎样处理?
			
			可以使用xml插件处理  xstream，它可以在java对象与xml之间做转换.
			
		xstream使用:
			1.导包
				2个.
			2.使用
				1.将java对象转换成xml
					XStream xs=new XStream();
					String xml=xs.toXML(java对象);
				问题:生成的xml中的名称是类的全名.
					两种方式:
						1.编码实现
							xs.alias("person", Person.class);
						2.使用注解(Annotation)
							@XStreamAlias(别名) 对类和变量设置别名
							@XStreamAsAttribute  设置变量生成属性
							@XStreamOmitField  设置变量 不生成到XML
							@XStreamImplicit(itemFieldName = “hobbies”) 设置集合类型变量 别名

							使注解生效 
							xStream.autodetectAnnotations(true);
							
	--------------------------------------------------------------------------
	作业:
		1.通过返回xml来完成省市联动.
		2.通过返回json来完成省市联动。
		