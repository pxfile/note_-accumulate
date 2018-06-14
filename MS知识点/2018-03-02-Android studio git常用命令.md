---
layout:     post
title:      Android studio git常用命令
subtitle:   git常用命令
date:       2018-03-02
author:     pxf
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - git常用命令
    - Android
---
# Android studio git常用命令

## git 不能pull或者push，报“Git Push Error - Could not resolve host name”
git remote set-url origin git@xxx:xxx(远程URL) 

## MAC 运行gradle报permission denied错误   命令行中输入：./gradlew xxxx  

出现下错误：

**zsh: permission denied: ./gradlew**   解决方式：

先输入：**chmod +x gradlew**，再输入你的命令行./gradlew xxxx 即可

## 将本地分支推到远程分支
git push origin search(本地分支):search/6.1.0_search(远程分支)

## 将远程分支删除
git push origin :search/6.1.0_search(远程分支)

## 修改组件项目打最新aar包
./gradlew clean upload

## gradlew打某环境的包
./gradlew assembleGmtest