WordOnline 前后端对接说明
==========

## 1 后端支持说明

后端给前端提供网盘文件的操作，具体有下面集中类型的操作请求：

1. 网盘文件常规操作
2. 登录passport
3. 网盘文件常规操作
4. 文档读写内容
5. 分享文件（公开/私密）
5. 共享文档（协同编辑）

## 2 登录passport

使用百度passport帐号登录，需要提交[申请](http://passport.sys.baidu.com)

## 3 网盘文件常规操作

网盘文件的操作，统一往后端提交请求，后端对请求做代理，返回请求结果给前端。

请求格式参考了网盘API [PCS_INNER_API](http://wiki.babel.baidu.com/twiki/bin/view/Com/Main/PCS_INNER_API)

代理接口包括以下几个：

1. 列出目录文件
2. 添加文件
3. 删除文件
4. 移动文件（重命名文件）
5. 复制文件

## 4 文档读写内容

wordonline 的文档使用个人云存储，保存在网盘 `/apps/wordonline` 目录下。

文档保存格式一个压缩包，里面bdjson内容和相关资源文件，压缩格式待定。

## 4.1 新增文档

### 4.1.1 上传文档

1. 前端文档提交文档给后端，后端把文档发送到rtcs服务，返回bdzip包。

2. 后端把bdzip包上传到网盘。

### 4.1.2 创建文档

1. 前端编辑内容，编辑过程会上传图片或文件到后端，后端把图片和文件保存在缓冲区，给前端返回一个临时可访问有权限控制的图片链接。

2. 前端文档内容编辑完成，发送bdjson字符串给后端，后端通过bdjson以及里面包含的临时链接（在缓冲区找到对应资源），压缩成bdzip包。

## 4.2 读取文档内容

### 4.2.1 从网盘读取文件

1. 前端列出网盘文件目录后，再通过path向后端请求文档。

2. 后端根据path，向网盘请求获取文件的bdzip内容，解压，把资源文件放到缓冲区，bdjson里的资源文件链接换成缓冲区的临时可访问链接，返回bdjson给前端。

### 4.2.2 上传文件并进入编辑

1. 上传过程按 4.1.1 的流程执行完成之后，再按 4.2.1 第2点返回给前端。

## 5 分享文件（公开/私密）

这一点依赖网盘是否提供以下三个接口：

1. **设置文件为分享文件（公开分享/私密分享）**
2. 如果设置了私密分享，还需要**设置其他人对分享文件的权限**
3. **已分享文件有权限的人可读内容的接口**

如果网盘提供了支持：

1. 分享文档和权限验证的过程可通过网盘接口完成。
2. 文档阅读操作，可以根据网盘文件的 `ACL`，判断已登录用户的权限，返回bdjson给前端。

如果网盘不提供支持：

1. 分享文档和权限验证，需要后端维护，需要一个存储空间存放已分享文档和权限控制。
2. 文档阅读过程，后端向该存储空间获取文档内容，把解析后的bdjson返回给前端。

## 6 共享文档（协同编辑）

共享文档需要协作者有读写内容的权限，网盘需要支持 **4.3分享文件** 里需要的接口以外，还需要以下接口：

1. **已分享文件有权限的人可写内容的接口**

网盘支持或不支持，后端需要实现与 **4.3分享文件类** 里的情况类同。
