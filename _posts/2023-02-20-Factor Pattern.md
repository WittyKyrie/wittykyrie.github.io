---
title: 游戏设计模式之工厂模式
author: wittykyrie
date: 2023-02-20 09:00:00 +0800
categories: [Unity, '设计模式']
tags: ['工厂模式']
render_with_liquid: false
---

当我们开发游戏时，我们通常需要创建大量的对象。有时候我们需要手动创建这些对象，但是这种方式容易出现错误，而且难以维护和扩展。在这种情况下，工厂模式就变得非常有用。

工厂模式是一种创建型设计模式，用于创建对象，将对象的创建过程封装在一个类中，以便在需要时创建对象。工厂模式可以通过将客户端代码与实际创建对象的过程分离来提高代码的可维护性、可扩展性和可读性。

在游戏开发中，工厂模式通常用于创建不同类型的游戏对象，例如武器、角色、道具等等。

## 工厂模式的类型
在工厂模式中，通常会使用以下三种类型：

### 简单工厂模式
简单工厂模式是最基本的工厂模式。在简单工厂模式中，我们只需要一个工厂类，它根据客户端的请求返回一个具体的对象实例。简单工厂模式通常只适用于创建单一的对象类型。

### 工厂方法模式
工厂方法模式是一种扩展简单工厂模式的模式，它定义了一个工厂方法接口，用于创建对象，这些对象由子类决定实例化哪一个类。工厂方法模式通常用于创建不同的对象类型，每个对象类型对应一个工厂方法。

### 抽象工厂模式
抽象工厂模式是一种创建一组相关或依赖对象的工厂模式。在抽象工厂模式中，客户端请求一个工厂，并从工厂中获取一个产品族中的一个对象。在抽象工厂模式中，工厂不仅仅是一个单独的类，而是一个由多个工厂组成的层次结构。

### 工厂模式的优点
使用工厂模式有以下优点：

1. 将对象创建过程与客户端代码分离，减少重复代码，提高代码的可维护性和可读性。

2. 隐藏实现细节，避免客户端代码对具体实现的依赖，提高代码的可扩展性。

3. 降低了耦合性，客户端只需要知道需要什么对象，而不需要知道对象的创建细节。

### 工厂模式的缺点
使用工厂模式有以下缺点：

1. 增加了代码的复杂度，需要创建更多的类。

2. 工厂模式通常需要使用额外的代码来实现，这增加了开发时间和成本。

## 工厂模式的应用

### 简单工厂模式

首先我们创建一个道具的基类

```c#
public class Item {
    protected string name;

    public virtual void use() {
        Debug.Log("使用了" + name);
    }
}
```

然后，我们可以定义几个具体的道具类，例如 HealthPotion、ManaPotion、SpeedBoost，它们都是 Item 的子类，并且实现了自己特定的 use 方法。

```c#
public class HealthPotion : Item {
    public HealthPotion() {
        name = "生命药水";
    }

    public override void use() {
        base.use();
        // 具体的使用逻辑
        Debug.Log("恢复了100点生命值");
    }
}

public class ManaPotion : Item {
    public ManaPotion() {
        name = "法力药水";
    }

    public override void use() {
        base.use();
        // 具体的使用逻辑
        Debug.Log("恢复了100点法力值");
    }
}

public class SpeedBoost : Item {
    public SpeedBoost() {
        name = "加速药剂";
    }

    public override void use() {
        base.use();
        // 具体的使用逻辑
        Debug.Log("增加了移动速度");
    }
}

```

接下来，我们可以定义一个简单工厂类 ItemFactory，该类包含一个用于创建道具的静态方法 CreateItem，根据传入的参数不同，创建不同的道具实例并返回。

```c#
public class ItemFactory {
    public static Item CreateItem(string itemName) {
        Item item = null;

        if (itemName == "health_potion") {
            item = new HealthPotion();
        }
        else if (itemName == "mana_potion") {
            item = new ManaPotion();
        }
        else if (itemName == "speed_boost") {
            item = new SpeedBoost();
        }

        return item;
    }
}
```
最后，我们可以在游戏中使用 ItemFactory 来创建道具实例并使用它们。

```c#
// 创建一个生命药水道具
Item healthPotion = ItemFactory.CreateItem("health_potion");
// 使用生命药水道具
healthPotion.use();

// 创建一个加速药剂道具
Item speedBoost = ItemFactory.CreateItem("speed_boost");
// 使用加速药剂道具
speedBoost.use();
```

通过使用简单工厂模式，我们可以将道具的创建过程封装在 ItemFactory 类中，使得代码更加清晰，易于维护和扩展。

### 工厂方法模式

道具类保持相同，首先我们可以定义一个工厂接口 ItemFactory，该接口包含一个用于创建道具的抽象方法 CreateItem，具体的工厂类实现该方法来创建具体的道具实例。

```c#
public interface ItemFactory {
    Item CreateItem();
}
```

我们可以针对每种道具类型，创建具体的工厂类，例如 HealthPotionFactory、ManaPotionFactory、SpeedBoostFactory，它们实现了 ItemFactory 接口，并且实现了自己特定的 CreateItem 方法来创建具体的道具实例。

```c#
public class HealthPotionFactory : ItemFactory {
    public Item CreateItem() {
        return new HealthPotion();
    }
}

public class ManaPotionFactory : ItemFactory {
    public Item CreateItem() {
        return new ManaPotion();
    }
}

public class SpeedBoostFactory : ItemFactory {
    public Item CreateItem() {
        return new SpeedBoost();
    }
}
```

最后，我们可以在游戏中使用具体的工厂类来创建道具实例并使用它们。

```c#
// 创建一个生命药水道具
ItemFactory healthPotionFactory = new HealthPotionFactory();
Item healthPotion = healthPotionFactory.CreateItem();
// 使用生命药水道具
healthPotion.use();

// 创建一个加速药剂道具
ItemFactory speedBoostFactory = new SpeedBoostFactory();
Item speedBoost = speedBoostFactory.CreateItem();
// 使用加速药剂道具
speedBoost.use();
```

### 抽象工厂模式

当使用抽象工厂模式时，通常会涉及到一组相关的产品。在游戏开发中，我们可以将不同类型的武器视为相关的产品，使用抽象工厂模式来创建不同种类的武器。

我们可以定义一个抽象的武器接口 Weapon，包含攻击方法 attack。然后，我们定义两种不同类型的武器：近战武器和远程武器。为了实现抽象工厂模式，我们需要为每个武器类型定义一个工厂接口，并实现相应的工厂类。

```c#
// 抽象工厂接口
public interface WeaponFactory {
    MeleeWeapon CreateMeleeWeapon();
    RangedWeapon CreateRangedWeapon();
}

// 近战武器工厂
public class MeleeWeaponFactory : WeaponFactory {
    public MeleeWeapon CreateMeleeWeapon() {
        return new Sword();
    }

    public RangedWeapon CreateRangedWeapon() {
        return new NullRangedWeapon();
    }
}

// 远程武器工厂
public class RangedWeaponFactory : WeaponFactory {
    public MeleeWeapon CreateMeleeWeapon() {
        return new NullMeleeWeapon();
    }

    public RangedWeapon CreateRangedWeapon() {
        return new Bow();
    }
}
```

然后我们定义具体的武器类，包括剑 Sword、弓 Bow、空近战武器 NullMeleeWeapon 和空远程武器 NullRangedWeapon。

```c#
// 抽象武器接口
public interface Weapon {
    void attack();
}

// 近战武器
public interface MeleeWeapon : Weapon { }

public class Sword : MeleeWeapon {
    public void attack() {
        Debug.Log("使用剑攻击");
    }
}

public class NullMeleeWeapon : MeleeWeapon {
    public void attack() {
        Debug.Log("无近战武器");
    }
}

// 远程武器
public interface RangedWeapon : Weapon { }

public class Bow : RangedWeapon {
    public void attack() {
        Debug.Log("使用弓箭攻击");
    }
}

public class NullRangedWeapon : RangedWeapon {
    public void attack() {
        Debug.Log("无远程武器");
    }
}
```

最后，我们可以在游戏中使用具体的工厂类来创建武器实例并使用它们。

```c#
// 创建一个近战武器
WeaponFactory meleeFactory = new MeleeWeaponFactory();
MeleeWeapon meleeWeapon = meleeFactory.CreateMeleeWeapon();
// 使用近战武器
meleeWeapon.attack();

// 创建一个远程武器
WeaponFactory rangedFactory = new RangedWeaponFactory();
RangedWeapon rangedWeapon = rangedFactory.CreateRangedWeapon();
// 使用远程武器
rangedWeapon.attack();
```


