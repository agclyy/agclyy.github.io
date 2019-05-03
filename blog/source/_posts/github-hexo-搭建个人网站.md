---
title: github + hexo 搭建个人网站
date: 2019-05-02 20:26:37
categories: github+hexo
tags: hexo
---

- #### 安装node

  > 直接网站下载，安装=>https://nodejs.org/en/download/

- #### 安装hexo

  > 直接命令 简单快捷：npm install -g hexo-cli

  但在我电脑上因为权限出现了错误，fail to execute 。。。修改后命令如下

  > 修改命令为=> npm install -g hexo-cli --unsafe-perm 

- #### github上注册账号

  > 网址：https://github.com/  具体如何注册这边就不阐述了
  >
  > 然后创建一个respository  **注册名.github.io** 必须这样命名 例如我的是agclyy.github.io


接下来就是一路的hexo的命令了，这边为了网站数据不丢失，创建了develop分支，用来存放所有数据

新建立一个目录，之后的博客都存储与这里：

家目录下执行：

```shell
$ mkdir gitblog 

$ cd gitblog

$ git clone -b develop https://github.com/agclyy/agclyy.github.io.git

```

然后在该目录下创建blog，之后生成的blog文章都存储于此

```shell
$ mkdir blog
$ cd blog
$ hexo init
$ npm install
$ hexo generate
$ hexo server
```

终端输出这些信息就说明OK了,浏览器可以输入，页面就出来啦

```shell
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```

#### Hexo配置

用编辑器打开 blog/ 下的配置文件_config.yml找到：

```yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: 
  repository:
```

到GitHub的 **注册名.github.io**仓库下，点击Clone or download,复制里面的HTTPS地址到repository:，添加branch: master。

```yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/注册名/注册名.github.io.git
  branch: master
```

#### 完成部署

最后一步，快要成功了，键入指令：

```shell
$ npm install hexo-deployer-git --save
$ hexo generate
$ hexo deploy
```

每次修改本地文件后，需要键入hexo generate才能保存，再键入hexo deploy上传文件。成功之后命令行输出大概是这样：INFO  Deploy done: git

将所有文件提交到git上的develop分支

```shell
$ git add .
$ git commit -m "..."
$ git push origin develop
```

#### 书写文章

之后书写文章只需要执行new命令，生成指定名称的文章至 blog\source_posts\文章标题.md 。

```shell
$ hexo new "文章标题" #新建文章
```

#### 参考资料&鸣谢：

1. [https://chaoo.oschina.io/2016/05/23/Hexo3-2-github%E6%90%AD%E5%BB%BA%E9%9D%99%E6%80%81%E5%8D%9A%E5%AE%A2.html](https://chaoo.oschina.io/2016/05/23/Hexo3-2-github搭建静态博客.html)
2. https://www.jianshu.com/p/77db3862595c