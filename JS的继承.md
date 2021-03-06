###JavaScript继承：

继承是OO语言中的一个最为人津津乐道的概念。许多OO语言都支持两种继承方式：**借口继承**和**实现继承**。接口继承只继承方法签名，而实现继承则继承实际的方法。由于JS中方法没有签名，在ECMScript中只支持实现类继承，而且其$实现继承$主要是依靠**原型链**实现的。

>Java中，方法签名包括函数名和参数列表，不包括返回值，编译器通过方法签名判断方法的调用，如果方法签名包括返回值，那么同名同参数不同返回值的方法被调用时编译器无法做出正确选择。
###概念
简单回顾构造函数、原型和实例的关系：

每个构造函数（constructor）都有一个原型对象（prototype），原型对象都包含一个指向构造函数的指针，而实例（instance）都包含一个指向原型对象的内部指针。

JS对象的圈子有这么个游戏规则：

如果试图引用对象(实例instance)的某个属性,会首先在对象内部寻找该属性,直至找不到,然后才在该对象的原型(instance.prototype)里去找这个属性.

如果让原型对象指向另一个类型的实例…..有趣的事情便发生了.

即: constructor1.prototype = instance2

鉴于上述游戏规则生效,如果试图引用constructor1构造的实例instance1的某个属性p1:

1. 首先会在instance1内部属性中找一遍;

2. 接着会在instance1.__proto__(constructor1.prototype)中找一遍,而constructor1.prototype 实际上是instance2, 也就是说在instance2中寻找该属性p1;

3. 如果instance2中还是没有,此时程序不会灰心,它会继续在instance2.__proto__(constructor2.prototype)中寻找…直至Object的原型对象

==搜索轨迹: instance1–> instance2 –> constructor2.prototype…–>Object.prototype==

这种搜索的轨迹,形似一条长链, 又因prototype在这个游戏规则中充当链接的作用,于是我们把这种实例与原型的链条称作 **原型链 **. 下面有个例子

```javascript
function Father(){
    this.property = true;
}
Father.prototype.getFatherValue = function(){
    return this.property;
}
function Son(){
    this.sonProperty = false;
}
//继承 Father
Son.prototype = new Father();//Son.prototype被重写,导致Son.prototype.constructor也一同被重写
Son.prototype.getSonVaule = function(){
    return this.sonProperty;
}
var instance = new Son();
alert(instance.getFatherValue());//true
```

instance实例通过原型链找到了Father原型中的getFatherValue方法.

注意: 此时instance.constructor指向的是Father,这是因为Son.prototype中的constructor被重写的缘故.

### 确定原型的实例的关系
使用原型链后，我们怎么去判断原型和实例的这种继承关系呢？方法一般有两种：

第一种是使用 **instanceof** 操作符, 只要用这个操作符来测试实例(instance)与原型链中出现过的构造函数,结果就会返回true. 以下几行代码就说明了这点.

```js
alert(instance instanceof Object);//true
alert(instance instanceof Father);//true
alert(instance instanceof Son);//true
```

由于原型链的关系, 我们可以说instance 是 Object, Father 或 Son中任何一个类型的实例. 因此, 这三个构造函数的结果都返回了true.

第二种是使用 isPrototypeOf() 方法, 同样只要是原型链中出现过的原型,isPrototypeOf() 方法就会返回true, 如下所示.

```js
alert(Object.prototype.isPrototypeOf(instance));//true
alert(Father.prototype.isPrototypeOf(instance));//true
alert(Son.prototype.isPrototypeOf(instance));//true
```

原理同上。

### 原型链的问题

原型链并非是非完美，它包含如下两个问题。

1.  当原型链中包含引用类型值的原型时,该引用类型值会被所有实例共享（~~子类型的某个实例修改会作用于所有实例~~）;

2.  在创建子类型(例如创建Son的实例)时,不能向超类型(例如Father)的构造函数中传递参数~~（在原型链的继承过程中，只有一个地方用到了父类型的构造函数，Son.prototype = new Father();）~~.

有鉴于此, 实践中很少会单独使用原型链.

为此,下面将有一些尝试以弥补原型链的不足.

### 借用构造函数
为解决原型链中上述两个问题, 我们开始使用一种叫做**借用构造函数**(constructor stealing)的技术(也叫经典继承).

==基本思想：即在子类型构造函数的内部调用超类的构造函数==。但是有一点要记住，这里其实并没有真的继承，仅仅是调用了父类的构造函数而已。也就是说，父类和子类没有任何关系。

```js
function Father(){
    this.colors = ["red","blue","green"];
}
function Son(){
    Father.call(this);//继承了Father,且向父类型传递参数
}
var instance1 = new Son();
instance1.colors.push("black");
console.log(instance1.colors);//"red,blue,green,black"

var instance2 = new Son();
console.log(instance2.colors);//"red,blue,green" 可见引用类型值是独立的
```

很明显,借用构造函数一举解决了原型链的两大问题:

1. 保证了原型链中引用类型值的独立,不再被所有实例共享;

2. 子类型创建时也能够向父类型传递参数.

随之而来的是, 如果仅仅借用构造函数,那么将无法避免构造函数模式存在的问题–方法都在构造函数中定义, 因此函数复用也就不可用了.而且超类型(如Father)中定义的方法,对子类型而言也是不可见的. 考虑此,借用构造函数的技术也很少单独使用~~（Father的原型对象中的共享属性和方法，Son没有办法获取。因为这个根本就不是真正的继承。）~~.

### 组合继承
组合继承, 有时候也叫做伪经典继承,指的是将原型链和借用构造函数的技术组合到一块,从而发挥两者之长的一种继承模式.

==基本思路: 使用原型链实现对原型属性和方法的继承,通过借用构造函数来实现对实例属性的继承.==

这样,既通过在原型上定义方法实现了函数复用,又能保证每个实例都有它自己的属性. 如下所示.

```js
function Father(name){
    this.name = name;
    this.colors = ["red","blue","green"];
}
Father.prototype.sayName = function(){
    alert(this.name);
};
function Son(name,age){
    Father.call(this,name);//继承实例属性，第一次调用Father()
    this.age = age;
}
Son.prototype = new Father();//继承父类方法,第二次调用Father()
Son.prototype.sayAge = function(){
    alert(this.age);
}
var instance1 = new Son("louis",5);
instance1.colors.push("black");
console.log(instance1.colors);//"red,blue,green,black"
instance1.sayName();//louis
instance1.sayAge();//5

var instance1 = new Son("zhai",10);
console.log(instance1.colors);//"red,blue,green"
instance1.sayName();//zhai
instance1.sayAge();//10
```

组合继承避免了原型链和借用构造函数的缺陷,融合了它们的优点,成为 JavaScript 中最常用的继承模式. 而且, instanceof 和 isPrototypeOf( )也能用于识别基于组合继承创建的对象.

同时我们还注意到组合继承其实调用了两次父类构造函数, 造成了不必要的消耗, 那么怎样才能避免这种不必要的消耗呢, 这个我们将在后面讲到.