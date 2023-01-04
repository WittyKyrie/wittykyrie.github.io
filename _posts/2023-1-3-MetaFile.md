---
title: Meta文件详解
author: wittykyrie
date: 2023-1-3 09:00:00 +0800
categories: ['Unity']
tags: ['meta','guid','file']
render_with_liquid: false
---

在Unity当中，当我们将一个资源导入到项目中，Unity将会执行几个操作。
1. 赋予这个资源一个唯一的ID
2. 对资源进行不同的处理
3. 创建一个同名的`.meta`伴生文件，记录1，2步的信息

## Meta文件解析

以下是一个常用的`.meta文件`用文本文件打开后的样子。

```yaml
fileFormatVersion: 2
guid: 374a5cce332c141b1af97e408b4bf3b4
NativeFormatImporter:
  externalObjects: {}
  mainObjectFileID: 100100000
  userData: 
  assetBundleName: 
  assetBundleVariant: 
```
在上面这个文件当中，比较重要的信息就是我们上文说的资源的ID和ImportSetting，我们也将从这两部分来进行了解。

## GUID
当一个文件或者一个文件夹导入到Unity的Asset文件夹的时候，我们会给这个资源一个独特的Id，被称为guid，在Unity当中我们确认资源身份不是靠资源路径也不是靠资源名称而是这个guid，这样的做的目的是，即使一个资源重命名了，或者更改了路径，游戏之间已经存在的引用关系不会因此而被破坏，增加了开发过程中的稳定性。

但是一个guid只对应一个资源，可是像是图集这类的资源里面有很多子资源就无法用guid来确定，Unity会自动为这些子资源创建LocalId，如下图所示。
```yaml
fileFormatVersion: 2
guid: 76f3ef1aba5e2e0408f32a9140fbacd7
TextureImporter:
  internalIDToNameTable:
  - first:
      213: 21300000
    second: fising_indicator_shadow_0
  - first:
      213: 21300002
    second: fising_indicator_shadow_1
  - first:
      213: 21300004
    second: fising_indicator_shadow_2
```

因此，当我们使用版本控制工具，比如svn和git来进行开发时，要特别关注meta文件guid的更改，很多文件的引用丢失都是由于guid的更改而造成的。

## ImportSetting
在Unity当中每一个资源导入需要进行不同的处理，例如对一个纹理文件和对一个CS文件处理的方式是不同的，因此Unity为我们集成了多种的[内置导入器（ImportSetting)](https://docs.unity3d.com/cn/current/Manual/BuiltInImporters.html)

以下是一个Font资源的`.meta文件`中关于ImportSetting的内容，其中的大部分内容都可以在Inspector窗口进行查看。这些记录的数据代表着Unity对这个Font资源进行的处理的参数，由此我们可以知道，如果我们想在不同的项目中按照一定的格式来传资源，可以同时将资源以及它的`.meta文件`一起进行传送。
```yaml
TrueTypeFontImporter:
  serializedVersion: 2
  fontSize: 16
  forceTextureCase: -2
  characterSpacing: 1
  characterPadding: 0
  includeFontData: 1
  use2xBehaviour: 0
  fontNames: []
  fallbackFontReferences: []
  customCharacters: 
  fontRenderingMode: 0
  userData: 
  assetBundleName: 
  assetBundleVariant: 

```



### 参考文章

[Unity官网资源元数据](https://docs.unity3d.com/cn/current/Manual/AssetMetadata.html)

[UInity文件系统的导入](https://blog.uwa4d.com/archives/USparkle_inf_UnityEngine.html)