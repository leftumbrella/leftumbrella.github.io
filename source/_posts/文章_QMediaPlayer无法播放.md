---
title: Qt 使用 QMediaPlayer 生成 Release 版本程序时无法正常播放的问题
date: 2022-1-7 15:30:25
tags: [C++,Qt]
categories: 问题拾遗
---

> 简述 : *Qt* 使用 *QMediaPlayer* 生成 *Release* 版本程序时无法正常播放
> 时间 : 2022年1月17日

开发 *Qt* 桌面应用程序时，遇到了播放 *.mp3* 文件的需求，使用 *Qt multimedia* 模块中的 *QMediaPlayer* 实现， *debug* 模式下直接通过 *Visual Studio* 启动， *.mp3* 文件播放正常无误:
```
// main.cpp
#include <QMediaPlayer>
QMediaPlayer* G_Player_ptr = new QMediaPlayer;
int main(){
    // Some Codes ...
}
```
<!--more-->
```
//FunctionCore.h
extern QMediaPlayer* G_Player_ptr;
static void QPlaySound(int type) {
	G_Player_ptr->setMedia(QUrl::fromLocalFile("path/mp3name.mp3"));
	G_Player_ptr->play();
}
```
但是，在使用 *Qt windeployqt.exe* 对生成版本添加相关依赖后，直接点击生成目录下的 *.exe* 文件却无法播放 *.mp3* 文件，经过排查，此问题是由此行代码所导致——```QMediaPlayer* G_Player_ptr = new QMediaPlayer;```，直接将```G_Player_ptr```定义为全局变量(指针)，并为其赋初始值为```new QMediaPlayer```，这将导致 *Qt multimedia* 模块在 ```main``` 函数前被使用，但此时程序并未加载该模块，从而导致程序启动时无法正常调用 *Qt5Multimedia.dll* 。因此，将 ```main.cpp``` 更改为如下代码时，能够正常播放 *.mp3* 文件：
```
// main.cpp
#include <QMediaPlayer>
QMediaPlayer* G_Player_ptr = nullptr;
int main(){
    G_Player_ptr= new QMediaPlayer;

    // Other Codes ...
}
```

但是为什么改动前直接通过 *Visual Studio* 启动调试时播放正常呢？可能是由于 *Visual Studio* 直接启动调试时会在所有步骤前先进行模块导入操作 ***(未证实)***