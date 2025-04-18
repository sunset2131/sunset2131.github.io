---
layout: config.default_layout
title: Sunshine + moonlight 息屏串流方案
date: 2025-04-18 09:27:13
updated: 2025-04-18 09:28:55
comments: true
tags: [远程串流,Sunshine,Moonlight]
categories: 杂项
---

# Sunshine + moonlight 息屏串流方案

## 前置

最近想玩的 Gal 有点多，但是安卓上的模拟器并没有完全够用，想起来之前有使用过本地串流，所以想着是否能使用串流来玩 Gal。

这样为什么不直接串流玩呢？因为串流的时候和显示器是同步的，也就是你手机串流的时候显示器屏幕也是跟着动，那你在床上悠闲的玩着 Gal 的时候，电脑屏幕也跟着动（雾）这不很奇怪吗？

所以现在要实现的方案是串流的时候，屏幕是黑的但是手机依旧能串流，这样大晚上推 Gal 也不会影响到室友们睡觉了。

![image.png](ciallo-3x.png)

## 环境

笔记本屏幕（1）、外接屏幕（2，可有可无）、虚拟屏幕（3，手机串流到的屏幕，一开始是没有的，后面会装）

## 实现

### 配置 sunshine

项目地址：https://github.com/LizardByte/Sunshine/releases/download/v2025.122.141614/sunshine-windows-installer.exe

下载完安装后，正常默认配置即可。

安装完毕后，浏览器输入：`https://localhost:47990/` ，能正常进入表示安装成功（第一次可能要设置账号密码）

![image.png](image.png)

设置中文

![image.png](image1.png)

其它设置默认即可

### 安装 moonlight

要在手机上安装，项目地址：https://github.com/moonlight-stream/moonlight-android/releases/tag/v12.1

也是直接安装即可

![image.png](image2.png)

### 初步测试

现在进行初步测试，电脑和手机连上**校园网**，或者电脑和手机处于一个**热点**中（不会用流量，因为用的是本地流量）

1. 查看电脑 `IP`
    
    `win + r` 打开 `CMD`，输入 `ipconfig`
    
    ![image.png](image3.png)
    
2. 打开 `moonlight`
    
    点击右上角 `+` 号
    
    ![image.png](image4.png)
    
    输入电脑的 `ip`
    
    ![image.png](image5.png)
    
3. 添加完毕后，需要在 Sunshine web 端输入验证码（会有弹窗），直接输入确认即可
4. 确认后点击刚刚添加的 PC 即可
    
    ![image.png](image6.png)
    
5. 点击后选择 `Desktop` 即可，然后你就会得到
    
    ![7d427ce4ecc2b6e0ead5037b872eaa7e.jpeg](7d427ce4ecc2b6e0ead5037b872eaa7e.jpeg)
    
    已经可以正常使用了。
    

### 添加虚拟屏幕

现在虽然可正常使用了，但是现在和屏幕是同步的，如果你想实现熄屏串流，那么继续往下做。

1. 下载驱动，项目链接：https://github.com/VirtualDrivers/Virtual-Display-Driver/releases/download/24.12.24/Virtual.Display.Driver-v24.12.24-setup-x64.exe
    
    全部下一步即可，安装完后会有一个
    
    ![image.png](image7.png)
    
2. 打开 设置-系统-屏幕 里面可以看到第三（二）个屏幕了。
3. 将它设置为 **拓展该显示器**
    
    ![image.png](image8.png)
    
4. 这样子就全部完成了

### 熄屏串流测试

1. 像 **初步测试** 那样**连接**上，并登陆进去（直接操控电脑进行登录）
2. 打开设置，来到 设置-系统-屏幕，将除了 **虚拟屏幕以外** 的屏幕都选择 **断开此显示器的连接**
    
    ![image.png](image9.png)
    
3. 这样子设置以后，`moonlight` 串流到屏幕就是我们安装的虚拟屏幕了
    
    这时候笔记本屏幕应该就是黑的，但是手机还在串流
    
    这时候就可以愉快的玩耍了
    
    ![image.png](image10.png)
    

### 假如现在不想要串流要怎么操作

同打开设置，来到 设置-系统-屏幕，将前面 **断开连接的屏幕** 全部选择 **将桌面拓展到该桌面**。

然后将 **虚拟屏幕** 选择 **断开此显示器的连接** 即可。

这样子就恢复了。

假如下次想继续串流直接和上边一样操作即可。从 **熄屏串流测试** 开始操作。

## 总结

这样子实现可能有些人觉得可能会麻烦，但是该方案算是非常简单的了，只需要在设置里面调屏幕 断开连接&连接 即可。
