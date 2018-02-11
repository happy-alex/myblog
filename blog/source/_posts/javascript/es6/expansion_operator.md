---
title: POST
date: 2018-02-11 20:30:41
tags: js es6
---

刚才在玩手机，发现一篇文章评论中有人说es6的展开运算符（...arg）是深拷贝，于是我在chrome控制台上做了个测试。

var cat = { parent: { age: 3 } };
var cat1 = {...cat};

cat === cat1;   // false, 从这儿看指向内存的地址不同，似乎是深拷贝

cat.parent.age++;
console.log(cat1);     // cat1.parent.age 为 4， 如果展开运算符是深拷贝，这儿的值应该为3.

得出结论：展开运算符不是真正的深拷贝。


截图如下：
![](http://img.blog.csdn.net/20180211210305601?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGh6eDI2NWJvYm8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)