---
title: "教师节特辑：图形学自学路线图（从 OpenGL 到 WebGPU）"
source: "https://mp.weixin.qq.com/s/_KFjTpJjdp07klJsHNvj8A"
author:
  - "[[东汉书院]]"
published:
created: 2026-11-18
description: "免费的永远是最贵的，反正学不会也跟他们没关系，毕竟是免费。"
tags:
  - "clippings"
---
东汉书院 内核观察 *2026年9月9日 08:30*

> 免费的永远是最贵的，反正学不会也跟他们没关系，毕竟是免费。实在是学不明白也没关系，东汉书院的《WebGPU萌谷手册》已经高调发布 `第1个` 视频啦

图形学这条路，是很多老师用三十年铺出来的。 **明天就是教师节，也是我们东汉书院开学季活动的最后一天了** 。今天想聊一聊图形学里那些“老师”。

![图片](https://mmbiz.qpic.cn/mmbiz_png/gnR2ZrVNns4oyib4AdqgWt5jZloaKR5eNeDobMNZfjxuceGxy2EMFdxLq2IHLe0yeqjwgcazibbUZbMWbtbPkgZdKuLuGn6VeJXwic4myk5wR4/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

来呀，快上车呀，每次的活动内容都有细微的差异，你就真的想错过这次绝佳的上车机会吗？

![图片](https://mmbiz.qpic.cn/mmbiz_png/gnR2ZrVNns6r12BgRRGkOj0hxtp73lQqHhClfgoNLI4SPXo453sENtRdIsdkSM9j6HN1VxlV6HS2ffynaGiaG8BvYZQ3rC2lphUfLuofugyA/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=1)

想学图形学的人，多半都有过这样的纠结：现在是不是该直接学 WebGPU？Vulkan 是不是更高级？OpenGL 是不是早就过时了，学了白学？

这种“追新”的心态，恰恰是自学路上最大的坑。图形学三十多年积累的知识体系，不会因为换一个新 API 就能跳过。OpenGL 之所以到今天仍是全世界大多数图形学课程的入门语言，不是因为它先进，而是因为它把“图形学到底要解决什么问题”讲得最清楚。

这篇文章就是一份自学路线图：从 OpenGL 出发，一路走到 WebGPU，图形学到底在学什么、这条路为什么要这么走、不同目标的人该怎么选。

## 图形学到底在学什么

图形学不是“学一个 API”。OpenGL、Vulkan、WebGPU 都只是工具，图形学的核心始终是一个问题：如何把三维世界，描述成屏幕上的像素。

它由几条主线组成：

- • 渲染管线：顶点怎么变成三角形，三角形怎么变成像素（光栅化），像素怎么着色；
- • 数学基础：线性代数（向量、矩阵、变换）是绝对主力，微积分与几何是左右手；
- • 着色与光照：颜色模型、光照方程、材质、纹理；
- • 场景管理：相机、层次结构、裁剪与剔除；
- • 现代扩展：GPU 并行计算、实时全局光照、神经渲染。

两个字：API 会过时，但“渲染管线 + 数学”这套骨架，从 1992 年到现在几乎没有变过。先搭骨架，再换武器。并且 **最可怕的是，东汉书院具备成体系的一条龙的教程** 。

## 图形 API 的演进

图形学入门最难的部分之一，是被一堆名词绕晕：OpenGL、WebGL、Vulkan、Metal、DirectX、WebGPU……它们不是平行关系，而是一条有先后、有继承的时间线。

- • 1992 年：SGI 发布 OpenGL 1.0，图形学第一次有了跨平台的标准 API；
- • 2004—2008 年：OpenGL 2.x 引入着色器（GLSL），固定管线让位可编程管线，3.0 确立 core profile；
- • 2011 年：WebGL 1.0 发布（基于 OpenGL ES 2.0），浏览器第一次能调用 GPU；
- • 2014—2016 年：Metal（苹果）、DirectX 12（微软）、Vulkan（Khronos）相继问世，“显式控制”成为原生 API 的方向；
- • 2017 年：OpenGL 4.6 发布，成为 OpenGL 最后的主要版本；同年 WebGL 2.0 发布（基于 OpenGL ES 3.0）；
- • 2022 年：WebGL 2.0 获得所有主流浏览器支持；
- • 2023 年：Chrome 113 默认启用 WebGPU，浏览器图形进入新阶段；
- • 2024—2026 年：Safari 17.4 完全支持、Firefox 跟进；W3C 规范进入候选推荐阶段，全球约 84% 的页面浏览已完全支持 WebGPU。

这条时间线说明三件事：

- • OpenGL 定义了“图形 API 应该长什么样”的范式，后来的 API 都在它的肩膀上做减法或加法；
- • 2010 年代之后，行业分成两条线：原生 API 走向显式控制（Vulkan/Metal/DX12），Web 从 WebGL 走向 WebGPU；
- • WebGPU 是第一次让 Web 和原生在“现代 GPU 抽象”上对齐——这也是它被称为“图形学下一代”的原因。

学习图形学，就是一级一级台阶往上走。还有什么好说的，干、梭哈就完了！真男人，要梭哈， **这是对自己的投资，且是一个无风险的投资** ！

## OpenGL是最好的入门老师

一个常见的说法是：OpenGL 过时了，别学。更准确的说法是：OpenGL 作为“生产 API”确实在退休，但作为“教学 API”，它依然是全世界图形学课程的主流选择。

三个理由：

- • 它只有一层抽象，问题暴露得最清楚。学 Vulkan 或 WebGPU，第一周你往往在跟队列、命令缓冲、管线布局搏斗，而不是在学习渲染本身。OpenGL 让你第一周就能画出三角形，把注意力放在“渲染管线是怎么转的”上；
- • 资料密度全网最高。各高校课程、经典教材几乎都以 OpenGL 为例，遇到问题，答案随手可查；其实感觉主要是因为高校老师主攻理论，所以他们只掌握了OpenGL，导致了跟着他们学就是OpenGL作为起手式了。
- • 概念可以平移。在 OpenGL 里理解的顶点缓冲、着色器、纹理、帧缓冲，在 WebGPU 里几乎一一对应，只是换了写法。学完 OpenGL 再看 WebGPU，你会发现它更像“一个更现代的 OpenGL”。

换句话说：OpenGL 不是终点，但它是大多数人的最佳起点。

## 从 OpenGL 到 WebGPU

### 阶段一：数学与基础（约 2—4 周）

- • 线性代数：向量、矩阵、点乘与叉乘、变换矩阵，这是图形学的第一语言；
- • 动手实现一个软件光栅化器：画三角形、做深度缓冲。这一步能让你从“第一性原理”理解 GPU 在做什么。

`想要快速掌握可以选择：东汉书院的《右手坐标系中的数学》这个课程，讲的就是上面的内容，不是盲目的学，而是只学对你工作有用的！`

### 阶段二：OpenGL 入门（约 4—8 周）

- • 掌握：窗口与上下文、着色器、纹理、变换、坐标系统、光照（Phong / Blinn-Phong）；
- • 里程碑：完成一个带光照、纹理和相机控制的 3D 场景。

想要快速掌握，同样东汉书院有着最齐全的OpenGL学习教程，从入门到精通，四大主流平台的OpenGL教程，任君挑选：Android、iOS、Windows、OSX

### 阶段三：WebGL 作为桥（可选，约 2—4 周）

- • 理解 OpenGL 概念如何搬到浏览器：GLSL 与 JavaScript 的交互、兼容性差异；
- • 做一个能分享的小作品：3D 小游戏或数据可视化。

`东汉书院通过《骨骼动画WebGL版》演示了WebGL中最核心的概念和样本了，一个教程打通WebGL`

### 阶段四：WebGPU（约 4—8 周）

- • 用 WGSL（WebGPU 的着色语言）重写之前学过的概念；
- • 掌握：设备与队列、管线、缓冲区、Bind Group、计算着色器；
- • 延伸：用 WebGPU 做 GPU 通用计算，比如流体模拟、浏览器端 AI 推理。

`东汉书院已经正式发布了《WebGPU萌谷手册》第一课时了，预计本月底将彻底更新完毕，2025年12月，我们已经发布过《Nanite的WebGPU实现版》了，如果看过的同学应该知道通过视频教程学习和通过传统书籍学习之间有多大的差距了吧！`

### 阶段五：按兴趣深化

- • 实时渲染方向：PBR、阴影、全局光照；
- • 引擎方向：Unity / Unreal，或自研小型引擎；
- • 研究方向：光线追踪、点云处理、神经渲染。

`嘿，别说，东汉书院全部都有，PBR、Nanite、Lumen、《游戏引擎开发》每一个都在这条学习路线上！你想不起飞都难了！`

整个周期大约 ` 6 个月到一年` 。别赶进度，里程碑比天数重要。

## 怎么选路径

- • 网页 3D / 数据可视化 / 前端进阶：WebGL 入门 → WebGPU 深耕；
- • 游戏开发：OpenGL 打基础 → 引擎（Unity / Unreal）→ 按需补 Vulkan / DirectX 12；
- • 图形学算法 / 研究：数学 + 软件光栅化器 → OpenGL → 论文复现（可配 WebGPU）；
- • 工业软件 / GIS / 医疗可视化：OpenGL 系优先，兼容性是第一考量；
- • AI 与 GPU 计算：图形基础 → WebGPU / Vulkan 计算管线。

图形学的学习，很像一场接力。OpenGL 是从 1992 年一路跑过来的老教师，它教了一代又一代人“像素是怎么来的”；WebGPU 是新一代接棒者，它更懂今天的 GPU，也更接近 Web 与原生交汇的未来。

路线图记不住没关系，记住一句话就够： **先理解管线，再选择武器。** OpenGL 让你理解，WebGPU 让你前进。

### 课程进度

**《游戏引擎开发实践》** `DLC` 方面，第一波Nanite和Lumen的教程 **更新完毕** ！后续进一步更新和补充其他章节的内容和可交互式教学工具来使得基础薄弱的同学能够更好地理解Lumen和Nanite！

![图片](https://mmbiz.qpic.cn/mmbiz_png/gnR2ZrVNns5TFVAXKfRzPnrOeymCHe9ujpGjxOGqk4eg2nsdWE1nxUzzGQg0vQTwuHupXmr1Pib0lO8PnOEh9whKDcgWmBGFOPqOCPXtiaG2E/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=3)

2026年为同学们安排的小课程进度：

- • 《Metal萌谷手册iOS版》(进行中，已经创建完毕PSO了)
- • 《Metal萌谷手册Mac版》(进行中)
- • 《WebGPU萌谷手册》(进行中，已经开张)
- • 《Vulkan硬件光线追踪》(进行中)
- • 《DXR硬件光线追踪》(进行中)

### 联系我们

- • 图形学技师研讨会1：460926233(满)
- • 图形学技师研讨会2：719446549
- • 微信：battlefireedu
- • 官网：https://edu.battlefire.cn
- • 做智能硬件渲染有瓶颈的欢迎把设备发过来，帮看渲染性能问题和瓶颈
- • GPU厂商欢迎发你们的产品过来，让我们把玩把玩，测评测评

**每天进步一点点，来东汉书院躺平，享不卷游戏人生！**

不干了，我们一起去犯罪吧 · 目录