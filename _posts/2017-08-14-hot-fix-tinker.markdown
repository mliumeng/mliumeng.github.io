---
layout:     post
title:      "热更新- 微信的Tinker 基于AndFix的阿里百川HotFix2.0 "
subtitle:   "整理 整理 整出真理。"
date:       2017-08-14 9:14:00
author:     "刘蒙"
header-mask: 0.3
header-img: "img/post-bg-hot-fix.jpg"
catalog:    true
tags:
    - android
    - hotfix
    - Sophix
    - tinker
    - 热更新
    - 热修复
---

# 热更新概念？
大家常说的热更新：热更新就是不用发布新版本，用下发补丁的方式更新或修改已发布的应用出现的新需求活 BUG 。
# 为啥要用热更新？

1. 用了热更新会不会觉得自己变牛 B 了？
2. 用了热更新找工作都多个筹码了
3. 已发布版本出现 BUG 可以云修复了。多吊（不用发布新版本）
4. 产品有新需求也能云部署了。屌不屌？（不用发布新版本）

正儿八经的讲，热修复必将是一个程序员的基础本领。产品对用户体验越来越重视。包括我们用户对产品的体验要求也是越来越高。

## 从修复 BUG 的层面讲

尽管我们开发者认真细致长得帅，但也不能保证百分之百没有 BUG 。尽管上线前我们的测试组认真负责的反复测试，却总难以揣测千万用户的逻辑。所以 BUG 真的会在上线的版本上出现。这就不可避免的影响到了一部分用户的使用体验。产品，研发，测试以及公司的每一个人都不愿看到用户抱怨的。所以：**热修复很重要**热修复就能避免了上线后遇到 BUG 不能及时修复的尴尬。

## 从产品新需求的层面

产品有了新需求，正常的流程评测 - 开发 - 测试 - 上线 - 得到数据模型 - 精准评估 - 发现不如不上 - 下一版本去掉- ----- 漫长的流程中终于去掉了。
而我们有了热修，本着对用户负责的态度我们这些流程其实并不能少，但是我们可以灰度下发，得到数据模型 - 精准评估 决定下一版本是否正式加入。能节省 6 - 30 天的时间。

# 流行的热更新框架
热更新在中华大地百花齐放。各领风骚。如下 ↓
还有很多很多，，，下面这个表示阿里给的。当然数据看起来是阿里的好。
【仅供参考】

|平台|阿里云HotFix|AndFix|Tinker|Qzone|美团Robust|
|:------ | :------ |:------ | :------ | :------ | :------ |
| 即时生效|yes|yes|no|no|yes|
|性能损耗|较小|较小|较大|较大|较小|
|侵入式打包|无侵入式打包|无侵入式打包|依赖侵入式打包|依赖侵入式打包|依赖侵入式打包|
|Rom体积|较小|较小|较大|较小|较小|
|接入复杂度|傻瓜式接入|比较简单|复杂|比较简单|复杂|
|补丁包大小|较小|较小|较小|较大|一般|
|全平台支持|yes|yes|yes|yes|yes|
|类替换|yes|yes|yes|yes|no|
|so替换|yes|no|yes|no|no|
|资源替换|yes|no|yes|yes|no|

那我们就说说这些有强大技术团队支持的几个：

- 阿里云HotFix（Sophix）:  AndFix 的升级版 ---> 阿里百川 HotFix 再升级 ---> 是阿里云的 HotFix（Sophix）

> 阿里云HotFix 
优势：
1. 补丁即时生效，不需要应用重启；
2. 补丁包同样采用差量技术，生成的 PATCH 体积小；
3. 对应用无侵入，几乎无性能损耗；
4. 傻瓜式接入。

- Tinker 

>  腾讯的Tinker 
优势：
文档完善，接入简单。代码开源。
缺点：（虽然缺点多，但是人家开源，这点我喜欢）
1. Tinker不支持修改AndroidManifest.xml，Tinker不支持新增四大组件；
2. 由于Google Play的开发者条款限制，不建议在GP渠道动态更新代码；
3. 在Android N上，补丁对应用启动时间有轻微的影响；
4. 不支持部分三星android-21机型，加载补丁时会主动抛出"TinkerRuntimeException:checkDexInstall failed"；
5. 对于资源替换，不支持修改remoteView。例如transition动画，notification icon以及桌面图标。
6. 不能及时生效
7. 占用Rom体积；这边大约是你修改Dex数量的1.5倍(dexopt与dex压缩成jar)的大小。
8. 一个额外的合成过程；虽然我们单独放在一个进程上处理，但是合成时间的长短与内存消耗也会影响最终的成功率。


**不要以为我是黑腾讯，找了这么多缺点。。。这都是腾讯官方文档上写的。其他的比较保守不列自己缺点，或者我就是不痛不痒的缺点**
**再次强调：仅供参考！**
# 都是咋实现的？

## 先说Tinker
它的名字来至 Dota 中的地精修补匠，我们希望发版本可以像它一样做到无限刷新。
![Cartoon Tinker](https://github.com/mliumeng/mliumeng.github.io/blob/master/img/article/hotfix/cartoon_tinker.jpg)
Tinker 的方案来源 gradle 编译的 instant run 与 buck 编译的 exopackage . 他们的思想都是**替换新的Dex** . 即我们使用了新的 Dex ，那样既不出现 Art 地址错乱问题，在 Dalvik 也无需插桩。
但是 instant run 是针对编译期，他可以将最后生成的所有变化直接考到手机端。对于线上方案，不可行。
![Tinker](https://github.com/mliumeng/mliumeng.github.io/blob/master/img/article/hotfix/icon_tinker.jpg)
 [Tinker 接入指南](https://github.com/Tencent/tinker/wiki/Tinker-接入指南)
大致是如图几种方案，而 Tinker 采用的是腾讯自己搞的算法 DexDiff 。
 [DexDiff 算法详解](https://www.zybuluo.com/dodola/note/554061) 这篇文章写得很详细。

## 阿里云的HotFix（Sophix）
[HotFix2.0（Sophix） 接入指南](https://help.aliyun.com/document_detail/53240.html) 详细的接入文档
[HotFix（Sophix） 原理](https://yq.aliyun.com/articles/103527)

# 未来

> 引用阿里云的话

**“ 未来，无限可能！** 
我们对于未来是很乐观的，Android的热修复领域不仅不会受到封杀，反而还有很大的发展空间。我们正在尝试支持各大加固厂商，目前阿里聚安全修复已经支持了Sophix，热修复结合安全加固，将会使得app的稳定性和安全性更加坚不可摧。甚至后续还可以与系统厂商合作，对系统app乃至系统组件进行修复，这样就可以避免频繁OTA升级。

因此，热修复所能发挥是价值将是十分巨大的。热修复还可以与其他领域进行碰撞，引发无限的可能性。在这里，我们欢迎所有应用厂商以及ROM厂商与我们合作，共同使得Android的生态更加完善。 **”**
 