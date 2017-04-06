# Os常用知识

### 在MAC终端下打开Finder
```
open .

指定打开文件的程序: -a

open -a Safari http://www.baidu.com
调用TextEdit编辑文件: -e

open -e main.c
打开finder,并且以某个文件为焦点

open -R main.c

```

### Mac电脑如何截图

```
截取全屏：快捷键（Shift＋Command＋3）
直接按“Shift＋Command＋3“快捷键组合，即可截取电脑全屏，图片自动保存在桌面。


截图窗口：快捷键（Shift+Command+4，然后按空格键）
▲直接按“Shift＋Command＋4“快捷键组合，会出现十字架的坐标图标；
▲将此坐标图标移动到需要截取的窗口上，然后按空格键；
▲按空格键后，会出现一个照相机的图标，单击鼠标，图片会自动保存在桌面。

截取任意窗口：快捷键（Shift＋Command＋4）
▲直接按“Shift＋Command＋4“快捷键组合，出现十字架的坐标图标；
▲拖动坐标图标，选取任意区域后释放鼠标，图片会自动保存在桌面。

```


### 在Terminal中查看当前目录下所有文件（包含文件夹）大小：
```
du -hs *
或者：

du -shc *
第二个命令能在最后显示一个Total大小，即当前目录的总大小。
```

