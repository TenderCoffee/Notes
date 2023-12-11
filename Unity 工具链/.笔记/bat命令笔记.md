批处理文件是一个“.bat”结尾的文本文件，这个文件的每一行都是一条DOS命令。



# 0.基本语法

```
@echo off  //表示不显示所有的命令行信息，包括此句
注释: rem xxx   或者 :: XXX
pause  //暂停,按任意键继续
echo xxx  //ToString输出
set num=123 //设置变量
xxx /?  //帮助信息 如XCOPY /? 或者 Help XCOPY
md TMP  //创建文件夹(Make Directory)
cd ../ //跳转到上一级目录
cd /d C:/Test //跳转,/d绝对路径

%cd%   //当前路径：
%~dp0  //当前bat文件路径：
%date%  //当前日期
%time%  //当前时间
```



# 1.复制/移动/删除 文件



## 复制

```
复制A文件夹 内所有txt格式文件到 B文件夹
XCOPY .\A\*.txt .\B\ /q /e /r /S /y

也可以使用for循环
for /r %%a in (.\A\*.txt) do copy %%a .\B

可以使用 XCOPY /? 查看帮助信息
  /Q           复制时不显示文件名。
  /E           复制目录和子目录，包括空目录。
  /S           复制目录和子目录，不包括空目录。
  /R           覆盖只读文件。
  /Y           取消提示以确认要覆盖

如果是整个文件夹一起复制
XCOPY .\A .\B\ /q /e /r /S /y
```



## 移动

```
从A移动B文件夹,并限定".txt"文件

move /Y .\A\*.txt .\B\
```



## 删除

```
删除A文件夹内所有的txt

del /S .\A\*.txt
删除文件夹A

rmdir /S .\A\
/S 显示删除记录

/Q 安静模式,不询问确认  //一般不开防止误删
```



# 2.For循环/目录遍历

```
For循环
格式:  FOR  %variable IN (set) DO command [command-parameters]
注意:%是cmd中运行,%%是bat中运行
```



## 1.数字遍历 /L 

```
in (起点,步长,终点)

0~5遍历

for /l %%i in (0,1,5) do echo %%i
```



## 2.目录遍历 /R

```
输出当前文件夹内所有文件

for /r %%i in (*) do echo %%i
```

```
输出B文件夹所有的路径

for /r .\B %%i in (*) do echo %%i

将结果写入到a.txt

for /r .\B %%i in (*) do echo %%i>>a.txt
```

```
利用Set保存For循环的信息

rem 设置 变量延迟扩展
if "%OS%"=="Windows_NT" setlocal enabledelayedexpansion
set allFile=
for /r .\B %%i in (*) do (
    set "var=%%i"
    set "allFile=!allFile! !var!"
)
echo %allFile%

如果只想获得相对路径而不是当前路径, 则可以使用字符串的替换,将前缀替换为空

set "bastPath=%~dp0"
set "allFile=!allFile! !var:%bastPath%=!"
```



## 3.字符串处理

### 截取

```
截取  str:~[起点][长度]

::截取前两个字符
echo %str:~0,2%
::截取至倒数第二个
echo %str:~0,-2%

```

### 替换

```
将str中x替换为y ,如"xyzx"替换后为"yyzy"

set "str=%str:x=y%"
相加

字符串aa+bb

set "aa=%aa%%bb%"
```



## 4.运行/读写



### 运行

```
start /d "E:\笔记" unity.txt  //打开|运行
call c:\code\run.bat   //使用新窗口,运行

::用ie 打开网页
start iexplore http://www.baidu.com

::使用记事本打开
start notepad.exe "E:\笔记" unity.txt

::使用Notepad++打开
start "C:\Program Files\Notepad++" notepad++.exe "E:\笔记\unity.txt"
```



### 写入文件

```
echo abc>.//1.txt   ::覆写
echo abc>>.//1.txt	 ::追加写入

```

### 打印

```
type 1.txt

逐行读取
@echo off
set target=
setlocal enabledelayedexpansion
for /f %%i in (1.txt)  do (
set "var=%%i"
set "target=!target! !var!"
)
echo %target%
pause
```



## 5.if 条件判断 & goto

```
if 条件 ( ... )  else ( ... )

@echo off
if %errorlevel% == 0 (
goto yes
) else (
goto no
)

:no
	echo 失败处理
	pause
:yes
	echo 成功
	pause


```



## 6.管理员权限启动

```
%1 start "" mshta vbscript:CreateObject("Shell.Application").ShellExecute("cmd.exe","/c %~s0 ::","","runas",1)(window.close)&&exit
echo 管理员模式启动成功
或者

@echo off
net.exe session 1>NUL 2>NUL && (
    goto gotAdmin
) || (
    goto UACPrompt
)

:UACPrompt
    echo Set UAC = CreateObject^("Shell.Application"^) > "%temp%\getadmin.vbs"
    echo UAC.ShellExecute "%~s0", "", "", "runas", 1 >> "%temp%\getadmin.vbs"
    "%temp%\getadmin.vbs"
    exit /B

:gotAdmin
    if exist "%temp%\getadmin.vbs" ( del "%temp%\getadmin.vbs" )

:begin

echo 管理模式启动成功
```



## 7.等待时间 & 多任务等待

### 等待时间

```
TIMEOUT /T 10 /NOBREAK ::等待10秒
TIMEOUT /T 10  ::等待10s,按任意键跳过
```



### 多任务 等待

```
start /wait 1.bat
start /wait 2.bat
echo 任务完成
```



## 8.git

### 常规命令

```
//查看当前状态
git status

//推送 force
git push origin develop2 --force

//检出
git checkout -b develop origin/develop

//拉取
git pull
git pull --progress "origin" //--progress 用于看日志

//获取
git fetch --progress "origin"

//查看远端分支
git branch -r
```



## 9.其他



### 中文乱码解决: 

```
编码选择为 GB2312

notepad++: 编码/编码字符集/中文/GB2312
```



### 设置环境变量:

```
setx '变量名' 'XXX' :: 加到用户变量
setx '变量名' 'XXX' \m ::加到系统变量
```




参考:

官方文档:https://learn.microsoft.com/zh-cn/windows-server/administration/windows-commands/windows-commands

知乎-常用批命令:https://zhuanlan.zhihu.com/p/446337414 作者：机智的小草yns https://www.bilibili.com/read/cv22973097/ 出处：bilibili