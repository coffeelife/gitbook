# ES6新特性学习

学习链接：[http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/)

## var、let和const

let与var类似是用来声明变量的，const用来声明常量(ES5只有全局作用域和函数作用域，没有块级作用域)，const也用来声明变量，但是声明的是常量，一旦声明，常量的值就不能改变。它和let一样只在声明的区域内有用。

关于使用`let`与`const`规则:

- 使用`let`声明的变量可以重新赋值,但是不能在同一作用域内重新声明
- 使用`const`声明的变量必须赋值初始化,但是不能在同一作用域类重新声明也无法重新赋值.

```
  {
      var a = 100;
      let b = 200;
  }
  console.log(a); //100
  console.log(b); //b is not defined -- Error

 {
     var a   = 100;
     const a = 200;
     console.log(a); // 报错
 }

 //const声明对象
 const a = {};
 a.name  = "Zhangsan"; 
 console.log(a.name);   //Zhangsan
 console.log(a);        //Object {name: "Zhangsan"}
```

**总结：let是块级作用域，只在let命令所在的代码块内有效，而var存在变量提升(变量提升代表无论声明在什么地方，都会被视为声明在顶部）**

### const对象冻结**

```
const q = Object.freeze({});
q.name  = "Zhangsan";
console.log(q.name);   //undefined
console.log(q);        //Object
```

## 模版字符串

使用模板和插入值是在字符串里面输出变量的一种方式。因此，在ES5，我们可以这样组合一个字符串：

```
var id = 1;
var first = 'g';
var last = 'm';
var name = 'Your name is ' + first + ' ' + last + '.';
var url = 'http://localhost:3000/api/messages/' + id;
```

在ES6中，我们可以使用新的语法$ {NAME}，并把它放在反引号里：

```
let id = 1;
let first = 'g';
let last = 'm';
let name = `Your name is ${first} ${last}. `;
let url = `http://localhost:3000/api/messages/${id}`;
```

### Multi-line Strings （多行字符串）in ES6

ES6的多行字符串是一个非常实用的功能。在ES5中，我们不得不使用以下方法来表示多行字符串：

```
var roadPoem = 'Then took the other, as just as fair,nt'
    + 'And having perhaps the better claimnt'
    + 'Because it was grassy and wanted wear,nt'
    + 'Though as for that the passing therent'
    + 'Had worn them really about the same,nt';
var fourAgreements = 'You have the right to be you.n
    You can only be you when you do your best.';
```

在ES6中，仅仅用反引号就可以解决了：　　

```
var roadPoem = `Then took the other, as just as fair,
    And having perhaps the better claim
    Because it was grassy and wanted wear,
    Though as for that the passing there
    Had worn them really about the same,`;
var fourAgreements = `You have the right to be you.
    You can only be you when you do your best.`;
```

## 解构赋值

在ES6中,可以使用**解构**从数组和对象提取值并赋值给独特的变量。

在ES5中是这样：

```
var person = {name: 'gm', age: 25};
var name = person.name;
var age = person.age;
console.log(name + age);
```

在ES6，我们可以使用这些语句代替上面的ES5代码:

```
var person = {name: 'gm', age: 25};
var { name, age } = person;
console.log(name + age);

//数组
const point = [10, 25, -34];
const [x, y, z] = point;
console.log(x, y, z);
```

### ...操作符

展开运算符（用三个连续的点 (`...`) 表示）是 ES6 中的新概念，使你能够将字面量对象展开为多个元素

```
    let str2 = ['苹果','梨子'];
    console.log(str2);//["苹果", "梨子"]
    console.log(...str2);//苹果 梨子

    const fruits = ["apples", "bananas", "pears"];
    const vegetables = ["corn", "potatoes", "carrots"];
    const produce = [...fruits,...vegetables];
    console.log(produce);
```

剩余操作符 使用展开运算符将数组展开为多个元素, 使用剩余参数可以将多个元素绑定到一个数组中.

ES5中

```
function sum() {
  let total = 0;  
  for(const argument of arguments) {
    total += argument;
  }
  return total;
}
```

ES6中

```
    fun(a,b,...c){
        console.log(a,b,...c);//...c指展开数组
    }
    fun('苹果','香蕉','橘子','梨子','李子');//苹果 香蕉 橘子 梨子 李子
```

## Enhanced Object Literals （增强的对象字面量）in ES6

使用对象文本可以做许多让人意想不到的事情！通过ES6，我们可以把ES5中的JSON变得更加接近于一个类。  
下面是一个典型ES5对象文本，里面有一些方法和属性：

```
var serviceBase = {port: 3000, url: 'azat.co'},
 getAccounts = function(){return [1,2,3]};
var accountServiceES5 = {
 port: serviceBase.port,
 url: serviceBase.url,
 getAccounts: getAccounts,
 toString: function() {
 return JSON.stringify(this.valueOf());
 },
 getUrl: function() {return "http://" + this.url + ':' + this.port},
 valueOf_1_2_3: getAccounts()
}
```

如果我们想让它更有意思，我们可以用Object.create从serviceBase继承原型的方法：

```
var accountServiceES5ObjectCreate = Object.create(serviceBase)
var accountServiceES5ObjectCreate = {
  getAccounts: getAccounts,
  toString: function() {
    return JSON.stringify(this.valueOf());
  },
  getUrl: function() {return "http://" + this.url + ':' + this.port},
  valueOf_1_2_3: getAccounts()
}
```

我们知道，accountServiceES5ObjectCreate 和accountServiceES5 并不是完全一致的，因为一个对象(accountServiceES5)在**proto**对象中将有下面这些属性：  

new1

为了方便举例，我们将考虑它们的相似处。所以在ES6的对象文本中，既可以直接分配getAccounts: getAccounts,也可以只需用一个getAccounts，此外，我们在这里通过**proto**（并不是通过’proto’）设置属性，如下所示：

```
var serviceBase = {port: 3000, url: 'azat.co'},
getAccounts = function(){return [1,2,3]};
var accountService = {
 __proto__: serviceBase,
 getAccounts,
```

另外，我们可以调用super防范，以及使用动态key值(valueOf_1_2_3):

```
 toString() {
 return JSON.stringify((super.valueOf()));
 },
 getUrl() {return "http://" + this.url + ':' + this.port},
 [ 'valueOf_' + getAccounts().join('_') ]: getAccounts()
};
console.log(accountService)
```

## 函数

### 定义函数

我们先来看一个基本的新特性，在javascript中，定义函数需要关键字function，但是在es6中，还有更先进的写法，我们来看：

es6写法：

```
var human = {
    breathe(name) {   //不需要function也能定义breathe函数。
        console.log(name + ' is breathing...');
    }
};
human.breathe('jarson');   //输出 ‘jarson is breathing...’
```

转成js代码：

```
var human = {
    breathe: function(name) {
        console.log(name + 'is breathing...');
    }
};
human.breathe('jarson');
```

### 函数默认参数

es5中

```
    var link = function (height, color, url) {
    var height = height || 50;
    var color = color || 'red';
    var url = url || 'http://azat.co';
    ...
    }
```

es6为参数提供了默认值。在定义函数时便初始化了这个参数，直接看代码。

```
    var link = function(height = 50, color = 'red', url = 'http://azat.co') {
      ...
    }
```

### 箭头函数

ES6之前,使用普通函数把其中每个名字转换为大写形式：

```
const upperizedNames = ['Farrin', 'Kagure', 'Asser'].map(function(name) { 
  return name.toUpperCase();
});
```

箭头函数表示:

```
const upperizedNames = ['Farrin', 'Kagure', 'Asser'].map(
  name => name.toUpperCase()
);
```

普通函数可以是**函数声明**或者**函数表达式**, 但是箭头函数始终都是**表达式**, 全程是**箭头函数表达式**, 因此因此仅在表达式有效时才能使用，包括：

- 存储在变量中，
- 当做参数传递给函数，
- 存储在对象的属性中。

```
const greet = name => `Hello ${name}!`;
```

可以如下调用:

```
greet('Asser');
```

如果函数的参数只有一个,不需要使用`()`包起来,但是只有一个或者多个, 则必须需要将参数列表放在圆括号内:

```
// 空参数列表需要括号
const sayHi = () => console.log('Hello Udacity Student!');

// 多个参数需要括号
const orderIceCream = (flavor, cone) => console.log(`Here's your ${flavor} ice cream in a ${cone} cone.`);
orderIceCream('chocolate', 'waffle');
```

一般箭头函数都只有一个表达式作为函数主题:

```
const upperizedNames = ['Farrin', 'Kagure', 'Asser'].map(
  name => name.toUpperCase()
);
```

这种函数表达式形式称为**简写主体语法**:

- 在函数主体周围没有花括号,
- 自动返回表达式

但是如果箭头函数的主体内需要多行代码, 则需要使用**常规主体语法**:

- 它将函数主体放在花括号内
- 需要使用 return 语句来返回内容。

```
const upperizedNames = ['Farrin', 'Kagure', 'Asser'].map( name => {
  name = name.toUpperCase();
  return `${name} has ${name.length} characters in their name`;
});
```

有了箭头函数在ES6中， 我们就不必用that = this或 self = this 或 _this = this 或.bind(this)。例如，下面的代码用ES5就不是很优雅：以前我们使用闭包，this总是预期之外地产生改变，而箭头函数的迷人之处在于，现在你的this可以按照你的预期使用了，身处箭头函数里面，this还是原来的this。

```
var _this = this;
$('.btn').click(function(event){
  _this.sendData();
})
```

在ES6中就不需要用 _this = this：

```
$('.btn').click((event) =>{
  this.sendData();
})
```

下面这是一个另外的例子，我们通过call传递文本给logUpperCase() 函数在ES5中：

```
var logUpperCase = function() {
  var _this = this;

  this.string = this.string.toUpperCase();
  return function () {
    return console.log(_this.string);
  }
}

logUpperCase.call({ string: 'ES6 rocks' })();
```

而在ES6，我们并不需要用_this浪费时间：

```
var logUpperCase = function() {
  this.string = this.string.toUpperCase();
  return () => console.log(this.string);
}
logUpperCase.call({ string: 'ES6 rocks' })();
```

### 闭包解释

闭包，官方对闭包的解释是：一个拥有许多变量和绑定了这些变量的环境的表达式（通常是一个函数），因而这些变量也是该表达式的一部分。闭包的特点：
　　1. 作为一个函数变量的一个引用，当函数返回时，其处于激活状态。
　　2. 一个闭包就是当一个函数返回时，一个没有释放资源的栈区。
　　简单的说，Javascript允许使用内部函数---即函数定义和函数表达式位于另一个函数的函数体内。而且，这些内部函数可以访问它们所在的外部函数中声明的所有局部变量、参数和声明的其他内部函数。当其中一个这样的内部函数在包含它们的外部函数之外被调用时，就会形成闭包。

**总结：闭包指的是：能够访问另一个函数作用域的变量的函数。清晰的讲：闭包就是一个函数，这个函数能够访问其他函数的作用域中的变量。**

```
function outer() {
     var  a = '变量1'
     var  inner = function () {
            console.info(a)
     }
    return inner    // inner 就是一个闭包函数，因为他能够访问到outer函数的作用域
}
```

**坑点1： 引用的变量可能发生变化**

```
function outer() {
      var result = [];
      for （var i = 0； i<10; i++）{
        result.[i] = function () {
            console.info(i)
        }
     }
     return result
}
```

看样子result每个闭包函数对打印对应数字，1,2,3,4,...,10, 实际不是，因为每个闭包函数访问变量i是outer执行环境下的变量i，随着循环的结束，i已经变成10了，所以执行每个闭包函数，结果打印10， 10， ..., 10  
怎么解决这个问题呢？

```
function outer() {
      var result = [];
      for （var i = 0； i<10; i++）{
        result.[i] = function (num) {
             return function() {
                   console.info(num);    // 此时访问的num，是上层函数执行环境的num，数组有10个函数对象，每个对象的执行环境下的number都不一样
             }
        }(i)
     }
     return result
}
```

**坑点2: this指向问题**

```
var object = {
     name: ''object"，
     getName： function() {
        return function() {
             console.info(this.name)
        }
    }
}
object.getName()()    // underfined
// 因为里面的闭包函数是在window作用域下执行的，也就是说，this指向windows
```

**坑点3：内存泄露问题**

```
function  showId() {
    var el = document.getElementById("app")
    el.onclick = function(){
      aler(el.id)   // 这样会导致闭包引用外层的el，当执行完showId后，el无法释放
    }
}

// 改成下面
function  showId() {
    var el = document.getElementById("app")
    var id  = el.id
    el.onclick = function(){
      aler(id)   // 这样会导致闭包引用外层的el，当执行完showId后，el无法释放
    }
    el = null    // 主动释放el
}
```

**技巧1： 用闭包解决递归调用问题**

```
function  factorial(num) {
   if(num<= 1) {
       return 1;
   } else {
      return num * factorial(num-1)
   }
}
var anotherFactorial = factorial
factorial = null
anotherFactorial(4)   // 报错 ，因为最好是return num* arguments.callee（num-1），arguments.callee指向当前执行函数，但是在严格模式下不能使用该属性也会报错，所以借助闭包来实现


// 使用闭包实现递归
function newFactorial = （function f(num){
    if(num<1) {return 1}
    else {
       return num* f(num-1)
    }
}） //这样就没有问题了，实际上起作用的是闭包函数f，而不是外面的函数newFactorial
```

** 技巧2：用闭包模仿块级作用域**  
es6没出来之前，用var定义变量存在变量提升问题，eg:

```
for(var i=0; i<10; i++){
    console.info(i)
}
alert(i)  // 变量提升，弹出10

//为了避免i的提升可以这样做
(function () {
    for(var i=0; i<10; i++){
         console.info(i)
    }
})()
alert(i)   // underfined   因为i随着闭包函数的退出，执行环境销毁，变量回收
```

当然现在大多用es6的let 和const 定义

## Class、 extends、 super

ES6提供了更接近传统语言的写法，引入了Class（类）这个概念。新的class写法让对象原型的写法更加清晰、更像面向对象编程的语法，也更加通俗易懂。

ES5写法：

```
function Human(name) {
    this.name = name;
    this.breathe = function() {
        console.log(this.name + ' is breathing');
    }
}
var man = new Human('jarson');
man.breathe();    //jarson is breathing
```

ES6中

```
class Human {
    constructor(name) {
        this.name = name;
    }
    breathe() {
        console.log(this.name + " is breathing");
    }
} 
var man = new Human("jarson");
man.breathe();    //jarson is breathing
```

extends用法

```
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color; // ReferenceError
    super(x, y);
    this.color = color; // 正确
  }
}
```

在子类的构造函数中，只有调用super之后，才可以使用this关键字，否则会报错。这是因为子类实例的构建，是基于对父类实例加工，只有super方法才能返回父类实例。父类的静态方法，也会被子类继承。

注意，super虽然代表了父类Point的构造函数，但是返回的是子类ColorPoint的实例，即super内部的this指的是ColorPoint，因此super()在这里相当于Point.prototype.constructor.call(this)。

super这个关键字，既可以当作函数使用，也可以当作对象使用。在这两种情况下，它的用法完全不同。  
作为函数时，super()只能用在子类的构造函数之中，用在其他地方就会报错。

```
    class A {}
    class B extends A {
      m() {
         super(); // 报错
      }
    }
```

第二种情况，super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。

```
     class A {
        p() {
         return 2;
       }
     }
     class B extends A {
        constructor() {
          super();
          console.log(super.p()); // 2
        }
     }
     let b = new B();
```

上面代码中，子类B当中的super.p()就是将super当作一个对象使用。这时，super在普通方法之中，指向A.prototype，所以super.p()就相当于A.prototype.p()。这里需要注意，由于super指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的。

## Modules(模块)

众所周知，在ES6以前JavaScript并不支持本地的模块。人们想出了AMD，RequireJS，CommonJS以及其它解决方法。现在ES6中可以用模块import 和export 操作了。  
在ES5中，你可以在 <script>中直接写可以运行的代码（简称IIFE），或者一些库像AMD。然而在ES6中，你可以用export导入你的类。下面举个例子，在ES5中,module.js有port变量和getAccounts 方法:

```
module.exports = {
  port: 3000,
  getAccounts: function() {
    ...
  }
}
```

在ES5中，main.js需要依赖require(‘module’) 导入module.js：

```
var service = require('module.js');
console.log(service.port); // 3000
```

但在ES6中，我们将用export and import。例如，这是我们用ES6 写的module.js文件库：

```
export var port = 3000;
export function getAccounts(url) {
  ...
}
```

如果用ES6来导入到文件main.js中，我们需用import {name} from ‘my-module’语法，例如：

```
import {port, getAccounts} from 'module';
console.log(port); // 3000
```

或者我们可以在main.js中把整个模块导入, 并命名为 service：

```
import * as service from 'module';
console.log(service.port); // 3000
```

## Promises

Promises 是一个有争议的话题。因此有许多略微不同的promise 实现语法。Q，bluebird，deferred.js，vow, avow, jquery 一些可以列出名字的。也有人说我们不需要promises，仅仅使用异步，生成器，回调等就够了。但令人高兴的是，在ES6中有标准的Promise实现。  

学习链接：https://blog.csdn.net/yaocong1993/article/details/85252151

网络请求事例

es5中

```
function zpost(url,param,doPost){
    if(!url.startWith("http://"))
        url = getSite() + "/" + url;
    $.ajax({
        type: 'POST',
        url: url,
        data: param,
        dataType: 'html',
        timeout: 30000,
        success: function(data){
            doPost(data);
        },
        error: function(xhr, type){
            hideLoading();
            zalert("网络不给力!");
        }
    });
}
```

es6中

```
this.$http('/api/getData').then((res) => {
res = res.data;
this.dataList = res.result;
}).catch((err) => {
...
});
```

下面是一个简单的用setTimeout()实现的异步延迟加载函数:

```
setTimeout(function(){
  console.log('Yay!');
}, 1000);
```

在ES6中，我们可以用promise重写:

```
var wait1000 =  new Promise(function(resolve, reject) {
  setTimeout(resolve, 1000);
}).then(function() {
  console.log('Yay!');
});
```

或者用ES6的箭头函数：

```
var wait1000 =  new Promise((resolve, reject)=> {
  setTimeout(resolve, 1000);
}).then(()=> {
  console.log('Yay!');
});
```

到目前为止，代码的行数从三行增加到五行，并没有任何明显的好处。确实，如果我们有更多的嵌套逻辑在setTimeout()回调函数中，我们将发现更多好处：

```
setTimeout(function(){
  console.log('Yay!');
  setTimeout(function(){
    console.log('Wheeyee!');
  }, 1000)
}, 1000);
```

在ES6中我们可以用promises重写：

```
var wait1000 =  ()=> new Promise((resolve, reject)=> {setTimeout(resolve, 1000)});
wait1000()
    .then(function() {
        console.log('Yay!')
        return wait1000()
    })
    .then(function() {
        console.log('Wheeyee!')
    });
```

## Generator生成器函数

ES6中非常受关注的的一个功能，能够在函数中间暂停，一次或者多次，并且之后恢复执行，在它暂停的期间允许其他代码执行，并可以用其实现异步。

### 简单使用

在异步编程中，还有一种常用的解决方案，它就是Generator生成器函数。顾名思义，它是一个生成器，它也是一个状态机，内部拥有值及相关的状态，生成器返回一个迭代器Iterator对象，我们可以通过这个迭代器，手动地遍历相关的值、状态，保证正确的执行顺序。

```
function *foo(x) {
    var y = 2 * (yield (x + 1));
    var z = yield (y / 3);
    return (x + y + z);
}

var it = foo( 5 );

console.log( it.next() );       // { value:6, done:false }
console.log( it.next( 12 ) );   // { value:8, done:false }
console.log( it.next( 13 ) );   // { value:42, done:true }
```

generator能实现好多功能，如配合for...of使用，实现异步等等

```
function* showWords() {
    yield 'one';
    yield 'two';
    return 'three';
}

var show = showWords();

show.next() // {done: false, value: "one"}
show.next() // {done: false, value: "two"}
show.next() // {done: true, value: "three"}
show.next() // {done: true, value: undefined}
```

如上代码，定义了一个showWords的生成器函数，调用之后返回了一个迭代器对象（即show），调用next方法后，函数内执行第一条yield语句，输出当前的状态done（迭代器是否遍历完成）以及相应值（一般为yield关键字后面的运算结果），每调用一次next，则执行一次yield语句，并在该处暂停，return完成之后，就退出了生成器函数，后续如果还有yield操作就不再执行了

### yield和yield*

```
function* showWords() {
    yield 'one';
    yield showNumbers();
    return 'three';
}

function* showNumbers() {
    yield 10 + 1;
    yield 12;
}

var show = showWords();
show.next() // {done: false, value: "one"}
show.next() // {done: false, value: showNumbers}
show.next() // {done: true, value: "three"}
show.next() // {done: true, value: undefined}
```

增添了一个生成器函数，我们想在showWords中调用一次，简单的 yield showNumbers()之后发现并没有执行函数里面的yield 10+1，因为yield只能原封不动地返回右边运算后值，但现在的showNumbers()不是一般的函数调用，返回的是迭代器对象，所以换个yield* 让它自动遍历进该对象

**注意的是，这yield和yield* 只能在generator函数内部使用，一般的函数内使用会报错**

### next()调用中的传参

```
function* showNumbers() {
    var one = yield 1;
    var two = yield 2 * one;
    yield 3 * two;
}

var show = showNumbers();

show.next().value // 1
show.next().value // NaN
show.next(2).value // 6
```

第一次调用next之后返回值one为1，但在第二次调用next的时候one其实是undefined的，因为generator不会自动保存相应变量值，我们需要手动的指定，这时two值为NaN，在第三次调用next的时候执行到yield 3 * two，通过传参将上次yield返回值two设为2

引用链接：https://www.cnblogs.com/imwtr/p/5913294.html , https://www.jianshu.com/p/393d2a203977

### for...of循环代替.next()

除了使用.next()方法遍历迭代器对象外，通过ES6提供的新循环方式for...of也可遍历，但与next不同的是，它会忽略return返回的值，如

```
const fruits = ['apple','coconut','mango','durian'];
//for循环数组，通过下标取得每一项的值
for (let i = 0; i < fruits.length; i++) {
    console.log(fruits[i]);
}

//数组的forEach方法，相对for循环语法更简单
fruits.forEach(fruit => {
    console.log(fruit);
})

//forEach有个问题是不能终止循环
fruits.forEach(fruit => {
    if(fruit === 'mango' ){
        break;                        //Illegal break statement
    }
    console.log(fruit);
})

//for...in循环，遍历数组对象的属性，MDN不推荐使用for...in遍历数组
//for...in循环会打印出非数字属性
const fruits = ['apple','coconut','mango','durian'];
fruits.fav = 'my favorite fruit';

for(let index in fruits){
    console.log(fruits[index]);   //...my favorite fruit
}

function* showNumbers() {
    yield 1;
    yield 2;
    return 3;
}

var show = showNumbers();

for (var n of show) {
    console.log(n) // 1 2
}

const fruits = ['apple','coconut','mango','durian'];
fruits.fav = 'my favorite fruit';

//ES6中的for...of循环，遍历属性值
for(let fruit of fruits){
    console.log(fruit);
}

//支持终止循环，也不会遍历非数字属性
for(let fruit of fruits){
    if(fruit === 'mango' ){
        break;
    }
    console.log(fruit);      //apple coconut durian
}

function* showNumbers() {
    yield 1;
    yield 2;
    return 3;
}

var show = showNumbers();

[...show] // [1, 2, length: 2]
```

## as和 ===

as表示把一个对象复制到另一个变量名称上，===

学习链接：[http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/)
