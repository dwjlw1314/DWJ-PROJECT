<p align="center">ping命令加入时间戳</p>

将以下代码复制到记事本中，格式后缀更改为vbs，实验中存放位置为C盘
```vbs
Dim args, flag, unsuccOut
args=""
otherout=""
flag=0

If WScript.Arguments.count = 0 Then
WScript.Echo "Usage: cscript tping.vbs [-t] [-a] [-n count] [-l size] [-f] [-i TTL] [-v TOS]"
WScript.Echo "                         [-s count] [[-j host-list] | [-k host-list]]"
WScript.Echo "                         [-r count] [-w timeout] destination-list"
wscript.quit
End if

For i=0 to WScript.Arguments.count - 1
args=args & " " & WScript.Arguments(i)
Next

Set shell = WScript.CreateObject("WScript.Shell")
Set re=New RegExp
re.Pattern="^Reply|^Request|^来自|^请求"

Set myping=shell.Exec("ping" & args)

while Not myping.StdOut.AtEndOfStream
   strLine=myping.StdOut.ReadLine()
'WScript.Echo  "原数据" & chr(9) & strLine
   r=re.Test(strLine)
   If r Then
WScript.Echo date & " "& time & chr(9) & strLine
flag=1
   Else
unsuccOut=unsuccOut & strLine
   End if
Wend

if flag = 0 then
WScript.Echo unsuccOut
end if
```

打开cmd，运行以下命令，即将具有时间戳的ping记录保存到了C盘的根目录，记录文件为status.log
> cscript C:\ping.vbs 58.251.82.205 -t -l 1024 >> C:\status.log
