---
title: Update和FixedUpdate
author: wittykyrie
date: 2022-11-11 09:00:00 +0800
categories: ['Update']
tags: ['Update','FixUpdate']
render_with_liquid: false
---

如果你已经接触过Unity的脚本，你一定对 `Update（）` 函数感到很熟悉，因为我们游戏的大部分逻辑都将在这个方法里面实现。如果你接触Unity有一段时间，那么你大概率会接触到 `FixedUpdate()`方法。

相对于`Update`的每一帧进行执行，`FixedUpdate()`的调用采用的是完全独立的计时器，这个独立的计时器将于Unity的物理系统同步。

因此，我们常常将使用`Rigidbody`的逻辑放置在`FixedUpdate`中进行调用。例如向物体施加力，移动或者跳跃等操作。

以上可能是选择何时使用不同的`Update`方法的结论，但是你最好要了解其背后发生的原理，以及如果不正确的使用会发生什么样的后果。

# FixedUpdate的运行原理

因为`FixedUpdate`由于使用的是独立的计时器来进行调用，因此在一帧的时间内，`FixedUpdate`有可能调用一次，两次，或者是一次都不调用，这完全取决于游戏的帧率。

举个例子，假如游戏的运行速率是60帧每一秒，而`FixedUpdate`运行是50次每一秒的话，这意味着，总有10帧的时候，游戏并不会运行`FixedUpdate`的逻辑。

同样的，假如游戏运行的速率是30帧每一秒的话，而`FixedUpdate`运行是50次每一秒的话，代表每秒至少有20帧执行调用两次`FixedUpdate`。

听完上面两段描述，你可能会认为，`FixedUpdate`的执行顺序也是根据游戏运行的帧率来进行的，但Unity的脚本生命周期则不是这样，而是按照特定的逻辑来进行运行。

在一个完整的脚本周期当中，`FixedUpdate`永远先于`Update`进行调用，但是在执行里面步骤之前需要进行时间上的判断。

