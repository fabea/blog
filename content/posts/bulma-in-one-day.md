---
title: "Bulma in One Day"
date: 2021-03-06T22:07:47+08:00
draft: false
toc: false
images:
tags: 
  - untagged
---

## 缘由

一直以来，我做的网页都很难看。这件事情说起来有点让人羞涩，二十年前开始玩电脑的时候，就知道html、css是干啥的。但是就是硬生生的不想学这个东西，一方面不想学，一方面有自己告诉自己“这个东西很简单，用得到的时候，随时都能学会。”“html根本就不是语言，太简单了。”

这种简单，就一下子简单了快二十年。每当需要用网页的时候，我就开始想办法“从X格式转换为html”，同时“自动生成css”，“不要使用js”。这样的要求，先先后后用orgmode和markdown+pandoc都填补过，但是效果很难让自己满意，毕竟自己无法控制的东西，永远都不会让人满意。

## 所以

最近，属于一段拾起那些眼高手低的东西的时间。

两个星期的连续不断的python学习有些疲惫，作为插曲，花了一天时间研究Bulma。一天过去了。至少自己能做出来还得过去的网页了。

bulma的设置很符合我的要求，做出来的效果不错，控制简易且精准。之前使用了Uikit，感觉过于复杂和繁琐，大量的class也记不住，然后又尝试了据说很多人用的semantic ui的后续版本fomantic ui，仍然是差不多的感觉。

在bulma之前，又研究了许许多多minimal css framework(?)和许多classless css framework(?)，很多都写的很巧妙，也很管用，但是很难找到一个真正能够满足需求的。

## 关于bulma

bulma的美，来自于一贯和精简的设计。简单的布局之后，将一些元素利用简单的js调动起来，就足以满足我并不高的需要了。

## Python学习下一步

Python的学习间断了两天，明天开始要继续啃了。出于对学过的内容的巩固，准备好好研究一下Flask常用的extension。下一步，准备把FastAPI的学习摆上议程，最近感觉到，学习的过程中，最好的方式确实是“learn by doing and learn by hard way”。上一周，基本上一直都在用Flask本身，Flask不怎么提供的功能，就通过自己想办法来实现，也学到了不少东西，下一步学习成熟的第三方库的时候，自己学过的东西，应该会起到很大的作用。

还有就是，也想利用闲暇的时间，把Javascript好好再学学，把之前不太会的东西都解决掉，顺便说一下，原生的js真的越来越好用了。难怪jquery已经混不下去了。



```javascript
document.addEventListener('DOMContentLoaded', () => {
            document.querySelector('#burger').addEventListener('click', () => {
                document.querySelector('#nav-links').classList.toggle('is-active')
            })
        })
```



 **Peace**