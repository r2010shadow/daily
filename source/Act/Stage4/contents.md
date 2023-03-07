# Stage4



## Windows服务创建与实例应用

### 使用winsw工具

```
# Windows Service Wrapper小工具
下载文件
https://repo.jenkins-ci.org/artifactory/releases/com/sun/winsw/winsw/2.9.0/winsw-2.9.0-bin.exe  
重命名服务名
rename winsw-2.9.0-bin.exe nginx-server.exe
```

### 新建服务xml

```xml
# nginx-service.xml
<service>
    <id>nginx</id>nginx-service
	<name>nginx</name>
	<description>nginx</description>
	<logpath>D:\ng\logs\</logpath>
	<logmode>roll</logmode>
	<depend></depend>
	<executable>D:\ng\nginx.exe</executable>
	<stopexecutable>D:\ng\nginx.exe -s stop</stopexecutable>
</service>
```

### 注册服务

```
# 在nginx安装目录下以管理员运行命令
.\nginx-service.exe install
```

### 管理服务

```
nginx-service.exe install 命令可注册对应的系统服务
nginx-service.exe uninstall 命令可删除对应的系统服务
nginx-service.exe stop 命令可停止对应的系统服务
nginx-service.exe start 命令可启动对应的系统服务

```

## 服务实例-NG

### 下载NG

```
http://nginx.org/download/nginx-1.23.2.zip
```

### 配置开放目录

```
# 备份
copy nginx.conf nginx.conf-bak
# 修改
        location / {
            root   D:\01;   # 开启目录名称
            index  index.html index.htm;
            autoindex on;  # 开启目录文件列表
            autoindex_exact_size off;  # 显示出文件的确切大小，单位是bytes
            autoindex_localtime on;  # 显示的文件时间为文件的服务器时间
            charset utf-8,gbk;  # 避免中文乱码			
        }
```

### 启动服务

```
start nginx.exe -c conf/nginx.conf
```

### 检查服务

```
tasklist /fi “imagename eq nginx.exe”
netstat -ano | findstr 0.0.0.0:80
```

### 服务维护

```
# 退出服务
nginx -s quit
nginx-service.exe stop 命令可停止对应的系统服务

# 重载服务
nginx.exe -s reload

# 查看日志
type logs\error.log
```

---
版权声明：本文为CSDN博主「zhy810302」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/zhy810302/article/details/122254477

