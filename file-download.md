# 文件的下载

文件下载的方式:
		1.超连接下载
		2.服务器端通过流下载(服务器端编程)
		
	1.超连接下载
		download1.jsp
		<a href='${pageContext.request.contextPath}/upload/a.bmp'>a.bmp</a><br>
		<a href='${pageContext.request.contextPath}/upload/a.doc'>a.doc</a><br>
		<a href='${pageContext.request.contextPath}/upload/a.txt'>a.txt</a><br>
		<a href='${pageContext.request.contextPath}/upload/tk.mp3'>tk.mp3</a><br>

		注意:如果文件可以直接被浏览器解析，那么会在浏览器中直接打开，不能被浏览器直接解析，就是下载操作。
             直接打开的要想下载 ，右键另存为。

		超连接下载，要求下载 的资源，必须是可以直接被浏览器直接访问的。
			
		客户端访问服务器静态资源文件时，静态资源文件是通过 缺省Servlet返回的，
		在tomcat配置文件conf/web.xml 找到 --- org.apache.catalina.servlets.DefaultServlet	
		
	2.在服务器端编程完成下载.
		1.创建download2.jsp
			<a href='${pageContext.request.contextPath}/download?filename=a.bmp'>a.bmp</a><br>
			<a href='${pageContext.request.contextPath}/download?filename=a.doc'>a.doc</a><br>
			<a href='${pageContext.request.contextPath}/download?filename=a.txt'>a.txt</a><br>
			<a href='${pageContext.request.contextPath}/download?filename=tk.mp3'>tk.mp3</a><br>
			
		2.创建DownloadServlet
			// 1.得到要下载 的文件名称
			String filename = request.getParameter("filename");
			
			//2.判断文件是否存在
			File file = new File("d:/upload/" + filename);
			if (file.exists())
		
			//3.进行下载 
				原理:就是通过response获取一个输出流，将要下载的文件内容写回到浏览器端就可以了.
				
		注意:要想通过编程的方式，实现文件下载，
			1.要设置mimetype类型
				resposne.setContextType(String mimeType);
				
				问题:怎样可以得到要下载文件的mimeType类型?
					ServletContext.getMimeType(String filename);
					
				如果设置了mimeType,浏览器能解析的就直接展示了，不能解析的，直接下载.
				
			2.设置一个响应头，设置后的效果，就是无论返回的是否可以被浏览器解析，就是下载 。
				response.setHeader("content-disposition","attachment;filename=下载文件名称");
				
		总结:服务器端编程下载:
			1.将下载的文件通过resposne.getOutputStream()流写回到浏览器端。
			2.设置mimeType  response.setContentType(getServletContext.getMimeType(String filename));
			3.设置响应头，目的是永远是下载操作
				response.setHeader("content-disposition","attachment;filename=下载文件名称");
				
		--------------------------------------
		文件下载时的乱码问题:
			
			1.关于下载时中文名称资源查找不到
				原因:<a href='${pageContext.request.contextPath}/download?filename=天空.mp3'>天空.mp3</a>
				  这是get请求。
				  
				  在服务器端:
				  String filename = request.getParameter("filename");
				  
				 解决: new String(filename.getBytes("iso8859-1"),"utf-8"); 
				 
			2.下载文件显示时的中文乱码问题
					response.setHeader("content-disposition", "attachment;filename="+filename);
					
					IE:要求filename必须是utf-8码
					firefox:要求filename必须是base64编码.
					
					问题:怎样判断浏览器?
						String agent=request.getHeader("user-agent");
					
						if (agent.contains("MSIE")) {
							// IE浏览器
							filename = URLEncoder.encode(filename, "utf-8");
							
						} else if (agent.contains("Firefox")) {
							// 火狐浏览器
							BASE64Encoder base64Encoder = new BASE64Encoder();
							filename = "=?utf-8?B?"
									+ base64Encoder.encode(filename.getBytes("utf-8"))
									+ "?=";
						}else {
							// 其它浏览器
							filename = URLEncoder.encode(filename, "utf-8");
						}
