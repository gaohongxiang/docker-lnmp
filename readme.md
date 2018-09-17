# 说明

本项目为docker-compose搭建lnmp环境
L：ubuntu18.04
N：nginx1.12
M：mysql5.7
P：php7.2

此项目只是用来搭建lnmp环境，具体项目请在www目录下完善。
如果部署博客项目可以参考https://github.com/gaohongxiang/www.blockchant.cn.git

如果还不太清楚docker，请参考本人博客[docker](http://www.blockchant.cn/tutorials/docker/71/docker-jian-jie)相关内容

# 必备工具

## docker 

[docker安装](http://www.blockchant.cn/tutorials/docker/72/docker-an-zhuang)

## docker-compose 

[docker-compose安装](http://www.blockchant.cn/tutorials/docker/87/compose-an-zhuang)

# 项目目录结构

```
docker
	---docker-compose.yml
	---mysql
		---data
		---mysql.conf
	---php
		---Dockerfile
		---php.ini
		---sources.list
	---nginx
		---nginx.conf
	---www
		---.env
		---public
			---index.php
	---readme.md
```

详解

## docker-compose.yml

docker-compose.yml为docker-compose的配置文件，用来编排容器。是核心文件！
特别注意各容器的volume挂载部分，代码的挂载和mysql数据的挂载。

## mysql

### data目录
data目录挂载到mysql容器的/var/lib/mysql目录下。容器里生成的数据都会在data目录中出现。

### mysql.conf配置文件
mysql.conf文件挂载到容器的/etc/mysql/mysql.conf.d/mysql.cnf文件上。相当于把mysql.conf文件里的内容写入容器的mysql.cnf文件里

## php

### Dockerfile文件
因为官方php镜像没有mysql、gd等模块。所以，采用dockerfile构建自己的镜像，并上传到了docker hub中。
所以，这里的Dockerfile文件不是必须的，docker-compose.yml文件中php镜像直接使用的上传到docker hub中的镜像。

### sources.list文件
sources.list文件为国内源文件，dockerfile文件构建镜像时换成国内源会比较快。构建完镜像后此文件也可不要。

当然，以上两个文件最好留着，如果构建的php镜像不合适，就需要修改dockerfile文件重新构建。

### php.ini文件
PHP的配置文件。挂载到了php容器中/etc/php/conf.d/目录中。


## nginx

### nginx.conf文件

nginx的配置文件，挂载到nginx容器中/etc/nginx/conf.d/default.conf文件上。

**注意文件中的root对应的项目根目录为/application/public。**本机代码在www目录下，www目录又挂载到了nginx容器的application目录下，相当于代码已经在application目录下了，入口文件就是application/public下的index.php文件。

## www

此目录为项目目录。根据自己的情况来完善此目录。

### public/index.php
一般php框架都有一个同一的入口文件，放在public目录下。这里模拟这种情况。对应的root目录请参考nginx部分说明。

### .env文件
laravel文件中，.env文件是不上传的，这里只是用来说明一个docker部署后的数据库连接的问题。

**DB_HOST=mysql**常规配置的话host本地的话php、mysql都在本地，php连接mysql的host就是localhost或127.0.0.1但docker部署的环境，php、mysql都是容器，相互是隔离的，所以php连接mysql时就是容器连接容器。host为mysql（mysql的容器名，取决于docker-compose.yml文件中的配置).而127.0.0.1只能指向各自容器！


# 操作

### 获取项目
```
git clone https://github.com/gaohongxiang/docker-lnmp.git
```
获取项目后可以把docker-lnmp目录改名，随意。

### 添加阿里镜像加速器
通过修改daemon配置文件/etc/docker/daemon.json来使用加速器
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://vym227em.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
这里的加速器是我注册的阿里云镜像仓库匹配的加速器，可以用。也可以换成自己的。

### 切换到docker-compose.yml所在目录
```
cd docker-lnmp
```
如果获取项目后改了docker-lnmp目录的名字，这里就是切换到改了名字的目录。

### 启动容器
```
docker-compose up -d
```

更多指令参考
[docker命令](http://www.blockchant.cn/tutorials/docker/43/docker-ming-ling)
[docker-compose命令](http://www.blockchant.cn/tutorials/docker/95/compose-ming-ling)

# 部署过程中的坑

#### mysql容器一直出入restarting的状态。

这种情况一般出现在一次部署不成功，又重新启动容器的情况。

启动容器的时候本机下/data目录（挂载到mysql容器中）必须是空的。否则会报错。
如果docker-compose.yml文件中自定义了顶级volumes，那么必须把此volume删除，然后再启动容器。否则也会报错。
删除volume操作详见docker指令。

#### mysql连接拒绝问题
这里的连接问题不是常规问题，与数据卷的挂载有关，详情参考[无法连接到mysql数据库：访问被拒绝 ](https://github.com/docker-library/mysql/issues/51)


# 一些使用技巧

备份好的sql文件www_blockchant_cn_080908.sql，在mysql容器里www_blockchant_cn数据库中还原步骤

1、备份的sql文件放在本机mysql/data目录下

2、进入mysql容器
```
docker exec -it mysql bash
```

3、进入data对应的/var/lib/mysql目录（即sql文件所在位置）
```
cd /var/lib/mysql
```

4、还原
```
mysql -uroot -p www_blockchant_cn < www_blockchant_cn_080908.sql
```

5、登录mysql查看
```
mysql -uroot -p

mysql> use www_blockchant_cn;

mysql> show tables;
```
