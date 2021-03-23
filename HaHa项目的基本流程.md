##  HaHa项目的基本流程

**路由基本**

**/api/{version}/app_name/请求**

####  haha/urls

*这是haha项目下的urls,它指向save_object下面的urls,他的功能是：*

```
先是判断app的信息，这一部分后面再说。。。
如果判断他不是一个文件夹的话，就把路由地址拼接在“v1”后面
	例如：
		{
			'v1':[
				'appname1',
				'appname2'
			]
		}
```

haha/urls ---> save_objects/urls --->  save_objects/views

在save_objects/urls中加入了route.urls



+ handler_upload_file

  //此函数用来处理上传的文件,根据请求方式处理

  //如果是DELETE，则把传来的response删除

  //如果是POST，则代表是response的增加，修改，删除

  可能是处理还留在服务器的文件，把他删除。

+ handle_response

  直接处理response,如果有错直接return

  没有错误，直接返回response

**这两个方法很重要**



####  save_objects/views.py下面是众多接口功能

+ find函数是查找函数，先是得到索引、method、parmes等直接在es中查出数据
+ create函数是增加函数。通过handle_response、handler_upload_file处理残留文件
+ destroy函数是删除函数。先直接向es发送删除请求，然后通过handle_response、handler_upload_file处理残留文件





####  EsClient

他是es的实例化对象，他直接向服务器发送请求对数据进行处理。

















