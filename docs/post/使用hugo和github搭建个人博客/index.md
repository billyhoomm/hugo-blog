
## 关于hugo

Hugo是一个用Go语言编写的静态网站生成器，它使用起来非常简单，相对于Jekyll复杂的安装设置来说，Hugo仅需要一个二进制文件hugo(hugo.exe)即可轻松用于本地调试和生成静态页面。

另外，个人觉得相对于hexo的话，hugo更具个人特色

> 官网： https://gohugo.io/

## 安装

本地hugo客户端

- 官网下载对应自己机器版本的客户端，我这里是macos：https://github.com/gohugoio/hugo/releases
- 得到名为hugo的exec文件
- 直接拖到 `/usr/local/bin` 文件夹
- 终端输入`hugo version`会看到版本信息输出

## 生成博客

终端运行`hugo new site myblog`会生成一个初始化的文件夹

```
▸ archetypes/
▸ content/
▸ layouts/
▸ static/
config.toml
```

config.toml是博客的配置文件，稍候会用到，现在先生成一篇文章试下

```
hugo new post/first_article.md
```

可以看到在content/post/下产生了一个first_article.md文件

另外，再在根目录创建一个docs文件夹，以用来支持github pages的运行

现在先按兵不动

## 安装主题

hugo官网推荐了各种特色的主题，可以随意选择，我目前使用的一个叫even的主题，在myblog根目录运行：

```
git clone https://github.com/olOwOlo/hugo-theme-even.git themes/even
```

在`/themes/even/exampleSite/`中会有一个`config.toml`配置文件，拷贝到根目录覆盖掉原来的`config.toml`

注意这里配置文件中的baseUrl为你最后的博客访问地址，默认为本地调试模式的http://localhost:1313/，如果后续使用github pages的域名或者自定义域名，都需要作修改

其他各种配置参数可以参考该主题在github上的README

## 启动

根目录运行

```
hugo server
```

浏览器打开http://localhost:1313/，即可访问到初始页面

## 部署

首先在博客根目录运行`hugo`，会生成博客运行所需要的静态页面文件

然后去github创建一个空的仓库

```
git init
git remote add origin https://github.com/billyhoomm/hugo-blog.git
git add .
git commit -m 'frist commit'
git push origin master
```

前往github，点击该项目的settings，会看到github pages相关的选项

![](https://cdn.billyhu.com/blogUploads/github_pages.png)

1、我这里是使用docs文件夹（前面说过），当然你也可以选择使用根文件夹，这样master分支就只能放静态页面文件了，我觉得这样不太方便

2、如果你不使用自定义域名，那么修改一下配置文件的baseUrl后提交，然后浏览器访问：https://billyhoomm.github.io/ ，就可以看到你的博客了，

如果你使用自定义域名，请往下看

## 自定义域名&HTTPS

首先你需要一个域名，国内服务商买需要备案，想省去备案麻烦的话，可以选择国外服务商，比如：https://sg.godaddy.com/zh

在上图第二个地方输入你的域名，第三个地方选择强制HTTPS。

完了后去配置DNS解析

以腾讯云为例：

![](https://cdn.billyhu.com/blogUploads/tencent_dns.png)



我这里添加了一个CNAME记录指向自己的github page地址

一个A记录指向自己github page地址对应的IP，其实这个IP现在一共有四个：

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

有条件的可以为根域名添加4个A记录到上面四个IP，但是腾讯云免费情况下只支持两个IP的负载均衡，于是加一个算了...

接下来查看自己的域名解析是否生效了，linux或者macos可以输入以下命令：

![](https://cdn.billyhu.com/blogUploads/dns_check.png)

查看到结果即为解析成功

## 博客相关配置

接下来配置下config.toml

`baseURL = "https://billyhu.com"`

运行`hugo`

然后push上去

大概现在再访问你的域名就可以看到部署好的网站了，如果还不行的话，可以静待一会儿，等其生效。
