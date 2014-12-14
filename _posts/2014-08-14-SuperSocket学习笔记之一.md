---
layout: post
title: "SuperSocket学习笔记之一"
description: ""
category: 工作生涯
tags: ""
---
###开篇，为什么选了SuperSocket？
目前的项目开发中都需要用到TCP协议的SOCKET编程，建立服务器端接收网络数据，这个是我之前用接触的比较少的点，前一个项目被这个SOCKET编程折腾了好久，最后是通过修改网络的程序才基本解决这个问题，之所以说是基本解决，是因为该服务器在使用过程中还是出现一些不稳定的现象。目前的项目对于TCP服务器要求更高，因此自己在也考虑换一种方式实现服务器。刚开始研究时是想采用IOCP模型进行开发，但是试用了一下之后，一方面由于自己技术能力有限，另一方面是自己试用了一下发现针对自己的应用还是有一些问题。再请教其他公司的同学时，他提到了SuperSocket这个框架，之后开始尝试学习之，最初进去时感觉学习难度挺大，想过放弃，停了一段时间之后，还是决定继续扎进去学习如何使用。因此也就有了这篇学习笔记。  
  
####什么是SuperSocket？  

这个网上可以百度到，在我看来就是这个由江大渔（牛人）开发的开源Socket框架，并且一直在升级优化同时提供使用中出现问题的答疑解惑的良心项目。使用这个框架进行socket编程，你可以不用去理解socket如何通信的细节（当然理解了就最好了，也应该去理解一下），根据文档就可以建立起自己的通信服务器，同时可以根据自己的需求进行拓展。如果有心，还可以研究源码去了解SS的实现机制，源码的风格也是非常优秀的，值得好好研究。总之，对于这个框架，就一个字“赞”。很多原理的东西我也说不是很透彻，可以参看这里[http://www.supersocket.net/](http://www.supersocket.net/)了解更详细的信息。 当然，使用这个框架时，必须要有一定的C#知识储备。

####SS使用记录
原理的东西认识的没有很透彻，我就根据自己学习使用过程中碰到的问题，学习到的知识做一个记录。  
先稍微讲一下我认识到的数据流动轨迹：  
1，客户端与服务器建立连接  
2，服务器监听并将每个连接封装成session对象  
3，通过接收过滤器（ReceiveFilter）对数据流进行格式化，将客户端请求实例化为请求实例，即RequestInfo类型，形式为key和body，parameters的格式  
4，RequestInfo中不同的key被对应的command执行。  

#####文档和代码例子的学习
这两项是学习SS必备的也是唯一的途径，边看文档边调例子，例子都调通了，基本的了解也就到位了。
######例子一：Basic系列
要使用SS框架，必须引入<b>SuperSocket.Common.dll, SuperSocket.SocketBase.dll, SuperSocket.SocketEngine.dll,log4net.dll</b>到项目中，在对应的源码文件夹下的bin/Debug目录下可以找到这些dll文件。这个例子可以直接启动，可以根据自己的要求修改一下端口试试，启动好服务器端，就可以通过客户端测试了，哪里来的客户端？Telnet，难以理解，有些人连这个也不知道，直接win键，搜索中输入telnet，就能找到telnet.exe了，运行即可。输入命令 <b>open localhost 2012</b>  就可以连上服务器了，这时就可以看例子的代码，边玩边调试。

之后还涉及到command的添加，SS中对数据的操作都由command来完成，不同的动作执行不同的command类，添加command类很简单。所有的Command都需要实现ICommand接口，接口中的Name属性用于匹配requestInfo中的key，根据key执行对应的command操作。有些命令无法直接当做类名，如“01”，需要在Command类中做如下操作进行转换：  
<code>  
    public override string Name  
    {  
        get { return "01"; }  
    }  
</code>  
这里要说一下StringRequestInfo类，这是经过ReceiveFilter处理后的数据格式，包含了（key,body,parameters),key就对应command，body是发送过来的消息体，parameters是经过分隔符(默认的分隔符是空格)分割后的数组。
key,body和parameters的分割可以自己定义，比如用‘；’分隔key和body，用‘,’分隔body中的参数，只需要在你的Server类的构造函数继承如下代码即可：  
<code>  
	public YourServer()  
        : base(new CommandLineReceiveFilterFactory(Encoding.Default, new BasicRequestInfoParser(":", ",")))  
    {  
    }
</code>

######例子二：Medium系列  
这里的例子涉及到了配置方式启动服务器的知识，跟上一个不同的是，这里的项目不能直接启动执行，需要先生成，然后都生成项目的Debug目录下找到SuperSocket.SocketService.exe文件执行程序结果。服务器的IP，端口信息全部在.config文件下进行设置。

这里涉及到了命令过滤器CommandFilter和连接过滤器ConnectionFilter的使用：  
CommandFilter用于命令过滤，有两个方法可以在命令执行前和执行时做一些操作。
ConnectionFilter用于过滤远程的连接信息，允许哪些连接进入。

基本上我用到的就是这些信息，有些没有用到的信息我就没有去理解了，有些内容在刚开始学的时候觉得是个新人难理解的点，现在由于有了一些认识，一下子就忘了是哪个点了，只能把自己能想起来的做个记录了。



####未完待续
对于以前学习的内容页只能回顾到这里了，还是有挺多觉得应该可以记录的东西却记录不了了，应该当时就将学习的疑问记录下来，然后将解答也记录下来的，回顾的时候就能想得比较全了。其实这些内容很多都是文档中提到的，但是对于刚接触SS的人来说，看文档还是不够细心，有些点没有注意到。下一篇还是根据自己现在正在修改使用的程序进行说明吧，可能整个流程会比较清晰一点。

