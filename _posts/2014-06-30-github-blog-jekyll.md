---
title: 在Github上部署自己的博客
layout: post
tags:
    - blog
    - jekyll
    - github
---

一直想建立自己的博客，但不知道选哪个博客平台比较好。

作为一个程序员，当然希望自己一手一脚把整个博客系统搭起来，买域名，租主机或者云平台，搭建WordPress，找漂亮模版，弄这个弄那个的。但精力实在有限，不想重复做个轮子，所以还是放弃了。不过如果你是一个IT专业的大学生，还是建议去折腾折腾，这也是找工作的筹码呀。

虽说放弃了自建的想法，但也对已有的博客平台不感冒。一是想把文章数据保存在自己那里，二是想要定制有个性化的界面，三是不想一个个平台去体验比较。

最后，一个偶然的机会发现了[Github](https://pages.github.com/)上可以写博客，使用jekyll静态站点生成器，以commit的方式发表新文章。一次满足三个愿望：

1. 免费又稳定的空间，还带文章版本管理。（Github）
2. 丰富的jekyll开源主题，你只需要关注如何写文章，还可以高度定制化，满足日后需要。
3. 发表新文章还能攒Github Contributions，一举两得。


----------

好，废话完毕，进入主题 － 如何在Github上部署博客。

**1st step:**

创建一个新的项目，命名为 *username*.github.io （*username*是Github的帐户）,具体参考[这里](https://pages.github.com/)，本文不再冗述。

**2nd step:**

创建好项目，git clone下来后就可以开始在本地写博客了，当然这个时候博客还是什么都没有的。Github Pages是基于jekyll系统的，我们需要先在本地安装jekyll，方便实时预览效果。

**安装 jekyll**

首先你要有ruby(>= 1.9.3)，```ruby -v``` 查看当前ruby版本。否则会提示以下错误：

```
require: no such file to load -- net/https (LoadError)
```

对于较新的操作系统一般没问题，如果是较旧的系统，则需要自己安装高版本的ruby。下面以**安装ruby 2.1.2为例**（ruby >= 1.9.3 请直接忽略）:

----------

1. 首先你要有autoconf(>2.67), ```autoconf --version``` 查看当前版本。 如果有，请忽略第2步：
2. 从[这里下载](http://ftp.gnu.org/gnu/autoconf/)，解压，安装三步曲：
>
```
$ cd autoconf           #解压后的目录
$ ./configure           #配置
$ make                  #编译
$ sudo make install     #安装
```

3. 其次， 你还需要有旧版本的ruby（-. -），否则会提示：
>
```
executable host ruby is required. use --with-baseruby option
```
4. 一般操作系统会预装，如果没有，就需要安装一个旧版的ruby。为了方便，这里以Ubuntu为例：
>
```
$ sudo apt-get install ruby
```
5. 再次，你还要有bison
>
```
$ sudo apt-get install bison
```
6. 有了以上准备，可以正式开始安装ruby了， 三步曲：
>
```
$ cd ruby
$ ./configure
$ make
$ sudo make install
$ ruby -v
```


----------

安装好ruby>=1.9.3后，安装jekyll：

```
$ sudo gem install jekyll
$ jekyll -v
```

**3rd step:**

安装好jekyll后，从 [jekyll wiki](https://github.com/jekyll/jekyll/wiki/Sites)，[jekyll themes](http://jekyllthemes.org/) 里挑选一个theme，作为自己博客的模版。
例如本博客使用的是[sext vi](http://lhzhang.com/)的theme（页面底部标注），当然大家要注意License版本。

**4th step:**

下载主题后，可以通过jekyll在本地预览效果。

```
$ cd theme-demo
$ jekyll serve --watch  #带上watch可以实时监控文件修改，自动渲染
```
在浏览器打开 http://localhost:4000 预览。

**5th step:**

接下来就可以根据个人喜好对theme进行修改，这里面会涉及到jekyll的语法，详细可以查看这里的[文档](http://jekyllrb.com/docs/pages/)，[中文版文档](http://jekyllcn.com/docs/pages/)。以下是jekyll基本的目录结构：

```
.
├── _config.yml         ＃配置文件
├── _drafts             ＃草稿箱
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes           ＃重用的组件
|   ├── footer.html
|   └── header.html
├── _layouts            ＃页面、文章模版
|   ├── default.html
|   └── post.html
├── _posts              ＃保存文章，文件名格式必须 YEAR-MONTH-DAY-title.MARKUP
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _site              # jekyll生成的静态页面
└── index.html         ＃ 首页
```


**final step:**

修改完成后，再次提交到Github上，就可以打开 http://*username*.github.io 浏览自己的博客，至此搭建完成。

