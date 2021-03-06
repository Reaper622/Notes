# JavaScript 有关提升

## 先有鸡还是先有蛋？

当我们执行下面一段代码：
```
a=2;
var a;
console.log(a);
```
他会输出什么呢？可能有部分人会认为是undefined，因为 var a 在 a=2 之后声明，所以会被重新赋值故为undefined。
But，告诉你一个不幸的消息，它真正的输出结果为2.

那么我们再来看下一段代码：
```
console.log(a);

var a = 2;
```
鉴于上一段代码的结果，我们主观性的认为它的结果也应该是2，也有人可能会说，a在使用之前没有声明，会抛出一个ReferenceError异常。但是我要很遗憾的告诉你，它的结果是 **undefined**

为了搞明白这个一点，我们需要深度剖析这个问题。

## 有关编译器
我们首先要了解，一段JavaScript代码在被引擎解释之前会先被编译器编译。编译的其中一项工作就是找到**所有**的声明，并且将它们与合适的作用域关联起来。

> 因此我们得知：变量和函数的所有声明都会在任何代码被执行前首先被编译器处理。

当我们看到 var a = 2;时我们可能会认为这只是一个声明。但在JavaScript眼中，它分为了"var a"和"a=2"两部分，"var a"会在编译阶段执行，而"a=2"则在代码执行到这里时再执行。

所以，第一段代码我们可以化为：
```
var a;
a = 2;
console.log(a);
```
第二段代码化为：
```
var a;
console.log(a);
a = 2;
```
在第二段代码执行console输出时，a还没有被赋值，故结果为undefined.

**我必须再提醒你们一次，当编译器对变量执行提升时，只有声明本身得到了提升，而赋值或者其他逻辑运行则会停留在原地，在代码执行到这一行时才会执行**

并且，*每个作用域*都会有提升操作，但它会只相对于它自身进行提升，一个作用域内的提升并不会提升到程序（全局）的最上方。

此外，函数声明也是会被提升的，但他的表达式却不会被提升。
```
foo();  //报出TypeError!而不是ReferenceError.
var foo = function bar(){
    //some code
}
```
这段代码的解释为，foo被提升并分配给所在的作用域，所以它不会报出ReferenceError，但是foo 后半段的赋值操作并没有得到提升，所以foo此时是undefined，对undefined进行函数调用时非法操作，故报出了TypeError异常。

## 函数是那个女士！
我们都知道一个国际性原则：女士优先！所以函数是那个女士也就表明了“函数优先”！
> 函数声明和变量声明都会被提升，但是函数会被首先提升，然后才是变量。

话不多说，让我们看下面这段代码
```
foo();  //结果为1

var foo;

function foo(){
    console.log(1);
}

foo = function(){
    console.log(2);
}
```
它输出的结果为1 并不是 2！Why！因为在引擎的眼中，这段代码是长这个样子的：
```
function foo(){
    console.log(1);
}

foo();

foo = function(){
    console.log(2);
}
```
**提醒一点，肃然var foo出现在function foo()之前，但是他是重复的声明从而会被忽略，因为函数优先！**

尽管重复的var声明会被忽略，但是后出现的函数声明可以覆盖从前的函数声明
```
foo(); //3

function foo(){
    console.log(1);
}
var foo = function(){
    console.log(2);
}
function foo(){
    console.log(3);
}
```
相信各位看到这些已经眼花缭乱，所以**在同一个作用域中进行重复定义**是非常不被提倡的，会出现各种意想不到的Bug。如果你不想开发的大部分时间都要去debug，我建议你不要在同一个作用域中重复定义。

## 总结一下
- 我们眼中的var a = 2;在引擎的眼中却是var a 和 a = 2这两个单独的声明，第一个是**编译**阶段执行，第二个是**执行**阶段执行。
- 函数声明会优先于变量声明，函数声明后的变量声明会被认为是多余的而忽略，但后面的函数声明可以覆盖之前的函数声明！
- 不要在同一个作用域中进行重复定义！不要在同一个作用域中进行重复定义！不要在同一个作用域中进行重复定义！