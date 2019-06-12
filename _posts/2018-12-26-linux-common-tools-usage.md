---
title: linux常用工具的用法模板总结
key: 20181226
tags: linux
---

在Linux环境下开发，会用到各种各样的工具。使用时经常忘记用法，有的工具help写的比较复杂，去查Google又不能一下子找到自己想要的命令，所以在这里这里总结了一些平时开发过程中用的比较多的命令模板，便于自己需要时查阅。

<!--more-->

### curl  

### screen

```bash
# 新建一个以<name>命名的窗口
screen -S <name>
# 分离<name>窗口,需要另开一个terminal
screen -d <name>
# 列出所有现存的screen窗口
screen -ls
# 回到某个screen窗口
screen -r <name>
```



### find

### tar

```bash
# 解压缩.tar.gz文件，解压后生成target_dir/input文件夹
tar -C target_dir zxfv input.tar.gz
# 打包成tar文件
tar -cf archive.tar foo bar
# 打包成gzip文件
tar -zcf archive.tar.gz foo bar
```

### ps

```bash
# 查看指定进程内存消耗情况
ps -eo size,pid,user,command --sort -size | grep http_ad| awk '{ hr=$1/1024 ; printf("%13.2f Mb ",hr) } { for ( x=4 ; x<=NF ; x++ ) { printf("%s ",$x) } print "" }' |cut -d "" -f2 | cut -d "-" -f1
# 查看服务频响
tail -f *http_ad*log | grep -P "CONTROL_SERVER" | awk -F'\t' '{split($0,a,/ ts=/); split(a[2],b,/ /);ts=b[1]; num+=1; tts+=ts; if(num==100){print tts,num,tts/num; num=0;tts=0}}'
# 根据日志计算QPS
tail -f *http_ad*.log | grep "CONTROL_SERVER" | awk '{print $3; system("")}' | uniq -c
```

### awk

### netstat

```bash
# 查看端口占用情况
netstat -anp | egrep ":88[0-9]{2} "
```



### ffmpeg  

`ffmpeg`是一款强大的视频处理工具。常用语法如下：

```bash
# 视频转化为音频
ffmpeg -i input.mp4 output.mp3
# 从视频中截取出图片, -ss表示开始时间， fps=1表示每隔一秒截取一张，-vframe表示截取3帧，输出为out1.png, out2.png, out3.png
ffmpeg -i input.mp4 -ss 00:00:01 -vf fps=1 -vframes 3 out%d.png
# 将多张图片转化为gif, -framerate 1/2表示生成的gif每张图持续2秒，in%d.png表示使用类似in1.png/in2.png这种命名格式的图片，将out.gif改为out.mp4可生成视频
ffmpeg -framerate 1/2 -i in%d.png out.gif
# 去除视频logo，加上show=1，生成的视频在logo处将有一个矩形框，用于调试时确定位置
ffmpeg -i input.mp4 -vf delogo=x=10:y=10:w=100:h=70:show=1 output.mp4
# 压缩视频大小，根据码率
ffmpeg -i input.mp4 -b:v 700k output.mp4
# 截取视频中指定的时间段，如从10秒开始截取20秒视频
ffmpeg -i input.mp4 -ss 00:00:10 -t 00:00:20 output.mp4
```

### go跨平台编译

```bash
# 在linux中编译，mac平台和windos 64平台使用
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build main.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go
# 在mac中编译，linux和windows 64平台使用
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go
# 在windows中编译，在mac和linux 64中使用
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build main.go

SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build main.go
```

### logrotate使用

```
# 在/etc/logrotate.d/中新建配置
/opt/logs/* {
  rotate 3
  copytruncate
  hourly
  nocompress
  missingok
  notifempty
  dateext
  dateformat .%Y-%m-%d_%H
}
```

