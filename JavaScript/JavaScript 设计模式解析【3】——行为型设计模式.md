# JavaScript 设计模式解析【3】——行为型设计模式



系列目录:
- [JavaScript 设计模式解析【1】——创建型设计模式](https://juejin.im/post/5d92079ce51d45780b568325)
- [JavaScript 设计模式解析【2】——结构型设计模式](https://juejin.im/post/5d9563cf6fb9a04e12714447)
- [JavaScript 设计模式解析【3】——行为型设计模式](https://juejin.im/post/5d96f52b6fb9a04e031bec9f)


## 策略模式

> 策略模式就是定义一系列的算法，把它们一个个封装起来，并且使它们可以相互替换。

策略模式的核心是讲算法的使用与算法的实现分离开。

一个策略模式的程序至少要包含两部分：

- 一组策略类，策略类封装了具体的算法，并且负责具体的计算过程
- 环境类，接收客户的请求，随后把请求委托给某一个策略类。要做到这点，说明环境类中要维持对某个策略对象的引用。



### 实现一个策略模式

假设我们要实现一个根据评分来打分的系统

```
// 等级与成绩的映射关系 
const levels = {
  S : 100,
  A : 90,
  B : 80,
  C : 70,
  D : 60
}

// 一组策略
let gradeBaseOnLevel = {
  S: () => {
    return `当前成绩为${levels['S']}`
  },
  A: () => {
    return `当前成绩为${levels['A']}`
  },
  B: () => {
    return `当前成绩为${levels['B']}`
  },
  C: () => {
    return `当前成绩为${levels['C']}`
  },
  D: () => {
    return `当前成绩为${levels['D']}`
  },
}

// 调用方法
function getStudentScore(level) {
  return levels[level] ? gradeBaseOnLevel[level]() : 0;
}

console.log(getStudentScore('S')); // 当前成绩为100
console.log(getStudentScore('A')); // 当前成绩为90
console.log(getStudentScore('B')); // 当前成绩为80
console.log(getStudentScore('C')); // 当前成绩为70
console.log(getStudentScore('D')); // 当前成绩为60
```



### 优缺点

- 优点： 可以有效地避免多重条件语句，将一系列方法封装起来也更直观，利于维护
- 缺点： 往往策略组会比较多，我们需要事先知道所有定义好的情况


## 迭代器模式

> 迭代器模式时指模块提供的一种方法去顺序访问一个集合对象中的各个元素，而又不需要暴露该对象的内部表示。迭代器模式也可以把迭代过程从业务逻辑中分离出来，使用迭代器模式后，即使不关心对象的内部构造，也可以按顺序访问其中的每个元素。

其实我们无形中已经使用了不少迭代器模式的功能，例如 JS 中数组的 `map`, 与 `forEach`已经内置了迭代器。

```
[1,2,3].forEach(function(item, index, arr) {
    console.log(item, index, arr);
});
```

同时迭代器分为两种：内部迭代器 与 外部迭代器

- 内部迭代器

> 内部迭代器在调用时非常方便，外界不会去关系其内部的实现。在每次调用时，迭代器的规则就已经制定完毕，如果遇到一些不同样的迭代规则，此时的内部迭代器就不是很清晰

- 外部迭代器

> 外部迭代器会显式地请求迭代下一个元素(`next`方法)，外部迭代器虽然增加了调用的复杂度，但是增强了迭代器的灵活性，我们可以手动地控制迭代过程或者顺序。就像`Generator`函数

#### 手写实现一个迭代器

我们可以通过代码实现一个简单的迭代器：

```javascript
// 创建者类
class Creator {
  constructor(list) {
    this.list = list;
  }
  // 创建一个迭代器来进行遍历
  creatIterator() {
    return new Iterator(this);
  }
}

// 迭代器类
class Iterator {
  constructor(creator) {
    this.list = creator.list;
    this.index = 0;
  }

  // 判断是否完成遍历
  isDone() {
    if (this.index >= this.list.length) {
      return true
    }
    return false
  }
  // 向后遍历操作
  next() {
    return this.list[this.index++]
  }
}

let arr = [1,2,3,4];

let creator = new Creator(arr);
let iterator = creator.creatIterator();
console.log(iterator.list) // [1,2,3,4]
while (!iterator.isDone()) {
  console.log(iterator.next()); 
}

// 执行结果为 1,2,3,4
```

#### ES6中的迭代器

JavaScript 中的有序数据集合包括：

- Array
- Map
- Set
- String
- typeArray
- arguments
- NodeList

！！ 注意 Object 不属于有序数据集合

以上的有序数据集合都部署`Symbol.iterator`属性，属性值作为一个函数，执行这个函数，返回一个迭代器，迭代器部署`next`方法来按顺序遍历访问子元素

以数组对象为例：

```javascript
let array = [1,2,3];

let iterator = arr[Symbol.iterator]();

console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```



### 总结

- 同时迭代器分为两种：内部迭代器 与 外部迭代器, 内部迭代器操作方便，外部迭代器可控性强。
- 任何部署了`[Symbol.iterator]`接口的数据都可以使用`for of`循环。
- 迭代器模式使目标对象和迭代器对象分离，符合了开放封闭原则。


## 观察者模式 （发布-订阅模式）

> 观察者模式也称作 发布—订阅模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，依赖于它的对象都将得到通知。在JavaScript开发中，我们一般用事件模型来替代传统的发布—订阅模式。

总的来说，这种模式的实质就是你可以对程序中的某个对象的状态进行观察，并且在其发生改变时能够得到通知。

当前已经有了很多实用观察者模式的例子，例如 `DOM事件绑定`就是一个非常典型的*发布—订阅模式*,还有 Vue.js 框架中的`数据双向绑定`，也利用了*观察者模式*。

所以一般观察者模式有两种角色：

- 观察者 （发布者）
- 被观察者 （订阅者）

下面我们举一个具体的例子，假设有三个报纸出版社，报社一，报社二，报社三，有两个订报人，分别是：订阅者1，订阅者2.此时出版社就是被观察者，订报人就是观察者。

我们先定义`报社类`:

```javascript
// 报社类
class Press {
  constructor(name) {
    this.name = name;
    this.subscribers = []; //此处存放订阅者名单
  }

  deliver(news) {
    let press = this;
    // 循环订阅者名单中所有的订报人，为他们发布内容
    press.subscribers.map(item => {
      item.getNews(news, press); // 向每个订阅者发送新闻
    })
    // 实现链式调用
    return this;
  }
}

```

接着我们定义`订报人类`

```javascript
// 订报人类
class Subscriber {
  constructor(name) {
    this.name = name;
  }

  // 获取新闻
  getNews(news, press) {
    console.log(`${this.name} 获取来自 ${press.name} 的新闻: ${news}`)
  }
  // 订阅方法
  subscribe(press) {
    let sub = this;
    // 避免重复订阅
    if(press.subscribers.indexOf(sub) === -1) {
      press.subscribers.push(sub);
    }
    // 实现链式调用
    return this;
  }

  // 取消订阅方法
  unsubscribe(press) {
    let sub = this;
    press.subscribers = press.subscribers.filter((item) => item !== sub);
    return this;
  }
}
```

之后我们通过实际操作进行演示：

```javascript
let press1 = new Press('报社一')
let press2 = new Press('报社二')
let press3 = new Press('报社三')

let sub1 = new Subscriber('订报人一')
let sub2 = new Subscriber('订报人二')

// 订报人一订阅报社一、二
sub1.subscribe(press1).subscribe(press2);
// 订报人二订阅报社二、三
sub2.subscribe(press2).subscribe(press3);

// 报社一发出新闻
press1.deliver('今天天气晴');
// 订报人一 获取来自 报社一 的新闻: 今天天气晴


// 报社二发出新闻
press2.deliver('今晚12点苹果发布会');
// 订报人一 获取来自 报社二 的新闻: 今晚12点苹果发布会
// 订报人二 获取来自 报社二 的新闻: 今晚12点苹果发布会

// 报社三发出新闻
press3.deliver('报社二即将倒闭，请大家尽快退订');
// 订报人二 获取来自 报社三 的新闻: 报社二即将倒闭，请大家尽快退订

// 订报人二退订
sub2.unsubscribe(press2);

press2.deliver('本报社已倒闭');
// 订报人一 获取来自 报社二 的新闻: 本报社已倒闭
```

上文我们提到了，`Vue.js`的双向绑定的原理是数据劫持和发布订阅，我们可以自己来实现一个简单的数据双向绑定

首先我们需要有一个页面结构

```
<div id="app">
    <h3>数据的双向绑定</h3>
    <div class="cell">
        <div class="text" v-text="myText"></div>
        <input class="input" type="text" v-model="myText" >
  </div>
</div>
```

接着我们创建一个类`aVue`

```
class aVue {
  constructor (options) {
    // 传入的配置参数
    this.options = options;
    // 根元素
    this.$el = document.querySelector(options.el);
    // 数据域
    this.$data = options.data;

    // 保存数据model与view相关的指令，当model改变时，我们会触发其中的指令类更新
    this._directives = {};
    // 数据劫持，重新定义数据的 set 和 get 方法
    this._obverse(this.$data);
    // 解析器，解析模板指令，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者
    // 一旦数据发生变动，收到通知，更新视图
    this._complie(this.$el);
  }
  // 对数据进行处理，重写set和get方法
  _obverse(data) {
    let val;
    // 进行遍历
    for(let key in data) {
      // 判断是否属于自身的属性
      if(data.hasOwnProperty(key)) {
        this._directives[key] = [];
      }

      val = data[key];
      if( typeof val === 'object') {
        // 递归遍历
        this._obverse(val);
      }

      // 初始化当前数据的执行队列
      let _dir = this._directives[key];

      // 重新定义数据的set 和 get 方法
      Object.defineProperty(this.$data, key, {
        // 可枚举的
        enumerable: true,
        // 可改的
        configurable: true,
        get: () => val,
        set: (newVal) => {
          if (val !== newVal) {
            val = newVal;
            // 触发_directives 中绑定的Watcher类更新
            _dir.map(item => {
              item._update();
            })
          }
        }
      })
    }
  }

  // 解析器，绑定节点，添加数据的订阅者，更新视图变化
  _complie(el) {
    // 子元素
    let nodes = el.children;
    for(let i = 0; i < nodes.length; i++) {
      let node = nodes[i];
      // 递归对所有元素进行遍历
      if (node.children.length) {
        this._complie(node);
      }

      // 如果有 v-text 指令， 监控 node 的值，并及时更新
      if (node.hasAttribute('v-text')) {
        let attrValue = node.getAttribute('v-text');
        // 将指令对应的执行方法放入指令集
        this._directives[attrValue].push(new Watcher('text', node, this, attrValue, 'innerHTML'))
      }

      // 如果有 v-model 属性，并且元素是 input 或者 textarea 我们监听input事件
      if( node.hasAttribute('v-model') && ( node.tagName === 'INPUT' || node.tagName === 'TEXTAREA')) {
        let _this = this;
        // 添加input事件
        node.addEventListener('input', (function(){
          let attrValue = node.getAttribute('v-model');
          // 初始化复制
          _this._directives[attrValue].push(new Watcher('input', node, _this, attrValue, 'value'));
          return function () {
            // 后面每次都会更新
            _this.$data[attrValue] = node.value;
          }
        })())
      }
    }
  }
}

```

`_observe`方法处理传入的data，重新改写data的`set`与`get`方法，保证我们可以跟踪到data的变化。

`_compile`方法本质是一个解析器，他通过解析模板指令，将每个指令对应的节点绑定更新函数，并添加监听数据的订阅者，数据发生变化时，就去更新视图变化。

接着我们定义订阅者类

```
class Watcher{
  /*
  * name 指令名称
  * el 指令对应的DOM元素
  * vm 指令所属的aVue实例
  * exp 指令对应的值，本例为"myText"
  * attr 绑定的属性值，本例为"innerHTML"
  */
  constructor(name, el, vm, exp, attr) {
    this.name = name;
    this.el = el;
    this.vm = vm;
    this.exp = exp;
    this.attr = attr;


    // 更新操作
    this._update();
  }

  _update() {
    this.el[this.attr] = this.vm.$data[this.exp];
  }
}
```

在`_compile`中，我们创建了两个`Watcher`实例，不过这两个对应的`_update`的操作结果不同，对于`div.text`的操作其实是`div.innerHTML = this.data.myText`,对于`input`的操作相当于`input.value = this.data.myText`，这样每次数据进行`set`操作时，我们会触发两个`_update`方法，分别更新`div`和`input`的内容。

![udbfdx.gif](https://user-gold-cdn.xitu.io/2019/10/4/16d95b26cef51baa?w=500&h=250&f=gif&s=31749)

Finally，我们成功地实现了一个简单的双向绑定。



> 示例Demo源码可在 [Observer](<https://github.com/Reaper622/JavaScript-DesignPatterns/tree/master/Observer>) 查看

### 总结

- 观察者模式可以使代码解耦合，满足开放封闭原则。
- 当过多地使用观察者模式时，如果订阅消息一直没有触发，但订阅者仍然一直保存在内存中。


## 命令模式

> 在软件系统里，`行为请求者`与`行为实现者`通常呈现一种*紧耦合*，但在某些场合，比如要对行为进行`记录、撤销/重做、事务`等处理时，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如果要将`行为请求者`与`行为实现者`解耦合，我们需要将一组行为抽象为对象，实现二者之间的松耦合，这就是`命令模式`。

我们需要在命令的发布者和接受者之间定义一个命令对象，命令对象会暴露一个统一的借口给命令的发布者而命令的发布者不需要去了解接受者是如何执行命令的，做到了命令发布者和接受者的解耦合。

[![u06DRP.png](https://user-gold-cdn.xitu.io/2019/10/4/16d95b33858a8e62?w=530&h=340&f=png&s=13313)](https://imgchr.com/i/u06DRP)

我们下面的例子中一个页面有三个按钮，每个按钮有不同的功能：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>命令模式</title>
</head>
<body>
    <div>
        <button id="btn1">Button1</button>
        <button id="btn2">Button2</button>
        <button id="btn3">Button3/button>
    </div>
    <script src="./Command.js"></script>
</body>
</html>
```

接下来我们定义一个命令发布者（执行者）的类

```javascript
class Executor {
    setCommand(btn, command) {
        btn.onclick = function () {
            command.execute();
        }
    }
}  
```

接着我们定义一个命令接受者，此例中为一个菜单

```javascript
// 定义一个命令接受者
class Menu {
    refresh() {
        console.log('刷新菜单')
    }

    addSubMenu() {
        console.log('增加子菜单')
    }

    deleteMenu() {
        console.log('删除菜单')
    }
}
```

之后我们将Menu的方法执行封装在单独的类中

```js

class RefreshMenu {
    constructor(receiver) {
        // 使命令对象与接受者关联
        this.receiver = receiver
    }
    // 暴露出统一接口给 Excetor
    execute() {
        this.receiver.refresh()
    }
}

// 定义一个接受子菜单的命令对象的类
class AddSubMenu {
    constructor(receiver) {
        // 使命令对象与接受者关联
        this.receiver = receiver
    }

    // 暴露出统一的接口给 Excetor
    execute() {
        this.receiver.addSubMenu()
    }
}

// 定义一个接受删除菜单对象的类
class DeleteMenu {
    constructor(receiver) {
        this.receiver  = receiver
    }

    // 暴露出统一的接口给 Excetor
    execute() {
        this.receiver.deleteMenu()
    }
}

```

之后我们分别实例华不同的对象

```javascript
// 首先获取按钮对象
let btn1 = document.getElementById('btn1')
let btn2 = document.getElementById('btn2')
let btn3 = document.getElementById('btn3')

let menu = new Menu()
let executor = new Executor()

let refreshMenu = new RefreshMenu(menu)

// 给 按钮1 添加刷新功能
executor.setCommand(btn1, refreshMenu) // 点击按钮1 显示"刷新菜单"

let addSubMenu = new AddSubMenu(menu)
// 给按钮2添加子菜单功能
executor.setCommand(btn2, addSubMenu)// 点击按钮2 显示"添加子菜单"

let deleteMenu = new DeleteMenu(menu)
// 给按钮3添加删除功能
executor.setCommand(btn3, deleteMenu)// 点击按钮3 显示"删除菜单"

```



### 总结

- 发布者与接受者实现了解耦合，符合单一职责原则。
- 命令可扩展，对请求可以进行排队或日志记录，符合开放—封闭原则。
- 但额外增加了命令对象，存在一定的多余开销。


## 状态模式

> 状态模式允许一个对象在其内部状态改变时改变行为，这个对象看上去像改变了类一样，但其实并没有。状态模式把所研究对象在其内部状态改变时，其行为也随之改变，状态模式需要对每一个系统可能取得的状态创立一个状态类的子类。当系统的状态变化时，系统改变所选的子类。

假设我们此时有一个电灯，在电灯上有个开关，在灯未亮时按下使灯开启，在灯已亮时按下则将灯关闭，此时行为表现时不一样的：

```javascript
class Light {
    constructor() {
        this.state = 'off'; //电灯默认为关闭状态
        this.button = null;
    }

    init() {
        let button = document.createElement('button');
        let self = this;
        button.innerHTML = '我是开关';
        this.button = document.body.appendChild(button);
        this.button.onclick = () => {
            self.buttonWasClicked();
        }
    }

    buttonWasClicked() {
        if (this.state === 'off') {
            console.log('开灯');
            this.state = 'on';
        } else {
            console.log('关灯');
            this.state = 'off';
        }
    }
}

let light = new Light();
light.init();
```

[![uD8U1A.gif](https://user-gold-cdn.xitu.io/2019/10/4/16d95b4057c37740?w=317&h=196&f=gif&s=26634)](https://imgchr.com/i/uD8U1A)

此时只有两种状态，我们尚且可以不使用状态模式，但当状态较多时，例如，当电灯出现弱光，强光档位时，以上的代码就无法满足需求。

###  ### 使用状态模式重构

状态模式的关键是把每种状态都封装成单独的类，跟此种状态有关的行为都被封装在这个类的内部，在按钮被按下时，只需要在上下文中，把这个请求委托给当前状态对象即可，该状态对象会负责渲染它自身的行为。

首先定义三种不同状态的类

```javascript
// 灯未开启的状态
class OffLightState {
    constructor(light) {
        this.light = light;
    }

    buttonWasClicked() {
        console.log('切换到弱光模式');
        this.light.setState(this.light.weakLightState);
    }
}

// 弱光状态
class WeakLightState {
    constructor(light) {
        this.light = light;
    }

    buttonWasClicked() {
        console.log('切换到强光模式');
        this.light.setState(this.light.strongLightState);
    }
}

// 强光状态
class StrongLightState {
    constructor(light) {
        this.light = light;
    }

    buttonWasClicked() {
        console.log('关灯');
        this.light.setState(this.light.offLightState);
    }
}
```

接着我们改写 Light 类，在内部通过`curState`记录当前状态

```javascript
class Light {
    constructor() {
        this.offLightState = new OffLightState(this);
        this.weakLightState = new WeakLightState(this);
        this.strongLightState = new StrongLightState(this);
        this.button = null;
    }

    init() {
        let button = document.createElement('button');
        let self = this;
        button.innerHTML = '我是开关';
        this.button = document.body.appendChild(button);
        this.curState = this.offLightState;
        this.button.onclick = () => {
            self.curState.buttonWasClicked();
        }
    }

    setState(state) {
        this.curState = state;
    }
}
```

之后实例化对象后，我们在页面中查看

```javascript
let light = new Light();
light.init();
```



[![uDGh2d.gif](https://user-gold-cdn.xitu.io/2019/10/4/16d95b401717810d?w=480&h=261&f=gif&s=35594)](https://imgchr.com/i/uDGh2d)



### 总结

- 状态模式通过定义不同的状态类，根据状态的改变而改变对象的行为。
- 不必把大量的逻辑都写在被操作的对象的类中，很容易增加新的状态。
- 符合开放-封闭原则。