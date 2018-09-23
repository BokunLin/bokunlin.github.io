---
title: Git SSH 配置
date: 2018-09-23 18:58:23
tags:
---

> 以下所有 `Bokun` 都是电脑账户名称 需要替换成自己的电脑账户名称
#1. 初始化配置信息
> `$ git config --global user.name "Bokun"`
> 设置用户名
>`$ git config --global user.email "ninebook@hotmail.com"`
> 设置邮箱
#2. 生成 SSH
>`$ ssh-keygen -t rsa -C "ninebook@hotmail.com"`
>会问到保存位置，账号密码类的 直接全部回车默认就可以了

> 如果已存在 会询问是否覆盖
`/c/Users/Bokun/.ssh/id_rsa already exists.Overwrite(yes/no)`

* 通过以上步骤 ssh 就已经配置完成了
#3. 用记事本打开 `id_rsa.pub` 文件， 里面就是你的 ssh 密钥了
>默认位置是在`C:\Users\Bokun\.ssh`

#4. 复制到相应的代码托管平台上去

