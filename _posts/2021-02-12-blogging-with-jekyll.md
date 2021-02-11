---
layout: post
title: 搭建基于Jekyll的静态博客网站
tags: [Blog, Jekyll]
excerpt_separator: <!--more-->
---

把我搭建这个个人博客的过程作为本博客的第一篇教程

<!--more-->

作为一个DIY至上的Engineering青年，我本意是不愿意花钱搭建个人网站的，之前用过学校提供的Wordpress，觉得不好用，整体洁面也不够美观，其他免费平台限制又特别多。在使用Jekyll搭建博客框架后，可以把精力集中在博客内容创作上。（感谢Type on Strap提供此模版）一来可以将自己的生活学习体验记录下来（平时可没有什么整理笔记的习惯），二来可以将学习到的知识分享给大家，贯彻极客精神，与各位交流。

我这里选择[Github](https://github.com/)作为平台发布，尽最大的可能减少开支，同时提供最大的自由度。

[Jekyll](https://jekyllrb.com/)可以理解为一个静态网页生成器，将Markdown编写的内容生成网页，日后不需要对结构排版等问题花费太多时间。

### 在本地使用Jekyll
为了方便调试以及修改，Jekyll提供了在本地浏览器中实时预览对功能，本教程主要以Catalina 10.15.6下的方法，其他系统下应该大同小异。

Jekyll是由Ruby编写，需要安装Ruby运行环境和RubyGem。Catalina默认安装了Ruby，输入 `Ruby -v` 查看Ruby版本。也可以手动安装一个最新版本。

```shell
brew install ruby
```

（有一些情况下还需要安装Command Line Tools, 具体方法是`xcode-select --install`）

安装Jekyll

```shell
sudo gem install jekyll
```

然后在网站文件夹目录下输入`bundle install`后再输入`bundle exec jekyll serve`即可得到一个本地地址（我的地址是`http://127.0.0.1:4000/`），在浏览器中打开即可预览，此网站是在本地服务器上架设运行的。（在修改`Gemfile`后还需要重新执行一次`bundle install`）

不熟悉Jekyll的朋友可以前往Jekyll的官方网站查找所需要的信息以及不同平台的安装方法。




### 数学表达式
本站使用KaTeX显示数学表达式，例如$$E=mc^2$$

在新的一行显示

$$
    x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}
$$

### Highlighting for code
{% highlight js %}
// count to ten
for (var i = 1; i <= 10; i++) {
    console.log(i);
}

// count to twenty
var j = 0;
while (j < 20) {
    j++;
    console.log(j);
}
{% endhighlight %}

### GitHub.io
创建一个名为`username.github.io`的仓库，将修改后的文件发布到`https://username.github.io`即可。


### 小结
我们可以基本使用Jekyll + GitHub 来零成本搭建一个静态的个人博客网站了，但这同时还存在很多的限制，例如插件，非动态等。不过这可以是学习博客搭建的第一步，希望本站的资料可以帮助到各位。当然，GitHub Pages帮助我们照顾了整个网站的后台，如果你还想要一个美观的洁面的话，这便成为了一个浩大的工程，就与我们使用Jekyll的初衷想违背了。