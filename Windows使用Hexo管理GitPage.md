---
title: (原)Windows使用Hexo管理GitPage
date: 2018-06-16 00:19:20
tags:
---
### 本文重点
```text
    1 如何一步一步按顺序搭建一个可用的git page
    2 着重讲解操作过程遇到的坑，以及网上帖子的不足或未更新之处
    3 对如何安装git、node等基础操作一笔带过，具体参考文中链接
```

### 各类问题
```text
    安装问题
    启动问题
    依赖问题
    图片问题
    发布问题
    子项目问题
    路径问题
```

### 账号准备
```text
    1 github
    2 sshkey
```

### 环境准备
```text
    windows
    1 git server
    2 nodejs server
    3 hexo
```

### hexo目录
```text
    1 安装
    2 配置
    3 使用
    4 部署
```

#### 安装
* [官方文档](https://hexo.io/zh-cn/docs/index.html)
* 安装前提
安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：
[Node.js](https://nodejs.org/en/)
[Git](http://git-scm.com/)
* 安装 Hexo
所有必备的应用程序安装完成后，即可使用如下 npm 命令安装 Hexo。
```bash
    $ npm install -g hexo-cli
```
* 报错
    * 1.网络问题如下图，解决办法：[换为国内源](https://www.jianshu.com/p/0deb70e6f395)
    ![](20180616003722.png)
    ```bash
        // 1.安装
        $ npm install express --registry=https://registry.npm.taobao.org
        // 2.设置
        $ npm config set registry https://registry.npm.taobao.org
        // 3.验证
        $ npm config get registry
    ```

* 安装 Hexo 插件
```bash
    $ npm install hexo-generator-index --save    #索引生成器
    $ npm install hexo-generator-archive --save  #归档生成器
    $ npm install hexo-generator-category --save #分类生成器
    $ npm install hexo-generator-tag --save      #标签生成器
    $ npm install hexo-server --save             #本地服务
    $ npm install hexo-deployer-git --save       #hexo通过git发布（必装）
    $ npm install hexo-renderer-marked --save    #渲染器
    $ npm install hexo-renderer-stylus --save    #渲染器
```


#### 配置
* 标准部署
```yaml
    deploy:
      type: git
      repo: git@github.com:yoursite/project.git
      branch: master
```
    * 注意
    type: git 不要写成 type: github

* [支持图片](https://www.jianshu.com/p/cf0628478a4e)
```yaml
    post_asset_folder: true
```
    * 图片位置
    source\\_posts\pic.jpg
    * 写法举例
    ```markdown
        ![图片说明](pic.jpg)
    ```

* [支持子项目](https://segmentfault.com/a/1190000003946969)
```yaml
    url: https://yoursite.github.io/project/
    root: /project/
```
    * GitHub Pages的分类及区别
    根据官方文档，GitHub Pages分为两类：
    个人/组织主页(yoursite)以及项目主页(project)

* [双线部署](https://www.jianshu.com/p/b0cae7168352)
```yaml
    deploy:
      type: git
      message: "hexo deploy generated pages"
      repo: 
        github: <github repo url>
        gitcafe: <gitcafe repo url> 
```

#### 使用
```bash
    $ hexo n == hexo new "blogname" # 新建文章
    $ hexo g == hexo generate       # 将写好的博客生成静态文件
    $ hexo s == hexo server         # 从本地启动服务进行预览
    $ hexo d == hexo deploy         # 部署到指定的远程目录
```
* 切换主题
[主题列表1](https://hexo.io/themes/)
[主题列表2](https://github.com/hexojs/hexo/wiki/Themes)

* [其他问题1](https://xuanwo.org/2014/08/14/hexo-usual-problem/)
* [其他问题2](https://blog.csdn.net/u013593306/article/details/53749055)

#### 部署
第一次发布可能会有10分钟的延迟。
* 报错
![](20180616013939.png)
原因：更新主题等导致
解决：再执行一次
```bash
    $ npm install hexo-deployer-git --save
```
* 报错
![](20180616015210.png)
原因：缺少.git文件夹
解决：
```bash
    // 第一次操作
    $ git init
    // 远程已有对应仓库
    $ git clone
```