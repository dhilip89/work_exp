## docker创建自定义网络
```
docker network create -o "com.docker.network.bridge.name"="docker1" --subnet 172.20.0.0/16 docker1


指令解析：

创建名为docker1的bridge
--subnet 设置子网段
-o "com.docker.network.bridge.name"="docker1" 给这个bridge起个名字，否则宿主机中看到的网桥名是一坨乱码。
然后就是见证奇迹的时刻：

docker run --rm --ip 172.20.100.100 --net docker1 ubuntu ifconfig

```