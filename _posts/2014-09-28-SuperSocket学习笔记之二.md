---
layout: post
title: "SuperSocket学习记录之二-TCP应用程序实现"
description: ""
category: 工作生涯
tags: ""
---

之前简单介绍了一些`SuperSocket`的入门的方法，整体来说不是很规范，没有按刚开始摸索时候的流程来说明，因为摸索的过程自己都有些遗忘，只能凭着记忆回顾一些知识点进行说明。开发人员在看寻求新的知识的过程中，有两种不同的方式，一种是把原理都先弄懂，对于SS来说，就是把SS的源码大致都进行阅读，了解整个实现的机制，然后再去实现；另一种就是直接找到demo，照着demo依葫芦画瓢，边实现边了解。我想，大部分人采用第二种方式去做，一来是这样更快做出东西来，有成就感；二来是项目紧，被逼着尽快出成果，没办法，只能改demo。嗯，是的，我也是通过第二种方式去学习和使用SS的。接下来还是对项目中开发的TCP应用的实现流程进行说明，原理的东西涉及的比较少，只是记录自己实现的方式。  

###TCP应用程序需求
`TCP服务器`主要的功能就是接收客户端发送上来的数据，对数据进行解析，然后将数据存储入库同时还要将信息发送到前台。

###实现流程  

####SS和Winform开发  

SS的作者相当牛，在源码中已经附带了一个基于GPS的服务器，用于接收GPS终端的数据，我们在开发时可以基于这个例子进行修改，能够省去很多的工作。在这个DEMO之上，我们需要做的就是将服务的启动和Winform程序结合起来，这需要通过调用SS的启动方式来实现，这里通过BootStrap启动方式来完成ss的启动，在Winform中点击按钮后调用BootStrap的方法实现SS的启动。具体的代码如下  


    public InitServer()
        {
            bootstrap = BootstrapFactory.CreateBootstrap();
            if (!bootstrap.Initialize())
            {
                Console.WriteLine("Failed to setup!" );
                //Console.ReadKey();
            }
        }

        public StartResult StartServer()
        {
            StartResult result = bootstrap.Start();
            return result;
        }

####定义接收过滤器ReceiveFilter
接收过滤器对请求数据进行过滤，如果不是指定的请求信息，将无法进入程序处理。本接收过滤器是跟终端请求直接打交道的一个接口，相当于一个门，只用符合规定的数据才能进入这个门，然后进行其他操作。因此这个门需要定义一个规定，这个规定在这里的表示就是协议。SS中实现了多个常用协议的模板，我们可以通过过滤器类继承这些自定义的协议模板，然后在过滤器类的构造方法中调用这个协议方法，就实现了我们系统中需要定义的协议标准。  

    class TerminalReceiveFilter : BeginEndMarkReceiveFilter<BinaryRequestInfo >
    {
        private readonly static byte[] BeginMark = new byte[] {0x7e};
        private readonly static byte[] EndMark = new byte[] {0x7e};
        public TerminalReceiveFilter()
            : base(BeginMark, EndMark)
        {
        }
    }


定义好了标准之后，我们需要在过滤器类中实现RequestInfo 实例的构建，就是将收到的二进制流定义成一个RequestInfo 实例，才能进行下一步的处理。本系统中我们需要将请求数据定义成一个`BinaryRequestInfo`实例，通过如下方法实现  

       protected override BinaryRequestInfo ProcessMatchedRequest(byte[] readBuffer, int offset, int length)
        {
            return new BinaryRequestInfo( BitConverter.ToString(readBuffer, offset + 1, 2), readBuffer.CloneRange(offset+1, length-2));
        }

####Command实现
在获得RequestInfo 实例后，根据该实例的不同key值，以该key值命名的command类会被执行。在本应用中，由于key值都是以数字的形式存在，不能将数据定义为command的类型，因此需要将在对类实现时，需要对command类的Name属性进行如下转换  

    public override string Name { get { return "01"; } } 

这样，key为01时，会对应执行上面的Command类。

每个command类其实对应这一种请求，这种实现方式能够将所有的请求隔离的非常清楚，对请求的处理都包含在一个独立的类中实现，对系统的实现和后期的修改维护都有非常大的帮助。
在本系统中，我们可以在command类中，对不同的请求的业务处理进行实现了，入库。

####第一阶段完成
至此，本应用中第一阶段的工作就完成了，目前为止，本应用可以接收由我们自己定义的数据信息，同时我们也可以对数据进行分析和数据库存储操作了。
但是应用还远没有完成，在本系统中，我们需要将终端发送过来的信息经过处理后发送到前台，实现数据的实时显示，前台在这里其实就是指浏览器，由winform程序发送数据到浏览器，还是有点麻烦的。之前的很多程序，在实现数据实时显示时采用的多数都是Ajax轮询方式，通过无刷新请求，模拟实时的效果。在本系统中，其实也可以这么实现，但是后来在查询资料的过程中碰到了一个好东西：WebSocket，接下来，又要开始继续完善系统了。

###WebSocket实现
WebSocket是HTML5中的一种新的协议，更多的资料可以看这里[
使用 HTML5 WebSocket 构建实时 Web 应用
](http://www.ibm.com/developerworks/cn/web/1112_huangxa_websocket/)，这可是一个非常好玩有用的技术，可以通过这种技术实现后台应用和前端页面的实时通信，而且还是基于TCP方式。在这里不多介绍这个东西，我们只需要知道websocket在实现前后端通信时需要做两部分工作，第一部分，实现一个加入的websocket协议的tcp服务器，相当于是在普通的TCP服务器上加入一个websocket的协议处理操作；第二部分，实现前台的接口调用，websocket提供了js的调用接口，很简单。
本系统中，我们在实现时，也是使用了现成的框架，基于SuperSocket的WebSocket服务器框架SuperWebSocket，其实就是在SS的基础上实现了WebSocket协议，网上有源码，我在使用时只需要直接调用SuperWebSocket.dll就可以了。服务器端完成之后，需要完成的是客户端，最开始，本系统直接使用的是js实现的API接口，直接在页面调用接收进行数据操作就能完成了。之后又发现有一个基于C#实现的WebSocket客户端WebSocket4Net，这个客户端实现将js进行了封装，使用起来更加灵活，这里是对WebSocket4Net的解释说明[WebSocket4Net](http://websocket4net.codeplex.com/)，我们在实现时需要调用的是WebSocket4Net.dll。至此，有了这两个dll文件，我们的WebSocket通信机制就可以完成了。
###新的问题（应用服务器通信）
但是，在具体系统实现时，我们还是遇到了问题，在上述的说明中，我们实现了两个TCP服务器，一个是对终端的TCP服务器，另一个是对前端的WebSocket服务器，在我们的应用中，需要实现的是TCP服务器处理后将信息发送到浏览器，因此需要TCP服务器首先将数据发送到WebSocket服务器，然后再由WebSocket服务器将数据发送到浏览器。因此，这两个服务器的通信成为我们要解决的问题。
###服务器通信实现
SuperSocket框架的好处是可以支持多个服务器实例的同时运行，有了这个，我们可以考虑将WebSocket服务器整合到TCP服务器中，在系统启动时同时启动这两个服务器，因为这两个服务器都是基于SuperSocket框架实现，因此整合起来很容易，只需要在Bootstrap启动时多调用一个服务就可以实现，由于是通过Bootstrap方式实现，因此只需要在配置文件的`servers`节点中添加WebSocket服务器的信息即可

        <server name="TerminalServer" serverTypeName="TerminalSocketService" ip="Any" port="40000" maxConnectionNumber="10000">
        </server>
        <server name="PageServer" serverTypeName="PageServerService" ip="Any" port="2012">
        </server>


整合完服务器之后，需要做的就是服务器的通信了。由于两个服务器在同一进程中执行，因此交互数据变得不是那么难（当然，前提是基于SuperSocket框架)。我们可以通过如下的代码实现数据的转发  

    interface IDespatchServer
    {
    void DispatchMessage(string sessionKey, string message);
    }

    public class MyAppServerB : AppServer, IDespatchServer
    {
    public void DispatchMessage(string sessionKey, string message)
    {
        var session = GetAppSessionByID(sessionKey);
        if (session == null)
            return;

        session.Send(message);
    }
    }

    public class MyAppServerA : AppServer
    {
    private IDespatchServer m_DespatchServer;

    protected override void OnStartup()
    {
        m_DespatchServer = this.Bootstrap.GetServerByName("ServerB") as IDespatchServer;
        base.OnStartup();
    }

    internal void DespatchMessage(string targetSessionKey, string message)
    {
        m_DespatchServer.DispatchMessage(targetSessionKey, message);
    }
    }



通过设置配置文件，在服务开启是同时开启ServerA和ServerB，A，B客户端分别连接到ServerA和ServerB。
首先定义转发服务接口类IDespatchServer，接口类中定义转发方法DispatchMessage()；A终端发送信息到达ServerA，ServerA中通过Bootstrap获取ServerB，并赋值到接口类，ServerA中实现DespatchMessage()方法，该方法内调用了ServerB中实现的具体发送到B客户端的方法DespatchMessage()，ServerA解析A发送过来的信息后到具体的Command中执行，Command中调用ServerA的DespatchMessage()方法，从而将A的信息发送到B。

通过上述代码，可以实现将数据的转发，完成数据由终端到后端再到前端的传送过程，至此，此应用程序要完成的功能基本上达到了。

###总结

本系统这样的实现其实还不是很完善，为了实现系统的性能和易用性，还需要做解决以下两个问题：  

* 目前来说，所有的业务流程都是在Command中直接实现，如果数据量大的话可能会成为系统的瓶颈，因此在这里建议是加入消息队列来处理，目前了解到微软的消息队列实现起来是比较方便的，后期用到时需要做一些研究；  
* 就易用性来说，Winform程序应该要能显示一些终端设备的连接信息和接收数据信息，目前还未能从SS的command类中将数据传递到winform界面，这是一个系统优化要解决的问题。

本文把应用SuperSocket实现一个TCP接收服务的应用程序的主要流程进行了说明，其实很多工作都是用SS来完成的，我只是进行了一些对框架的学习和实现的过程，不能不说SS确实是一个非常好用的东西，值得好好去琢磨。
这个系统中只用了部分功能，还有好些功能没有用上，估计在应用真正广泛使用时会需要更多的实现，目前也只能先放下了。







