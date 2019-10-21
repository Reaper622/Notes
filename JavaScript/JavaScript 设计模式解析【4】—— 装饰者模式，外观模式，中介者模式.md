#JavaScript 设计模式解析【4】—— 装饰者模式，外观模式，中介者模式



系列目录:

- [JavaScript 设计模式解析【1】——创建型设计模式](https://juejin.im/post/5d92079ce51d45780b568325)
- [JavaScript 设计模式解析【2】——结构型设计模式](https://juejin.im/post/5d9563cf6fb9a04e12714447)
- [JavaScript 设计模式解析【3】——行为型设计模式](https://juejin.im/post/5d96f52b6fb9a04e031bec9f)
- [JavaScript 设计模式解析【4】——装饰者模式，外观模式，中介者模式](https://juejin.im/post/5da7baa66fb9a04de237b0fa)

## 装饰者模式

> 装饰者模式是在不改变对象自身的基础上，在程序运行期间给对象动态地添加方法。
>
> 本章节的示例需要 babel 支持修饰器模式

装饰者模式非常贴合 JavaScript 动态语言的特性，因为我们可以轻易的改变某个对象，但同时，因为函数是`一等公民`，所以我们会避免直接改写某个函数，来保护代码的可维护性和可扩展性。

其实就像我们拍照后添加的`滤镜`，不同的滤镜给照片赋予了不同的意境，这就是装饰者模式，通过滤镜装饰了照片，而并没有修改照片本身就给其添加了功能。

下面来举一个例子：

#### 初始实例

加入此时你是以为勇者 但勇者当前只有初始数值

```javascript
class Warrior {
    constructor(atk=50, def=50, hp=100, mp=100) {
        this.init(atk,def,hp,mp)
    }

    init(atk, def, hp, mp) {
        this.atk = atk
        this.def = def
        this.hp = hp
        this.mp = mp
    }

    toString() {
        return `攻击力:${this.atk}, 防御力: ${this.def}, 血量: ${this.hp}, 法力值: ${this.mp}`
    }
}


const Reaper = new Warrior()

console.log(`勇者状态 => ${Reaper}`) // 勇者状态 => 攻击力:50, 防御力: 50, 血量: 100, 法力值: 100
```

#### 装饰器的使用

之后我们为勇者配备一把圣剑，创建一个 `decorateSword` 方法，注意这个方法是装饰在`init` 上的

```javascript
// 本质是使用了 Object.defineProperty 方法
function decorateSword(target, key, descriptor) {
    // 首先获取到 init 方法
    const initMethod = descriptor.value
    // 宝剑添加攻击力 100 点
    let moreAtk = 100
    let returnObj
    descriptor.value = (...args) => {
        args[0] += moreAtk
        returnObj = initMethod.apply(target, args)
        return returnObj
    }
}



class Warrior {
    constructor(atk=50, def=50, hp=100, mp=100) {
        this.init(atk,def,hp,mp)
    }
    @decorateSword
    init(atk, def, hp, mp) {
        this.atk = atk
        this.def = def
        this.hp = hp
        this.mp = mp
    }

    toString() {
        return `攻击力:${this.atk}, 防御力: ${this.def}, 血量: ${this.hp}, 法力值: ${this.mp}`
    }
}


const Reaper = new Warrior()

console.log(`勇者状态 => ${Reaper}`) // 勇者状态 => 攻击力:150, 防御力: 50, 血量: 100, 法力值: 100
```

#### 装饰器的叠加

现在，我们这位勇者的防御力还太低了，我们需要为勇者再增添一个护甲

```typescript

// 省略decorateSword

function decorateArmour(target, key, descriptor) {
    // 首先获取到 init 方法
    const initMethod = descriptor.value
    // 护甲添加防御力 100 点
    let moreDef = 100
    let returnObj
    descriptor.value = (...args) => {
        args[1] += moreDef
        returnObj = initMethod.apply(target, args)
        return returnObj
    }
}


class Warrior {
    constructor(atk=50, def=50, hp=100, mp=100) {
        this.init(atk,def,hp,mp)
    }
    @decorateSword
    @decorateArmour
    init(atk, def, hp, mp) {
        this.atk = atk
        this.def = def
        this.hp = hp
        this.mp = mp
    }

    toString() {
        return `攻击力:${this.atk}, 防御力: ${this.def}, 血量: ${this.hp}, 法力值: ${this.mp}`
    }
}


const Reaper = new Warrior()

console.log(`勇者状态 => ${Reaper}`) // 勇者状态 => 攻击力:150, 防御力: 150, 血量: 100, 法力值: 100
```

我们成功的让勇者升级了，他终于可以打败大魔王了（没准还是个史莱姆）

### 总结：

装饰者一般也用来实现 AOP （面向切面编程）利用AOP可以对业务逻辑的各个部分进行隔离，也可以隔离业务无关的功能比如日志上报、异常处理等，从而使得业务逻辑各部分之间的耦合度降低，提高业务无关的功能的复用性，也就提高了开发的效率。

装饰者模式与代理模式类似，但代理模式的意图是不直接访问实体，为实体提供一个替代者，实体内定义了关键功能，而代理提供或拒绝对实体的访问，或者一些其他额外的事情。而装饰者模式的作用就是为对象动态地加入行为。



## 外观模式

> 外观模式为一组复杂的子系统接口提供一个更高级的统一接口，通过这个接口使得对子系统接口的访问更容易，不符合单一职责原则和开放封闭原则。

其实外观模式很常见，它其实就是通过一个单独的函数，来简化对一个或多个更大型，更为复杂的函数的访问，是一种对复杂操作的封装。

### 封装Ajax

初始化一个原生 Ajax 请求其实是复杂的，我们可以通过封装来简化

```javascript
function ajaxCall(type, url, callback, data) {
    let xhr = (function(){
        try {
            // 标准方法
            return new XMLHttpRequest()
        }catch(e){}

        try {
            return new ActiveXObject("Msxm12.XMLHTTP")
        }catch(e){}
    }())
    STATE_LOADED = 4
    STATUS_OK = 200

    // 一但从服务器收到表示成功的相应消息，则执行所给定的回调方法
    xhr.onreadystatechange = function () {
        if (xhr.readyState !== STATE_LOADED) {
            return
        }
        if (xhr.state == STATUS_OK) {
            callback(xhr.responseText)
        }
    }

    // 发出请求
    xhr.open(type.toUpperCase(), url)
    xhr.send(data)
}
```

封装之后，我们发送请求的样子就变成了

```javascript
// 使用封装的方法
ajaxCall("get", "/url/data", function(res) {
    document.write(res)
})
```



### 总结

	外观模式适用于当需要同时有多个复杂操作时，通过把复杂操作封装，调用时直接用方法调用，提高代码的可阅读性和可维护性。


## 中介者模式

> 中介者模式的主要作用是解除对象之间的强耦合关系，通过增加一个中介者，让所有的对象通过中介者通信，而不是相互引用，所以当一个对象发生改变时，只需要通知中介者对象即可。

中介者可是使网状的多对多关系变为相对简单的一对多关系。

### 情景示例

假设此时有两队人在玩 英雄联盟 ，必须团灭对方所有玩家才能获得胜利。下面将分为蓝红方：

```javascript
class Player {
    constructor(name, teamColor) {
        this.name = name // 英雄名称
        this.teamColor = teamColor // 队伍颜色
        this.teammates = [] // 队友列表
        this.enemies = [] // 敌人列表
        this.state = 'alive' // 存活状态
    }
    // 获胜
    win() {
        console.log(`Vicotry! ${this.name}`)
    }
    // 失败
    lose() {
        console.log(`Defeat! ${this.name}`)
    }
    // 死亡方法
    die() {
        // 团灭标志
        let ace_flag = true
        // 设置玩家状态为死亡
        this.state = 'dead'
        // 遍历队友列表，若没有团灭，则还未失败
        for(let i in this.teammates) {
            if (this.teammates[i].state !== 'dead') {
                ace_flag = false
                break
            }
        }
        // 如果已被团灭
        if (ace_flag === true) {
            // 己方失败
            this.lose()
            for(let i in this.teammates) {
                this.teammates[i].lose()
            }
            // 敌方胜利
            for(let i in this.enemies) {
                this.enemies[i].win()
            }
        }
    }
}
// 玩家列表
const Players = []

// 定义一个工厂函数来生成玩家

function playerFactory (name, teamColor) {
    let newPlayer = new Player(name, teamColor)
    // 通知所有玩家 新角色加入
    for(let i in Players) {
        if (Players[i].teamColor === teamColor) {
            Players[i].teammates.push(newPlayer)
            newPlayer.teammates.push(Players[i])
        } else {
            Players[i].enemies.push(newPlayer)
            newPlayer.enemies.push(Players[i])
        }
    }
    Players.push(newPlayer)
    return newPlayer
}

// 开始比赛
// 蓝色方
let hero1 = playerFactory('盖伦', 'Blue')
let hero2 = playerFactory('皇子', 'Blue')
let hero3 = playerFactory('拉克丝', 'Blue')
let hero4 = playerFactory('剑姬', 'Blue')
let hero5 = playerFactory('赵信', 'Blue')

// 红色方
let hero6 = playerFactory('诺手', 'Red')
let hero7 = playerFactory('德莱文', 'Red')
let hero8 = playerFactory('卡特琳娜', 'Red')
let hero9 = playerFactory('乌鸦', 'Red')
let hero10 = playerFactory('赛恩', 'Red')


// 红色方被团灭
hero6.die()
hero7.die()
hero8.die()
hero9.die()
hero10.die()


/* 运行结果:
Defeat! 赛恩
Defeat! 诺手
Defeat! 德莱文
Defeat! 卡特琳娜
Defeat! 乌鸦
Vicotry! 盖伦
Vicotry! 皇子
Vicotry! 拉克丝
Vicotry! 剑姬
Vicotry! 赵信
*/
```



但这只是一局比赛的情况，倘若我们此时存在掉线或者更换队伍的情况，那么上面的这种形式是晚觉无法解决的，所以此时我们需要一个中介者来统管所有的玩家。

### 使用中介者模式重构

- Player 和 palyerFactory 基础操作不变
- 将操作转角给中介者对象

```javascript
const GameManager = ( function() {
    // 存储所有玩家
    const players = []
    // 操作实体
    const operations = {}
    // 新增玩家
    operations.addPlayer = function (player) {
        let teamColor = player.teamColor
        players[teamColor] = players[teamColor] || []; // 如果该颜色的玩家还没有成立队伍，则新成立一个队伍 
        players[teamColor].push(player); // 添加玩家进队伍
    }
    // 玩家掉线
    operations.playerDisconnect = function (player) {
        // 玩家队伍颜色
        let teamColor = player.teamColor
        let teamPlayer = players[teamColor]
        for(let i in teamPlayer) {
            if (teamPlayer[i].name = player.name) {
                teamPlayer.splice(i, 1)
            }
        }
    }

    // 玩家死亡
    operations.playerDead = function (player) {
        let teamColor = player.teamColor
        teamPlayers = players[teamColor]
        // 团灭标志
        let ace_flag = true
        // 设置玩家状态为死亡
        this.state = 'dead'
        // 遍历队友列表，若没有团灭，则还未失败
        for(let i in teamPlayers) {
            if (teamPlayers[i].state !== 'dead') {
                ace_flag = false
                break
            }
        }
        // 如果已被团灭
        if (ace_flag === true) {
            // 己方失败
            for(let i in teamPlayers) {
                teamPlayers[i].lose()
            }
            // 敌方胜利
            for(let color in players) {
                if (color !== teamColor) {
                    let teamPlayers = players[color]
                    teamPlayers.map(player => {
                        player.win()
                    })
                }
            }
        }
    }

    function reciveMessage (message, player) {
        operations[message](player)
    }

    return {
        reciveMessage: reciveMessage
    }
})()




class Player {
    constructor(name, teamColor) {
        this.name = name // 英雄名称
        this.teamColor = teamColor // 队伍颜色
        this.state = 'alive' // 存活状态
    }
    // 获胜
    win() {
        console.log(`Vicotry! ${this.name}`)
    }
    // 失败
    lose() {
        console.log(`Defeat! ${this.name}`)
    }
    // 死亡方法
    die() {
        // 设置玩家状态为死亡
        this.state = 'dead'
        // 向中介者发送死亡的宣告
        GameManager.reciveMessage('playerDead', this)
    }
    // 玩家掉线
    disconnect() {
        GameManager.reciveMessage('playerDisconnect', this)
    }
}
// 玩家列表
const Players = []

// 定义一个工厂函数来生成玩家

function playerFactory (name, teamColor) {
    let newPlayer = new Player(name, teamColor)
    // 通知中介者新增玩家
    GameManager.reciveMessage('addPlayer', newPlayer)
    return newPlayer
}

// 开始比赛
// 蓝色方
let hero1 = playerFactory('盖伦', 'Blue')
let hero2 = playerFactory('皇子', 'Blue')
let hero3 = playerFactory('拉克丝', 'Blue')
let hero4 = playerFactory('剑姬', 'Blue')
let hero5 = playerFactory('赵信', 'Blue')

// 红色方
let hero6 = playerFactory('诺手', 'Red')
let hero7 = playerFactory('德莱文', 'Red')
let hero8 = playerFactory('卡特琳娜', 'Red')
let hero9 = playerFactory('乌鸦', 'Red')
let hero10 = playerFactory('赛恩', 'Red')


// 红色方被团灭
hero6.die()
hero7.die()
hero8.die()
hero9.die()
hero10.die()

/* 运行结果:
Defeat! 赛恩
Defeat! 诺手
Defeat! 德莱文
Defeat! 卡特琳娜
Defeat! 乌鸦
Vicotry! 盖伦
Vicotry! 皇子
Vicotry! 拉克丝
Vicotry! 剑姬
Vicotry! 赵信
*/



```



### 何时使用？

 中介者模式就是用来降低耦合度的，所有如果你的代码或者模块中耦合度较高，依赖过度，对实际调用和维护产生了影响，那么就可以通过中介者模式来降低耦合度。



### 总结

- 中介者模式符合最少知识原则
- 中介者模式降低了对象和模块之间的耦合度
- 中介者模式使复杂的网状多对多模型转换为相对简单的一对多关系。
- 中介者模式也存在一定的缺点，例如中介者对象相对复杂且庞大，有时往往中介者本身难以维护。