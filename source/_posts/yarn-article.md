---
title: Yarn,或将取代npm客户端
categories:
- 技术
- 工具
tags:
- yarn
- npm
- node
date: 2017/3/29
---

Yarn 是一个新的包管理器，用于替代现有的 npm 客户端或者其他兼容 npm 仓库的包管理工具。Yarn 保留了现有工作流的特性，优点是更快、更安全、更可靠。

### 为什么用yarn
#### 非常快，非常非常快
yarn 缓存了每次你下载的模块，所以同样模块同样的版本不会发送第二次下载请求，对于没有缓存的模块， yarn 也可以通过并行的网络请求最大限度利用网络资源。现在真的是没有什么几十秒安装不完的依赖的。一个 50 个依赖的 webpack + babel 项目可以在 20 秒左右安装完成。
#### 安全
yarn在开始安装一个包之前会先用 checksums 来验证，你不用担心本地的缓存的包被破坏了导致安装失败。
#### 可靠
被一群喜欢喵星人的开发者维护，以及有 FaceBook 在 production 环境中使用。完善的测试和基于 flow type 的 code base。保证各平台依赖的一致性。
#### 网络优化
力求网络资源最大利用化，让资源下载完美队列执行，避免大量的无用请求，下载失败会自动重新请求，避免整个安装过程失败
#### 扁平化模式
对于不匹配的依赖版本的包创立一个独立的包，避免创建重复的。
#### 以及很多令人感动的小改进
1. 有些 npm 包会抛出 warning 提示信息，在一起 npm install 的时候只有一个名字你完全不知道是哪个包的哪个包的哪个包抛出的这个信息，而 yarn 改善了这一点。
1.  `yarn ls` 会高亮出所有在 package.json 的 dependencies 里的依赖，增强可读性。
1. 每一条命令都会显示执行的时间。
1. 默认生成 lockfile . 保证 yarn 每次安装相同版本的依赖，npm shrinkwrap 会丧失掉同步性如果你忘了生成它。
1.  `yarn why <name>` 这条命令可以告诉你为什么一个依赖会被安装到你的项目中。


### 安装
安装yarn其实非常简单，可以使用npm进行安装yarn包。其[官网](https://yarnpkg.com/zh-Hans/)也有相应的安装文档:[yarn docs](https://yarnpkg.com/zh-Hans/docs)
```bash
$ npm install -g yarn
```
安装完成之后可以查看版本号
```bash
$ yarn --version
```
或者
```bash
$ yarn -v
```

### 使用

#### 初始化
切换到自己的项目目录，执行
```bash
$ yarn init
```
然后照着提示依次输入参数就OK了。
如果觉得麻烦，可以直接执行
```bash
$ yarn init -y
```
这样可以直接创建最简的一个项目。其中包含了package.json文件。

#### 添加包依赖
接下来可以直接在此添加包依赖了,会自动安装最新版本，注意会覆盖指定版本号
```bash
$ yarn add [package]@[version]
```
或者
```bash
$ yarn add [package]@[tag]
```
你会发现，在客户端会打印一出一堆信息如下，添加jquery依赖：
```bash
$ yarn add jquery
yarn add v0.21.3
info No lockfile found.
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
█████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░
█████████████████████████████████░░░░░░░░░░░░░
████████████████████████████████████████
████████████████████████████████████████
[4/4] Building fresh packages...
success Saved lockfile.
success Saved 1 new dependency.
└─ jquery@3.2.1
Done in 4.28s.
```
这时在当前目录中自动添加了一个yarn.lock文件，用于记录所有的包依赖信息。

#### 更新包依赖
更新某包
```bash
$ yarn update [package]
```
更新指定版本的包
```bash
$ yarn update [package]@[version]
```
更新到指定某个标签上的包
```bash
$ yarn update [package]@[tag]
```

#### 移除依赖
```bash
$ yarn remove [package]
```
后面的参数可以与更新包的类似。

#### 开工
在多人团队协作的时候，拉到代码之后，可以类似于npm一样，执行
```bash
$ yarn install
```
即可以开工啦！

### 总结
yarn管理器有一个很重要的文件需要注意，就是yarn.lock，这个是用来依赖的正确性，快速可靠安装的；是执行cli的时候自动生成的，在项目的根目录下，需要保留！！！！不要编辑它，这是自动生成的