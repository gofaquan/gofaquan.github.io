---
title: Rust 环境搭建 及 常见问题
tags: rust
---

## Rust 环境搭建 及 常见问题

### 下载  Rust 

* 安装 rustup

  先配置国内镜像加个速

  ````shell
  export RUSTUP_DIST_SERVER= https:// mirrors.ustc.edu.cn/rus t-static export RUSTUP_UPDATE_ROOT= https:// mirrors.ustc.edu.cn/rus t-static/rustup
  ````

  * 然后执行安装脚本

```
curl https://sh.rustup.rs -sSf | sh
```

  安装过程中会让选择安装方式，使用默认方式安装即可，默认安装`cargo`。安装之后需要设置两个目录到PATH变量中:

- $HOME/.cargo/bin，cargo的bin目录
- $HOME/.cargo/env，为shell配置的目录

* 配置环境变量

```shell
 source $HOME/.cargo/env
```

* 加入标准库

```shell
rustup component add rust-src
```





### 下面的也可以看看



#### 解决 standard library 找不到的问题

那当然是没下啊。。。

```shell
rustup component add rust-src
```

#### 解决 update crates.io 过慢的问题 

更改rust的文件源来加快速度

* 首先进入电脑的cargo目录，`Linux` 默认安装在 ` ~/.cargo` 下： 创建一个 `config` 文件

```sh
sudo vim  ~/.cargo/config
```



* 将下面的代码贴进去：

```sh
# 放到 `$HOME/.cargo/config` 文件中
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"

# 替换成你偏好的镜像源
replace-with = 'sjtu'
#replace-with = 'ustc'

# 清华大学
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

# 中国科学技术大学
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"

# 上海交通大学
[source.sjtu]
registry = "https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index"

# rustcc社区
[source.rustcc]
registry = "git://crates.rustcc.cn/crates.io-index"
```

* 然后按esc，输入:wq即可保存退出，现在再试试 ，是不是速度炒鸡快！！！！