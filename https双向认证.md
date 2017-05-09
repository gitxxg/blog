#  TOMCAT-SSL配置HTTPS双向认证
SSL (Secure Socket Layer - 安全套接字层)
功能：保障在Internet上数据传输之安全，利用数据加密(Encryption)技术，确保数据在网络上之传输过程中不会被截取及窃听，防止篡改。

在D盘创建目录mykeys，启动命令行，并转移到 d:/mykeys

## 证书生产
### 1、创建服务器密钥
其密钥库为 d:/mykeys/server.keystore，注意keypass和storepass保持一致，它们分别代表 密钥密码和密钥库密码，
注意 CN=localhost 中，localhost表示要配置SSL的主机名，不能任意指定
```
keytool -genkey -v -alias serverKey -dname "CN=localhost" -keyalg RSA -keypass server123 -keystore server.keystore -storepass server123
```
### 2、创建客户端密钥
其密钥库为 d:/mykeys/client.p12，注意这个密钥库的后缀名，注意密钥库类型PKCS12
```
keytool -genkey -v -alias clientKey -dname "CN=SomeOne" -keyalg RSA -keypass client321 -keystore client.p12 -storepass client321 -storetype PKCS12
```
### 3、将客户端密钥导出为证书文件
```
keytool -export -alias clientKey -file clientKey.cer -keystore client.p12 -storepass client321 -storetype PKCS12
```
### 4、将上述客户端密钥文件导入服务器证书库，并设置为信任证书
注意会问你是否信任该证书，回答 y 即可
```
keytool -import -v -alias clientKey -file clientKey.cer -keystore server.keystore -storepass server123
```
### 5、通过 list 命令查看服务器的证书库
可以看到两个证书：一个是服务器证书，一个是受信任的客户端证书
```
keytool -list -keystore server.keystore
输入密钥库口令:

密钥库类型: JKS
密钥库提供方: SUN

您的密钥库包含 2 个条目

serverkey, 2017-5-9, PrivateKeyEntry,
证书指纹 (SHA1): 6E:CC:7B:0B:55:E7:24:77:2F:AF:0A:1F:69:3A:E8:6A:89:9A:82:BF
clientkey, 2017-5-9, trustedCertEntry,
证书指纹 (SHA1): DC:09:2D:03:0A:24:A4:A2:81:7F:69:62:E2:72:6B:96:3E:D1:D9:45
```

## 证书使用
### 1、配置tomcat
打开Tomcat根目录下的/conf/server.xml，找到Connector port="8443"配置段，修改为如下：
```
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
    SSLEnabled="true" maxThreads="150" scheme="https"
    secure="true" clientAuth="true" sslProtocol="TLS"
    keystoreFile="d:/mykeys/server.keystoree" keystorePass="server123"
    truststoreFile="d:/mykeys/server.keystore" truststorePass="server123" />
```
属性说明：
clientAuth:设置是否双向验证，默认为false，设置为true代表双向验证
keystoreFile:服务器证书文件路径
keystorePass:服务器证书密码
truststoreFile:用来验证客户端证书的根证书，此例中就是服务器证书
truststorePass:根证书密码

### 2、导入客户端证书
双击client.p12,然后一直下一步，导入证书到《个人》
### 3、导入客户端证书
双击clientKey.cer,然后一直下一步，导入证书到《受信任的根证书颁发机构》
### 4、浏览器 https://localhost:8443/
