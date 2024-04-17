## linux系统重装nginx

#### 1. [apt-get和yum包管理工具](https://worktile.com/kb/ask/437645.html)
看到网上很多使用`apt-get`命令的文章，包括直接使用`apt-get install nginx`，我自己试了一下，是报错的，后来发现`linux`也是有不同的内核的，有的内核使用`apt-get`,又得使用`yum`，而我的就是基于`Red Hat`所以使用`yum`

#### 2. 删除已经安装的nginx
使用命令
```bash
# 找出所有关于nginx的文件
find / -name nginx
/usr/local/nginx
xxx


# 删除找到的文件
rm -rf /usr/local/nginx
rm -rf xxx

# 执行
yum remove nginx
# 返回以下内容，所以删除成功
# 参数 nginx 没有匹配
# 不删除任何软件包
```

#### 3. 安装nginx
1. 存储下载软件
```bash
mkdir  /soft && mkdir /soft/nginx/
```

2. 使用wget下载
```bash
wget https://nginx.org/download/nginx-1.25.5.tar.gz

# 如果报错，使用yum安装
yum -y install weget
```

3. 解压
```bash
tar -xfvz nginx-1.25.5.tar.gz
```

4. 安装nginx需要的依赖包
```bash
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

5. 进入到解压目录
```bash
cd /nginx-1.25.5.tar.gz
./configure --prefix=/soft/nginx/
```

6. 编译
```bash
make && make install
```

7. 可以再soft/nginx中查看一些配置文件
```bash
ll
```

8. 修改配置文件
```bash
vim conf/nginx.conf
```

9. 启动nginx
在这一步，我卡了很久，因为我始终找不到sbin在哪里，我一直以为要退出当前目录，其实你在/usr/local/nginx这个目录下，输入cd sbin就可以了，make install这一步绝对不能少
```
sbin/nginx -s reload
```

参考文档
- https://blog.csdn.net/qq_68163788/article/details/134519218
- https://blog.csdn.net/studio_1/article/details/128534769#Nginx_1