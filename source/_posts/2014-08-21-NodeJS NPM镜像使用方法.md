title: NodeJS NPM 镜像使用方法
date: 2014-08-21 19:22
tags: node.js npm 淘宝镜像
categories: 个人学习
avatar: "https://avatars2.githubusercontent.com/u/3118988?v=3&s=40"
---
通过改变默认npm镜像代理服务，以下三种办法任意一种都能解决问题，建议使用第三种，将配置写死，下次用的时候不用重新配置。

1. 通过config命令
  ```
	npm config set registry https://registry.npm.taobao.org
	npm info underscore (如果上面配置正确这个命令会有字符串response)
  ```

2. 命令行指定
  ```
	npm --registry https://registry.npm.taobao.org info underscore 
  ```

3. 编辑 ~/.npmrc 加入下面内容
  ```
    registry = https://registry.npm.taobao.org
  ```

搜索镜像：[https://npm.taobao.org](https://npm.taobao.org)