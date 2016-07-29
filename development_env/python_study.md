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