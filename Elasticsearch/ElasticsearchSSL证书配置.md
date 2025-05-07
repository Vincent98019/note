
### 生成CA证书（信任根）

```bash
# 生成CA证书（有效期10年） 
./bin/elasticsearch-certutil ca --pem --days 3650 --out config/certs/ca.zip

# 解压
unzip config/certs/ca.zip -d config/certs/
```

> 解压后会得到 `ca/ca.crt` 和 `ca/ca.key`（私钥）

### 生成节点证书

```bash
# 为单节点生成证书（包含本地地址）
./bin/elasticsearch-certutil cert \
--pem --ca-cert config/certs/ca/ca.crt \
--ca-key config/certs/ca/ca.key \
--name "es-test" \
--dns localhost \
# --dns xxx 可以使用通配符，也可以有多个，*.example.com
--ip 127.0.0.1 \
# --ip xxx docker内部地址，以及局域网地址等，可以有多个
--out config/certs/es-test.zip

# 解压
unzip config/certs/es-test.zip -d config/certs/`
```

> 生成 `localhost.crt`（证书）和 `localhost.key`（私钥）

### 配置安全参数

`elasticsearch.yml` 中修改并重启：

```yaml
xpack.security.http.ssl:
  enabled: true
  key: certs/es-test/es-test.key
  certificate: certs/es-test/es-test.crt
  certificate_authorities: [ "certs/es-test/ca.crt" ]

xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  key: certs/es-test/es-test.key
  certificate: certs/es-test/es-test.crt
  certificate_authorities: [ "certs/es-test/ca.crt" ]
```


### Java连接

下载刚刚生成的 `es-test.crt` 文件，将证书导入到Java中，有两种导入方式：
#### 全局导入

```bash
keytool -importcert -trustcacerts \
-keystore $JAVA_HOME/lib/security/cacerts \
-storepass changeit \
-noprompt -alias es-cert \
-file /path/to/certificate.cer
```

找到Java的 `cacerts` 文件，将证书导入并且命名为 `es-cert` ，密码默认是 `changeit` 。


#### 自定义信任库

如果不想修改全局的 `cacerts` ，也可以自己创建一个 `truststore.jks` ，然后在启动时使用：

```bash
keytool -importcert \
  -file es-test.crt \
  -alias es-test-cert \
  -keystore truststore.jks \
  -storepass changeit \
  -noprompt
```

在启动 Java 程序时，添加以下 JVM 参数：

```bash
-Djavax.net.ssl.trustStore=路径/truststore.jks \
-Djavax.net.ssl.trustStorePassword=changeit
```

例如：

```bash
java -Djavax.net.ssl.trustStore=/path/to/truststore.jks \
     -Djavax.net.ssl.trustStorePassword=changeit \
     -jar your-app.jar
```


### Kibana连接

修改Kibana的配置文件， `kibana.yml` ：

```yaml
server.host: "0.0.0.0"
server.shutdownTimeout: "5s"
elasticsearch.hosts: ["https://192.168.0.163:9200"]
elasticsearch.ssl.certificateAuthorities: ["/usr/share/kibana/config/ca.crt"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "es1019"
monitoring.ui.container.elasticsearch.enabled: true
```

将对应的地址、账号、密码、证书配置上去重启即可，这里主要的证书是第一次生成的 `ca.crt` 。