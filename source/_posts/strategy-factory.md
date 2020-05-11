---
title: 🌀策略模式与工厂模式
keywords: Design Pattern
thumbnail: 'https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511161033.png'
tags: Design Pattern
categories:
  - Design Pattern
date: 2020-02-18 12:37:56
---

以需求为导向，以分析为路径，理解设计模式。

<!-- MORE -->

## 提出需求
开发一个超市收银计算器，计算顾客购买的商品总原价，通过优惠政策计算之后，得到实际需要支付的价格。
背景：超市在不同时间会搞各种活动。有时候打八折，有时候满减，有时候购物积分，有时候甚至又打折又满减。在以后也会陆续推出各种各样的优惠政策。
## 思考与实现
首先，弄清楚需求中变与不变的部分。
变：计算时的算法。不变：由原价计算得到实付价格。
超市会经常变更优惠计算的方式，我们需要注意算法替换与新增的扩展性。
在这里，我们会使用到多种计算方式，但它们都是在做同一件事情——由原价计算得到实付价格。
这里可以应用到策略模式：
> 策略模式：它定义了算法家族，分别封装起来，让它们之间可以相互替换，此模式让算法的变化不会影响到使用算法的客户。    —— 《设计模式：可复用面向对象软件的基础》

我们先来进行实践，首先实现计算的三种算法，分别是：原价购买、八折、满300减100。

```java
abstract class CashSuper {
  public abstract double acceptCash(double money);
}

class CashNormal extends CashSuper {
  @Override
  public double acceptCash(double money) {
    return money;
  }
}

class CashRebate extends CashSuper {
  public CashRebate (double rate) {
    this.moneyRebate = rate;
  }
  private double moneyRebate = 1d;


  @Override
  public double acceptCash (double money) {
    return money * this.moneyRebate;
  }
}

class CashReturn extends CashSuper {
  private double moneyCondition = 0.0d;
  private double moneyReturn = 0.0d;
  public CashReturn (double moneyCondition, double moneyReturn) {
    this.moneyCondition = moneyCondition;
    this.moneyReturn = moneyReturn;
  }

  @Override
  public double acceptCash (double money) {
    double result = money;
    if (money > this.moneyCondition) {
      result = money - Math.floor(money / moneyCondition) * this.moneyReturn;
    }
    return result;
  }
}
```

三个算法的类与方法定义好之后，我们需要创建一个 Context 类来维护来维护对一个 Strategy 的引用。

```java
class Context {
  public CashSuper strategy;
  public Context (Strategy strategy) {
      this.strategy = strategy;
  }
  public double calcCash(double money) {
    return this.strategy.acceptCash(money);
  }
}
```

现在，我们可以使用策略模式完成我们的需求，接下来我们看看主函数部分的代码。

```java
class StrategyMode {
  public static void main(String[] args) {
    double money = 320.3d;
    Context ctx;
    ctx = new Context(new CashReturn(300, 100)); // 注入策略 满300减100
    double ret = ctx.calcCash(money);
    System.out.println(ret);
  }
}
```

此时，我们只需要替换 Context 实例化时接收的策略，即可达到替换优惠计算算法的目的。

### 问题
在上面的代码中，我们每次使用一次策略都要显式地选择创建一个策略实例。这显然可以使用工厂模式来解决。
我们来创建优惠计算策略工厂

```java
class CashFactory {
  public static CashSuper createCashAccept (String type) {
    CashSuper cs = null;
    switch (type) {
      case "正常收费":
        cs = new CashNormal();
        break;
      case "满300减100":
        cs = new CashReturn(300, 100);
        break;
      case "八折":
        cs = new CashRebate(0.8);
        break;
    }
    return cs;
  }
}
```

Context 只要接收一个字符串，由工厂创建实例化的策略对象。

Context:
```java
class Context {
  public CashSuper strategy;
  public Context (String type) {
    this.strategy = CashFactory.createCashAccept(type); // 这里使用工厂创建
  }
  public double calcCash(double money) {
    return this.strategy.acceptCash(money);
  }
}
```

Main:
```java
class StrategyMode {
  public static void main(String[] args) {
    double money = 320.3d;
    Context ctx;
    ctx = new Context("满300减100");
    double ret = ctx.calcCash(money);
    System.out.println(ret);
  }
}
```

这样，利用工厂模式与策略模式相结合，代码更加简单明了，使得收费算法彻底分离。

## Tips

- 策略模式是一种定义一系列算法的方法，从概念上来看所有这些算法都是完成相同的工作，只是实现不同，它可以以相同的方式调用所有的算法，减少了各种算法类与使用算法类之间的耦合。
- 策略模式 Strategy 类层次为 Context 定义了一系列的可供重用的算法或行为。继承有助于析取出这些算法中的公共功能。
- 策略模式简化了单元测试，因为每个算法都有自己的类，可以通过自己的接口单独测试。
- 当不同的行为堆砌在一个类中，就很难避免使用条件语句来选择合适的行为。将这些行为封装在一个个独立的 Strategy 类中，可以在使用这些行为的类中消除条件语句。
- 策略模式封装了变化。策略模式，不仅可以用来封装算法，甚至可以用它来封装几乎任何类型的相同的规则，只要在分析过程中听到在不同时间应用不同的规则，就可以考虑使用策略模式进行处理这种变化的可能性。
- 在基本的策略模式中，选择所使用具体实现的职责由客户端对象承担，并转给策略模式的 Context 对象。

## 参考资料

- 《大话设计模式》
- 《设计模式：可复用面向对象软件的基础》
- 《设计模式解析》
