# JavaScript 设计模式解析【1】——创建型设计模式



系列目录:

- [JavaScript 设计模式解析【1】——创建型设计模式](https://juejin.im/post/5d92079ce51d45780b568325)
- [JavaScript 设计模式解析【2】——结构型设计模式](https://juejin.im/post/5d9563cf6fb9a04e12714447)
- [JavaScript 设计模式解析【3】——行为型设计模式](https://juejin.im/post/5d96f52b6fb9a04e031bec9f)

## 工厂模式

工厂模式是用来创建对象的一种设计模式。

它不会暴露创建对象的具体逻辑，而是将逻辑封装在一个函数中，那么这个函数便成为了工厂，同时根据抽象程度的不同分为`简单工厂`、`工厂方法`



### 简单工厂模式

 通过一个工厂创建一种对象类的实力。主要用来创建同一类的对象。

```
// 简单工厂模式
// Pet 类
class Pet {
  // 构造函数
  constructor(props) {
    this.species = props.species;
    this.sound = props.sound;
  }

  // 静态实例创建方法
  static getProps(pet) {
    switch (pet) {
      case 'dog':
        return new Pet({species: 'dog', sound: 'woof'});
      case 'cat':
        return new Pet({species: 'cat', sound: 'meow'});
      case 'bird':
        return new Pet({species: 'bird', sound: 'chirping'});
      }
  }
}

let Adog = Pet.getProps('dog');
console.log(Adog.sound); // woof
let Acat = Pet.getProps('cat');
console.log(Acat.sound); // meow
let Abird = Pet.getProps('bird');
console.log(Abird.sound); // chirping
```

简单工厂让我们只需要传递一个参数给工厂函数即可获取到对应的实例对象，而不需要知道细节，但简单工厂模式只能用于对象数量少，对象创建逻辑不复杂的情况。



### 工厂方法模式

工厂方法模式让我们将创建实例的过程推迟到了*子类*中,这样我们的核心类就变成的抽象类，并且将构造函数和创建者分离，对 new 操作进行了封装。

```
// 工厂方法模式
class Pet {
  constructor(species = '', sound = '') {
    this.species = species;
    this.sound = sound;
  }
}

// 工厂子类
class PetShop extends Pet {
  constructor(species, sound) {
    super(species, sound);
  }
  create(pet) {
    switch (pet) {
      case 'dog':
        return new PetShop('dog','woof');
      case 'cat':
        return new PetShop('cat','meow');
      case 'bird':
        return new PetShop('bird','chirping');
      }
  }
}

let thePetShop = new PetShop();
// 通过创建者的方法进行实例创建
let shopDog = thePetShop.create('dog');
console.log(shopDog.sound); // woof
let shopCat = thePetShop.create('cat');
console.log(shopCat.sound); // meow
let shopBird = thePetShop.create('bird');
console.log(shopBird.sound); // chirping
```

我们可以将工厂方法看做一个实例化对象的工厂，它只做实例化对象这一件事。



#### 总结

- 工厂模式将构造函数和创建者分离
- 符合开放封闭原则


## 单例模式

> 单例模式的核心思想是：保证一个类仅有一个实例，并提供一个访问它的全局访问点

JavaScript 的单例模式不同于面向对象的应用，而在实际的开发中却有很多用途，例如提高页面性能，避免不必要的DOM操作。例如在我们点击登录后出现的登录浮窗，无论点击多少次登录按钮，这个浮窗都只会被创建一次。这里就可以用惰性单例模式来创建。

> 惰性单例是值在需要的时候才创建对象实例。

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Single Pattern</title>
</head>
<body>
  <button id="btn">点我登录</button>
  <script>
    class Login {
      createLayout() {
        console.log('invoked')
        let theDiv = document.createElement('div');
        theDiv.innerHTML = '我是浮窗'
        document.body.appendChild(theDiv)
        theDiv.style.display = 'none'
        return theDiv
      }
    }
    class Single {
      getSingle(fn) {
        let result;
        return function () {
          return result || (result = fn.apply(this, arguments))
        }
      }
    }

    let theBtn = document.getElementById('btn')
    let single = new Single()
    let login = new Login()
    // 由于闭包， createLoginLayer 对 result 的引用， 所以当single.getSingle 函数执行完之后，内存中并不会销毁 result
    // 之后点击按钮是，根据 createLoginLayer 函数的作用域链中已经存在result，所以直接返回result
    let createLoginLayer = single.getSingle(login.createLayout)
    theBtn.onclick= function() {
      let layout = createLoginLayer()
      layout.style.display = 'block'
    }
  </script>
</body>
</html>
```

此时我们可以连续点击登录按钮，但 `invoked`只会输出一次，代表着实例只创建了一次并且之后使用的都是唯一的那个实例。



### 总结

- 单例模式的主要思想： 实例已经创建，就直接返回，反之，则创建新的实例。
- 符合开放封闭原则。


## 原型模式

> 通过原型实例指定创建对象的类型，并且通过拷贝原型来创建新的对象。

在 JavaScript 中，实现原型模式的方法是`Object.create`方法,通过使用现有的对象来提供新创建的对象的`_proto_`。

- 每一个函数数据类型（普通函数，类）都有一个天生自带的属性：`prototype`，并且这个属性是一个对象数据类型的值。
- 并且在prototype上浏览器天生添加了一个构造函数`constructor`，属性值是当前函数（类）本身。
- 每一个对象数据类型叶天生自带一个属性`proto`,属性值是当前实例所属类的原型。

```javascript
var prototype = {
  name: 'Reaper',
  getName: function() {
    return this.name;
  }
}
// 同时注意 Object.create为浅拷贝
var obj = Object.create(prototype, 
  {
    skill: {value: 'FE'}
  }
)

console.log(obj.getName()); //Reaper
console.log(obj.skill); // FE
console.log(obj.__proto__ === prototype); //true

```

prototype 的几种方法

#### 1.Object.create()方法

```
let proto = {a:1}
let propertiesObject = {
  b: {
    value: 2
  }
}

let obj = Object.create(proto, propertiesObject)
console.log(obj.__proto__ === proto); // true
```



#### 2.方法继承

```
// 方法继承
let  proto = function() {}
proto.prototype.excute = function() {}
let child = function() {}

// 让child 继承proto的所有原型方法
child.prototype = new proto()
```

#### 3.函数对Object的默认继承

```
let Foo = function() {}
console.log(Foo.prototype.__proto__ === Object.prototype); // true
```

#### 4.isPrototypeOf

```
prototypeObj.isPrototypeOf(obj)
```



#### 5.instanceof

contructor.prototype是否出现在obj的原型链上

```
obj instanceof contructor
```

#### 6.getPrototypeOf

Object.getPrototypeOf(obj) 方法返回指定对象obj的原型（内部[[Prototype]]属性的值）

```
Object.getPrototypeOf(obj)
```

#### 7.setPrototypeOf

设置一个指定的对象的原型 ( 即, 内部[[Prototype]]属性）到另一个对象或 null

```
var obj = {}
var prototypeObj = {}
Object.setPrototypeOf(obj, prototypeObj)
console.log(obj.__proto__ === prototypeObj)  // true
```



> 仓库源代码: [JavaScript-DesignPatterns](https://github.com/Reaper622/JavaScript-DesignPatterns/tree/master/Strategy)