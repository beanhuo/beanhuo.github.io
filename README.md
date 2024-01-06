

### 使用手册

[Jekyll搭建个人博客](http://baixin.io/2016/10/jekyll_tutorials1/)  :  使用Jekyll搭建个人博客的教程，以及如果把博客模板修改成你自己的博客，里面也有大量的评论，及 Jekyll 搭建博客出现过的问题。

[HEXO搭建个人博客](http://baixin.io/2015/08/HEXO%E6%90%AD%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/) : 使用 HEXO 基于 Github Page 搭建个人博客， 教程里面累计了大量提问和评论，如果你在搭建博客时遇到问题，可以看看这个教程。 


### 使用条件

Jekyll 支持 Mac 、Windows、ubuntu 、Linux 操作系统                     
Jekyll 需要依赖：Ruby、bundler


#### 安装Jekyll

[Jekyll中文官方文档](http://jekyll.bootcss.com/) ， 如果你已经安装过了 Jekyll，可以忽略此处。

> $ gem install jekyll

#### 获取博客模板

> $ git clone https://github.com/leopardpan/leopardpan.github.io.git

或者直接[下载博客](https://github.com/leopardpan/leopardpan.github.io/archive/master.zip)   

进leopardpan.github.io/ 目录下， 开启本地服务 

> $ jekyll server

在浏览器输入 [127.0.0.1:4000](127.0.0.1:4000) ， 就可以看到博客效果了。


### 提示

>* 如果你想使用我的模板，请把 _posts/ 目录下的文章都去掉。
>* 修改 _config.yml 文件里面的内容为你自己的个人信息。

如果在部署博客的时候发现问题，可以直接在[Issues](https://github.com/leopardpan/leopardpan.github.io/issues)里面提问。        


### 把这个博客变成你自己的博客

根据上面【提示】修改过后，在你的github里创建一个username.github.io的仓库，username指的值你的github的用户名。      
创建完成后，把我的这个模板使用git push到你的username.github.io仓库下就行了。
搭建博客如果遇到问题可以看看我教程[Jekyll搭建个人博客](http://baixin.io/2016/10/jekyll_tutorials1/)。



本博客在[Vno Jekyll](https://github.com/onevcat/vno-jekyll)基础上修改的。  

### Jekyll vs Hugo

Jekyll vs. Hugo, side by side
Comparison	Jekyll	Hugo
First release	2008	2013
Built with	Ruby	Go
Templating engine	Liquid	Go Templates
Plugins	Yes, via Ruby Gems	No, but still extendable with a bit more work
GitHub stars	42K+	49K+
Themes	Yes	Yes
Open source licence	MIT	Apache 2.0
Installation requirements	Ruby must be installed.	None, Hugo is a single binary.
Asset pipeline	SASS and CoffeeScript out of the box, plugins for almost everything else.	SASS, JS, images, bundling… Hugo does a LOT out of the box. But no plugin ecosystem.
Build times	“Fast-ish” since version 4.0	Fastest in class
Multilingual and i18n	Through plugins	Built in
Shortcodes	Yes	Yes
Easy WordPress et al. converter	Yes: Jekyll converter	Yes: Hugo converter
Learning curve	Gentle, especially if you have Ruby experience.	Less gentle. Go templating is subjectively harder.
Supported content types	Markdown, HTML + more through plugins	Markdown, AsciiDoc, RST, HTML
Support communities	Jekyll Talk Gitter	Hugo Discourse
CLI tool	Yes	Yes
Hot reloading dev server	Yes	Yes
Twitter accounts	Jekyll Twitter	Hugo Twitter
GitHub repos	Jekyll GitHub	Hugo GitHub
