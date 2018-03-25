---
layout: photo
title:  hexo + github pages搭建个人博客
date: 2017-02-23 12:20:48
categories: 
- Hexo 
tags: 
- 博客 
- hexo

photos:
- http://bruce.u.qiniudn.com/2013/11/27/reading/photos-0.jpg

---
[Hexo官网](https://hexo.io/)
### hexo报错解决

<!-- more -->

* hexo init  报错  
	
	 ```bash
	 npm uninstall dtrace-provider 
	 ```

* hero server 报错

	```bash 
	  npm install hexo --no-optional
	```
	通常这个命令能解决，（我这解决不了 就找到报错的地方，看了下这个报错没啥影响就注释掉了，不然强迫症）
	
	
### github pages

#### 在github 新建仓库
仓库命名： username.github.io, 此时就可以访问 username.github.io了

#### github pages 和 hexo 关联

- hexo -g 会生成一个静态网站（第一次会生成一个public目录），这个静态文件可以直接访问。
- 需要将hexo生成的静态网站，提交(git commit)到github上。

#### 通过hexo 命令将静态站点push到github

* 安装插件hexo-deployer-git

 ```bash
 npm install hexo-deployer-git --save
 ```

* 修改 hexo的 _config.yml
		
	```bash
	deploy:
	  type: git
	  repo: https://github.com/username/username.github.io.git
	  branch: master
	```
	  
	  
#### 更换hexo主题
* 克隆主题到themes路径下

 ```bash 
 git clone https://github.com/theme-next/hexo-theme-next.git themes/next
 ```



  注意：主题文件夹放到themes下 文件夹命名和配置文件里保持一致

* 修改站点配置文件 _config.yml

	```bash 
	theme: next
	```
  
  
#### hexo添加分类页面
* 新建分类页
	
	```bash
	hexo new page categories
	```
* 在 source/categories 目录的 index.md 中修改:

		title: 分类
		date: 2015-12-02 12:44:45
		type: 'categories'

		 
#### hexo 常用命令

 * 新建一篇博客
 
	```bash		
	hexo new filename
	```

* 起本地服务
	
	```bash	
	hexo s 
	```

* 生成静态站点

 ```bash  
   	hexo g
   	```
	   
* 发布
	
	```bash  	
	hexo d 
	```
	   
* 清除本地缓存
	
	```bash   
	hexo clean
	``` 
	
#### 使用技巧
* git deploy 面用户名和密码提交github 
在github 添加ssh key 公钥后 通过ssh 的方式提交代码

	```bash
	deploy:
	  type: git
	  repo: git@github.com:xxx
	  branch: master
	```




