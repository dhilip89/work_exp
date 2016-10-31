##Linux Command

### 查看系统用户及用户组
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

### Linux Shell 1>/dev/null 2>&1 含义
>`/dev/null`: 代表空设备文件

> `>`: 代表重定向到哪里，例如：echo "123" > /home/123.txt

> `1`: 表示stdout标准输出，系统默认值是1，所以">/dev/null"等同于"1>/dev/null"

> `2`: 表示stderr标准错误

> `&`: 表示等同于的意思，2>&1，表示2的输出重定向等同于1

`1 > /dev/null 2>&1` 语句含义：

	1 > /dev/null ： 首先表示标准输出重定向到空设备文件，也就是不输出任何信息到终端，说白了就是不显示任何信息。

	2>&1 ：接着，标准错误输出重定向（等同于）标准输出，因为之前标准输出已经重定向到了空设备文件，所以标准错误输出也重定向到空设备文件。

### shell压缩解压命令：gzip、zip、tar
  gzip 和 gunzip
   
	压缩:
	gzip  filename 文件即会被压缩，并被保存为 filename.gz
	
	解压缩:
	gunzip filename.gz filename.gz 会被删除，而继之以 filename
	可以通过命令man gzip 和man gunzip获得命令的详细说明.
	
	
zip 和 unzip

	要使用 zip 来压缩文件，在 shell 提示下键入下面的命令：
	zip -r filename.zip filesdir 
	在这个例子里，filename.zip 代表你创建的文件，filesdir 代表你想放置新 zip 文件的目录。-r 选项指定你想递归地（recursively）包括所有包括在 filesdir 目录中的文件。
	
	要解压缩 zip 文件的内容，键入以下命令：
	unzip filename.zip
	
	你可以使用 zip 命令同时处理多个文件和目录，方法是将它们逐一列出，并用空格间隔：

	zip -r filename.zip file1 file2 file3 /usr/work/school

	上面的命令把 file1、file2、 file3、以及 /usr/work/school 目录的内容（假设这个目录存在）压缩起来，然后放入 filename.zip 文件中。

tar

	tar 文件是几个文件和（或）目录在一个文件中的集合。这是创建备份和归档的佳径。

	tar 使用的选项有：

	-c — 创建一个新归档。

	-f — 当与 -c 选项一起使用时，创建的 tar 文件使用该选项指定的文件名；当与 -x 选项一起使用时，则解除该选项指定的归档。

	-t — 显示包括在 tar 文件中的文件列表。

	-v — 显示文件的归档进度。

	-x — 从归档中抽取文件。

	-z — 使用 gzip 来压缩 tar 文件。

	-j — 使用 bzip2 来压缩 tar 文件。

	要创建一个 tar 文件，键入：

	tar -cvf filename.tar directory/file

	在以上的例子中，filename.tar 代表你创建的文件，directory/file 代表你想放入归档文件内的文件和目录。

	你可以使用 tar 命令同时处理多个文件和目录，方法是将它们逐一列出，并用空格间隔：

	tar -cvf filename.tar /home/mine/work /home/mine/school

	上面的命令把 /home/mine 目录下的 work 和 school 子目录内的所有文件都放入当前目录中一个叫做 filename.tar 的新文件里

	以下是一个解压的例子

	tar -zvxf  filename.tar.gz  
	
### 用shell切分文件--split
有个文件要处理，因为很大，所以想把它切成若干份，每份N行，以便并行处理。怎么搞呢？查了下强大的shell，果然有现成的工具--split。
下面记录下基本用法：

split [-bl] file [prefix]  
参数说明：
> -b, --bytes=SIZE：对file进行切分，每个小文件大小为SIZE。可以指定单位b,k,m。
> 
> -l, --lines=NUMBER：对file进行切分，每个文件有NUMBER行。

> prefix：分割后产生的文件名前缀

示例：

1.假设要切分的文件为test.2012-08-16_17，大小1.2M，12081行。
1)

	split -l 5000 test.2012-08-16_17  
	
	生成xaa，xab，xac三个文件。
	wc -l 看到三个文件行数如下：
	5000 xaa
	5000 xab
	2081 xac
	12081 总计
2) 
	split -b 600k test.2012-08-16_17  
	
	生成xaa，xab两个文件
	ls -lh 看到 两个文件大小如下：
	600K xaa
	554K xab
	
3)
	split -b 500k test.2012-08-16_17 example 
	
	得到三个文件，文件名的前缀都是example
	ls -lh 看到文件信息如下：
	500K exampleaa
	500K exampleab
	154K exampleac
	
### Linux中find常见用法示例
ind   path   -option   [   -print ]   [ -exec   -ok   command ]   {} \;
find命令的参数；

> pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。

> -print： find命令将匹配的文件输出到标准输出。

> -exec： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' { } \;，注意{ }和\；之间的空格。

> -ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

> -print 将查找到的文件输出到标准输出

> -exec   command   {} \;      —–将查到的文件执行command操作,{} 和 \;之间有空格

> -ok 和-exec相同，只不过在操作前要询用户
例：find . -name .svn | xargs rm -rf

### 查看Linux内核版本
```
cat /etc/issue  //CentOS release 6.4 (Final)
uname -r //2.6.32-358.6.1.el6.i686
``` 


## cat uniq

```
cat words.txt mf.keyword.txt | sort  | uniq -c
```

反顺序

```
cat words.txt mf.keyword.txt | sort  -r | uniq -c
```