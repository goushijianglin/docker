docker learning memory

切换root   su - root  


开机自启： sudo systemctl enable docker  


## images
拉images： docker pull nginx  
看images： docker images  

## Container
<img width="167" alt="Screenshot 2024-06-18 at 23 52 24" src="https://github.com/goushijianglin/docker/assets/83333650/b9219833-dcba-45c1-8bd4-a72d83bc4ef7">  

查看已经停止的容器 
```sh
docker ps -a
```
删除所有Container
```sh
docker rm -f $(docker ps -aq)
```  
运行container
```sh
docker run -d --name mynginx -p 80:80 nginx
```  
    -d      Run container in background and print container ID  
    --name  Assign a name to the container  
    -p      Publish a container's port(s) to the host  
进入docker内部
```sh
docker exec -it mynginx /bin/bash  
```

## Bind mount a volume 目录挂载
```sh
docker run -d --name mynginx -p 80:80 -v /app/nghtml:/usr/share/nginx/html --name app nginx
```
## 卷映射
```sh
docker run -d -p 99:80 -v /app/nhtml:/usr/share/nginx/html -v ngconf:/etc/nginx  --name app nginx
```
映射目录  cd /var/lib/docker/volumes/ 
```sh
cd /var/lib/docker/volumes/ngconf
```
## 自定义网络
第一个container
```sh
docker run -d -p 99:80 --name app1 nginx
```
第二个container
```sh
docker run -d -p 90:80 --name app2 nginx
```
①直接拿本机端口连接  
<img width="949" alt="Screenshot 2024-06-27 at 13 14 12" src="https://github.com/goushijianglin/docker/assets/83333650/6bb8a0df-d8b2-4326-8cac-1ec287fb848c">  
这样访问可以 但是很奇怪 是在docker app1 container 到 本机(lunix) 在到docker app2 container  
②docker内部网络连接  
所以 docker给了一个docker0的默认网络 每个container都有一个内部ip，可以用  
```sh
docker container inspect app1
```
查看详细信息  
<img width="929" alt="Screenshot 2024-06-27 at 13 26 58" src="https://github.com/goushijianglin/docker/assets/83333650/adeb2399-9d82-4821-a4d2-d975e8613c19">
"IPAddress":就是docker内部ip地址  
<img width="552" alt="Screenshot 2024-06-27 at 13 41 53" src="https://github.com/goushijianglin/docker/assets/83333650/f3d25b40-222b-43e4-8b38-79784d4467ac">  
但是这样还是有问题 ip有可能会变 查IP还很麻烦 所以  
③创建自定义网络 用域名（就是容器名）直接访问  
创建自定义网络  
```sh
docker network create mynet
```
第一个container
```sh
docker run -d -p 99:80 --name app1 --network mynet nginx
```
第二个container
```sh
docker run -d -p 90:80 --name app2 --network mynet nginx
```
这样可以直接使用域名访问 
```sh
curl http://app2:80
```
<img width="735" alt="Screenshot 2024-06-27 at 14 08 37" src="https://github.com/goushijianglin/docker/assets/83333650/85cadecd-4ce5-441c-b301-ebe2efe81657">
  
## Docker Compose  

不使用compose：  

mysql:  
```sh
docker run -d -p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
-e MYSQL_DATABASE=wordpress \
-v mysql-data:/var/lib/mysql \
-v /app/mysql:/etc/mysql/conf.d \
--restart always --name mysql \
--network blog \
```
wordpress:一个blog的服务  
```sh
docker run -d -p 8080:80 \
-e WORDPRESS_DB_HOST=mysql \
-e WORDPRESS_DB_USER=root \
-e WORDPRESS_DB_PASSWORD=123456 \
-e WORDPRESS_DB_NAME=wordpress \
-v wordpress:/var/www/html \
--restart always --name wordpress-app \
--network blog \
```

使用compose：  

```sh
name: myblog
services:
  mysql:
    container_name: mysql
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - MYSQL_DATABASE=wordpress
    volumes:
      - mysql-data:/var/lib/mysql
      - /app/myconf:/etc/mysql/conf.d
    restart: always
    networks:
      - blog
  wordpress:
    image: wordpress
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: root 
      WORDPRESS_DB_PASSWORD: 123456
      WORDPRESS_DB_NAME: wordpress

```
