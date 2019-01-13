---
title: linux常用工具的用法模板总结
key: 20181226
tags: linux
---

在Linux环境下开发，会用到各种各样的工具。使用时经常忘记用法，有的工具help写的比较复杂，去查Google又不能一下子找到自己想要的命令，所以在这里这里总结了一些平时开发过程中用的比较多的命令模板，便于自己需要时查阅。

<!--more-->

### curl  

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



### awk

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
ffmpeg -i inout.mp4 -vf delogo=x=10:y=10:w=100:h=70:show=1 output.mp4
```

