### MVC(Model View Controller)
- Model
	- Business Model: process business logic
	- Data Model: encapsulate the data
- View: present the data.ex.html or jsp
- controller: interaction with client

![](https://miro.medium.com/v2/resize:fit:1400/1*1jcvxPpuGuGdVY9vMdIdaw.png)
- (1.) Client 發request 給 DispatcherServlet
- (2.)(3.)DispatcherServlet調用HandlerMapping找到具體Handler（後端控制器），返回HandlerExecutionChain( Handler Object 跟 Handler Inception 攔截器）
- (4.)(5.)(6.)(7.) 調用HandlerAdapter執行Handler返回Model and view
- (8.)請求給View Resolver 將字串解析成View 物件 $\text{String viewName} \rightarrow \text{View}$ 填入對應參數
- (10.)交給前端controller($\text{JSP}$)渲染($\text{view.render()}$) 將資料填進request


https://medium.com/@hayato.chang/spring-mvc-筆記-1-簡介-d5caad598ab5

