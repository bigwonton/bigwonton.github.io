---
title: hexo+github搭建博客
date: 2019-08-14 22:05:15
tags: Tools
toc: true
---

## 准备工作

1. github账号
2. 安装`node.js` 和`npm`
3. 安装git

## 新建仓库

新建 `bigwonton.github.io`的仓库，以后的访问网站就是`https://bigwonton.github.io`

## 安装hexo

```
npm install -g hexo
npm install hexo-deploy-git --save //安装插件
```

<!--more-->

## 初始化

```
hexo init //初始化
```

## 配置`_config.yml`
```
//下载主题，放在themes文件夹下
git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

修改配置文件
```
## 换主题
theme: yilia 

deploy:
  type: git
  repo: git@github.com:bigwonton/bigwonton.github.io.git
  branch: master
```


## 上传到github
```

hexo new 'newArticleName' //新建文章

hexo g   // 生成 html

hexo d   // 上传到github

```

## 其他命令

```
hexo clean // 清理public文件夹下的内容

hexo new page 'newPageName' //新建页面

hexo s   // 开启本地预览服务 localhost:4000

```

## 多终端写博客


### 创建hexo分支


配置gitignore文件

```
/.deploy_git
/public
```
初始化仓库及提交

```
git init  //初始化本地仓库
git remote add origin git@github.com:bigwonton/bigwonton.github.io.git  //添加远程仓库 <server> 是指在线仓库的地址 origin是本地分支,remote add操作会将本地仓库映射到云端
git add . //添加本地所有文件到仓库        
git commit -m "更新说明" //添加commit
git checkout -b hexo //创建hexo分支
git push origin hexo //将本地仓库的源文件推送到远程仓库
```

### 新终端初始化

```
git clone git@github.com:bigwonton/bigwonton.github.io.git  //克隆hexo分支
npm install 
```


### 写博客提交方法

```
git checkout hexo //切换hexo分支

git pull //每次写博客之前，先拉取hexo分支最新代码
```

编辑md文件，提交

```
git commit -m "添加新博客"
git push origin hexo
hexo g -d //部署
```





