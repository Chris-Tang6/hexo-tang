---
title: VLab部署指南
tags: [项目, Java, Vue, Sass]
date:	2022-4-26
---

本文档旨在帮助客户将VLab平台部署到你们的服务器上，并提供了管理员的使用介绍。

## 配置环境⚙

![image-20220426102607392](https://s2.loli.net/2022/04/26/ykpGAu7RT2HOQKg.png)

### 安装jdk1.8

[有道云笔记 (youdao.com)](https://note.youdao.com/ynoteshare/index.html?id=0d342eb40a5f6989f739c896ef04e785&type=note&_time=1647671319026)

### 安装docker

[CentOS Docker 安装 | 菜鸟教程 (runoob.com)](https://www.runoob.com/docker/centos-docker-install.html)

```bash
#启动docker
sudo systemctl start docker

#docker需要用户具有duso权限，为了避免每次命令都输入sudo可以把用户加入Docker用户组
sudo usermod -aG docker $USER

#查看镜像
docker images

#开机自启
systemctl enable docker

#阿里云镜像加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
	"registry-mirrors":["https://exjaocnn.mirror.aliyuncs.com/"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 使用docker安装mysql8.0

```bash
# install
docker pull mysql:8.0

# check whether installed
docker images

# use docker run mysql
# -p 3306:3306: the front means server's port, the last one means docker's port
# –name：给新创建的容器命名，此处命名为mysql
# -e：配置信息，此处配置mysql的root用户的登陆密码
# -p：端口映射，此处映射主机3306端口到容器“mysql”的3306端口
# -d：成功启动容器后输出容器的完整ID，例如上图 73f8811f669ee...
# 最后一个mysql指的是mysql镜像名字
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0
# 输出：9aea5372e9657777d1203a9532328fb17acaf05296c93d75a793aec6d1daf0a9

# check whether run success
docker ps

# 进入MySQL容器
docker exec -it 9aea5372e965 /bin/bash

# 登陆MySQL
mysql -uroot -p123456

# 修改密码，首先需要先登陆进mysql
set password for root@localhost = password('123456')
# 或者
alter user 'root'@'%' identified by '123456';
flush privileges;
```

![image-20220320135315072](https://s2.loli.net/2022/04/26/xIuBYbeNfadrAMP.png)

<img src="https://s2.loli.net/2022/04/26/ZgeaYvx9wHdb1NO.png" alt="image-20220320135904010" style="zoom: 67%;" />

### 客户端连接mysql

在服务器上配置好数据库后，可以使用你的电脑远程连接服务器的数据库，这里使用DBeaver工具。

<img src="https://s2.loli.net/2022/04/26/rO9om8vFGXesT31.png" alt="image-20220320140015207" style="zoom:80%;" />

<img src="https://s2.loli.net/2022/04/26/6nBaS9wApdRE35c.png" alt="image-20220320141503647" style="zoom:67%;" />

报错了：最简单的解决方法是在连接后面添加 allowPublicKeyRetrieval=true

在dbeaver中解决办法是修改这个属性为true

![image-20220320141613902](https://s2.loli.net/2022/04/26/EvhWusJc7mRjMrb.png)

![image-20220320141636581](https://s2.loli.net/2022/04/26/AluKjZ7GYFvdCMx.png)

### 导入MySQL数据库

这里要导入的数据就是我们前面提供的sql文件。

![image-20220320144026446](https://s2.loli.net/2022/04/26/5NGHdmuVCOq6ZfA.png)

<img src="https://s2.loli.net/2022/04/26/gDI6WlYSz1nxyJ9.png" alt="image-20220320143940954" style="zoom:80%;" />

### 修改数据表中的值

因为数据库中存储的许多数据都使用了文件的URL，而现在我们的系统平台已经部署到其它的服务器上去了，所以我们需要更新这些URL，具体做法是将URL中的ip更改为新服务器的ip。

例如下面的更新语句，使用SQL中的***REPLACE***函数。

```sql
UPDATE vlab.experiment SET `input` = REPLACE(`input` , '127.0.0.1','124.71.14.213');
```

我们要更新的属性列有下面这些：

```
exp_template：filefold_dir, icon

experiment：input, output

file：	url
```

### 修改vlab-back配置

修改**application.yml**文件，三个地方要修改：server.ip，datasource.url，file，分别表示服务器的ip、数据库位置、文件存储位置。

![image-20220426103202781](https://s2.loli.net/2022/04/26/OpezKjL4PASxgMh.png)

```yml
server:
  ip: 124.71.14.213
#  ip: 127.0.0.1
  port: 9090
  front-port: 8080

spring:
  servlet:
    multipart:
      max-file-size: 200MB
      max-request-size: 200MB
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
#    url: jdbc:mysql://127.0.0.1:3306/vlab?characterEncoding=utf-8&serverTimezone=GMT%2b8
    url: jdbc:mysql://124.71.14.213:3306/vlab?characterEncoding=utf-8&serverTimezone=GMT%2b8
    username: root
    password: 123456
mybatis:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

#    path: H:/bioinfo_vlab/files/
files:
  upload:
#    path: H:/bioinfo_vlab/files/
    path: /home/bioinfo_vlab/files/
  code:
#    path: H:/bioinfo_vlab/files/code/
    path: /home/bioinfo_vlab/files/code/
  temp:
#    path: H:/bioinfo_vlab/files/temp/
    path: /home/bioinfo_vlab/files/temp/
```

### 传输文件

这些文件是原本项目中已有的一些文件和算法文件。

将files文件夹使用**Xftp**工具传输到服务器的对应位置——这个位置就是你刚在[application.yml](###修改vlab-back配置)中配置的那个位置。

![image-20220426104033623](https://s2.loli.net/2022/04/26/HoKiskxrhMcRQql.png)

### 打包springboot项目vlab_back

```bash
# 首先在控制台进入到项目所在的根目录
mvn clean package -DskipTests

# 启动jar包
java -jar xxx.jar
```

### 服务器上启动jar包

```bash
# 首先进入到jar包所在目录
# 增加权限
chmod 777 xxx.jar

# 启动（使用下面的后台启动）
java -jar xxx.jar

# 后台启动
nohup java -jar xxx.jar &

# 查看日志
tail -500f nohup.out

# 查看是否成功启动
ps -ef | grep java

# 关闭进程
kill -9 [进程号]
```

![image-20220320161156112](https://s2.loli.net/2022/04/26/l8JZUdymBcPoACz.png)

### 打包vue项目vlab_front

```bash
# 首先进入到vue项目所在的目录，输入下面的命令，成功后会生成一个dist目录
npm run build

# 要启动vue项目，首先需要安装一个anywhere插件-前端静态资源服务器
npm install anywhere -g

# 进入到上面生成的dist目录下，输入下面的命令来启动
anywhere -p 8080
```

### 安装nginx

```bash
# 安装nginx所需的依赖和相关库
yum -y install gcc-c++ zlib-devel openssl-devel libtool

# 下载
cd /usr/local
wget https://nginx.org/download/nginx-1.14.0.tar.gz

# 解压
tar -zxvf nginx-1.14.0.tar.gz
rm -rf nginx-1.14.0.tar.gz

# 配置
cd nginx-1.14.0/
./configure --prefix=/usr/local/nginx

# 安装
make && make install

# 启动
cd ../nginx/sbin
./nginx

# 查看
ps -ef | grep nginx

```

### vue-dist nginx配置

```bash
# 进入到配置文件目录
cd /usr/local/nginx/conf

# 编辑配置文件
vim nginx.conf
# 配置如下内容
#    location / {
#	    root /home/bioinfo_vlab/dist;
#    	index index.html index.htm;
#	    try_files $uri $uri/ /index.html;
#    }
        
# 重启nginx
cd ../sbin
./nginx -s reload

```

![image-20220321112748641](https://s2.loli.net/2022/04/26/cFBQsR5zD2mpW3a.png)

配置好后，就将前面打包的vue-dist文件夹放在这个目录下即可，我配置是"/home/bioinfo_vlab/dist"这个目录。

### 安装python

因为已有的实验流程都是基于python的，这里根据他们需要的依赖安装所需的库文件，并修改数据库中exp_template表中compiler_dir的值。

例如我这里安装的是python3.6，配置好环境后可以使用命令“python3”或“python36”来直接使用python3.6，所以在数据表中我就需要将这些compiler_dir改成“python3”或“python36”。

需要安装的库有下面这些：

![image-20220426105916023](https://s2.loli.net/2022/04/26/uYB9b8pTh6Z7xlP.png)

## 可能会遇到的问题🤨

1. 运行MDA实验时，报错找不到bz2这个模块

   [解决ModuleNotFoundError: No module named '_bz2'_小小北漂的博客-CSDN博客](https://blog.csdn.net/u014589856/article/details/89175609)

2. 测试MDA实验时，报错

   ```
   /usr/local/python36/lib/python3.6/site-packages/pandas/compat/__init__.py:120: UserWarning: Could not import the lzma module. Your installed Python is incomplete. Attempting to use lzma compression will result in a RuntimeError.  warnings.warn(msg)
   ```

   [python安装pandas库出现 No module named ‘_lzma’_就是sang飞飞啦的博客-CSDN博客_lzma module](https://blog.csdn.net/sangfei18829896970/article/details/97754635?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-97754635-blog-112849662-2~default~CTRLIST~default-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-97754635-blog-112849662-2~default~CTRLIST~default-1.pc_relevant_default&utm_relevant_index=2)



## 管理员使用指南✨

这里我们着重介绍信息管理模板，信息管理也是主要提供给管理员使用的功能。

![image-20220426110940945](https://s2.loli.net/2022/04/26/HxbRMhEeTd6DgZX.png)

### 用户管理

中心部分是以表格展示的用户信息，表格上方是工具栏，表格中每行都有对应的操作按钮，可以对该行进行编辑和删除。最下方是分页栏，可以修改页面大小和页号。

![image-20220426111227954](https://s2.loli.net/2022/04/26/TvIEgm91w7f4Z6u.png)

#### 给用户修改角色

角色是为了控制每名用户的权限所设计的，默认角色是普通用户。

点击编辑按钮，在出现的对话框中修改角色即可。

![image-20220426111702377](https://s2.loli.net/2022/04/26/TuIjxFNX18w2C5G.png)

### 角色管理

每一名用户都对应一种角色，默认创建一个账号后是”普通用户“的角色。这里我们可以修改每种角色的权限等信息。

如果要修改用户角色请参考——[给用户修改角色](##给用户修改角色)

![image-20220426111518489](https://s2.loli.net/2022/04/26/YUlEhAGxaRLnSmo.png)

这里我们**不建议**你修改唯一标识或者是删除已有的这些角色。

![image-20220426112519545](https://s2.loli.net/2022/04/26/LbvC79UaIu8RsJc.png)

一般可以接受的操作是修改描述、给已有的角色分配菜单已经新增角色。

![image-20220426112541298](https://s2.loli.net/2022/04/26/4jPia8CXVtTRJnG.png)

### 菜单管理

VLab支持管理员对平台的菜单栏进行动态的管理，例如：名称、图标、子菜单等。

![image-20220426112649759](https://s2.loli.net/2022/04/26/bzVrEgX2pY8Goeh.png)

当你没有给某一菜单设置访问路径时，就可以对该菜单设置二级菜单，具体做法是点击操作栏的”新增子菜单“按钮。

![image-20220426112721325](https://s2.loli.net/2022/04/26/UuFvlYwyb47daIx.png)

![image-20220426112853949](https://s2.loli.net/2022/04/26/kdPhCT21QMfLGyc.png)

注意其中的访问路径和页面路径的区别：

- 访问路径：是设置在浏览器的地址栏中显示的路径名
- 页面路径：不能随意设置，这里填写的名称必须是你写的vue组件的名称。

### 文件管理

![image-20220426113100499](https://s2.loli.net/2022/04/26/6DJKr5AtwRVZqdU.png)

### 实验模板审核

该页面主要是进行模板审核的。

在该页面，三种角色所浏览、可操作的内容都各不相同。

审核员：模板浏览、模板审核

部署员：模板浏览、模板测试、模板审核

超级管理员：所有功能呢

![image-20220426113119211](https://s2.loli.net/2022/04/26/FmgiEl49ZrnqjOe.png)

点击表格中某一行，即可跳转到该模板的详细内容

![查看模板](https://s2.loli.net/2022/04/26/Rl6o1qx5u3zOb4m.gif)

点击审核按钮，可以对该模板进行审核、以及设置编译器路径

![image-20220426113618065](https://s2.loli.net/2022/04/26/DmYFSHKaCWo4iql.png)

![image-20220426113701824](https://s2.loli.net/2022/04/26/qb9U7pmZTVrX21c.png)

点击测试按钮，可以运行该模板

![image-20220426113628594](https://s2.loli.net/2022/04/26/mqPwplOCF1D7yTK.png)

![image-20220426114159347](https://s2.loli.net/2022/04/26/N2w4eQLG3cItdvB.png)

下载按钮可以下载该模板的算法压缩文件

![image-20220426113648493](https://s2.loli.net/2022/04/26/aBCAY37ixpFGdRL.png)