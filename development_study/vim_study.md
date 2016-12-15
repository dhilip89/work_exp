## vim 技巧记录

### 批量替换：
vim 批量替换

XXX是需要替换的字符串,YYY是替换后的字符串。

只对当前行进行替换：

Example
1
:s/XXX/YYY/g
,如果需要进行全部替换：

Example
1
:%s/XXX/YYY/g
如果需要对指定部分进行替换,可以用V进入visual模式,再进行

Example
1
:s/XXX/YYY/g
或者可以指定行数对指定范围进行替换:

Example
1
:10,31s/XXX/YYY/g
若需要显示行号，在vim下

Example
1
:set nu
取消显示行号：

Example
1
:set nonu

	%s/\/Users\/rick\/local\/sphinx\/sphinx_github/\/service\/sphinx/g 
	
	vi/vim 中可以使用 ：s 命令来替换字符串§以前只会使用一种格式来全文替换，今天发现该命令有很多种写法（vi 真是强大啊飕还有很多需要学习），记录几种在此，方便以后查询
	
	：s/vivian/sky/ 替换当前行第一个 vivian 为 sky

	：s/vivian/sky/g 替换当前行所有 vivian 为 sky
	
	：n，$s/vivian/sky/ 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky
	
	：n，$s/vivian/sky/g 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky, n 为数字，若 n 为 .，表示从当前行开始到最后一行

	：%s/vivian/sky/（等同于 ：g/vivian/s//sky/） 替换每一行的第一个 vivian 为 sky

	：%s/vivian/sky/g（等同于 ：g/vivian/s//sky/g） 替换每一行中所有 vivian 为 sky

可以使用 # 作为分隔符，此时中间出现的 / 不会作为分隔符
	
	：s#vivian/#sky/# 替换当前行第一个 vivian/ 为 sky/

	：%s /oradata/apras/ /user01/apras1 （使用 来 替换 / ）： /oradata/apras/替换成/user01/apras1/

	：s/vivian/sky/ 替换当前
	
## Vim 使用 regex 將 "," 取代成換行
```
:%s/,/\r,/g
```