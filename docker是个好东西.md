删除所有镜像
docker rm $(docker ps -a)
addr查看ip端口
docker exec -it ID/NAME 
查看镜像版本
cat /etc/issue
cat /etc/os-release
端口映射
-p 9999:8060
停止
docker stop  ID
删除
docker rm  ID
开始
docker start  ID
本机端口：docker端口 /bin/bash
docker run -it -p
进入容器 
docker attach ID
重启ssh
/etc/init.d/ssh restart

systemctl stop docker.socket
systemctl stop docker
systemctl start docker
systemctl enable docker
systemctl restart docker





