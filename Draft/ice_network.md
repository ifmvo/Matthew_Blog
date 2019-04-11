---
title: Android ICE 网络请求完美封装
date: 2017-03-16
tags: ICE
---



由于公司启动一个新项目，使用 ICE 作为项目系统的中间件，包括后台、移动端iOS、Android 还有 Web 端都使用 ICE，而 Android 端的网络请求部分就是使用 ICE 来实现。

其实我之前也没有听说个 ICE 这个东西，Google 后发现 ICE 在 Android 方面的资料极少，有的话也是一个简单的 Demo，而且官方 Github 上面提供的 Demo 也只是告诉你如何使用，并没有为你提供什么封装好的网络请求库，所以完全由自己从零开始一步一步进行封装，这对于一个刚刚入门的我来说着实不易。

---------

## ICE为何物？

`ZeroC ICE` 是 `ZeroC` 公司的 `ICE（Internet Communications Engine）`中间件平台。对于客户端和服务端程序的开发提供了很大的便利。

**官网自己的描述：**

 - 灵活
 - 安全、稳定
 - 快速
 - 多语言多平台`（Develop in C++, C#, Java, JavaScript, Objective-C, PHP, Python, and Ruby. Deploy on Linux, macOS, Windows, Android, and iOS.）`

目前稳定版本为 3.6 ， 3.7 版本正在持续开发。

---------

## 如何安装？

无论是 `Linux` `Window` 还是 `Mac` 安装步骤都是很方便的，只要按照官方文档使用几个命令就可以了。请参考[官方文档](https://doc.zeroc.com/)
## Demo
我对 Android 中使用 ICE ***网络请求部分进行简单封装***,可以提供参考.

github 链接:
https://github.com/ifmvo/ICEAndroidDemo

### 调用接口姿势:
实现传统 Callback 回调方式
```
IceClient.getUsers("ifmvo", "123456", new IceClient.Callback<String>() {
            @Override
            public void onStart() {
                // showLoading();

            }

            @Override
            public void onFailure(String msg) {
                //closeLoading();
                //showErrorMsg();
                tvInfo.setText("链接异常信息" + msg);
            }

            @Override
            public void onSuccess(String result) {

                //closeLoading();
                //showResult();
                tvInfo.setText("返回成功信息" + result);
            }
        });
```
### 添加接口方式:
//IceClient.java
```
/**
 * 定义的借口就可以这样写
 * @param username 参数
 * @param password 参数
 * @param callback 回调
 */
public static void getUsers(String username, String password, final Callback callback){
    if (requestPre(callback) != OK) return ;

    helloPrx.begin_getUsers(username, password, new Callback_Hello_getUsers() {
        @Override
        public void response(int ret, String message) {
            handleSuccess(message, callback);
        }

        @Override
        public void exception(LocalException e) {
            handleException(callback, e);
        }
    });
}
```
