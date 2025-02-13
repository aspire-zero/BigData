## Gitbook简明安装

## 1.Gitbook 简介

[Gitbook (opens new window)](https://www.gitbook.com/)是一个使用 Git 和 Markdown 来构建电子书籍的开源工具。它既可以生成一个静态网站，也可以输出内容作为电子书（ePub，Mobi，PDF）。Gitbook 可以在本地、Github、VPS 等平台上部署，本文采用的是借助 Node.js 在本地部署然后推送到 Github Page 托管的方式。

环境：

```
Node.js 环境：Gitbook 在本地部署时所需要的环境。
Markdown 编辑器：推荐使用 Typora，用来编写 Gitbook 文档的具体内容。
Git 工具 +Github 账号：用于将 Gitbook 托管到 Github 存储库上。
翻墙工具：如果没有，可以私信我推荐一个我正在用的
```

## 2.Gitbook本地部署

**安装node.js**

- 因为 GitBook 是基于 Node.js 的，所以我们首先需要下载安装 [Node.js (opens new window)](https://nodejs.org/en/download/)（对应平台的`.msi`版本即可）**注意node版本太高会影响后续安装。**

- 测试：打开cmd，输入`node -v`和`npm -v`，若显示node和npm的版本号即安装成功。作者的分别是v20.18.0和10.8.2

  - 截止到目前的 Gitbook V3.2.3版本，需要使用NodeJs的v10+版本，否则会产生各种报错。

    这里建议下载`v10.23.1`版本，官网最新版本我试了也是不行的。[Index of /download/release/v10.23.1/](https://nodejs.org/download/release/v10.23.1/)

- 安装Gitbook

  ```shell
  npm install -g gitbook-cli # cmd中运行
  ```

- 安装完之后，就会多了一个 gitbook 命令（如果没有，请确认上面的命令是否加了 `-g`）

**安装Git**

- 官网下载 Git：[https://www.git-scm.com/download/win(opens new window)](https://www.git-scm.com/download/win)
- 安装过程中一路 next 即可，安装成功后桌面右键单击出现 Git GUI Here 以及 Git Bash Here

**安装typora** 

其他md编辑器也行，typora收费了，但有教程破解

- 官网下载 Typora：[https://www.typora.io/(opens new window)](https://www.typora.io/)
- 说明：Typora 可以配合 pandoc 插件将 Markdown 转换成其他格式的文件

## 3.Gitbook本地使用

**初始化Gitbook**

* 新建一个存储Gitbook的文件夹，切换到该目录下，然后初始化

  ```shell
  gitbook init 
  # 执行完后，你会看到多了两个文件 —— README.md 和 SUMMARY.md，它们的作用如下：
  README.md —— 书籍的介绍写在这个文件里
  SUMMARY.md —— 书籍的目录结构在这里配置
  ```

**用md编辑器编辑一个示例**

* 编辑SUMMARY.md

  ```shell
  # 目录
  
  - [前言](README.md)
  - [第一章](Chapter1/README.md)
    - [第1节：衣](Chapter1/衣.md)
    - [第2节：食](Chapter1/食.md)
    - [第3节：住](Chapter1/住.md)
    - [第4节：行](Chapter1/行.md)
  - [第二章](Chapter2/README.md)
  - [第三章](Chapter3/README.md)
  - [第四章](Chapter4/README.md)
  ```

**在本地部署Gitbook**

- 回到命令行，在 Gitbook 的工作区文件夹中再次执行 `gitbook init` 命令。GitBook 会查找 SUMMARY.md 文件中描述的目录和文件，如果没有则会将其创建。
- 接着我们再执行 `gitbook serve` 命令，将其部署在本地，打开 Chrome 访问：[http://localhost:4000 (opens new window)](http://localhost:4000/)，即可看到本地部署的 Gitbook（注：serve 命令可以指定端口 `gitbook serve --port 2333`）
- 当你写得差不多，你可以执行 `gitbook build` 命令构建书籍，默认将生成的静态网站输出到 _book 目录。实际上，这一步也包含在 `gitbook serve` 里面（注：build 命令可以指定路径 `gitbook build [书籍路径] [输出路径]`，如果你想查看输出目录详细的记录，可使用 `gitbook build ./ --log=debug --debug` 来查看）

问题：

node.js版本问题

- 问题描述：编译时报错 `polyfills.js:287 if (cb) cb.apply(this, arguments) TypeError: cb.apply is not a function`

- 产生原因：因开发前端项目时升级了node.js版本所致，不支持这个版本的gitbook。

- 解决办法：一种是把node.js降级回去，第二种方式是注释掉报错代码（这个函数的作用是用来修复node.js的一些bug，用处不大）

  打开`C:\Users\xxx\AppData\Roaming\npm\node_modules\gitbook-cli\node_modules\npm\node_modules\graceful-fs\polyfills.js`文件，找到287行的statFix函数，把它的调用处注释掉即可。

## 4.Gitbook配置文件

如果你想对你的网站有更详细的个性化配置或使用插件，那么需要使用配置文件。配置文件写完后，需要重启服务或者重新打包才能应用配置。Gitbook 的配置文件是`book.json`， 请在项目的根目录处自行创建。

### 4.1 配置文件主要参数

- title：标题
- author：作者
- description：描述，对应 Gitbook 网站的 description
- language：使用的语言，`zh-hans` 是简体中文，会对应到页面的`<html lang="zh-hans" >`
- structure：指定 Readme、Summary、Glossary 和 Languages 对应的文件名
- plugins：使用的插件列表，所有的插件都在这里写出来，然后使用 `gitbook install` 来安装。
- pluginsConfig：插件的配置信息，如果插件需要配置参数，那么在这里填写。
- links：目前可以给侧导航栏添加链接信息
- styles：自定义页面样式，各种格式对应各自的 css 文件

### 4.2 Gitbook 的常用插件

#### 4.2.1 Gitbook 插件简介

**[1] 为什么要用插件？**

Gitbook 默认自带以下 5 个插件：highlight：代码高亮、search：导航栏查询功能（不支持中文）、sharing：右上角分享功能、font-settings：字体设置（最上方的"A"符号）、livereload：为 Gitbook 实时重新加载。Gitbook 插件可以解决一些网站不太方便的地方，如侧边栏导航不能收缩，自带搜索不支持中文等问题。

**[2] 去哪里找插件？**

除了下文推荐的插件之外，Gitbook 还支持许多其他插件，可以从[NPM (opens new window)](https://www.npmjs.com/)上搜索 Gitbook 相关的插件。

**[3] 插件安装方法：**

- Step1：在项目的根目录中创建 `book.json` 文件，然后在 plugins 参数中添加插件名。
- Step2：使用 `gitbook install` 来安装插件，重启服务 `gitbook serve` 或者重新打包 `gitbook build` 就能看见效果。

注意：

- 编写 json 时字符串不能用“单引号”括起，最后的那个不能有“逗号”。
- 如果要卸载自带的 `font-settings`，插件处应写成 `-fontsettings`，中间不要加 `-`。
- `gitbook install` 命令有时会出现问题，多试几次可能就好了。
- `gitbook install` 命令安装慢，而且是全部插件都安装一遍，如果只安装一个插件的话建议使用npm命令安装。

#### 4.2.2 Gitbook 插件推荐

**[1] 支持中文的搜索框**

```JSON
{
    "plugins": [
         "-lunr", 
         "-search", 
         "search-pro"
    ],
}
```

**[2] 左侧章节目录可折叠**

```JSON
{
    "plugins": [
         "expandable-chapters-small"
    ],
}
```

**[3] 侧边栏宽度可调节**

```json
{
    "plugins": [
    	"splitter"
    ],
}
```

**[4] 回到顶部按钮**

```json
{
    "plugins": [
        "back-to-top-button"
    ],
}
```

**[5] 右上角添加 github 图标跳转**

```json
{
    "plugins": [
        "-sharing",
        "github"
    ],
    "pluginsConfig": {
        "github": {
            "url": "项目仓库地址"
        },
    }
}
```

**[6] 隐藏元素**

可以隐藏不想看到的元素，`hide-element` 是通过 HTML 元素的 `class` 名字来查找要隐藏的元素，想要隐藏元素找到元素的样式类名加到插件配置里面就可以隐藏元素了。

```json
{
    "plugins": [
        "hide-element"
    ],
    "pluginsConfig": {
        "hide-element": {
            "elements": [".gitbook-link"]
        },
    }
}
```

经过该配置之后，导航栏中 `Published by GitBook` 就被隐藏了。

**[7] 代码块行号、复制**

```json
{
    "plugins": [
        "code"
    ],
}
```

**[8] 修改标题栏图标**

```json
{
    "plugins": [
        "favicon"
    ],
    "pluginsConfig": {
        "favicon": {
            "shortcut": "assets/images/favicon.ico",
            "bookmark": "assets/images/favicon.ico",
            "appleTouch": "assets/images/apple-touch-icon.png",
            "appleTouchMore": {
                "120x120": "assets/images/apple-touch-icon-120x120.png",
                "180x180": "assets/images/apple-touch-icon-180x180.png"
            }
        }
    }
}
```

shortcut通常可以被所有可以显示favicon的浏览器读取。

bookmark是在收藏夹中显示自己的图标。

apple-touch-icon是一个类似网站favicon的图标文件，用来在iphone和iPad上创建快捷键时使用。apple-touch-icon这个文件应当是png格式，默认：57x57像素大小。如果准备的文件不是57x57的话，它会自己缩放的。

**[9] 添加悬浮目录**

```json
{
    "plugins" : [ 
        "page-toc-button" 
    ],
    "pluginsConfig": {
        "page-toc-button": {
            "maxTocDepth": 2,
            "minTocSize": 2
        },
    }
}
```

maxTocDepth：标题的最大深度（最大支持到 2，即为 h1+h2+h3）

minTocSize：显示 toc 按钮的最小 toc 条目数

**[10] 添加版权信息和最后修改时间**

```json
{
    "plugins": [
       "tbfed-pagefooter"
    ],
    "pluginsConfig": {
        "tbfed-pagefooter": {
            "copyright":"Copyright &copy 版权信息",
            "modify_label": "该文件修订时间：",
            "modify_format": "YYYY-MM-DD HH:mm:ss"
        },
    }
}
```

**[11] 添加RSS订阅**

```json
{
    "plugins": [ 
   		 "rss" 
     ],
    "pluginsConfig": {
        "rss":{
            "title": "标题",
            "description": "描述信息",
            "author": "作者",
            "feed_url": "https://xxx.xxx.xxx/rss",
            "site_url": "https://xxx.xxx.xxx/"
        }
    }
}
```

## 4.3 成品配置

以下是我配好的 `book.json` 文件，仅供参考。

```json
{
    "title": "标题",
    "description": "描述信息",
    "author": "作者",
    "output.name": "site",
    "language": "zh-hans",
    "gitbook": "3.2.3",
    "root": ".",
    "links": {
        "sidebar": {
            "sidebar标题": "sidebar地址"
        }
    },
    "plugins": [
         "-lunr", 
         "-search",
         "-sharing",
         "-fontsettings",
         "highlight",
         "livereload",
         "search-pro",
         "expandable-chapters-small",
         "splitter",
         "back-to-top-button",
         "github",
         "hide-element",
         "code",
         "custom-favicon",
         "page-toc-button",
         "tbfed-pagefooter",
         "rss"
    ],
    "pluginsConfig": {
        "github": {
            "url": "项目仓库地址"
        },
        "hide-element": {
            "elements": [".gitbook-link"]
        },
        "favicon": {
            "shortcut": "assets/images/favicon.ico",
            "bookmark": "assets/images/favicon.ico",
            "appleTouch": "assets/images/apple-touch-icon.png",
            "appleTouchMore": {
                "120x120": "assets/images/apple-touch-icon-120x120.png",
                "180x180": "assets/images/apple-touch-icon-180x180.png"
            }
        },
       "page-toc-button": {
            "maxTocDepth": 2,
            "minTocSize": 2
        },
        "tbfed-pagefooter": {
            "copyright":"Copyright &copy 版权信息",
            "modify_label": "该文件修订时间：",
            "modify_format": "YYYY-MM-DD HH:mm:ss"
        },
        "rss":{
            "title": "标题",
            "description": "描述信息",
            "author": "作者",
            "feed_url": "https://xxx.xxx.xxx/rss",
            "site_url": "https://xxx.xxx.xxx/"
        }
    }
}
```

## 5.托管到Github Page

这部分需要使用 Git 和 Github 网站

由于 Gitbook 生成的项目跟文档的源码是两个部分，所以可以把文档放到 master 分支上，部署的网站放到 gh-pages 分支。

### 5.1 本地项目提交到 Github 仓库

**[1] 创建git忽略文件**

有些文件是不需要纳入到版本控制之中的，在项目根目录创建一个名为`.gitignore`的git忽略文件，编辑内容如下：

```text
# 忽略gitbook生成的项目目录
_book
```

下面是一些`.gitignore`文件忽略的匹配规则：

```text
*.a       # 忽略所有 .a 结尾的文件
!lib.a    # 但 lib.a 除外
/TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/    # 忽略 build/ 目录下的所有文件
doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```

注意：`.gitignore`只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改`.gitignore`文件是无效的。那么解决方法就是先把本地缓存删除（改变成未track状态），然后再提交。

**[2] SSH KEY**

由于本地 Git 仓库和 Github 仓库之间的传输是通过 SSH 加密的，所以连接时需要设置一下：

Step1：先看一下 `C:\Users\xxx` 有没有.ssh 目录，有的话看下里面有没有 id_rsa 和 id_rsa.pub 这两个文件，有就跳到下一步，没有就通过下面命令创建:

```text
$ ssh-keygen -t rsa -C "xxx@xxx.com"
```

然后一路回车，这时你就会在用户下的.ssh 目录里找到 id_rsa 和 id_rsa.pub 这两个文件

Step2：登录 Github,找到右上角的图标，打开点进里面的 Settings，再选中里面的 SSH and GPG KEYS，点击右上角的 New SSH key，然后 Title 里面随便填，再把刚才 id_rsa.pub 里面的内容复制到 Title 下面的 Key 内容框里面，最后点击 Add SSH key，这样就完成了 SSH Key 的加密。

**[3] 初次部署**

```text
[1] 设置用户名和邮箱
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
[2] 连接到远程仓库并设置记住用户名和密码
git remote add origin URL # 连接到远程仓库并创建别名
git config --global credential.helper store # 设置自动记住用户名和密码（第一次push会被记住，后续无需再输入）
[3] 解决使用git add命令时报错LF will be replaced by CRLF的问题
git config auto.crlf true
[4] 将项目从本地提交到仓库
git init # 创建git工作区
git add . # 提交所有文件到暂存区
git status # 查看git状态
git commit -m 'description' # 提交到仓库，''内的是描述信息
[5] 将代码推送至远程仓库
git pull origin master --allow-unrelated-histories # 取回远程仓库分支的更新，再与本地的分支合并
git push -u origin master # 将本地仓库推送至远程仓库
```

说明：

1. 用户名和邮箱设置：在 github 仓库主页显示谁提交了该文件，要根据 github 的注册信息来填写，不要填错了。

2. `git init` 后，在文件夹内生成.git 文件（如果没有，则 `查看——勾选“隐藏的项目”`）

3. 仓库地址可在 clone or download 按钮下取得（即仓库的浏览器地址栏末尾加 `.git`）

4. 使用 git pull origin master 命令报错：`fatal:refusing to merge unrelated histories`

   - 错误原因：如果合并了两个不同的开始提交的仓库，在新的 git 会发现这两个仓库可能不是同一个，为了防止开发者上传错误，于是就出现了此提示。

   - 解决办法：如我在 Github 新建一个仓库，写了 License，然后把本地一个写了很久仓库上传。这时会发现 github 的仓库和本地的没有一个共同的 commit 所以 git 不让提交，认为是写错了 origin ，如果开发者确定是这个 origin 就可以使用 --allow-unrelated-histories 告诉 git 允许不相关历史合并

5. 使用 git push origin master 命令报错：`error:failed to push some refs to URL`

   - 错误原因：直接在 GitHub 上修改后，内容已经和本地不一致了，必须要合并（merge）

   - 解决办法：先使用 git pull origin master 命令下载到本地并合并，自动弹出的 vim 编辑器（按 i 进行编辑，说明为什么合并，可选择不修改，ESC 进入命令行模式然后输入 `:wq` 退出），再 git push origin master

6. 使用 git add .命令报错：warning: LF will be replaced by CRLF

   - 原因：在 Unix 系统中，行尾用换行（LF）表示。在窗口中，用回车（CR）和换行（LF）（CRLF）表示一行。当您从 unix 系统上载的 git 中获取代码时，它们将只有 LF。

   - 解决办法：如果您是在 Windows 计算机上工作的单个开发人员，并且您不关心 git 自动将 LF 替换为 CRLF，则可以通过在 git 命令行中键入以下内容来关闭此警告

     ```text
     git config core.autocrlf true
     ```

**[4] 后续使用**

```text
git config auto.crlf true # 解决使用git add命令时报错LF will be replaced by CRLF的问题
git add . # 提交所有文件到暂存区
git status # 查看git状态
git commit -m 'description' # 提交到仓库，''内的是描述信息
git pull origin master # 取回远程仓库分支的更新，再与本地的分支合并
git push origin master # 将本地仓库推送至远程仓库
```

为了提交方便，可以创建一个脚本文件 `commit.sh` 来自动执行，内容如下：

```shell
目录结构：
--shell
----commit.sh
----deploy.sh
--content
--gh-pages
```



```shell
# 切换到上一级目录
echo '切换到上一级目录\n'
cd ..

# 解决使用git add命令时报错LF will be replaced by CRLF的问题
echo '执行命令：git config auto.crlf true\n'
git config auto.crlf true

# 保存所有的修改
echo '执行命令：git add -A\n'
git add -A

# 把修改的文件提交
echo "执行命令：git commit -m 'update gitbook'\n"
git commit -m 'update gitbook' 

# 将本地仓库推送至远程仓库
echo '执行命令：git push origin main\n'
git push origin main

# 返回到上一次的工作目录
echo "回到刚才工作目录"
cd -
```

编写好后，在终端运行以下命令即可：

```text
$ bash commit.sh
```

### 5.2 上传到 Github 仓库的 gh-pages 分支

打包命令太多，为了部署方便，可以创建一个脚本文件 `deploy.sh` 来自动执行，内容如下：

```shell
# 切换到上一级目录
echo '切换到上一级目录\n'
cd ../gh-pages

git checkout gh-pages

# 解决使用git add命令时报错LF will be replaced by CRLF的问题
# echo '执行命令：git config auto.crlf true\n'
# git config auto.crlf true

# 保存所有的修改
echo "执行命令：git add -A"
git add -A

# 把修改的文件提交
echo "执行命令：commit -m 'deploy gitbook'"
git commit -m 'deploy gitbook'

# 发布到 https://<USERNAME>.github.io/<REPO>
echo "执行命令：git push -f 仓库地址.git main:gh-pages"
git push git@github.com:aspire-zero/BigData.git gh-pages

# 返回到上一次的工作目录
echo "回到刚才工作目录"
cd -
```

注意脚本文件代码中仓库地址要替换成你自己的地址。

文件保存后，在终端执行如下命令，把生成的项目推送到 github 仓库上的 `gh-pages` 分支：

```text
$ bash deploy.sh
```

执行成功后，打开你的 github 仓库，然后选择 `branch` 分支，会发现多了一个 `gh-pages` 分支，打开这个分之后，里面会有一个 `index.html` 文件，说明部署的代码上传成功了。

### 5.3 查看 GitHub Page 网站

在 Github 网站上的项目仓库右侧，有一个 Environments，点进去之后再点击 View deployment，即可查看部署好的 GitHub Page。至此，我们的 Gitbook 就已经搭建完成了。 