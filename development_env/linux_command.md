##Linux Command

###查看系统用户及用户组
```
俺的centos vps上面不知道添加了多少个账户，今天想清理一下，但是以前还未查看过linux用户列表，google了一下，找到方便的放：
一般情况下是

cat /etc/passwd 可以查看所有用户的列表
w 可以查看当前活跃的用户列表
cat /etc/group 查看用户组

但是这样出来的结果一大堆，看起来嘿负责，于是继续google
找到个简明的layout命令

cat /etc/passwd|grep -v nologin|grep -v halt|grep -v shutdown|awk -F":" '{ print $1"|"$3"|"$4 }'|more
```