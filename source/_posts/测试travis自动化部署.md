---
title: 测试travis 自动部署
date: 2018-03-25 16:21:50
categories: 
- javascript 
tags: 

---
## 自动化部署

hexo 是一套基于nodejs的静态博客生成系统，这往往需要有一套本地的hexo环境，将你本地编写的markdown 文件 生成静态网站需要的文件。
这使得写博客依赖于这套环境，如果你换个电脑，要么重新装一遍环境，要么没法发布。

这里说的自动化部署最终效果是我在github上在线编写一片博文，提交后就会自动构建生成博客文件并上传到指定的git仓库，构建部署完成，博客就发布生效了。
可以做到只要有网络你就可以发布博客。而且以后你只需要关心写博客，不需要关心构建部署。

具体细节不赘述，原理就是利用了 travis 和 github 的webhook。 github 上提交时出发travis 的自动构建部署。 之所以选travis 是因为他免费。

<!-- more -->

## 问题记录
### travis login 报错

* 报错信息
     
       Last Exception
        An error occurred running `travis login`:
          TypeError: no implicit conversion of nil into String
              from /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/json/common.rb:155:in `initialize'
              ...
 
 * 解决方案： 升级ruby 版本至2.5
      
        # 安装rvm
        curl -L get.rvm.io | bash -s stable
        source ~/.rvm/scripts/rvm

### ssh 权限问题

* 报错信息

      Warning: Permanently added the RSA host key for IP address '192.30.253.113' to the list of known hosts.
      To github.com:foreverwang/foreverwang.github.io.git
      
大概是要报github 这个ip 地址加入到known hosts 文件，这个电脑在第一次访问时会自己添加，不知道怎么在travis的服务器上添加。其实到这一步travis 服务器已经能通过ssh方式连接我的github仓库了。但是优于这个问题的存在，导致在hexo deploy时 无法push到仓库。尝试解决未果。

* 解决方案： 改用github token 方式连接 ok了

    具体配置信息看[这里](https://github.com/foreverwang/foreverwang.github.io/blob/hexo/.travis.yml)


### 参考资料

* http://kchen.cc/2016/11/12/hexo-instructions/
* https://blog.csdn.net/woblog/article/details/51319364


