#NDK总结

###1.什么是NDK？
NDK全称是:Native Development Kit。它是一套开发工具的总称，可以将C/C++("native code")内嵌到你的安卓app中，NDK大致有以下的应用场景：

* 将App在不同的平台间移植.
* 使用其他程序员开发好的第三方库。
* 用于在特定场景上(例如游戏)提升性能

**NDK关键字解释:**

**LOCAL_PATH := $(call my-dir)**