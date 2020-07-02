# Linux自动部署jar包，注册系统服务

## 1、创建service 

```
在 /usr/lib/systemd/system 下创建xxx.service
```

## 2、写入

```
[Unit]
Description=mall
After=syslog.target

[Service]
ExecStart=/usr/local/JDK/jdk1.8.0_171/bin/java -jar /usr/local/mall/mall.jar

ExecReload=/bin/kill -s HUP $MAINPID
ExecStop= ps -ef | grep doctor-pc-0.0.1| grep -v grep |awk '{print $2}' | xargs sudo kill -9
[Install]

WantedBy=multi-user.target

```

## 3、给service权限

```
chmod 777 xxx.service
```

## 4、启动，关闭，开机自启，关闭开机自启

```
systemctl start xxx.service
systemctl stop xxx.service
systemctl enable  xxx.service
systemctl disenable  xxx.service
```

