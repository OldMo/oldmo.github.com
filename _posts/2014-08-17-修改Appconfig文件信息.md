---
layout: post
title: "C#修改App.config配置信息"
description: ""
category: 工作生涯
tags: ""
---

###实现目的
开发Winform应用程序时，为了配置方便，需要设置一些数据库连接信息和服务器ip信息，因此需要读取和修改app.config文件。很多操作这个文件的方法都是基于ConfigurationManger类，这个类对于操作appSettings和connectionStrings是比较简单的，但是如果碰上了自定义的一些节点，就会发现不够用了，需要用到xml的解析，本来config文件也就是xml文件的结构，因此用xml解析也是比较容易的。


下面的代码信息是App.config中需要读取和修改信息，分别通过两种方式去实现，对connectionStrings 节点的操作用ConfigurationManager类实现，对superSocket节点的操作用xml方式实现。
###App.config文件信息

     <connectionStrings>
         < add name ="PublicSecurityPatrolDbEntities " connectionString=" metadata=res://*/PSPIDbModel.csdl|res://*/PSPIDbModel.ssdl|res://*/PSPIDbModel.msl;provider=System.Data.SqlClient;provider connection string= &quot;data source=192.168.1.118,1433;initial catalog=CarMonitor;persist security info=True;user id=sa;password=123456;multipleactiveresultsets=True;application name=EntityFramework&quot;" providerName=" System.Data.EntityClient" />
     </connectionStrings>

    <superSocket>
        < servers>
     
          < server name ="GPSServer " serverTypeName ="GPSSocketService " ip ="Any " port ="50000 " maxConnectionNumber=" 100">
          </ server>
          < server name ="SuperWebSocket " serverTypeName ="PageServerService " ip ="Any " port ="2012 ">
          </ server>
    
        </ servers>

    < serverTypes>
      < add name ="GPSSocketService " type ="CarMonitor.GPSServer,CarMonitor "/>
      < add name ="PageServerService " type ="CarMonitor.PageServer,CarMonitor "/>
    </ serverTypes>
  </superSocket>   


###XML方式读取和更新自定义节点信息
     /// <summary>
        /// 获取配置服务器配置信息并显示到前台
        /// </summary>
        public void getConfigValue()
        {
            XmlDocument xDoc = new XmlDocument();
            try
            {
                string path = System.Windows.Forms.Application .ExecutablePath + ".config";
                string ss = "GPSServer" ;
                string sws = "SuperWebSocket" ;
                xDoc.Load(path);
                XmlNode supersocketNode, serverNode;
                XmlElement ssElem = null , swsElem = null;
                supersocketNode = xDoc.SelectSingleNode( "//superSocket");
                serverNode = supersocketNode.SelectSingleNode( "servers");

                foreach (XmlNode item in serverNode.ChildNodes)
                {
                    ssElem = ( XmlElement)item.SelectSingleNode("//server[@name='" + ss + "']");
                    swsElem = ( XmlElement)item.SelectSingleNode("//server[@name='" + sws + "']");
                }
                if (ssElem != null )
                {
                    ssip.Text = ssElem.GetAttribute( "ip");
                    ssport.Text = ssElem.GetAttribute( "port");
                    swsip.Text = swsElem.GetAttribute( "ip");
                    swsport.Text = swsElem.GetAttribute( "port");
                }
            }
            catch
            {
                throw;
            }
        }

        /// <summary>
        /// 更新前端服务器和后端服务器信息
        /// </summary>
        /// <param name="path"></param>
        /// <param name="ss"> 后端服务器名 </param>
        /// <param name="sws"> 前端服务器名 </param>
        /// <param name="ssIP"> 后端ip</param>
        /// <param name="ssPort"> 后端端口</param>
        /// <param name="swsIP"> 前端ip</param>
        /// <param name="swsPort"> 前端端口</param>
        public void updateConfigValue(string path, string ss, string sws, string ssIP, string ssPort, string swsIP, string swsPort)
        {
            XmlDocument xDoc = new XmlDocument();
            try
            {
                xDoc.Load(path);
                XmlNode supersocketNode, serverNode;
                XmlElement ssElem = null , swsElem = null;
                supersocketNode = xDoc.SelectSingleNode( "//superSocket");
                serverNode = supersocketNode.SelectSingleNode( "servers");

                foreach (XmlNode item in serverNode.ChildNodes)
                {
                    ssElem = ( XmlElement)item.SelectSingleNode("//server[@name='" + ss + "']");
                    swsElem = ( XmlElement)item.SelectSingleNode("//server[@name='" + sws + "']");
                }
                if (ssElem != null )
                {
                    ssElem.SetAttribute( "ip", ssIP);
                    ssElem.SetAttribute( "port", ssPort);
                    swsElem.SetAttribute( "ip", swsIP);
                    swsElem.SetAttribute( "port", swsPort);
                    xDoc.Save(path);
                }
            }
            catch
            {
                throw;
            }
        }

###ConfigurationManager方式实现读取和更新connectionString节点信息
        ///<summary>
        ///读取连接字符串 
        ///</summary>
        private string getConnectionStrings()
        {
            string conStr = ConfigurationManager .ConnectionStrings["PublicSecurityPatrolDbEntities"].ConnectionString.ToString();
            int serverPosStart = conStr.IndexOf("source=" );
            int serverPosEnd = conStr.IndexOf(",1433" );
            string serverStr = conStr.Substring(serverPosStart+7, serverPosEnd-serverPosStart-7);
            int dbnamePosStart = conStr.IndexOf("catalog=" );
            int dbnamePosEnd = conStr.IndexOf(";persist" );
            string dbName = conStr.Substring(dbnamePosStart+8,dbnamePosEnd-dbnamePosStart-8);
            int idPosStart = conStr.IndexOf("id=" );
            int idPosEnd = conStr.IndexOf(";password=" );
            string dbUserName = conStr.Substring(idPosStart+3,idPosEnd-idPosStart-3);
            int pwPosEnd = conStr.IndexOf(";mul" );
            string dbPassWord = conStr.Substring(idPosEnd+10,pwPosEnd-idPosEnd-10);
           
            return serverStr+":" +dbName+":"+dbUserName+ ":"+dbPassWord;
        }

        ///<summary>
        ///更新连接字符串 
        ///</summary>
        ///<param name="name"> 连接字符串名称 </param>
        ///<param name="newConString"> 连接字符串内容 </param>
        ///<param name="newProviderName"> 数据提供程序名称 </param>
        private static void updateConnectionStringsConfig( string name, string newConString, string newProviderName)
        {
            bool isModified = false ;    //记录该连接串是否已经存在,存在的话在更新前需要先删除 
            if (ConfigurationManager .ConnectionStrings[name] != null)
            {
                isModified = true;
            }
            ConnectionStringSettings mySettings = new ConnectionStringSettings(name, newConString, newProviderName);
            Configuration config = ConfigurationManager .OpenExeConfiguration(ConfigurationUserLevel.None);
            if (isModified)
            {
                config.ConnectionStrings.ConnectionStrings.Remove(name);
            }
            config.ConnectionStrings.ConnectionStrings.Add(mySettings);
            config.Save( ConfigurationSaveMode.Modified);                // 保存对配置文件所作的更改 
            ConfigurationManager.RefreshSection("ConnectionStrings" );   // 强制重新载入配置文件的ConnectionStrings配置节
        }

###主函数
    public static void Main(String[] args)
        {

            string file = System.Windows.Forms.Application .ExecutablePath+".config";
            string ss = "GPSServer" ;
            string sws = "SuperWebSocket" ;
            updateConfigValue(path, ss, sws, ssIP, ssPort, swsIP, swsPort);

         string conName = "PublicSecurityPatrolDbEntities" ;
            string conStr = "metadata=res://*/PSPIDbModel.csdl|res://*/PSPIDbModel.ssdl|res://*/PSPIDbModel.msl;provider=System.Data.SqlClient;provider connection string=&quot;data source=" + dbip + ",1433;initial catalog=" + dbname + ";persist security info=True;user id=" + userid + ";password=" + pw + ";multipleactiveresultsets=True;application name=EntityFramework&quot;" ;
            string provider = "System.Data.EntityClient" ;
            updateConnectionStringsConfig(conName, conStr, provider);
        }
