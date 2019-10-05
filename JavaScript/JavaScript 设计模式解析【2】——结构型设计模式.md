# JavaScript 设计模式解析【2】——结构型设计模式



系列目录:

- [JavaScript 设计模式解析【1】——创建型设计模式](https://juejin.im/post/5d92079ce51d45780b568325)
- [JavaScript 设计模式解析【2】——结构型设计模式](https://juejin.im/post/5d9563cf6fb9a04e12714447)
- [JavaScript 设计模式解析【3】——行为型设计模式](https://juejin.im/post/5d96f52b6fb9a04e031bec9f)


## 适配器模式

> 适配器模式主要用来解决两个已有接口之间不匹配的问题，它不考虑这些接口是怎么实现的，也不考虑他们将来会如何演化。适配器模式不需要改变当前已有的接口，就能让他们协同作用。

适配器的别名也叫包装器(wrapper)，这是一个并不复杂的模式，在日常开发中有许多这样的场景：例如当我们试图调用某个模块或者某个对象的接口时，却发现这个接口的格式并不符合目前的需求，这时就有两种解决方法，第一种使我们直接修改原来的接口实现，但如果原来的模块或者对象很复杂，亦或是我们拿到的已经是已压缩过的代码，那么去修改原接口就显得不现实了。第二种方法就是我们要讲到的适配器，将原接口转换成你希望拿到的接口，而你只需要通过适配器即可得到，并不需要去修改原模块或对象。

举一个抽象的例子，例如当我们有两台电脑需要充电：

```
class ThinkPad {
  charge() {
    console.log('ThinkPad 开始充电');
  }
}


class MacBook {
  charge() {
    console.log('MacBook 开始充电')
  }
}
// 电源充电
function PowerToCharge(laptop) {
  if(laptop.charge instanceof Function) {
    laptop.charge()
  }
}

PowerToCharge(new ThinkPad()) // ThinkPad开始充电
PowerToCharge(new MacBook()) // MacBook开始充电
```

但是如果MacBook不能直接用一种电源接口充电，可能我们就需要一种转接器，这里也就使用的适配器模式，我们不能直接更改电脑上的接口，但我们可以通过一层转接（封装），来实现充电

```
class ThinkPad {
  charge() {
    console.log('ThinkPad 开始充电');
  }
}


class MacBook {
  type_cCharge() {
    console.log('MacBook 开始充电')
  }
}

// 定义适配器类，来实现对MacBook的转接
class TypeCToDp {
  charge() {
    let macbook = new MacBook();
    return macbook.type_cCharge()
  }
}
// 电源充电
function PowerToCharge(laptop) {
  if(laptop.charge instanceof Function) {
    laptop.charge()
  }
}

PowerToCharge(new ThinkPad()) // ThinkPad开始充电
PowerToCharge(new TypeCToDp()) // MacBook开始充电
```



#### 总结

- 适配器模式虽然是一种相对简单的模式，但适配器在JS或者BFF层使用的场景很多。
- 但同时，我们要意识到适配器模式其实一种补救措施，它用来解决的是一些古老不可维护或者已经在稳定版本的两个接口不兼容的问题，而在开发初期应该减少或者不使用这种模式，而是要规划好接口的一致性。
- 适配器不会改变原有的接口，而是一个对象对另一个对象的包装。
- 适配器模式符合开放封闭原则。



## 代理模式

> 代理模式是为一个对象提供一个代用品或者占位符，以便控制对它的访问。

代理模式是一种极为有趣的模式，生活中我们可以找到很多代理模式的场景。比如明星的经纪人，一般的商业活动不会直接跟明星接触，而是和经纪人谈，经纪人会把工作内容和报酬谈好之后在交给明星。

### 代理模式的简单实现

下面我们用一个明星买包的例子来解释下代理

 当明星没有经济人 自己买包

```javascript
// 定义一个包类
class Bags {
  constructor(props) {
    this.name = props;
  }
  getName() {
    return this.name;
  }
}

// 定义一个明星对象
class Star {
  buyBag(bag) {
    console.log(`买到了一个${bag.getName()}包`);
  }
}

// 创建一个明星实例
let star = new Star();
star.buyBag(new Bags('Coach')); //买到了一个Coach包
```

当明星让自己的助理去买包

```javascript
// 定义一个包类
class Bags {
  constructor(props) {
    this.name = props;
  }
  getName() {
    return this.name;
  }
}

// 定义一个助理对象
class Assistant {
  constructor(props) {
    this.star = props;
  }
  buyBag(bag) {
    this.star.buyBag(bag);
  }
}

// 定义一个明星对象
class Star {
  buyBag(bag) {
    console.log(`买到了一个${bag.getName()}包`);
  }
}



// 创建一个明星实例
let star = new Star();
let assistant = new Assistant(star);
assistant.buyBag(new Bags('Coach')); //买到了一个Coach包
```

此时我们就实现了一个简单的代理模式，但一般我们不会在这么简单的场景使用代理，反而徒增了代码的复杂度。



### 代理的使用场景

图片的懒加载是前端中比较会用到代理模式的场景，当网络不好的时候，图片的加载需要一段时间，这就会产生空白，影响用户体验，这时候在图片真正加载完之前，使用一张loading占位图片，等图片真正加载完在设置`src`属性，来实现图片的懒加载。

```javascript
class ABigImage {
  constructor() {
    this.img = new Image();
    document.body.appendChild(this.img);
  }
  setSrc(src) {
    this.img.src = src;
  }
}

class ProxyImage {
  constructor() {
    this.proxyImage = new Image();
  }

  setSrc(src) {
    let bigImageObj = new ABigImage();
    bigImageObj.img.src = './local.png'; // 低清晰度图片url 或者本地图片
    this.proxyImage.src = src;
    this.proxyImage.onload = function() {
      bigImageObj.img.src = src;
    }
  }
}

var proxyImage = new ProxyImage();
proxyImage.setSrc('图片Url')
```

在演示中我们可以感受到 （启用chrome的 Network 中的 Disable cache 并设置网速为 slow 3G 会让我们有更直观的感受）

![uG2uWQ.gif](https://user-gold-cdn.xitu.io/2019/10/3/16d8f8fadb1c3b6d?w=1842&h=862&f=gif&s=465203)

到这里图片的预加载已经实现，并且当我们的网速非常好，已经达到可以取消图片预加载的一些问题，那么我们可以直接使用`ABigImage`的`setSrc`方法，并且删除代理类，这样我们根本就不许要改动本体代码，这是易于维护的。

```
// 网速很快时
let aImage = new ABigImage();
aImage.setSrc('图片url')
```

#### 总结

- 代理模式符合开放封闭原则。
- 代理模式会让代码易于维护。
- 本体对象一般和代理对象拥有相同的方法，所以使用者并不知道请求的是本体对象还是代理对象。



> 代理模式中所有的例子均可在 [Demo](<https://github.com/Reaper622/JavaScript-DesignPatterns/tree/master/Proxy>)查看


> 仓库源代码: [JavaScript-DesignPatterns](https://github.com/Reaper622/JavaScript-DesignPatterns)