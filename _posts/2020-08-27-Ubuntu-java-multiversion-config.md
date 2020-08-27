---
layout:     post
title:    Ubuntu 18.04 config java multiversion
author:   风止
category: blog
description: 工作中需要用到多版本的Java，为减少设备的使用，记录一下系统配置多版本的Java，从而可以根据项目需要切换不同版本的Java。
---


## 0x00 Introduction

由于工作中的项目需要不同版本的JDK，且设备有限，因此需要设备支持多版本的Java。设备充裕的同行们，直接略过。


## 0x01 Install Java

下载所需版本的JDK（Oracle），略过。，这里使用的版本是1.8.0_191。

配置Java Environment，编辑`/etc/profile`文件。完成后，重启系统即可。

```bash
export JAVA_HOME="/your_java_path/jdk1.8.0_191"
export CLASSPATH=".:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar"
export PATH="$JAVA_HOME/bin:$PATH"
```

校验Java的安装：

```bash
$ java -version
java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
```

## 0x02 Config OS Java

将上述安装的Java配置到系统中，使用`update-alternatives`：

```bash
sudo update-alternatives --install /usr/bin/java java /your_java_path/jdk1.8.0_191/bin/java 300
sudo update-alternatives --install /usr/bin/javac javac /your_java_path/jdk1.8.0_191/bin/javac 300
```

注：`update-alternatives`工具支持Java安装到系统，也支持系统切换Java。

## 0x03 Switch Java

根据需要，切换系统中Java的版本，使用`update-alternatives`：

```bash
sudo update-alternatives --config java
```


## 0x04 Conclusion

系统对多版本的Java的支持，主要使用了`update-alternatives`工具，该工具带上不同的参数，即可完成系统中Java的安装和切换。

使用`--help`即可查看该工具的支持功能：

```bash
$ update-alternatives --help
Usage: update-alternatives [<option> ...] <command>

Commands:
  --install <link> <name> <path> <priority>
    [--slave <link> <name> <path>] ...
                           add a group of alternatives to the system.
  --remove <name> <path>   remove <path> from the <name> group alternative.
  --remove-all <name>      remove <name> group from the alternatives system.
  --auto <name>            switch the master link <name> to automatic mode.
  --display <name>         display information about the <name> group.
  --query <name>           machine parseable version of --display <name>.
  --list <name>            display all targets of the <name> group.
  --get-selections         list master alternative names and their status.
  --set-selections         read alternative status from standard input.
  --config <name>          show alternatives for the <name> group and ask the
                           user to select which one to use.
  --set <name> <path>      set <path> as alternative for <name>.
  --all                    call --config on all alternatives.

<link> is the symlink pointing to /etc/alternatives/<name>.
  (e.g. /usr/bin/pager)
<name> is the master name for this link group.
  (e.g. pager)
<path> is the location of one of the alternative target files.
  (e.g. /usr/bin/less)
<priority> is an integer; options with higher numbers have higher priority in
  automatic mode.

Options:
  --altdir <directory>     change the alternatives directory.
  --admindir <directory>   change the administrative directory.
  --log <file>             change the log file.
  --force                  allow replacing files with alternative links.
  --skip-auto              skip prompt for alternatives correctly configured
                           in automatic mode (relevant for --config only)
  --verbose                verbose operation, more output.
  --quiet                  quiet operation, minimal output.
  --help                   show this help message.
  --version                show the version.
```

## Reference

1. [Install latest version of Java on Ubuntu 18.04](https://www.journaldev.com/33878/install-latest-java-ubuntu)
2. [Ubuntu多个JDK版本配置和切换](https://blog.csdn.net/goodmentc/article/details/80959686)