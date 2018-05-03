# CentOS 安装 kong  

KONG的使用基本和NGINX如出一辙。默认的工作目录是/usr/local/kong，默认的配置文件是/etc/kong/kong.yml。
如果需要工作在1024以下的端口，则要求root权限。  
KONG的启动命令很简单，就是kong start  
但是启动之前需要编辑配置文件。  
需要关注的主要是几个点：  
发送匿名统计报告，（默认打开，一定要关闭）  
后端数据库的配置，（默认Cassandra，建议Postgres）  
内存缓存大小配置，（默认128M，建议1G）  
出口端口（默认HTTP 8000，HTTPS 8443，建议直接改成80和443）  
管理端口（默认8001，我改成8888比较吉利）  
配置好之后，启动KONG即可。  

## lua：
- lua安装: yum install lua 

- lua包管理工具安装: yum install -y luarocks 

```
tar -zxvf luarocks-2.1.0.tar.gz
cd luarocks-2.1.0
./configure --prefix=/usr/local/luarocks
make build
make install  
OK了,luarock的可执行文件被安装到了 /usr/local/luarocks/bin/luarocks.
```

- lua-resty-jwt安装: luarocks install lua-resty-jwt


## kong插件安装
- kong插件安装目录: /usr/local/share/lua/5.1/kong/plugins (keycloak-auth这个插件)

```
+- plugins
|  +- keycloak-auth
```

安装完插件后修改配置: vi /etc/kong/kong.conf

custom_plugins = keycloak-auth

然后重启kong，安装的插件就能生效了
