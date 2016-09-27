```
./pip install virtualenv
sudo apt-get install python-virtualenv
./virtualenv /Users/rick/work_space/python_2
./pip install pycrypto
./pip install pyDes
./pip install redis
./pip install hiredis
./pip install MySQL-python
./pip install elasticsearch
./pip install numpy
./pip install pandas
./pip install psycopg2
```


```
Q: Error: pg_config executable not found.
A: for centOs: yum install postgresql-devel

Q: Error: You need to install postgresql-server-dev-X.Y for building a server-side extension or libpq-dev for building a client-side application.
A: sudo apt-get install libpq-dev
sudo apt-get install postgresql postgresql-contrib
```


print "%.2f" % a 

datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

```
yesterday = datetime.date.fromordinal(datetime.date.today().toordinal()-1)

time_from_str =  '%s 00:00:00' % yesterday.strftime("%Y-%m-%d")

time_from = int(time.mktime(time.strptime(time_from_str, "%Y-%m-%d %H:%M:%S")))

time_to_str = '%s 23:59:59' % yesterday.strftime("%Y-%m-%d")
	
time_to = int(time.mktime(time.strptime(time_to_str, "%Y-%m-%d %H:%M:%S")))

3.时间戳转换为指定格式日期:
	方法一:
		利用localtime()转换为时间数组,然后格式化为需要的格式,如
		timeStamp = 1381419600
		timeArray = time.localtime(timeStamp)
		otherStyleTime = time.strftime("%Y-%m-%d %H:%M:%S", timeArray)
		otherStyletime == "2013-10-10 23:40:00"

	方法二:
		import datetime
		timeStamp = 1381419600
		dateArray = datetime.datetime.utcfromtimestamp(timeStamp)
		otherStyleTime = dateArray.strftime("%Y-%m-%d %H:%M:%S")
		otherStyletime == "2013-10-10 23:40:00"
		注意：使用此方法时必须先设置好时区，否则有时差
```


python 字典（dict）的特点就是无序的，按照键（key）来提取相应值（value），如果我们需要字典按值排序的话，那可以用下面的方法来进行：

1 下面的是按照value的值从大到小的顺序来排序。

```
dic = {'a':31, 'bc':5, 'c':3, 'asd':4, 'aa':74, 'd':0}
dict= sorted(dic.iteritems(), key=lambda d:d[1], reverse = True)
print dict
```

输出的结果：

```
[('aa', 74), ('a', 31), ('bc', 5), ('asd', 4), ('c', 3), ('d', 0)]
```

下面我们分解下代码
print dic.iteritems() 得到[(键，值)]的列表。
然后用sorted方法，通过key这个参数，指定排序是按照value，也就是第一个元素d[1的值来排序。reverse = True表示是需要翻转的，默认是从小到大，翻转的话，那就是从大到小。

2 对字典按键（key）排序：

```
dic = {'a':31, 'bc':5, 'c':3, 'asd':4, 'aa':74, 'd':0}

dict= sorted(dic.iteritems(), key=lambda d:d[0]) d[0]

表示字典的键
print dict
```

### Three Ways to Read A Text File Line by Line in Python
1.One easy way to read a text file and parse each line is to use the python statement “readlines” on a file object. 

```
## Open the file with read only permit
f = open('myTextFile.txt', "r")

## use readlines to read all lines in the file
## The variable "lines" is a list containing all lines
lines = f.readlines()

## close the file after reading the lines.
f.close()

```

2.Read a Text File Line by Line Using While Statement in Python

```
## Open the file with read only permit
f = open('myTextFile.txt')
## Read the first line 
line = f.readline()

## If the file is not empty keep reading line one at a time
## till the file is empty
while line:
    print line
    line = f.readline()
f.close()
```

3.Read a Text File Line by Line Using an Iterator in Python

```
f = open('myfile.txt')
for line in iter(f):
    print line
f.close()
```


##Final 使用技巧：
pycharm python 代码格式化： command + alt + l


### Add alternatives to '%matplotlib inline' in ipython

```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt


goog = pd.read_csv("/Users/rick/Downloads/goog.csv")
goog['Log_Ret'] = np.log(goog['Close'] / goog['Close'].shift(1))
goog['Volatility'] = pd.rolling_std(goog['Log_Ret'], window=252)*np.sqrt(252)

#for linux
%matplotlib inline goog[['Close', 'Volatility']].plot(subplots=True, color='blue', figsize=(8, 6))

#for mac
%matplotlib osx goog[['Close', 'Volatility']].plot(subplots=True, color='blue', figsize=(8, 6))


ts = pd.Series(np.random.randn(1000), index=pd.date_range('1/1/2000', periods=1000));
ts = ts.cumsum()
ts.plot()
```



### OS X 10.8.x, uWSGI 2.0.3 linking error

```
C_INCLUDE_PATH=/usr/local/include LIBRARY_PATH=/usr/local/lib pip install uwsgi


uwsgi --socket 0.0.0.0:8800 --protocol=http -w wsgi

uwsgi --http-socket :9090 --plugin python --wsgi-file app/index.py
```