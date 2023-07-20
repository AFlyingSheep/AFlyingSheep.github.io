---
title: Respeaker 4-mic录制与播放
date: 2023-3-1 21:55:06
tags:
- [Raspberry Pi]
categories: 
- [Raspberry Pi]
mathjax: true
description: 树莓派使用Respeaker 4-mic的录制(arecord)与播放(aplay)功能。
top_img: 
cover: /image/covers/speaker.jpg
---

# 调节音量

```shell
$ alsamixer

# 使用F6选中声卡(JieLi BR17)并点回车,使用方向键↑↓调节声音大小，按ESC退出
```

# 录制

```shell
# 查看设备
$ arecord -l

# 找到BR17，查看它是哪个card。card会对应好几个device(但是BR17只有一个device)，记住所需要的device
# 录制
$ arecord --device=plughw:1,0 -f s32_LE -r 16000 -c 8 filename.wav
# 其中，--device=plughw代表使用插入的设备，后面对应的1,0代表card == 1, device == 0，所以需要根据上面的情况进行更改
# -f代表保存的参数，使用s32_LE
# -r代表频率，-c代表通道数
# filename可以改为你所需要的文件名

# 点击回车开始录制，点击ctrl+c暂停录制
```

# 播放

```shell
# 查看设备
$ aplay -l

# 找到BR17，查看它是哪个card。card会对应好几个device(但是BR17只有一个device)，记住所需要的device
# 播放
$ aplay --device=plughw:1,0 -f s32_LE filename.wav
# 其中，--device=plughw代表使用插入的设备，后面对应的1,0代表card == 1, device == 0，所以需要根据上面的情况进行更改
# -f代表播放的参数，使用s32_LE（s32_LE是一种封装，调用这个格式会自动帮你更改参数，比如波特率、大端小端等）
# filename是录制的文件名

# 点击回车开始播放，点击ctrl+c暂停播放
```

