# Linux系统下安装tomcat

### 一、前置条件

  先安装jdk，[详见参考文章](https://www.cnblogs.com/li150dan/p/12532700.html)

### 二、Linux上安装tomcat

1. 下载Apache tomcat

  [tomcat官网下载地址](https://tomcat.apache.org/download-80.cgi)

  在左边，可以选择下载各种版本的tomcat。根据服务器操作系统选择下载。Linux操作系统就下载tar.gz包。

  我下载的文件名是：apache-tomcat-8.5.37.tar.gz

2. 上传服务器、解压缩tomcat

  使用Xftp等工具将tar.gz压缩包上传到 /usr/local 目录中，以下操作命令都需要在root账户下操作。

  进入目录解压安装包

  进入目录：cd /usr/local

  解压文件：tar -zxvf apache-tomcat-8.5.37.tar.gz` ` 

  将目录apache-tomcat-8.5.37重命名为tomcat，方便一些

  重命名文件：mv apache-tomcat-8.5.37 tomcat

3. 正常启动

  运行 /usr/local/tomcat/bin/startup.sh 就会启动tomcat，在浏览器中访问http://ip:8080了，能看到如下界面，说明访问成功。

4. 正常关闭

  运行 /usr/local/tomcat/bin/shutdown.sh 就会关闭tomcat

5. 修改tomcat配置

  进入 /usr/local/tomcat/conf 目录修改server.xml中的tomcat端口

  运行 vi server.xml，用“/8080”定位到8080端口。　

  可以修改端口，默认8080，然后按ESC键，然后按 :wq（注意有一个英文冒号），保存并退出。

6. 启动失败解决办法

  如果不能访问，说明没有开启8080端口

  先查看防火墙开放端口列表 firewall-cmd --zone=public --list-ports

  如果没有8080端口，添加端口

-  如果是firewalld防火墙 ，开启防火墙端口 firewall-cmd --zone=public --add-port=8080/tcp --permanent 并且重新加载防火墙 firewall-cmd --reload
-  如果是iptables防火墙 ，执行 vi /etc/sysconfig/iptables 加入下面内容 -A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT

7. 其他一些问题 

  启动以后，可以利用命令 ps -ef|grep tomcat 查看tomcat是否正常启动。 

  如果正常启动以后，无法访问，可以先不修改8080端口，尝试用8080端口访问。如果可以通过8080端口正常访问，则说明80端口被其他应用占用。 

  或者使用 netstat -an|grep 80 命令查看是否有程序占用80端口。我的80端口就是被另外一个程序给占用了。