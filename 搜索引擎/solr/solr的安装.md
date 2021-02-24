# Solr安装

### 下载地址

http://archive.apache.org/dist/lucene/solr/

### window上安装

​		只需解压，然后进入bin目录，执行solr start就可以启动了。

### linux上安装

   1.确保tomcat启动没有问题。

2. 解压solr安装包：tar -zxvf solr-XX.tgz    移动到自己的软件安装目录： mv solr-XX  /software/solr

3. 将solr的war包部署到tomcat上：cp -r /software/soft/dist/solr-XX.war  /software/tomcat/webapps/solr.war

4. 启动tomcat：bin目录下使用./startup.sh ，这时候 war包被解压成solr   然后关闭tomcat:./shutdown.sh
   删除solr.war：rm -f solr.war

5. 拷贝solr启动需要的jar包  jar包位置：example/lib/ext/下的所有jar包: cp * /software/tomcat/webapps/solr/WEB-INF/lib/

6. 拷贝solrhome  solr/example/solr目录就是一个solrhome。复制此目录到/tomcat/solrhome   

7. 关联solr及solrhome

   需要修改solr工程的web.xml文件  vi /software/tomcat/webapps/solr/WEB-INF/web.xml

   ```xml
       <env-entry>
          <env-entry-name>solr/home</env-entry-name>
          <env-entry-value>/software/tomcat/solrhome</env-entry-value>
          <env-entry-type>java.lang.String</env-entry-type>
       </env-entry>
   ```

   8.启动tomcat   访问 ip:8080/solr

### ik分词器配置

1、 下载IK Analyzer 2012FF_hf1.zip压缩包。

下载网址：http://code.google.com/p/ik-analyzer/downloads/list

2、 将IK Analyzer 2012FF_hf1.zip解压，并把解压后的文件夹中的IKAnalyzer2012FF_u1.jar复制到D:\Tomcat6.0\webapps\solr\WEB-INF\lib目录下，也就是solr.war部署的地方。

3、 在D:\Tomcat6.0\webapps\solr\WEB-INF目录下创建classes文件夹，并把IK Analyzer 2012FF_hf1.zip解压包中的IKAnalyzer.cfg.xml和stopword.dic复制到新创建的classes目录中。

4、 配置D:\solr\collection1\conf目录中的schema.xml配置文件。

加入如下配置项：

```
<!-- IKAnalyzer 中文分词 -->       
 <fieldType name="text" class="solr.TextField">        
 
        <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer">     
                  </analyzer>     
                      </fieldType>
```

5、 启动Tomcat服务器，在浏览器中输入网址：

http://localhost:8983/solr/#/collection1/analysis

