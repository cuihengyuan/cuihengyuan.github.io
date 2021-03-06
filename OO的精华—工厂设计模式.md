##                           OO的精华—工厂设计模式（一）—简单工厂和工厂方法

本系列有三篇文章

1.工厂设计模式（一）—简单工厂方法和工厂方法

2.工厂设计模式（二）—抽象工厂方法和与工厂方法的区别

3.工厂设计模式（三）—工厂设计模式在Android开发中的应用

本文参考经典书籍《Head First 设计模式》

### 前言：

很多初学者小伙伴在学习工厂设计模式之前都有一点疑惑，为什么这个常常听说的神秘莫测的工厂模式可以以其他的方法创建对象而不用new 关键字，？？？？图片

其实没这么神秘，只要学过一点java基础的同学都知道，java就那十几个常用关键字，创建一个对象就是new 关键字，没有其他的的关键字可以创建对象 ，那么究竟是怎么回事呢？别急接下来就来看看着“传说中”的工厂模式

### 一，“new”关键字有啥“问题”吗？

![img](https://img.doutula.com/production/uploads/image/2019/05/13/20190513727868_ErOujh.jpg)

答案是：当然没有问题啊（诞生了十几年的java关键字怎么可能有问题.................）。你们现在的心情：我靠，这不是标题党吗？

且慢。**这里的意思并非是new关键字本身的作用有问题，而是我们在使用它创建对象的时候代码有“问题”**

我们都知道核心功能代码（就是一个程序核心功能的主要完成部分，一般逻辑比较复杂，不好也一般不会去大幅度修改它）如果绑着具体的类会使代码脆弱，缺乏弹性，有点抽象，所以直接上代码

```java


//牛奶基类
class Milk {

    void prepare() {
    //牛奶准备
    }

    void wrap() {
        //牛奶包装
    }

}

class ChocolateMilk extends Milk {

}

class StraBerryMilk extends Milk {

}

class BananaMilk extends Milk {


}


class main {
    public static void main(String[] args) {

        boolean isChocolate = false;
        boolean isStraBerry = false;
        boolean isBanana = true;

        Milk milk;

        if (isChocolate) {
            milk = new ChocolateMilk();
        } else if (isStraBerry) {
            milk = new StraBerryMilk();
        } else if (isBanana) {
            milk = new BananaMilk();
        }
    }

}

```



想要利用java**多态性**，要创建一个有着一群相关具体类的时候，我们通常就会这样写，不同的“人”，要在运行时才可以知道到底实例化哪一个。**但是如果一旦需要删除或者添加新的具体类时，我们就必须要重新打开这段代码然后修改。**也许你可能认为修改就修改呗，这里又不复杂........................  

emmmm................但是这只是最最最最简单的一个示例而已啊，真正的大项目中，代码量以万行为单位，具体类多到爆炸，逻辑很复杂，而且如果不止一个地方需要这样创建对象的话，那么还要去一个一个改，如果不小心改错了那么很可能程序就crash了。

这样的代码我们就认为可扩展性不高，**一个好的程序设计应该对扩展开放，对修改关闭。**

![img](https://img.doutula.com/production/uploads/image/2019/06/14/20190614486724_XfIKFg.gif)

很多小伙伴就要问了，这段创建对象的代码无论如何是必须存在的，难道不用这段代码？当然不是。

**但是我们可以将实例化的具体类的代码从应用中抽离封装，成为一个独立的部分来提供调用**，这是很重要的一个思想。



所以我们创建这样一个方法

```java
   public Milk GetMilk(Kinds kinds) {
        Milk milk;
        switch (kinds) {
            case Banana:
                milk = new BananaMilk();
            case Chocolate:
                milk = new ChocolateMilk();
            case StraBerry:
                milk = new StraBerryMilk();
            default:
                milk = new ChocolateMilk();
        }
       //创建牛奶对象后对牛奶进行加工
       milk.prepare();
        milk.wrap();
       return milk;
    }

```

这里的Kinds是一个枚举类

```java
enum Kinds {

    Chocolate,
    StraBerry,
    Banana
}
```

这样的话，需要创建牛奶对象的时候就传入想要的类型然后调用方法就可以了。

但是问题还是没有彻底解决，如果要添加和删除一些种类的牛奶那么还是要打开这个方法修改代码。

现在我们知道无论是那种类型的牛奶，prepare和wrap都是必须的，不变的（假设这个时候不变），变化的就是创建的牛奶的类型。

所以现在最好将创建牛奶对象的代码移动到这个方法之外，让另一个对象来专职创建牛奶对象。

我们称这个**对象叫做工厂**。

工厂处理创建对象的细节，而需要创建对象的时候就让工厂去做。

```javascript
//牛奶工厂
class MilkFactory {


    public Milk CreatMilk(Kinds kinds) {
        Milk milk;
        switch (kinds) {
            case Banana:
                milk = new BananaMilk();
            case Chocolate:
                milk = new ChocolateMilk();
            case StraBerry:
                milk = new StraBerryMilk();
            default:
                milk = new ChocolateMilk();
        }
        return milk;

    }


}
```



我们把刚刚的得到牛奶的方法改为一个专门的类（牛奶商店），这个类不仅可以创建牛奶，还可以对牛奶进行一系列操作，提供了两个空方法，卖牛奶和牛奶收钱

```javascript
class MilkStore {
    MilkFactory milkFactory;

    public MilkStore(MilkFactory milkFactory) {

        this.milkFactory = milkFactory;
    }

    public Milk GetMilk(Kinds kinds) {
        //这里就把new操作符替换为工厂对象创建对象的方法
        Milk milk = milkFactory.CreatMilk(kinds);
        milk.prepare();
        milk.wrap();
        return milk;
    }

    public void sell(Milk milk) {
        //牛奶商店卖牛奶
    }

    public void charge(Milk milk) {
        //牛奶商店根据牛奶收钱

    }

}
```

到这里我们一开始的问题就解决了，我们用工厂对象的创建对象的方法来代替new 关键字，让牛奶商店不需要关心是什么牛奶，全部交给工厂去创建。这其实就在一步步**解耦**

现在如果要添加或者删除牛奶品种，只需要修改工厂就可以了，其他地方不需要修改。

这里可能很多朋友就要问了，你这样做和之前没有工厂之前的那个创建牛奶对象的方法有什么区别呢？还不是一样的逻辑，而且你这个还多了一个类。

表面上看起来确实是这样，但是注意，这里我为了好讲解，把牛奶类写的非常简单。但是实际情况是一个工厂要创建一个牛奶需要很多的步骤和原料，也就是一系列的其他操作，如果这些在工厂的方法里面进行会逻辑清晰，但是

如果像之前的方法那样直接new 的话还要进行一系列操作，这就显得很不聪明了。

![img](https://img.doutula.com/production/uploads/image/2018/02/16/20180216749013_ajwMmW.png)

这个工厂我 们就称作**简单工厂**



好了，现在看似已经可以应对牛奶的删除和添加了，但是还不不够。

现在，假如商店扩大，需要在其他的地方开店，不同的地方有不同口味，需要不同的原料来增加牛奶口味，也就是说工厂不止一个，需要三个工厂来制作不同风味的牛奶，每个工厂都有自己的那种原料。

那么，可以这样写啊，用三个工厂在商店的对象中，但是三个还好，如果多了呢？需要用很多的if 或者swich判断。**这是我们不想看到的**。

或者说写三个工厂，分别给三个地方的商店用，但是这样就使三个商店类型都差不多只是工厂变化了，代码复用性不高，而且如果这样的话其他的商店除了用你规定的工厂来创造特定的牛奶之外，你身为主店是没办法控制其他的分店的其他流程的，就是说，分店可以不包装，用其他的的自己的做法来创建自己的牛奶。

比如这个分店：

```javascript
class AnotherMilkStore {
    AnotherMilkFactory anotherMilkFactory;

    public MilkStore(AnotherMilkFactory anotherMilkFactory) {

        this.anotherMilkFactory = anotherMilkFactory;
    }

    public Milk GetMilk(Kinds kinds) {
        Milk milk = anotherMilkFactory.CreatMilk(kinds);
        //这个分店偷工减料，把prepare这一步删了
        //milk.prepare();
        milk.wrap();
        return milk;
    }

    public void sell(Milk milk) {
    }

    public void charge(Milk milk) {
//可能另一个分店收钱要便宜
    }

}
```



这个咋办啊？我一个主店竟然无法控制分店的行为（除了牛奶种类之外），万一分店做一些小把戏把我这个主店名誉破坏了可咋办，万一把我反超了可咋办？

![img](https://img.doutula.com/production/uploads/image/2019/07/03/20190703146757_UBEncQ.jpg)

这是我们不想看到的。所以有没有一种模式可以让你的分店有一定的自由度可以做自己的特色，但是又必须要有主店的一些必须的步骤呢？（比如prepare和warp)  。就是让主店可以控制分店必须遵守一些步骤。

 **工厂方法可以办到**

直接上代码

```java
abstract class MilkStore {


    public Milk getMilk(Kinds kinds) {
        Milk milk = CreatMilk(kinds);
       
        //无论分店做的那种牛奶，都要走这两个步骤
        milk.prepare();
        milk.wrap();
        return milk;
    }

    //这个方法就是工厂方法
    abstract Milk CreatMilk(Kinds kinds);
}
```

  就是把牛奶商店改为一个抽象基类。

里面有一个抽象的方法CreatMilk返回牛奶对象，这个方法子类必须重写，实现自己地区的milk制作。

在GetMilk方法里面并不知道CreatMilk返回的是那种Milk，但是又对milk做出了操作，这就叫**解耦**



继承这个抽象商店，实现CreatMilk方法，写入分店自己的牛奶风味的逻辑，**超类不管细节，通过实例化正确的milk类，子类自行执行一切，这就是工厂方法**

这样主店就可以控制分店的必须步骤并保持分店一定的自由度和弹性。

![img](https://img.doutula.com/production/uploads/image/2017/10/22/20171022673435_XJynQY.jpg)



可能到这里还不能体会到工厂方法的强大之处，比较这里只有一个例子而已，下篇文章会更具体的讨论。



