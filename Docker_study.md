### Study Docker on Ubuntu 13.10

> 以下内容中是根据Yongboy~的[blog](http://www.blogjava.net/yongboy/)内容，经过本人尝试后所写，其中有部分是原帖中没有，而在我尝试过程中遇到的问题，在此提供解决的方法。

[Docker](http://www.docker.io "Docker website")旨在提供一种应用程序的自动化部署解决方案，在 Linux 系统上迅速创建一个容器（轻量级虚拟机）并部署和运行应用程序，并通过配置文件可以轻松实现应用程序的自动化安装、部署和升级，非常方便。因为使用了容器，所以可以很方便的把生产环境和开发环境分开，互不影响，这是 docker 最普遍的一个玩法。更多的玩法还有大规模 web 应用、数据库部署、持续部署、集群、测试环境、面向服务的云计算、虚拟桌面 VDI 等等。

![Docker](http://atlassian.wpengine.netdna-cdn.com/wp-content/uploads/docker-logo.png "Docker logo")


> “Docker is an open-source engine which automates the deployment of applications as highly portable, self-sufficient containers which are independent of hardware, language, framework, packaging system and hosting provider.”

Here is a [demo](http://www.youtube.com/watch?v=wW9CAH9nSLs) on youtube and [examples](http://docs.docker.io/en/latest/examples/hello_world.html) on their site

Docker provides a streamlined user interface and API over the very cool [lxc](http://lxc.sourceforge.net/ "Linux Containers") (Linux Containers) and [AUFS](http://en.wikipedia.org/wiki/Aufs) (advanced multi layered unification filesystem) to configure, spin up, down, set and replay light weight vm-like containers.

主观的印象：Docker 使用 Go 语言编写，用 cgroup 实现资源隔离，容器技术采用 LXC. 提供了能够独立运行Unix进程的轻量级虚拟化解决方案。它提供了一种在安全、可重复的环境中自动部署软件的方式。LXC命令有些复杂，若感兴趣，这里有一篇我以前写的基于LXC，（从无到有，搭建一个简单版的JAVA PAAS云平台），可以提前复习一下。

有关实现原理、相关理论、运用场景等，会在本系列后面书写，这里先来一个浅尝辄止，完全手动，基于Docker搭建一个Tomcat运行环境。先出来一个像模像样Demo，可以见到效果，可能会让我们走的更远一些。

####环境
本文所有环境，Virtualbox上运行ubuntu-13.10-server-amd64,注意是64位系统.

####安装Docker
Docker 0.7版本需要linux内核 3.8支持，同时需要AUFS文件系统。

```bash
# 检查一下AUFS是否已安装
sudo apt-get update
sudo apt-get install linux-image-extra-`uname -r`
# 添加Docker repository key
sudo sh -c "wget -qO- https://get.docker.io/gpg | apt-key add -" 
# 添加Docker repository，并安装Docker
sudo sh -c "echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
sudo apt-get update
sudo apt-get install lxc-docker
# 检查Docker是否已安装成功
sudo docker version
# 终端输出
Client version: 0.7.3
Go version (client): go1.2
Git commit (client): 8502ad4
Server version: 0.7.3
Git commit (server): 8502ad4
Go version (server): go1.2
Last stable version: 0.7.3
```

####去除掉sudo
在Ubuntu下，在执行Docker时，每次都要输入sudo，同时输入密码，很累人的，这里微调一下，把当前用户执行权限添加到相应的docker用户组里面。

```bash
# 添加一个新的docker用户组
sudo groupadd docker
# 添加当前用户到docker用户组里，注意这里的yongboy为ubuntu server登录用户名
sudo gpasswd -a yongboy docker
# 重启Docker后台监护进程
sudo service docker restart
# 重启之后，尝试一下，是否生效
docker version
#若还未生效，则系统重启，则生效
sudo reboot
```

####安装一个Docker运行实例-ubuntu虚拟机
Docker安装完毕，后台进程也自动启动了，可以安装虚拟机实例（这里直接拿官方演示使用的learn/tutorial镜像为例）：
```bash
docker pull learn/tutorial
```
安装完成之后，看看效果
```bash
docker run learn/tutorial /bin/echo hello world
```
交互式进入新安装的虚拟机中
```bash
docker run -i -t learn/tutorial /bin/bash
```

===
#####我是小插曲
当安装完成后，并且验证以上的命令都可以正常执行。但是当我运行一下的命令时：
```bash
docker run -i -t learn/tutorial /bin/bash
#输出：
Thu Oct 31 02:39:00 UTC 2013
root@523b8a0d4ebd:/# 2013/10/31 02:39:05 Error: Cannot start container 523b8a0d4ebd: The container failed to start due to timed out.
```
无法进入交互模式，竟然报错说是timed out。这不是玩呢吗？直接问Google
找到一帖[Docker fails to start on Ubuntu 13.10](https://github.com/dotcloud/docker/issues/2476)
看了半天，说是当前的lxc的版本在ubuntu13.10上work的有问题，难不成要downgrade到ubuntu13.04（整个是官方提出的宿主机版本），不过终于找到[解决方案](https://gist.github.com/crosbymichael/7255065 "upgrade-lxc.sh")

别着急运行这个Bash脚本，经过测试，千万别执行`sudo apt-get remove lxc`，因为这会导致无法使用docker，必须要要不docker卸载掉，过程有些麻烦，这次就不说了。

直接执行以下步骤更新对应的**lxc**和**liblxc0**就行了
```bash
# Download the new pakcages with the updated
wget https://launchpad.net/ubuntu/+source/lxc/1.0.0~alpha1-0ubuntu12/+build/5171688/+files/lxc_1.0.0~alpha1-0ubuntu12_amd64.deb
wget https://launchpad.net/ubuntu/+source/lxc/1.0.0~alpha1-0ubuntu12/+build/5171688/+files/liblxc0_1.0.0~alpha1-0ubuntu12_amd64.deb
 
# Install the updated packages
sudo dpkg -i liblxc0_1.0.0~alpha1-0ubuntu12_amd64.deb
sudo dpkg -i lxc_1.0.0~alpha1-0ubuntu12_amd64.deb
```
安装之后，可以使用一下command看看版本是不是已经更新了
```bash
dpkg -l |grep lxc
#输出
ii  liblxc0                                   1.0.0~alpha1-0ubuntu12                  amd64        Linux Containers userspace tools (library)
ii  lxc                                       1.0.0~alpha1-0ubuntu12                  amd64        Linux Containers userspace tools
ii  lxc-docker                                0.7.3                                   amd64        Linux container runtime
ii  lxc-docker-0.7.3                          0.7.3                                   amd64        Linux container runtime
ii  lxc-templates                             1.0.0~alpha1-0ubuntu11                  all          Linux Containers userspace tools (templates)
ii  python3-lxc                               1.0.0~alpha1-0ubuntu11                  amd64        Linux Containers userspace tools (Python 3.x bindings)

```
顺便在给出我扒出来的以上两个deb文件的URL: https://launchpad.net/ubuntu/+source/lxc/1.0.0~alpha1-0ubuntu12/+build/5171688/

===


会看到：
```bash
root@ed8574616b2a:/# 
```
说明已经进入交互式环境。

安装SSH终端服务器，便于我们外部使用SSH客户端登陆访问
```bash
apt-get update
apt-get install openssh-server
which sshd
/usr/sbin/sshd
mkdir /var/run/sshd
passwd #输入用户密码，我这里设置为123456，便于SSH客户端登陆使用
exit #退出
```
获取到刚才操作的实例容器ID
```bash
docker ps -l
#输出
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
ed8574616b2a        ubuntu:12.04        /bin/bash           About an hour ago   Exit 0                                  elegant_fermi 
```
可以看到当前操作的容器ID为：ed8574616b2a。注意了，一旦进行所有操作，都需要提交保存，便于SSH登陆使用：
```bash
docker commit ed8574616b2a learn/tutorial
```
以后台进程方式长期运行此镜像实例：
```bash
docker run -d -p 22 -p 80:8080 learn/tutorial /usr/sbin/sshd -D
```
ubuntu容器内运行着的SSH Server占用22端口，-p 22进行指定。-p 80:8080 指的是，我们ubuntu将会以8080端口运行tomcat，但对外（容器外）映射的端口为80。

这时，查看一下，是否成功运行。
```bash
docker ps
#输出
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS              PORTS                                         NAMES
9783a49b21ed        learn/tutorial:latest   /usr/sbin/sshd -D   7 seconds ago       Up 6 seconds        0.0.0.0:49153->22/tcp, 0.0.0.0:80->8080/tcp   high_albattani 
```
注意这里的分配随机的SSH连接端口号为***49153***：
```bash
ssh root@127.0.0.1 -p 49153
```
输入可以口令，是不是可以进入了？你一旦控制了SSH，剩下的事情就很简单了，安装JDK，安装tomcat等，随你所愿了。以下为安装脚本：

#####在ubuntu 12.04上安装oracle jdk 7
```bash
apt-get install python-software-properties
add-apt-repository ppa:webupd8team/java
apt-get update
apt-get install -y wget
apt-get install oracle-java7-installer
java -version
```
#####下载tomcat 7.0.47
```bash
wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-7/v7.0.47/bin/apache-tomcat-7.0.47.tar.gz
```
#####解压，运行
```bash
tar xvf apache-tomcat-7.0.47.tar.gz
cd apache-tomcat-7.0.47
bin/startup.sh
```
默认情况下，tomcat会占用8080端口，刚才在启动镜像实例的时候，指定了 -p 80:8080，ubuntu镜像实例/容器，开放8080端口，映射到宿主机端口就是80。知道宿主机IP地址，那就可以自由访问了。在宿主机上，通过curl测试一下即可：
```bash
curl http://192.168.190.131
```
可以使用浏览器访问啦。



###Reference:
+ [Docker学习笔记之一，搭建一个JAVA Tomcat运行环境](http://www.blogjava.net/yongboy/archive/2013/12/12/407498.html)
+ [Docker学习笔记之二，基于Dockerfile搭建JAVA Tomcat运行环境](http://www.blogjava.net/yongboy/archive/2013/12/16/407643.html)
+ [Docker学习笔记之三，有关状态的记录](http://www.blogjava.net/yongboy/archive/2013/12/29/408173.html)
+ [Docker学习笔记之四，构建一个Redis as a Service(RAAS)](http://www.blogjava.net/yongboy/archive/2013/12/31/408297.html)
+ [Deploy Java Apps With Docker = Awesome](http://blogs.atlassian.com/2013/06/deploy-java-apps-with-docker-awesome/)


