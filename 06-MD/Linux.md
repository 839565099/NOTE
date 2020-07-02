# Linux

# 一、常用命令

- 查看防火墙：


```
systemctl status firewalld
```

- 关闭防火墙


```
systemctl stop firewalld
```

- 开启防火墙


```
systemctl start firewalld
```

- 永久关闭防火墙：


```
systemctl disable firewalld
```

- 将a文件改为a1文件，修改文件名：


```
cat redis.conf > redis8005.conf
```

- 将redis8005文件拷贝一个成为redis8006，并且把文件内容8005改为8006


```
 sed "s/8005/8006/g" redis8005.conf > redis8006.conf 
```

- 如果Linux强制关机，出现这个问题：


![1585205172366](SpringCloud.assets\1585205172366.png)

解决方法：

```
xfs_repair -v -L /dev/dm-0
```

- 使命令能到处运行

  ```
  使/usr/bin/ruby指向真正的/usr/local/ruby/bin/ruby命令
  ln -s /usr/local/ruby/bin/ruby /usr/bin/ruby
  ```

  