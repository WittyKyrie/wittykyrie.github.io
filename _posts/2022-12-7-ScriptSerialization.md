---
title: Script serialization
author: wittykyrie
date: 2022-12-7 09:00:00 +0800
categories: ['Unity']
tags: ['Script','serialization']
render_with_liquid: false
---

`Serialization`序列化是指将数据结构或者GameObject状态转变成可以被Untiy可以储存以及重建的格式。

在 Unity 项目中组织数据的方式会影响 Unity 序列化数据的方式，这可能会对项目的性能产生重大影响。本页概述了 Unity 中的序列化以及如何针对它优化项目。

## Serialization rules序列化规则

Unity中的序列化器是专门为在运行时高效运行而设计的。正因为如此，Unity中的序列化与其他编程环境中的序列化表现不同。Unity中的序列化器直接作用于c#类的字段，而不是它们的属性，所以有一些规则你的字段必须符合才可以序列化。下面的部分概述了如何在Unity中使用字段序列化。

要使用字段序列化，你必须保证字段：
+ Is public, or has a SerializeField attribute
+ isn’t static
+ isn’t const
+ isn’t readonly
+ Has a field type that can be serialized:
    + Primitive data types (int, float, double, bool, string, etc.)
    + Enum types (32 bites or smaller)
    + Fixed-size buffers
    + Unity built-in types, for example, Vector2, Vector3, Rect, Matrix4x4, Color, AnimationCurve
    + Custom structs with the Serializable attribute
    + References to objects that derive from UnityEngine.Object
    + Custom classes with the Serializable attribute. (See Serialization of custom classes).
    + An array of a field type mentioned above
    + A List<T> of a field type mentioned above 
  