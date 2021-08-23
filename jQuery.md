jQuery：
1. jQuery是js库，
   库：相当于java的工具类，库是存放东西的， jQuery是存放js代码的地方， 放的用js代码写的function

2. dom对象和jquery对象
   dom对象，使用javascript的语法创建的对象叫做dom对象， 也就是js对象。
	var obj=  document.getElementById("txt1");  obj是dom对象，也叫做js对象
	obj.value;

   jquery对象： 使用jquery语法表示对象叫做jquery对象， 注意：jquery表示的对象都是数组。
        例如  var jobj =  $("#txt1") , jobj就是使用jquery语法表示的对象。 也就是jquery对象。 它是一个数组。
	                               现在数组中就一个值。

   dom对象可以和jquery对象相互的转换。
     dom对象可以转为jquery ， 语法： $(dom对象)
     jquery对象也可以转为dom对象， 语法： 从数组中获取第一个对象， 第一个对象就是dom对象， 使用[0]或者get{0).

   为什么要进行dom和jquery的转换：目的是要使用对象的方法，或者方法。
     当你的dom对象时，可以使用dom对象的属性或者方法， 如果你要想使用jquery提供的函数，必须是jquery对象才可以。

 3.选择器： 就是一个字符串， 用来定位dom对象的。定位了dom对象，就可以通过jquery的函数操作dom
   常用的选择器：
    1） id选择器， 语法： $("#dom对象的id值")
        通过dom对象的id定位dom对象的。 通过id找对象， id在当前页面中是唯一值。

    2） class选择器， 语法： $(".class样式名)
        class表示css中的样式， 使用样式的名称定位dom对象的。


    3） 标签选择器， 语法： $("标签名称")
        使用标签名称定位dom对象的


4. 表单选择器，
   使用<input>标签的type属性值，定位dom对象的方式。
	语法： $(":type属性值")
	例如： $(":text") ,选择的是所有的单行文本框，
	       $(":button") ,选择的是所有的按钮。

5.过滤器：在定位了dom对象后，根据一些条件筛选dom对象。
  过滤器有是一个字符串，用来筛选dom对象的。
  过滤器不能单独使用， 必须和选择器一起使用。

  1）$("选择器:first") : 第一个dom对象
  2）$("选择器:last"): 数组中的最后一个dom对象
  3）$("选择器:eq(数组的下标)") ：获取指定下标的dom对象
  4）$("选择器:lt(下标)") ： 获取小于下标的所有dom对象
  5）$("选择器:gt(下标)") ： 获取大于下标的所有dom对象

6.表单属性过滤器： 根据表单中dom对象的状态情况，定位dom对象的。
   启用状态， enabled , 
	不可用状态  disabled
	选择状态 ， checked ， 例如radio， checkbox 


7.each语法
  1). 可以对 数组， json ，dom数组循环处理。 数组， json中的每个成员都会调用一次处理函数。
     var arr = { 1, 2, 3} //数组
	  var json = {"name":"lisi","age":20 } 
     var obj = $(":text");

     语法： $.each( 循环的内容， 处理函数 ) ：表示使用jquery的each，循环数组，每个数组成员，
             都会执行后面的“处理函数”一次。
    
     $: 相当于是java的一个类名
      each:就是类中的静态方法。
      静态方法调用，可以使用 类名.方法名称 
    
      处理函数:function(index, emelent) :
         index, element都是自定义的形参， 名称自定义。
    	  index：循环的索引
    	  element：数组中的成员
    
     js循环数组：
      for(var i=0;i<arr.length;i++){
         var item = arr[i]; //数组成员
    		//操作数组成员
    		shuchu( i , item);
      }
    
     function shuchu(index, emlemnt){
        输出index ，element
      }

  2).循环jquery对象， jquery对象就是dom数组
    jquery对象.each( function(index,element) {} )


8. jquery中给dom对象绑定事件
    1） $(选择器).事件名称( 事件的处理函数)
      $(选择器)：定位dom对象， dom对象可以有多个， 这些dom对象都绑定事件了
	   事件名称：就是js中事件去掉on的部分， 例如 js中的单击事件 onclick(),
	             jquery中的事件名称，就是click，都是小写的。
      事件的处理函数：就是一个function ，当事件发生时，执行这个函数的内容。

		例如给id是btn的按钮绑定单击事件
		$("#btn").click(funtion(){
     	 alert("btn按钮单击了")
		})

  2) on 事件绑定
     $(选择器).on( 事件名称 , 事件的处理函数)

	  事件名称： 就是js事件中去掉on的部分， 例如js中onclick ,这里就是click
	  事件的处理函数： function 定义。
	
	  例如， <input type="button" id="btn">
	   $("#btn").on("click", function() { 处理按钮单击 } )

 9. 使用jquery的函数，实现ajax请求的处理。
    没有jquery之前，使用XMLHttpRequest做ajax ， 有4个步骤。  jquery简化了ajax请求的处理。
	 使用三个函数可以实现ajax请求的处理。

	 1） $.ajax() : jquery中实现ajax的核心函数。
	 2） $.post() : 使用post方式做ajax请求。
	 3） $.get() : 使用get方式发送ajax请求。

	 $.post()和$.get() 他们在内部都是调用的 $.ajax() 


	 介绍 $.ajax函数的使用， 函数的参数表示请求的url， 请求的方式，参数值等信息。
	 $.ajax()参数是一个json的结构。
	
	  例如： $.ajax(  {名称:值， 名称1:值1..... } )
	
	         例如： $.ajax(  {  async:true , 
				                   contentType:"application/json" , 
				                   data: {name:"lisi",age:20 },
										 dataType:"json",
										 error:function(){
	                              请求出现错误时，执行的函数
										 },
										 success:function( data ) {
	                             // data 就是responseText, 是jquery处理后的数据。
	
										 },
										 url:"bmiAjax",
										 type:"get"
									  }  
										 
								  )
	
	 json结构的参数说明：
	
	  1)async:是一个boolean类型的值， 默认是true ，表示异步请求的。可以不写async这个配置项
	        xmlHttp.open(get,url,true),第三个参数一样的意思。
	  2)contentType: 一个字符串，表示从浏览器发送服务器的参数的类型。 可以不写。
	                例如你想表示请求的参数是json格式的， 可以写application/json
	 3)data: 可以是字符串，数组，json，表示请求的参数和参数值。 常用的是json格式的数据
	  4)dataType: 表示期望从服务器端返回的数据格式，可选的有： xml ， html ，text ，json
	      当我们使用$.ajax()发送请求时， 会把dataType的值发送给服务器， 那我们的servlet能够
			读取到dataType的值，就知道你的浏览器需要的是 json或者xml的数据，那么服务器就可以
			返回你需要的数据格式。
	  5)error: 一个function ，表示当请求发生错误时，执行的函数。
	    error:function() {   发生错误时执行  }  
	
	 6)sucess:一个function , 请求成功了，从服务器端返回了数据，会执行success指定函数
	          之前使用XMLHttpRequest对象， 当readyState==4 && status==200的时候。
	
	  7)url:请求的地址
	  8)type:请求方式，get或者post， 不用区分大小写。 默认是get方式。

  主要使用的是 url , data ,dataType, success .