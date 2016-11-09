---
title: the first commit of vue (or element)
date: 2016-11-08 
---

2013年7月28日，作者提交了第一个版本，那时命名为 element, 还不能真正的说是vue,但确是vue这个项目的起点。

首先来看一下项目的目录结构：

* src(源码目录，包含一个main.js文件)
* test(测试代码)
* .gitignore
* .jshintrc
* component.json
* Gruntfile.js
* package.json

这个时候还没有什么实质性的代码，main.js中只有短短的一行`module.exports = 123`，但却包含了一个项目所必须的东西。

* 版本控制(git/.gitignore)
* 构建工具(Grunt)
* 测试(mocha)
* 代码质量检测(jshint)

版本控制和构建工具基本是现在项目的标配了，但是测试和代码质量检测通常会忽视，尤其是测试部门常被忽略，个人写业务代码时基本不会写测试的，但是作为开源的框架或是库，测试还是很有必要的。

一般都会觉得写测试过于麻烦，想着等项目稳定了在慢慢加测试，但其实测试是很有必要的，可以提前发现许多问题，避免许多不必要的bug.