# CentOS 安装 docker  

Docker 运行在 CentOS 7 上，要求系统为64位、系统内核版本为 3.10 以上  

1.查看当前系统内核版本：  
[root@docker ~]# uname -r
3.10.0-514.el7.x86_64  

2.安装docker
[root@docker ~]# yum -y install docker-io  

3.启动 Docker 后台服务  
[root@docker ~]# service docker start  

4.镜像加速

鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决，我使用的是阿里的镜像地址：

在/etc/docker/daemon.json文件中添加如下内容.

{
  "registry-mirrors": ["https://wghlmi3i.mirror.aliyuncs.com"]
}
 至此，docker安装完成

5.运行时报错处理  
```
报错：Is 'docker -d' running on this host  

查看docker日志：
vi /var/log/docker   
/usr/bin/docker: relocation error: /usr/bin/docker: symbol dm_task_get_info_with_deferred_remove,   
version Base not defined in file libdevmapper.so.1.02 with link time reference  

解决办法：yum upgrade device-mapper-libs  
```

三、卸载docker
```
列出你安装过的包

[root@docker ~]# yum list installed | grep docker  
docker.x86_64                        2:1.13.1-53.git774336d.el7.centos @extras  
docker-client.x86_64                 2:1.13.1-53.git774336d.el7.centos @extras  
docker-common.x86_64                 2:1.13.1-53.git774336d.el7.centos @extras  

删除软件包

[root@docker ~]# yum -y remove docker.x86_64
[root@docker ~]# yum -y remove docker-client.x86_64
[root@docker ~]# yum -y remove docker-common.x86_64

docker开启远程访问  

vim /etc/sysconfig/docker  
添加配置:  
DOCKER_OPTS="-H unix:///var/run/docker.sock -H 0.0.0.0:5555"  
```

## springboot + docker + maven

- 不使用docker-maven-plugin 和 dockerfile-maven-plugin 进行docker images
的编译生成。

项目目录结构:  
```
+- mq-control
|  +- src
|  |  +- docker
|  |  |  \- Dockerfile
|  |  |  \- start.sh
|  |  +- main
|  |  |  +- java
|  |  |  +- resource
```

pom.xml

```
     <!-- springboot插件，用于编译spring项目 -->
     <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <excludeDevtools>false</excludeDevtools>
          <classifier>exec</classifier>
        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>repackage</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
	  
	  <!-- 将和docker相关的资源文件copy到docker输出目录 -->
	  <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-resources-plugin</artifactId>
        <inherited>false</inherited>
        <executions>
          <execution>
            <id>copy-bin</id>
            <phase>package</phase>
            <goals>
              <goal>copy-resources</goal>
            </goals>
            <configuration>
              <outputDirectory>${project.build.directory}/docker/package-exec/bin</outputDirectory>
              <resources>
                <resource>
                  <directory>${project.basedir}/src/docker</directory>
                  <include>*.sh</include>
                </resource>
              </resources>
              <includeEmptyDirs>true</includeEmptyDirs>
            </configuration>
          </execution>
          <execution>
            <id>copy-dockerfile</id>
            <phase>package</phase>
            <goals>
              <goal>copy-resources</goal>
            </goals>
            <configuration>
              <outputDirectory>${project.build.directory}/docker</outputDirectory>
              <resources>
                <resource>
                  <directory>${project.basedir}/src/docker</directory>
                  <include>Dockerfile</include>
                </resource>
              </resources>
              <includeEmptyDirs>true</includeEmptyDirs>
            </configuration>
          </execution>
        </executions>
      </plugin>

	  <!-- srpingboot项目包解压到到docker输出目录 -->
      <plugin>
        <artifactId>maven-antrun-plugin</artifactId>
        <executions>
          <execution>
            <phase>package</phase>
            <configuration>
              <tasks>
                <unzip src="${project.build.directory}/${project.artifactId}-${project.version}-exec.jar"
                  dest="${project.build.directory}/docker/package-exec/lib"/>
              </tasks>
            </configuration>
            <goals>
              <goal>run</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
```

编译后的docker目录结构:  
```
+- target
|  +- docker
|  |  +- package-exec
|  |  |  +- bin
|  |  |  |  \- start.sh 
|  |  |  +- lib
|  |  |  |  +- BOOT-INF
|  |  |  |  +- META-INF
|  |  |  |  +- org
|  |  |  \- Dockerfile
```

Dockerfile 内容
```
FROM java:8

USER root

EXPOSE 8080

RUN mkdir -p /opt/mq-control

COPY package-exec /opt/mq-control

RUN chmod +x /opt/mq-control/bin/*.sh

CMD ["/opt/mq-control/bin/start.sh"]
```

start.sh 内容
```
#!/bin/bash

BIN_DIR=`dirname $0`
BASE_DIR=`cd ${BIN_DIR}/.. && pwd`

if [ "$1" = "migration" ]; then

    CP=${BASE_DIR}/lib/BOOT-INF/classes
    for filename in ${BASE_DIR}/lib/BOOT-INF/lib/*.jar; do
      CP="${CP}:${filename}"
    done

    exec ${JAVA_HOME}/bin/java \
      -server -d64 \
      -cp ${CP} \
      com.ericsson.automotive.apimgr.migration.Migrator $2

else

    JAVA_OPTS=${JAVA_OPTS:-}

    exec ${JAVA_HOME}/bin/java \
      -server -d64 ${JAVA_OPTS} \
      -cp ${BASE_DIR}/lib \
      org.springframework.boot.loader.JarLauncher

fi
```