### Eclipse 配置Solr 4.x 源码开发

###### 1. 下载

在[solr](https://lucene.apache.org/solr/)官方网站下载solr 4.x src源码`solr-4.6.1-src.tgz `，下载后解压 到本地目录。

###### 2. 解压，构建

由于Solr是基于[Ant](http://ant.apache.org/)和[Ivy](https://ant.apache.org/ivy/)做管理的，所以需要安装有Ant和Ivy，这里不做介绍。解压后，在解压后的根目录中执行ant elipse。

Ant 构建开始
![solr-src-build-start](https://raw.github.com/kiahhan/KeepThing/master/content_solr-src-build/solr-src-build-start.png)

Ant 构建成功
![solr-src-build-successful](https://raw.github.com/kiahhan/KeepThing/master/content_solr-src-build/solr-src-build-successful.png)

###### 3. 导入工程

打开eclipse，通过File > Import > Existing Projects into Workspace导入到Eclipse中。

![eclipse-package-explorer-4-solr-src.png](https://raw.github.com/kiahhan/KeepThing/master/content_solr-src-build/eclipse-package-explorer-4-solr-src.png)

###### 4. 运行代码

在Eclipse中，使用Open Type(Ctrl+Shift+T)找到StartSolrJetty 这个类，修改main方法里面的setPort参数为默认的8983，以及ContextPath,War

```
War为”solr/webapp/web/”
```


修改后的代码应该是
```java
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.apache.solr.client.solrj;

import org.eclipse.jetty.server.Connector;
import org.eclipse.jetty.server.Server;
import org.eclipse.jetty.server.bio.SocketConnector;
import org.eclipse.jetty.webapp.WebAppContext;

/**
 * @since solr 1.3
 */
public class StartSolrJetty 
{
  public static void main( String[] args ) 
  {
    //System.setProperty("solr.solr.home", "../../../example/solr");

    Server server = new Server();
    SocketConnector connector = new SocketConnector();
    // Set some timeout options to make debugging easier.
    connector.setMaxIdleTime(1000 * 60 * 60);
    connector.setSoLingerTime(-1);
    connector.setPort(8983);
    server.setConnectors(new Connector[] { connector });
    
    WebAppContext bb = new WebAppContext();
    bb.setServer(server);
    bb.setContextPath("/solr");
    bb.setWar("solr/webapp/web");

//    // START JMX SERVER
//    if( true ) {
//      MBeanServer mBeanServer = ManagementFactory.getPlatformMBeanServer();
//      MBeanContainer mBeanContainer = new MBeanContainer(mBeanServer);
//      server.getContainer().addEventListener(mBeanContainer);
//      mBeanContainer.start();
//    }
    
    server.setHandler(bb);

    try {
      System.out.println(">>> STARTING EMBEDDED JETTY SERVER, PRESS ANY KEY TO STOP");
      server.start();
      while (System.in.available() == 0) {
        Thread.sleep(5000);
      }
      server.stop();
      server.join();
    } 
    catch (Exception e) {
      e.printStackTrace();
      System.exit(100);
    }
  }
}

```

设置solr.solr.home，并运行，在run configure中Arguments > VM arguments中写入

```
-Dsolr.solr.home=your/path/of/example/solr
```

当然也可以在代码中修改

```java
System.setProperty("solr.solr.home", "your/path/of/example/solr");
```

使用solr自带的一个example作为solr配置的根目录，也可以设置其他的solr配置目录。点击run即可运行Solr，debug也可以用。

最后，在浏览器中输入http://localhost:8983/solr 查看Solr Admin页面

![solr-startup-successful](https://raw.github.com/kiahhan/KeepThing/master/content_solr-src-build/solr-startup-successful.png)