---
title: Mac 实现自动复制手机验证码
date: 2021-07-11 17:40:28
tags: [mac, shell, crontab]
---
> 这个方案依赖 `Apple iMessage` 来实现，看之前确保你的手机是 `iPhone` 电脑是 `Mac`。

## 背景
如今各大手机都能实现短信验证码来了，输入法自动识别，然后填充到文本框里。

在使用 `Mac` 进行办公的时候常常也需要收个验证码，这时候需要打开手机解锁然后背下 `6` 位数的验证码，在电脑上一个个输入。

以前还不觉得麻烦，还以为大家都这样，直到有一天在用 `Safari` 到时候发了条验证码，突发现 `Safari` 可以自动识别 `iPhone` 收到的短信验证码，然后一键填充到文本框，简直不要太方便。

然后因为这个功能我强行从 `Chrome` 切换到了 `Safari` ，但使用过程实在难受，前端调试、各大网站的适配低等等，整体性能体验不如 `Chrome` （也有可能是那些个网站不针对 `Safari` 优化），想了想还是用回 `Chrome` 吧，但切回去以后收短信又很难受了，那咋办呢。

还好我成年了，可以选择全都要；随即开始找方案。

> 其实最早的需求是自动转发短信给企业微信机器人，然后找了半天，，发现以前 `AppleScript` 可以自动转发消息，但是某个版本以后苹果删除掉了这个功能，只能另寻他路。

然后就开始各种 `Google`，直到看到这个老哥的[文章](https://www.cnblogs.com/herberts/p/13191760.html)，就可以开始一步步照着做了，各种细节不再赘述（懒得写了，自己看自己`Google`吧。

## 直接抄
自己也简单的写了个 `Shell`，懒得看的可以直接抄，分几个步骤。

1、`iPhone` 里找到 `Settings` - `Messages` - `Text Message Forwarding` 选择上你的 Mac （需要同时登录同一个 `Apple ID`。

2、创建脚本
* （比如路径在 `/Users/${Your Name}/Shells/AutoCheckCode.sh`）
> 脚本执行一次为 1 分钟，step 为每次执行间隔，默认 3 秒检查一次 `iMessage` 有没有收到带 验证码 的信息，并且默认复制最前面的 4-6 个数字，偷懒写的比较简单，规则后续建议自己调整。

    #!/bin/bash

    step=3;
    # 每 step 秒执行一次
    checkCode(){
        echo "starting to check code";
        # 路径中的 Bokun 记得改成自己电脑的名字
        # 通过 Sqlite3 查 1 条 iMessage 最近 5 秒收到消息（iMessage 收到消息的时间可能有延迟，这里实际冗余多了 2 秒）
        #! 注意这个时间和 step 要匹配好，不然可能每 step 秒都会查出消息，导致重复通知。
        #! /Users/Bokun/Library/Messages/chat.db 
        #！这个 DB 文件和目录记得给开权限，默认是不给读的。
        result=$(sqlite3 /Users/Bokun/Library/Messages/chat.db 'SELECT text FROM message WHERE datetime(date/1000000000 + 978307200,"unixepoch","localtime") > datetime("now","localtime","-5 second") ORDER BY date DESC LIMIT 1;')

        name="验证码";
        
        # 看下最近有没有收到消息
        if [ ! $result ]; then
            echo "latest $step second is not receive code messsages";
            return 
        fi

        echo "result is $result";
        echo "keyword is $name";
            
        # 如果短信中包含验证码则取前 4-6 个数字
        if [[ "$result" =~ "$name" ]]; then
            code=`echo $result | grep -o "[0-9]\{4,6\}"`;

            # 将获取到的数字输出到剪贴板
            echo "$code\c" | pbcopy;

            # 发个系统通知，展示内容，同时提醒你可以 Command + v 粘贴了。 
            osascript -e "display notification \"$result\" with title \"短信验证码\"";

        fi
    }

    # 每分钟执行一次，一次执行 60/step 次，规避 crontab 最低只能 1 分钟执行一次的问题
    for (( i = 0; i < 60; i=(i+step) )); do 
        checkCode;
        echo "i now is $i";
        sleep $step;
    done


3、创建自动任务（`Crontab`）
> 执行前务必手动跑跑脚本，不然出问题脚本在后台无限跑要找出来杀掉挺麻烦
* 打开 `Terminal`

        crontab -e
* 设置定时任务（每分钟跑一次）

        */1 * * * * sh /Users/Bokun/shells/autoCopyCode.sh >> /Users/Bokun/cronlog/autoCopyCode.log 2>&1

> 确认好自己文件的位置，出问题可以去看后半段的 log
        
## 大功告成，后面收到短信验证码直接 `command + v` 就可以粘贴了，各种参数可以自己调整。