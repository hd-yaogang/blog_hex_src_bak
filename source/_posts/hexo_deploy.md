---
title: hexo+githup 个人博客搭建
categories: centos6
tags:
  - centos6
  - hexo
  - githup
---

## 闲话
**优点：**
* 快速高效
* 支持markdown
* 布局自定义简单，无广告
* 部署简单
---

*因为想开始写博客，但又找不到好的博客平台，平时都看博客园和开源中国看博客文章，但博客园的那个皮肤是真有点难受，所以就想自己打个博客平台用着，然后blog的话还是发表到博客园，博客园叫 姚刚，有兴趣的关注一下，以后会出一些关于运维和oracle的文章*

---

**本次环境：
个人笔记本+centos6.9 + node.js v8.9.4 + theme（NexT.Mist v5.1.4）+ githup + gitment **


> 因为第一次开始搭建，主要记录一下操作步骤，可能对原理和其他分析不太全面，请原谅，有不当的或错误的见解希望指正。反正博客刚开始也没有访问量，所以就不打算购买域名，将个人博客部署到githup就好。

## 正文

### 安装git
**先安装git 或者node.js都行**
```
#直接yum进行安装、
yum -y install git-core
#初始化
git config --global user.name "yao gang"
git config --global user.email "xxx@qq.com"
```
### 安装node.js
```
#下载适合自己平台的进行解压：https://nodejs.org/zh-cn/download/
#我装了minimal版，下载node.js源码进行编译安装，开始报错，没有gcc g++
yum -y install gcc gcc-c++    #注意是 gcc-c++ 而不是 g++
#开始编译
./configure --prefix=/usr/local/node.js.8.9.4
#报警告  WARNING: C++ compiler too old, need g++ 4.9.2 or clang++ 3.4 (CXX=g++)
#升级c++编译器gcc
#下载源码包进行编译安装 https://ftp.gnu.org/gnu/gcc/
#我下载 4.9.2，因为我怕版本太高其他依赖又可能出问题

wget http://gcc.skazkaforyou.com/releases/gcc-4.9.2/gcc-4.9.2.tar.gz;
tar -zxvf gcc-4.9.2.tar.gz
cd gcc-4.9.2
mkdir build
cd build
yum install gmp-devel mpfr-devel libmpc-devel
../configure --prefix=/usr
make && make install     # 如果期间还出现问题，小的可以忽略，但需要安装成功，否则自行百度解决

#返回node.js.8.9.4目录下
./configure --prefix=/usr/local/node8.9.4
make && make install       #直到安装成功
```
### 安装hexo
```
# 编辑 /etc/profile (使用vim)
vim /etc/profile
# 在底部添加 PATH 变量
export PATH=$PATH:/usr/local/node8.9.4/bin
# 保存退出，先按exit键，再按shift+：
wq
# 最后保存并使其生效即可
source /etc/profile
# 执行命令有输出即代表安装和环境配置成功
node --version  #  v8.9.4
```
### 配置hexo
```
# 创建目录
mkdir blog
# 切换目录
cd blog
# 安装 Hexo
npm install -g hexo-cli
# 查看
执行命令：  hexo version # 有输出：
hexo-cli: 1.0.4
os: Linux 2.6.32-696.el6.x86_64 linux x64
http_parser: 2.7.0
node: 8.9.4
v8: 6.1.534.50
uv: 1.15.0
zlib: 1.2.11
ares: 1.10.1-DEV
modules: 57
nghttp2: 1.25.0
openssl: 1.0.2n
icu: 59.1
unicode: 9.0
cldr: 31.0.1
tz: 2017b

#安装hexo 插件
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked --save
npm install hexo-renderer-stylus --save
npm install hexo-generator-feed --save
npm install hexo-generator-sitemap --save
后面部署会用到 hexo-deployer-git --save，其他的好像不安装也行，我当时使用简单，并没有用到。


# 初始化 Hexo
hexo init
ls  # 生成hexo工程： _config.yml  db.json  node_modules  package.json  package-lock.json  public  scaffolds  source  themes

# _config.yml  hexo的主要配置文件 
# db.json         hexo的文件存储，以后的博客文章文件应该就是存储在这里
# node_modules   node的模块
# package.json      hexo的插件
# package-lock.json  node的相关依赖
# public                  后面部署就是部署这里的文件
# scaffolds             模板文件，可以编译下面的模板供后续使用,post、page 和 draft,默认是post
# source          使用hexo命令写作的博客页面都是在这下面，可以手动拷贝到此路劲下指定位置
# themes         hexo的主题配置文件
```
>这里介绍几个hexo命令

```
hexo init   # 在当前路劲初始化hexo博客框架工程
hexo generate/hexo g  # 后面是简写，讲hexo的 .md 博客文件通过 node.js 渲染生成静态页面
 hexo deploy/hexo d # 将生成到public下面的静态文件部署到xx/blog/_config.yml 指定的地方，待会我会配置为githup的一个用户名仓库，省的购买域名，需要安装git部署插件
hexo server/hexo s # 启动hexo服务器，访问入口http://localhost:4000（注意防火墙）
hexo clean # 删除上面的db.json数据存储文件
hexo d -g # 先生成，接着部署到制定位置
hexo s -g # 先生成，接着在本地启动服务器
hexo new "我的hexo博客文章1"  # 这就是博客的文章内容
hexo new page "测试页面1" # hexo new page "categories" hexo new page "categories"(主要用到这2个页面，当然你可以在丰富一些，这两个页面主要是用来做博客菜单按钮的跳转页面使用的)
```
> 这部分有点杂乱，但也是为了给新手说明一下配置文件，也可以看官网文档 https://hexo.io/zh-cn/docs/

上面我们初始化以后，执行 hexo s -g 会在public里面按默认的post布局生成静态的html、css、js等文件，访问 localhost：4000，即可看到原生最简单的hexo主页了。ctrol + c关闭服务器。
>到这里，我们的hexo服务器已经基本完成，是不是很简单呢？如果你购买了域名的话，再设置一下喜欢的主题，就完成了。

### 配置github
*由于我的博客没有人气，所以舍不得买域名，就把它部署到github，使用它的用户域名吧，虽然在国外，其实速度也还好啦。你可以点击[我的博客](https://hd-yaogang.github.io "我的博客")试试，可能你要翻墙才能访问*
```
# 首先，需要注册github，然后创建仓库，仓库名需要 重点 注意一下： 用户名 + .github.io
# 例子：我的仓库名为： hd-yaogang.github.io 我的用户名是 ：hd-yaogang
# 以后你的博客域名就是 xxx.github.io，我的就是 hd-yaogang.github.io

#设置hexo所在服务器可以使用git免密ssh部署到 github
#在hexo服务器生成rsa免密私钥和公钥
ssh-keygen  -t  rsa # 一直回车即可，然后执行命令
cat /root/.ssh/id_rsa.pub # 将输出的ssh-rsa到结束的一段内容，将这段内容保存到github

点进进入github仓库，再依次点击 settings-->Deploy keys-->add deploy key，然后随意设置key名字，将 /root/.ssh/id_rsa.pub 的内容粘贴到里面，点击保存完成即可。

```
### 部署hexo
```
# 首先测试一下git，执行命令
mkdir git_test
cd  git_test
git clone git@git@github.com:hd-yaogang/hd-yaogang.github.io.git
默认会克隆主分支master，虽然是空的目录，但是也能验证git是否免密登录了

# 编辑hexo主要配置文件
[root@yg blog]# vim _config.yml  # 注意你所在位置
# 找到下面，模仿着修改一下，注意type是git，注意：后面 有空格，注意配置文件的属性颜色能提示你对不对

deploy:
  type: git
  repo: git@github.com:hd-yaogang/hd-yaogang.github.io.git
  branch: master
  message: 'yg-blog 站点更新:{{now("YYYY-MM-DD HH/mm/ss")}}'

# 执行命令
hexo new “我的hexo 测试博客1”
hexo s -g # 访问http://localhost:4000
# 然后部署到git
hexo d -g # 访问 https://hd-yaogang.github.io ，已经显示博客了
```
### 配置hexo theme
```
[root@yg blog]# git clone --branch v5.1.2 https://github.com/iissnan/hexo-theme-next themes/next
# 下载next主题，也可以自己下载其他喜欢的，放到theme下面即可，这主题不错，后续根据自己需要再改一些
# 启用next主题
[root@yg blog]# vim _config.yml  # 找到theme编辑为：      theme: next
# 重新部署
hexo d -g # 刷新git的博客域名，多刷几次，可能有点慢，主题已经改变的很简洁。
```
>主题的细节设置参考，很详细了：
http://theme-next.iissnan.com/theme-settings.html
http://www.jeyzhang.com/next-theme-personal-settings.html
http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html
http://michael728.github.io/2015/11/30/hexo-next-optimize/

### 配置gitment
```
hexo有很多评论插件，当时也没多研究，就直接上了gitment，可能看的教程简单吧，哈哈哈，但是他评论时候需要登录githup，而且刚开始部署时，这个插件时有时无，有时你刷一下博客评论就不见了。样式的话还不错，可以看我博客评论的样子 https://hd-yaogang.github.io
```
```
首先到github登录，然后依次点击-->账户-->settings-->Developer settings-->New Aouth App，名字叫 Gitment ，homeURL和AuthorizationURL都填写你的域名：https://hd-yaogang.github.io，后面会生成 client ID和 Client Secret，需要配置到hexo的主题配置文件中

然后创建一个空的仓库，名字是：gitment-comments
然后编辑主题配置文件：# 修改相应的，其余的和空着的不用管

gitment:
  enable: true
  mint: true 
  count: true 
  lazy: false
  cleanly: false
  language: zh-Hans # 这是指定插件为中文
  github_user: hd-yaogang  # 这是github用户名
  github_repo: gitment-comments  # 这是空的仓库名，用来存储评论内容
  client_id: cc592528b859ff59a848  # 这是创建app时的id
  client_secret: 80fe54cd8f5f7d58997cc268e376e1552c7020c7 # 这是创建app时的秘钥
  proxy_gateway: 
  redirect_protocol: 

# 再次部署
hexo  d  -g

```

## 结语
**由于每个人能力与想法、环境各都不同，所以可能嫌我写的差，写的啰嗦，写的不好
但是慢慢来，希望有错误的话帮忙指出，谢谢！**

