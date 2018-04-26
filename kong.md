# kong  


## lua：
lua安装: yum install lua

lua包管理工具安装: yum install -y luarocks 

lua-resty-jwt安装: luarocks install lua-resty-jwt

## kong插件安装
kong插件安装目录: /usr/local/share/lua/5.1/kong/plugins (keycloak-auth这个插件)

```
+ plugins
  + keycloak-auth
```

安装完插件后腰修改配置: vi /etc/kong/kong.conf

custom_plugins = keycloak-auth

然后重启kong，安装的插件就能生效了
