---
layout: post
title: 如何使用Charles进行调试
date: 2017-12-21 14:20:20 +0800
description: A Simple Guide for charles. # Add post description (optional)
img: charles.jpg # Add image post (optional)
tags: [Guide, Charels]
---
Charles是一款代理服务器，通过过将自己设置成系统（电脑或者浏览器）的网络访问代理服务器，然后截取请求和请求结果达到分析抓包的目的。

## 为什么要使用Charles
在前端开发过程中，浏览器给予了开发者们强大的devtools，然而在webview内嵌的h5页面调试中，由于无法直接获取到接口返回数据，以及进行hotfix时无法在线对webview内的页面进行调试，所以需要charles这款强大的工具来实现

## 基本使用
最基本的使用方法当然是作为代理服务器使用了。
1. 先确保手机/电脑在同一个路由器下，然后打开Charles，找到电脑所在的ip地址
2. 在手机的wifi设置里手动配置代理，输入电脑的ip地址，以及端口号8888
3. 此时Charles内会收到一条连接的提示信息，选择Allow即可
4. 此时Charles即可抓取该手机发出的所有http请求啦！

## Https请求的抓取
1. 将手机连接至Charles
2. 在手机浏览器访问chls.pro/ssl，安装证书（ios下需要进行信任）
3. 在Proxy菜单下选择SSL Proxying Settings，勾选Enable
4. 添加你需要抓取的https请求域名，端口号输入443
5. 此时Charles即可抓取该手机发出的指定域名的https请求啦！

## 断点测试(BreakPoints)
1. 点击工具条上六边形的按钮，变红则为开始断点
2. Charles会拦截所有的Request以及Response，可以进行查看或者编辑
3. 可以通过编辑返回数据来查看页面在不同状态下的展示情况

## 映射调试(Map)
### 场景1
webview内出现了意料之外的兼容性问题，然而你无法确定哪种解决方案可行，又不希望每次改动都发布测试进行查看<br />
#### 解决方案：
1. 手机连接至Charles，进入页面，找到获取js文件的请求
2. 右键点击请求，找到Map Remote/Map Local
3. 点击Map Remote，输入你本地devServer下的同一个js地址/点击Map Local，选择本地的同一个js文件
4. 重新进入页面，然后执行的就是我们本地的js文件啦，可以尽情debug！

### 场景2
页面投放在自身app以及合作方app内部，然而由于双方webview对iPhoneX适配方案不一样，导致固定在屏幕底部的按钮位置不对<br />
#### 解决方案
同上，将css文件map成本地的css文件，然后对app内的userAgent进行判断，设置不同的样式以达到兼容效果


