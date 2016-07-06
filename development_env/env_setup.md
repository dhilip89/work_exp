# Development setup



##1.node
	下载二进制文件， 解压后放到目录 ，并指定路径可以直接使用

##2.python
###install python2 on mac
>	with ssl:
	
>Before starting the installation of Python I edited the Modules/Setup.dist uncommenting the following lines (not sure if it was necessary, but was suggested on several threads I came across):
>
```	
vi Modules/Setup.dist
SSL=/my/path/for/openssl/lib
_ssl _ssl.c \
    -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
    -L$(SSL)/lib -lssl -lcrypto
```   

./configure —prefix=‘xxx’ --enable-shared
		
```
./pip install pycrypto
./pip install pyDes
./pip install redis
./pip install hiredis
./pip install MySQL-python
./pip install elasticsearch
```

###install python3.5.1 on mac

```
export PYTHONHOME=/Users/rick/local/python_3
export PYTHONPATH=$PYTHONHOME:$PYTHONHOME/lib/python_3:$PYTHONHOME/lib:$PYTHONHOME/lib/python_3/site-packages
export PATH=$PATH:$PYTHONHOME:$PYTHONPATH

./configure --prefix=/Users/rick/local/python3 --enable-shared --disable-ipv6
```

> make 出现错误：
> 
Include/pyport.h:243: error: #error "This platform's pyconfig.h needs to define PY_FORMAT_LONG_LONG"
修改pyconfig.h: 找到#undef PY_FORMAT_LONG_LONG 处加：#define PY_FORMAT_LONG_LONG "ll"

```
install settools
~/local/python_3/bin/python3 ~/src/setuptools-23.0.0/setup.py build

~/local/python_3/bin/python3 ~/src/setuptools-23.0.0/setup.py install

install pip
~/local/python_3/bin/python3 ~/src/pip-8.1.2/setup.py build

~/local/python_3/bin/python3 ~/src/pip-8.1.2/setup.py install
```

	
##3.openssl for mac
	./Configure darwin64-x86_64-cc shared --openssldir=/usr/local/ssl/macos-x86_64
	make depend
	sudo make install
