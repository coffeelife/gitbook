# 前端干货：JS的执行顺序

```jsx
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
    console.log('async2');
}

console.log('script start');

setTimeout(function() {
    console.log('setTimeout');
}, 0)

async1();

new Promise(function(resolve) {
    console.log('promise1');
    resolve();
}).then(function() {
    console.log('promise2');
});
console.log('script end');


/*script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout*/
```

## 1. 单线程的JavaScript

> js是单线程的，基于事件循环，非阻塞IO的。  
> **特点：** 处理I／O型的应用，不适合CPU运算密集型的应用。  
> **说明：** 事件循环中使用一个事件队列，在每个时间点上，系统只会处理一个事件，即使电脑有多个CPU核心，也无法同时并行的处理多个事件。因此，node.js在I／O型的应用中，给每一个输入输出定义一个回调函数，node.js会自动将其加入到事件轮询的处理队列里，当I／O操作完成后，这个回调函数会被触发，系统会继续处理其他的请求。

- 然而单线程不应该是自上而下按照顺序执行的吗？
- 下面的代码输出顺序就被打乱了

```jsx
function fn(){
    console.log('start');
    setTimeout(()=>{
        console.log('setTimeout');
    },0);
    console.log('end');
}

fn() // 输出 start end setTimeout
```

---

## 2. JavaScript中的同步异步

> js的同步异步是如何实现的?  
> js中包含诸多创建异步的函数如:  
> seTimeout，setInterval，dom事件，ajax，Promise，process.nextTick等函数

![](https://upload-images.jianshu.io/upload_images/16418648-c0320e9b97d337de.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/636/format/webp)

timg.jpeg

1. 因为单线程，所以代码自上而下执行，所有代码被放到`执行栈`中执行；
2. 遇到异步函数将回调函数添加到一个`任务队列`里面；
3. 当`执行栈`中的代码执行完以后，会去循环`任务队列`里的函数;
4. 将`任务队列`里的函数放到`执行栈`中执行;
5. 如此往复，称为`事件循环`;  
   [图片上传失败...(image-5babc5-1559665370997)]
- 这样分析，上一段的代码就得到了合理的解释；
- 再来看一下这段代码；

```jsx
function fn() {
    setTimeout(()=>{
        console.log('a');
    },0);
    new Promise((resolve)=>{
        console.log('b');
        resolve();
    }).then(()=>{
        console.log('c')
    });
}
fn() // b c a
```

##### Promise和async中的立即执行

我们知道Promise中的异步体现在then和catch中，所以写在Promise中的代码是被当做同步任务立即执行的。而在async/await中，在出现await出现之前，其中的代码也是立即执行的。那么出现了await时候发生了什么呢？

##### await做了什么

从字面意思上看await就是等待，await 等待的是一个表达式，这个表达式的返回值可以是一个promise对象也可以是其他值。

很多人以为await会一直等待之后的表达式执行完之后才会继续执行后面的代码，实际上await是一个让出线程的标志。await后面的表达式会先执行一遍，将await后面的代码加入到microtask中，然后就会跳出整个async函数来执行后面的代码。  
由于因为async await 本身就是promise+generator的语法糖。所以await后面的代码是microtask。所以对于开始面试题中的

```jsx
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
```

等价于

```jsx
async function async1() {
    console.log('async1 start');
    Promise.resolve(async2()).then(() => {
                console.log('async1 end');
        })
}
```

## 3. 宏任务和微任务

> 两任务在同步异步中处于什么地位？  
> 两个任务分别处于`任务队列`中的`宏队列`与`微队列`中;  
> `宏队列`与`微队列`组成了任务队列;  
> `任务队列`将任务放入`执行栈`中执行

### 宏任务：

> 宏队列，macrotask，也叫tasks。  
> 异步任务的回调会依次进入macro task queue，等待后续被调用，  
> 这些异步任务包括：

- setTimeout
- setInterval
- setImmediate (Node独有)
- requestAnimationFrame (浏览器独有)
- I/O
- UI rendering (浏览器独有)

### 微任务：

> 微队列，microtask，也叫jobs。  
> 异步任务的回调会依次进入micro task queue，等待后续被调用，  
> 这些异步任务包括：

- process.nextTick (Node独有)
- Promise
- Object.observe
- MutationObserver

![](https://upload-images.jianshu.io/upload_images/16418648-c1884721402632a2?imageMogr2/auto-orient/strip|imageView2/2/w/710/format/webp)

图片

> 1. 执行全局Script同步代码，这些同步代码有一些是同步语句，有一些是异步语句（比如setTimeout等）；
> 2. 全局Script代码执行完毕后，`执行栈`Stack会清空；
> 3. 从`微队列`中取出位于队首的回调任务，放入`执行栈`Stack中执行，执行完后`微队列`长度减1；
> 4. 继续循环取出位于`微队列`的任务，放入`执行栈`Stack中执行，以此类推，直到直到把`微任务`执行完毕。注意，如果在执行`微任务`的过程中，又产生了`微任务`，那么会加入到`微队列`的末尾，也会在这个周期被调用执行；
> 5. `微队列`中的所有`微任务`都执行完毕，此时`微队列`为空队列，`执行栈`Stack也为空；
> 6. 取出`宏队列`中的任务，放入`执行栈`Stack中执行；
> 7. 执行完毕后，`执行栈`Stack为空；
> 8. 重复第3-7个步骤；

***以上才是一个完整的事件循环***

#### 回到面试题

> 1.首先，事件循环从宏任务(macrotask)队列开始，这个时候，宏任务队列中，只有一个script(整体代码)任务；当遇到任务源(task source)时，则会先分发任务到对应的任务队列中去。

> 2.然后我们看到首先定义了两个async函数，接着往下看，然后遇到了 console 语句，直接输出 script start。输出之后，script 任务继续往下执行，遇到 setTimeout，其作为一个宏任务源，则会先将其任务分发到对应的队列中

> 3.script 任务继续往下执行，执行了async1()函数，前面讲过async函数中在await之前的代码是立即执行的，所以会立即输出async1 start。  
> 遇到了await时，会将await后面的表达式执行一遍，所以就紧接着输出async2，然后将await后面的代码也就是console.log('async1 end')加入到microtask中的Promise队列中，接着跳出async1函数来执行后面的代码

> 4.script任务继续往下执行，遇到Promise实例。由于Promise中的函数是立即执行的，而后续的 .then 则会被分发到 microtask 的 Promise 队列中去。所以会先输出 promise1，然后执行 resolve，将 promise2 分配到对应队列

> 5.script任务继续往下执行，最后只有一句输出了 script end，至此，全局任务就执行完毕了。  
> 根据上述，每次执行完一个宏任务之后，会去检查是否存在 Microtasks；如果有，则执行 Microtasks 直至清空 Microtask Queue。  
> 因而在script任务执行完毕之后，开始查找清空微任务队列。此时，微任务中， Promise 队列有的两个任务async1 end和promise2，因此按先后顺序输出 async1 end，promise2。当所有的 Microtasks 执行完毕之后，表示第一轮的循环就结束了

> 6.第二轮循环依旧从宏任务队列开始。此时宏任务中只有一个 setTimeout，取出直接输出即可，至此整个流程结束

- 来一个稍微复杂点的代码

```jsx
function fn(){
    console.log(1);

    setTimeout(() => {
        console.log(2);
        Promise.resolve().then(() => {
            console.log(3);
        });
    },0);

    new Promise((resolve, reject) => {
        console.log(4);
        resolve(5);
    }).then(data => {
        console.log(data);
    });

    setTimeout(() => {
        console.log(6);
    },0);

    console.log(7);
}
fn(); //
```

> 流程重现

1. 执行函数同步语句；
- step1
  
  ```cpp
  console.log(1); 
  ```
  
  执行栈： [ console ]  
  宏任务： []  
  微任务： []
  
  > 打印结果：  
  > 1

- step2
  
  ```jsx
  setTimeout(() => {
      // 这个回调函数叫做callback1，setTimeout属于宏任务，所以放到宏队列中
      console.log(2);
      Promise.resolve().then(() => {
          console.log(3)
      });
  });
  ```
  
  执行栈： [ setTimeout ]  
  宏任务： [ callback1 ]  
  微任务： []
  
  > 打印结果：  
  > 1

- step3
  
  ```jsx
  new Promise((resolve, reject) => {
      // 注意，这里是同步执行的
      console.log(4);
      resolve(5)
  }).then((data) => {
      // 这个回调函数叫做callback2，promise属于微任务，所以放到微队列中
      console.log(data);
  });
  ```
  
  执行栈： [ promise ]  
  宏任务： [ callback1 ]  
  微任务： [ callback2 ]
  
  > 打印结果：  
  > 1  
  > 4

- step4
  
  ```jsx
  setTimeout(() => {
      // 这个回调函数叫做callback3，setTimeout属于宏任务，所以放到宏队列中
      console.log(6);
  })
  ```
  
  执行栈： [ setTimeout ]  
  宏任务： [ callback1 , callback3 ]  
  微任务： [ callback2 ]
  
  > 打印结果：  
  > 1  
  > 4

- step5
  
  ```cpp
  console.log(7)
  ```
  
  执行栈： [ console ]  
  宏任务： [ callback1 , callback3 ]  
  微任务： [ callback2 ]
  
  > 打印结果：  
  > 1  
  > 4  
  > 7
2. 同步语句执行完毕，从`微队列`中依次取出任务执行，直到`微队列`为空
- step6
  
  ```cpp
  console.log(data)       // 这里data是Promise的成功参数为5
  ```
  
  执行栈： [ callback2 ]  
  宏任务： [ callback1 , callback3 ]  
  微任务： []
  
  > 打印结果：  
  > 1  
  > 4  
  > 7  
  > 5
3. 这里`微队列`中只有一个任务，执行完后开始从`宏队列`中取任务执行
- step7
  
  ```cpp
  console.log(2);
  ```
  
  执行栈： [ callback1 ]  
  宏任务： [ callback3 ]  
  微任务： []
  
  > 打印结果：  
  > 1  
  > 4  
  > 7  
  > 5  
  > 2
  
  但是执行`callback1`的时候遇到另一个Promise,Promise异步执行完毕以后在`微队列`中又注册了一个`callback4`函数

- step8
  
  ```jsx
  Promise.resolve().then(() => {
      // 这个回调函数叫做callback4，promise属于微任务，所以放到微队列中
      console.log(3);
  });
  ```
  
  执行栈： [ Promise ]  
  宏任务： [ callback3 ]  
  微任务： [ callback4 ]
  
  > 打印结果：  
  > 1  
  > 4  
  > 7  
  > 5  
  > 2
4. 取出一个宏任务macrotask执行完毕，然后再去微任务队列microtask queue中依次取出执行
- step9
  
  ```cpp
  console.log(3)
  ```
  
  执行栈： [ callback4 ]  
  宏任务： [ callback3 ]  
  微任务： []
  
  > 打印结果：  
  > 1  
  > 4  
  > 7  
  > 5  
  > 2  
  > 3
5. `微队列`全部执行完，再去`宏队列`中取第一个任务执行
- step10
  
  ```cpp
  console.log(3)
  ```
  
  执行栈： [ callback3 ]  
  宏任务： []  
  微任务： []
  
  > 打印结果：  
  > 1  
  > 4  
  > 7  
  > 5  
  > 2  
  > 3  
  > 6
6. 以上全部执行完毕，`执行栈`，`宏队列`，`微队列`均为空
   
   执行栈： []  
   宏任务： []  
   微任务： []
   
   > 打印结果：  
   > 1  
   > 4  
   > 7  
   > 5  
   > 2  
   > 3  
   > 6
- 再来一段复杂代码

```jsx
function fn(){
    console.log(1);

    setTimeout(() => {
        console.log(2);
        Promise.resolve().then(() => {
            console.log(3)
        });
    });

    new Promise((resolve, reject) => {
        console.log(4)
        resolve(5)
    }).then((data) => {
        console.log(data);

        Promise.resolve().then(() => {
            console.log(6)
        }).then(() => {
            console.log(7)

            setTimeout(() => {
                console.log(8)
            }, 0);
        });
    })

    setTimeout(() => {
        console.log(9);
    })

    console.log(10);
}
fn();
```

## 4. NodeJS中的事件循环

![](https://upload-images.jianshu.io/upload_images/16418648-afae1d5cb784749d?imageMogr2/auto-orient/strip|imageView2/2/w/800/format/webp)

图片

### NodeJS中的`宏任务`和`微任务`

> NodeJS的`Event Loop`中，执行`宏队列`的回调任务有6个阶段，如下图：

![](https://upload-images.jianshu.io/upload_images/16418648-138a0419114175da?imageMogr2/auto-orient/strip|imageView2/2/w/670/format/webp)

图片

#### 各个阶段执行的任务如下：

- **timers阶段**：这个阶段执行setTimeout和setInterval预定的callback
- **I/O callback阶段**：执行除了close事件的callbacks、被timers设定的callbacks、setImmediate()设定的callbacks这些之外的callbacks  
  idle, prepare阶段：仅node内部使用
- **poll阶段**：获取新的I/O事件，适当的条件下node将阻塞在这里
- **check阶段**：执行setImmediate()设定的callbacks
- **close callbacks阶段**：执行socket.on('close', ....)这些callbacks

#### NodeJS的宏队列：

- Timers Queue
- IO Callbacks Queue
- Check Queue
- Close Callbacks Queue

> 这4个都属于`宏队列`，但是在`浏览器`中，可以认为只有一个`宏队列`，所有的`宏任务`都会被加到这一个`宏队列`中，但是在NodeJS中，不同的`宏任务`会被放置在不同的`宏队列`中

#### NodeJS的微队列：

- Next Tick Queue：是放置process.nextTick(callback)的回调任务的
- Other Micro Queue：放置其他`微任务`，比如Promise等

> 在`浏览器`中，也可以认为只有一个`微队列`，所有的`微任务`都会被加到这一个`微队列`中，但是在NodeJS中，不同的`微任务`会被放置在不同的`微队列`中

![](https://upload-images.jianshu.io/upload_images/16418648-fafebdb6ddc9480e?imageMogr2/auto-orient/strip|imageView2/2/w/951/format/webp)

图片

#### NodeJS中的`事件循环`过程

1. 执行全局的同步代码；
2. 执行`微任务`先执行`next tick queue`所有任务，再执行`other micro tasks queue`中的所有任务；
3. 开始执行`宏任务`，共6个阶段，从第1个阶段开始执行相应每一个阶段`宏队列`中的所有任务，  
   ***注意，这里是所有每个阶段宏任务队列的所有任务，在浏览器的Event Loop中是只取宏队列的第一个任务出来执行***，  
   每一个阶段的`宏任务`执行完毕后，开始执行`微任务`，回到步骤2；

> `Timers Queue` -> 步骤2 ->  
> `I/O Queue` -> 步骤2 ->  
> `Check Queue` -> 步骤2 ->  
> `Close Callback Queue` -> 步骤2 ->  
> `Timers Queue`

- 再看两张图  
  
  ![](https://upload-images.jianshu.io/upload_images/16418648-741f6a4eba742d1f?imageMogr2/auto-orient/strip|imageView2/2/w/420/format/webp)
  
  图片

---

![](https://upload-images.jianshu.io/upload_images/16418648-da128a930d268d00?imageMogr2/auto-orient/strip|imageView2/2/w/676/format/webp)

图片

- 代码又来了

```jsx
function fn(){

    console.log('start');

    setTimeout(() => {              // callback1
        console.log(111);

        setTimeout(() => {          // callback2
            console.log(222);
        }, 0);

        setImmediate(() => {        // callback3
            console.log(333);
        });

        process.nextTick(() => {    // callback4
            console.log(444);     
        });

    }, 0);

    setImmediate(() => {            // callback5
        console.log(555);

        process.nextTick(() => {    // callback6
           console.log(666); 
        });
    });

    setTimeout(() => {              // callback7
        console.log(777);

        process.nextTick(() => {    // callback8
            console.log(888);
        });
    }, 0);

    process.nextTick(() => {        // callback9
        console.log(999);
    });

    console.log('end');
}
fn();   

//  before version 11.0.0  start end 999 111 777 444 888 555 333 666 222
//  after  version 11.0.0  start end 999 111 444 777 888 555 666 333 222
```

***PS：[版本不同导致运行结果不同](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fnodejs%2Fnode%2Fpull%2F22842)***

### 总结：

- 浏览器的Event Loop和NodeJS的Event Loop是不同的，实现机制也不一样，不要混为一谈。
- NodeJS可以理解成有4个宏任务队列和2个微任务队列，但是执行宏任务时有6个阶段。先执行全局Script代码，执行完同步代码调用栈清空后，先从微任务队列Next Tick Queue中依次取出所有的任务放入调用栈中执行，再从微任务队列Other Microtask Queue中依次取出所有的任务放入调用栈中执行。然后开始宏任务的6个阶段，每个阶段都将该宏任务队列中的所有任务都取出来执行（注意，这里和浏览器不一样，浏览器只取一个），每个宏任务阶段执行完毕后，开始执行微任务，再开始执行下一阶段宏任务，以此构成事件循环。
- MacroTask包括： setTimeout、setInterval、 setImmediate(Node)、requestAnimation(浏览器)、IO、UI rendering。
- Microtask包括： process.nextTick(Node)、Promise、Object.observe、MutationObserver。
- v11以前 是上面说的那样；v11以后将Node环境的事件循环和浏览器的统一了。
- process.nextTick 上限是1000？
- 写一个休眠函数 达到阻塞目的

### 附图

> 浏览器中的EventLoop

![](https://upload-images.jianshu.io/upload_images/16418648-c94f9d0d1e3803a0.gif?imageMogr2/auto-orient/strip|imageView2/2/w/611/format/webp)

d1ca0d6b13501044a5f74c99becbcd3d4043.gif

> NodeJS中的EventLoop （v11以前，v11以后和浏览器一致）
> 
> ![](https://upload-images.jianshu.io/upload_images/16418648-72e58c982216a077.gif?imageMogr2/auto-orient/strip|imageView2/2/w/598/format/webp)
> 
> 963090bd3b681de3313b4466b234f4f02474.gif
