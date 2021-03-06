## **<a id="t1">将spring-cloud项目打包成docker镜像并启动</a>**
1. [项目地址](spring-cloud/spring-cloud-eureka/spring-cloud-eureka-server01/spring-cloud-eureka-server)

2. 步骤
   - pom.xml 里面加入如下配置:

     ```xml
     
         <build>
             <finalName>spring-cloud-eureka-server01-service</finalName>
             
             <plugins>
                 <plugin>
                     <groupId>com.spotify</groupId>
                     <artifactId>docker-maven-plugin</artifactId>
                     <version>0.4.13</version>
                     <configuration>
                         <!-- leegoo0820/spring-cloud-eureka-server01-service(name):0.0.1(tag)-->
                         <imageName>leegoo0820/spring-cloud-eureka-server01-service:0.0.1</imageName>
                         <!-- 基础镜像 -->
                         <baseImage>java</baseImage>
                         <entryPoint>["java","-jar","/${project.build.finalName}.jar"]</entryPoint>
                         <resources>
                             <resource>
                                 <targetPath>/</targetPath>
                                 <!-- 目录地址,也就是target生成的地址 -->
                                 <directory>${project.build.directory}</directory>
                                 <!--需要复制的jar包 -->
                                 <include>${project.build.finalName}.jar</include>
                             </resource>
                         </resources>
                     </configuration>
                 </plugin>
             </plugins>
         </build>
     ```

   - 将项目文件传到linux 服务器:

   - 使用maven命令进行打包(需要提前配置[maven(3.5版本已经失效,需要自己去apache官网查看最新的下载地址)](<https://www.cnblogs.com/jimisun/p/8054819.html>) 环境,和[java](<https://www.cnblogs.com/xiaohao95/p/7615248.html>)环境才能进行打包):

     ```shell
     mvn clean package docker:build
     ```

   - 然后使用`docker images` 会看到 `spring-cloud-eureka-server01-service` 镜像

   - 随后启动 

     ```shell
     [root@localhost spring-cloud-eureka-server]# docker run -p 8761:8761 --name my_eureka leegoo0820/spring-cloud-eureka-server01-service:0.0.1
     ```


   - 如果希望只执行`maven clean package` 达到 `mvn clean package docker:build`的效果 ,那么可以在`pom.xml`文件里面增加如下配置,简单理解为就是配置了`execution`后,会执行对应的命令,当前命令是`build`

     ```xml
      <build>
             <finalName>spring-cloud-eureka-server01-service</finalName>
     
             <plugins>
                 <plugin>
                     <groupId>com.spotify</groupId>
                     <artifactId>docker-maven-plugin</artifactId>
                     <version>0.4.13</version>
     				
                     <!-- 新加部分开始 -->
                     <executions>
                         <execution>
                             <id>build-images</id>
                             <phase>package</phase>
                             <goals>
                                 <goal>build</goal>
                             </goals>
                         </execution>
                     </executions>
                      <!-- 新加部分结束 -->
     
                     <configuration>
                         <!-- leegoo0820/spring-cloud-eureka-server01-service(name):0.0.1(tag)-->
                         <imageName>leegoo0820/spring-cloud-eureka-server01-service:0.0.1</imageName>
                         <!-- 基础镜像 -->
                         <baseImage>java</baseImage>
                         <entryPoint>["java","-jar","/${project.build.finalName}.jar"]</entryPoint>
                         <resources>
                             <resource>
                                 <targetPath>/</targetPath>
                                 <!-- 目录地址,也就是target生成的地址 -->
                                 <directory>${project.build.directory}</directory>
                                 <!--需要复制的jar包 -->
                                 <include>${project.build.finalName}.jar</include>
                             </resource>
                         </resources>
                     </configuration>
                 </plugin>
     
             </plugins>
         </build>
     ```
     
## **<a id="t2">安装`docker-compose`(linux)</a>**     
1. 安装`docker-compose`(linux)

   - 下载`docker-compose` (前提必须有docker)

   ```shell
   sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   
   ```
   - 授权`docker-compose`

     ```shell
     sudo chmod +x /usr/local/bin/docker-compose
     
     ```

   - 建立软链接(类似于在path 增加了`docker-compose`)

     ```shell
     sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
     ```

   - 检查是否安装完成

     ```shell
     [root@localhost my_files]# docker-compose --version
     docker-compose version 1.24.0, build 0aa59064
     ```

   - 增加`docker-compose`自动补全功能 (Place the completion script in `/etc/bash_completion.d/`)

     ```shell
      sudo curl -L https://raw.githubusercontent.com/docker/compose/1.24.0/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
     
     ```

## **<a id="t3">使用docker-compose 进行服务编排</a>**          
1. 
   - 首先在项目下创建`docker-compose.yml`文件, 比如我的项目名是 `spring-cloud-eureka-server`  ,那么就是`spring-cloud-eureka-server/docker-compose.yml`

   - 编辑`docker-compose.yml`内容

     ```yaml
     version: "2"
     
     services:
       eurela:
         image: spring-cloud-eureka-server_eurela
         volumes:  #挂载目录
           - "/usr/tmp/data/eureka"
         container_name: spring-cloud-eureka-server_eurela
         build: .  #找Dockerfile文件位置,  .代表当前文件下的Dockerfile文件
         ports:  #开放端口
           - "8767:8761"
     ```

   - 将项目传入到linux 服务器,随后到达项目的目录下 ,执行`docker-compose `命令

     ```shell
     docker-compose up  -d
     #-d 后台运行
     #如果要删除刚运行的容器
     # docker-compose stop   项目名 没有的话默认停止所有
     # docker-compose rm    项目名 没有的话默认删除所有
     ```

## **<a id="t4">使用`docker-compose` 同时部署多个docker 应用</a>**          
1.  使用`docker-compose` 同时部署多个docker 应用

   - 修改`spring-cloud-eureka-client01`中`pom.xml`配置,加入如下内容 [链接](spring-cloud/spring-cloud-eureka/spring-cloud-eureka-client01/pom.xml):

   - 修改`spring-cloud-eureka-server01`中的 `pom.xml`,加入如下内容 [链接](spring-cloud/spring-cloud-eureka/spring-cloud-eureka-server01/spring-cloud-eureka-server/pom.xml),

   - 修改`spring-cloud-eureka-server02`中的 `pom.xml`.加入如下内容 [链接](spring-cloud/spring-cloud-eureka/spring-cloud-eureka-server02/spring-cloud-eureka-server/pom.xml),

     ```xml
     
         <build>
             <plugins>
     
                 <plugin>
                     <groupId>com.spotify</groupId>
                     <artifactId>docker-maven-plugin</artifactId>
                     <version>0.4.13</version>
     
                     <configuration>
                         <!-- leegoo0820/spring-cloud-eureka-server01-service(name):0.0.1(tag)-->
                         <imageName>leegoo0820/${project.artifactId}:${project.version}</imageName>
                         <forceTags>true</forceTags>
                         <!-- 基础镜像 -->
                         <baseImage>java</baseImage>
                         <entryPoint>["java","-jar","/${project.build.finalName}.jar"]</entryPoint>
                         <resources>
                             <resource>
                                 <targetPath>/</targetPath>
                                 <!-- 目录地址,也就是target生成的地址 -->
                                 <directory>${project.build.directory}</directory>
                                 <!--需要复制的jar包 -->
                                 <include>${project.build.finalName}.jar</include>
                             </resource>
                         </resources>
                     </configuration>
                 </plugin>
     
             </plugins>
         </build>
     
     ```

   - 修改 `spring-cloud-eureka-client01` 中 `application.yml`配置

   - 修改 `spring-cloud-eureka-server01中 `application.yml`配置

   - 修改 `spring-cloud-eureka-server02` 中 `application.yml`配置

     ```yaml
     eureka:
       client:
         service-url:
         	#此处新加配置 ,因为docker 中使用localhost是不能找到eureka服务器的,宿主机都是单独的ip段 
         	#client 需要绑定多个eureka
         	#但是servier01 和但是servier02 只需要对方的 服务编排 "名字"即可,下方会增加服务编排 spring-cloud-eureka-server01 和 spring-cloud-eureka-server02配置
           defaultZone: http://spring-cloud-eureka-server01:8761/eureka/,http://spring-cloud-eureka-server02:8761/eureka/
       instance:
         prefer-ip-address: true  #ip注册
     ```

   - 增加`docker-compose.yml` ,内容如下,[直达链接](spring-cloud/spring-cloud-eureka/docker-compose.yml)

     ```yaml
     version: "2"
     
     services:
       #此处就是服务编排的名称
       spring-cloud-eureka-server01:
         #image: 此处指定生成的image ,如果没有的话可能会去docker.io下载
         image: leegoo0820/spring-cloud-eureka-server01-service:0.0.1-SNAPSHOT
         volumes:  #挂载目录
           - "/usr/tmp/data/spring-cloud-eureka-server01"
         container_name: spring-cloud-eureka-server_eureka01
         build: .
         ports:  #开放端口
           - "8761:8761"
         #links:
         #  - spring-cloud-eureka-server02:discovery
         #environment:  设置环境参数
         #  - spring.profile.active=dev
     
       spring-cloud-eureka-server02:
         #image: java 指定镜像名或者id,没有就会下载
         image: leegoo0820/spring-cloud-eureka-server02-service:0.0.1-SNAPSHOT
         volumes:  #挂载目录
           - "/usr/tmp/data/spring-cloud-eureka-server02"
         container_name: spring-cloud-eureka-server_eureka02
         build: .
         ports:  #开放端口
           - "8762:8762"
     
       spring-cloud-eureka-client01:
         image: leegoo0820/spring-cloud-eureka-client01:0.0.1-SNAPSHOT
         volumes:  #挂载目录
           - "/usr/tmp/data/spring-cloud-eureka-client01"
         container_name: spring-cloud-eureka-client01
         build: .
         ports:  #开放端口
           - "8846:8846"
     
     
     ```

   - 访问`http://192.168.109.133:8846/say`  返回servier结果



------
