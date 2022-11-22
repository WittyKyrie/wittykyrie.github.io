---
title: Update和`FixedUpdate`
author: wittykyrie
date: 2022-11-11 09:00:00 +0800
categories: ['Update']
tags: ['Update','FixUpdate']
render_with_liquid: false
---

如果你已经接触过Unity的脚本，你一定对 `Update（）` 函数感到很熟悉，因为我们游戏的大部分逻辑都将在这个方法里面实现。如果你接触Unity有一段时间，那么你大概率会接触到 ``FixedUpdate`()`方法。

相对于`Update`的每一帧进行执行，``FixedUpdate`()`的调用采用的是完全独立的计时器，这个独立的计时器将于Unity的物理系统同步。

因此，我们常常将使用`Rigidbody`的逻辑放置在`FixedUpdate`中进行调用。例如向物体施加力，移动或者跳跃等操作。

以上可能是选择何时使用不同的`Update`方法的结论，但是你最好要了解其背后发生的原理，以及如果不正确的使用会发生什么样的后果。

## FixedUpdate的运行原理

因为`FixedUpdate`由于使用的是独立的计时器来进行调用，因此在一帧的时间内，`FixedUpdate`有可能调用一次，两次，或者是一次都不调用，这完全取决于游戏的帧率。

举个例子，假如游戏的运行速率是60帧每一秒，而`FixedUpdate`运行是40次每一秒的话，这意味着，总有20帧的时候，游戏并不会运行`FixedUpdate`的逻辑，如下图所示。

![Alt text](/assets/2022-11-21/2.png){: .shadow style="max-width: 70%" }

同样的，假如游戏运行的速率是30帧每一秒的话，而`FixedUpdate`运行是50次每一秒的话，代表每秒至少有20帧执行调用两次`FixedUpdate`。

听完上面两段描述，你可能会认为，`FixedUpdate`的执行顺序也是根据游戏运行的帧率来进行的，但Unity的脚本生命周期则不是这样，而是按照特定的逻辑来进行运行。

在一个完整的脚本周期当中，`FixedUpdate`永远先于`Update`进行调用，但是在执行里面步骤之前需要进行时间上的判断。

![Alt text](/assets/2022-11-21/1.png){: .shadow style="max-width: 40%" }

1. 首先Unity会计算两帧之间的间隔DeltaTime，然后将其加到总时间Time内
2. 然后Unity会对FixedTime进行判断，判断其落后于Time的时间是否小于`FixedUpdate`的执行间隔FixedDeltaTime（默认是0.02s）
3. 如果落后的时长大于执行间隔则会对`FixedUpdate`进行调用，并将执行间隔FixedDeltaTime加到总时长FixedTime里面，之后再执行步骤二
4. 如果落后的时长小于执行间隔，则会按顺序调用Update，并重复步骤一

## 如何修改FixedUpdate的速率

在Unity当中我们可以点击**Project Setting -> Time -> FixedTime Step**来进行控制。如图所示。
![Alt text](/assets/2022-11-21/3.png){: .shadow style="max-width: 80%" }

或者我们可以在脚本中用代码进行控制。

```c#
Time.fixedDeltaTime = 1 / 50;//0.02s
```

## FixedUpdate速率在不同情况可能遇到的问题和好处

### FixedUpdate速率过慢

当`FixedUpdate`的速率在10帧每秒的时候，如果我们将控制人物移动的代码放在`FixedUpdate`当中进行实现，就会发现物体移动起来会十分的生涩。且由于每一秒只对`FixedUpdate`进行10次的调用，物理引擎只能用更少的数据来进行计算，因此精度会降低。

但于此相反的是，我们的整个游戏的性能开销则会有显著的提升。

### FixedUpdate速率过快

和上面所说的一样，当`FixedUpdate`速率过快的时候，物体移动起来会更加的流畅，且每一秒钟调用的次数越多，所获得的数据越多，处理的精度就会越高。但是游戏的性能开销则会有着显著的提升。甚至在极端情况下可能会导致游戏直接崩溃的情形。

原因是因为，游戏运行的帧率很大程度的取决于CPU的计算量，计算量越高，帧率越小，因此当我们的`FixedUpdate`速率过高时，每一帧将会多次执行我们的物理逻辑，从而导致计算量过大，所以在下一帧的时候，我们的帧率则会下降，但是当帧率下降的时候，我们又需要调用更多的`FixedUpdate`来追赶上FixedTime和Time之间的距离，参考上面提到的`FixedUpdate`的运行原理，所以就会形成恶性循环，当这样恶性循环持续一段时间后，Unity会因为过于卡顿而导致游戏的崩溃。

但是Unity显然也不会允许这样的事情发生，所以在刚刚提到的菜单的第二行。也就是Maximum Allowed Timestep当中，我们可以设置`FixedUpdate`调用的最大速率，从而防止我们因为物理引擎导致的计算量的无限增长。

![Alt text](/assets/2022-11-21/3.png){: .shadow style="max-width: 80%" }

## 该把哪些东西放在FixedUpdate当中

一般来说，将物体通过刚体的方式进行移动的逻辑，以及持续施加力的逻辑是要放到`FixedUpdate`里面的，这样可以与Unity的物理系统保持一致，即使游戏突然卡了，导致帧率下滑，也能保证基于物理的移动操作是保持同步的，因为`FixedUpdate`的调用频率是绝对的。而对于单次的力的施加，或者说基于物理逻辑的判断，比如说射线检测放在Update当中进行更新是完全没有问题的。

