docker learning memory

切换root   su - root  


开机自启： sudo systemctl enable docker  


## images
拉images： docker pull nginx  
看images： docker images  

## Container
<img width="167" alt="Screenshot 2024-06-18 at 23 52 24" src="https://github.com/goushijianglin/docker/assets/83333650/b9219833-dcba-45c1-8bd4-a72d83bc4ef7">  

查看已经停止的容器 docker ps -a  
运行container
```sh
docker run -d --name mynginx -p 80:80 nginx
```  
    -d      Run container in background and print container ID  
    --name  Assign a name to the container  
    -p      Publish a container's port(s) to the host  
进入 docker exec -it mynginx /bin/bash
