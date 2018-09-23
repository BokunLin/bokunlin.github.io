---
title: Vue2.0 自定义指令 Directive 记录
date: 2018-09-23 19:00:13
tags:
---

## 前言
算了，懒得写了，就这样吧。


## Code

> 组件内写法

    //template
    <div v-tap="{methods: func, args: [1,2,3]}"></div>

    // js
    directives: {
      tap(el, binding) {
        const handle = binding.method;
        const args = binding.args || [];
        //do something
        handle.call(el, args);
      }
    }


突然没啥心情写了，就这样看吧，有空再补充。