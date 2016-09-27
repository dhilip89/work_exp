# CentOS设置服务开机启动的方法


CentOS设置服务开机启动的两种方法

1、利用 chkconfig 来配置启动级别
在CentOS或者RedHat其他系统下，如果是后面安装的服务，如httpd、mysqld、postfix等，安装后系统默认不会自动启动的。就算手动执行 /etc/init.d/mysqld start 启动了服务，只要服务器重启后，系统仍然不会自动启动服务。 在这个时候，我们就需要在安装后做个设置，让系统自动启动这些服务，避免不必要的损失和麻烦。 其实命令很简单的，使用chkconfig即可。
[天涯PHP博客]-[http://blog.phpha.com]
比如要将mysqld设置为开机自动启动：

1
chkconfig mysqld on
要取消掉某个服务自动启动，只需要将最后的参数 “on” 变更为 “off” 即可。
比如要取消 postfix 的自动启动：

1
chkconfig postfix off
值得注意的是，如果这个服务尚未被添加到 chkconfig 列表中，则现需要使用 –-add 参数将其添加进去：

1
chkconfig –-add postfix
如果要查询当前所有自动启动的服务，可以输入：

1
chkconfig -–list
如果只想看指定的服务，只需要在 “–-list” 之后加上服务名就好了，比如查看httpd服务是否为自动启动：

1
chkconfig –-list httpd
1
httpd 0:off 1:off 2:off 3:off 4:off 5:off 6:off
此时0~6均为off，则说明httpd服务不会在系统启动的时候自动启动。我们输入：

1
chkconfig httpd on
则此时为：

1
httpd 0:off 1:off 2:on 3:on 4:on 5:on 6:off
这个时候2~5都是on，就表明会自动启动了。

2、修改 /etc/rc.d/rc.local 这个文件：
例如将 apache、mysql、samba、svn 等这些服务的开机自启动问题一起搞定：

1
2
3
4
5
6
7
[天涯PHP博客]-[http://blog.phpha.com]
vi /etc/rc.d/rc.local
#添加以下命令
/usr/sbin/apachectl start
/etc/rc.d/init.d/mysqld start
/etc/rc.d/init.d/smb start
/usr/local/subversion/bin/svnserve -d
