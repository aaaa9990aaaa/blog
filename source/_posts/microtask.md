---
title: JS用一行代码新建线程？前端异步机制深入剖析！
---
## 线程与任务
&emsp;&emsp;我们都知道，js是单线程的（别骂我标题党），而浏览器作为js的运行环境，它的线程结构又是怎样的呢？
&emsp;&emsp;在了解异步机制之前，不妨先看一下浏览器的线程处理。
### 1.浏览器有哪些线程？
|  线程 | 说明  |
|  :----  | :----  |
| GUI渲染线程  | 负责渲染浏览器界面，和JS引擎线程互锁，即同时只能运行一个。 |
| 事件触发线程  | 如onClick等 |
| 定时触发线程  | setTimeout和setInterval |
| 异步http请求线程  | 即XMLHttpRequest |
| JS引擎线程  | 负责处理javascript脚本程序，和GUI线程互锁，即同时只能运行一个。 |

### 2.宏任务与微任务   
|  宏任务(Macrotask)  |  微任务(Microtask) |
|  :----  | :----  |
|  setTimeout  | requestAnimationFrame  |
|  setInterval  | MutationObserver  |
|  MessageChannel  |  Promise.[then/catch/finally] |
|  I/O,事件队列  |  process.nextTick(Node环境) |
|  setImmediate（Node环境)  |  queueMicrotask |
|  script(整体代码块) |  |

&emsp;&emsp;通过对比可以发现，宏任务更多的是历史包袱，它们大多诞生自浏览器早期，甚至享有单独的线程，强依赖于浏览器或Node环境的支持。而微任务主要依赖于JS引擎自身的worker机制，不会另开线程。  
[了解Worker&gt;&gt;](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker)   
如果不方便理解，这里可以举一个芯片领域的例子：宏内核和微内核   
宏内核对应的是复杂指令集，拥有更强的兼容性，但效率相对低下。   
微内核对应的是精简指令集，兼容性不如宏内核，但效率更高。   

## 常用的异步处理
### 1.为什么需要异步处理？
&emsp;&emsp;上面提到，JS引擎与GUI渲染不能同时进行，这就导致了如果js的线程被阻塞，会造成页面渲染的停滞（我们习惯将js脚步插入到之后就是这个原因）
&emsp;&emsp;试想一下，假如我们不使用异步回调来处理接口返回的数据，当用户点击按钮等待请求结果后，整个页面直接定格卡死（GUI冻结）,等逻辑处理完成后，才终于又变成一个“活着”的页面，这种体验恐怕没有几个用户能够忍受吧？起码首先我会说一句：真卡！
&emsp;&emsp;从setTimeOut到ajax到axios再到observe、async await...甚至是LazyLoad 异步处理可以说渗透到前端开发的方方面面。
关于异步处理的例子还有很多，这里我不想过多描述，我想说的是<font color='red'>它很重要，必须要理解</font>。

### 2.任务的先后机制
&emsp;&emsp;宏任务与微任务虽然都能实现异步处理，但由于实现方式完全不同，在执行顺序上面也是有明显的先后的。
&emsp;&emsp;最先执行的当然是scrip代码块（js脚本本身属于一个宏任务），在script代码块中创建的任务会被处理成两个队列，它们的执行顺序如下：
![](https://media.liulangya.xyz/images/2021/04/06/08b2b4f72b1812b512cdb92c8cf09785.png)
实际应用中我们可以观察以下几个例子：
```
setTimeout(()=>{
  console.log(1)
},0);

Promise.resolve().then(()=>{
  console.log(2);
);
//输出结果： 2,1
```
这里可能有人要问了，不是说宏任务在微任务前面执行吗，为什么promise先于setTimeOut执行了？   
别急，这里还有一个隐藏的概念是W3C没有讲清楚的：<font color="red">每一个宏任务执行完都会清空当前的微任务队列，然后才会执行下一个宏任务。</font>
我们知道script代码块本身就是一个宏任务，而setTimeout可以看做是下一个宏任务节点。
观察以下代码：
```
console.log(1);
//微任务1，属于宏任务1
Promise.resolve().then(()=>{
  console.log(0.1);
});

//宏任务2 （宏任务1是整个script代码块）
setTimeout(() => {
  //宏任务4
  setTimeout(() => {
    console.log(4)
	//微任务4，输入宏任务4
    Promise.resolve().then(() => {
      console.log(0.4)
    });
  });
  console.log(2);
  //微任务2，属于宏任务2
  Promise.resolve().then(() => {
    console.log(0.2)
  })
},0)

//宏任务3
setTimeout(() => {
  console.log(3);
  //微任务3，属于宏任务3
  Promise.resolve().then(() => {
    console.log(0.3);
  });
});
//输出结果：1 0.1 2 0.2 3 0.3 4 0.4
```
所以实际执行队列的时候顺序如下：
![](https://media.liulangya.xyz/images/2021/04/06/-2.png)

### 3.ajax和fetch
&emsp;&emsp;ajax和fetch（或axios）有什么区别？面试时你一定也经常被问到这个问题，认真读完这篇文章，将能够帮助你更透彻的理解这个问题，以及这个问题真正想要表达的东西。
&emsp;&emsp;axios基于fetch，而fetch提供了与XMLHttpRequest截然不同的请求方式，虽然同样依赖于网络请求线程，但其本身并不属于宏任务和微任务，而是在调用请求后会返回一个promise对象。在使用时我们通过调用promise.then方法来创建一个微任务，这个被创建的微任务会被加入到微任务队列来等待执行，这也是为什么promise的链式调用时不会出现线程紊乱的现象。(队列是一个一个执行的)
如果难以理解，可以看一下下面的图解：
![](https://media.liulangya.xyz/images/2021/04/06/-1.png)

## 更简单的多线程（伪）创建
```
queueMicrotask(() => {
  console.log("我会在宏任务空闲时执行")
})
console.log("我会先执行");
```

值得注意的是：
1.使用这种方法仅仅只能创建一个平行的线程（伪），虽然利用宏、微切换的方式能够实现复杂的线程，但应该很少有这样的应用场景。
2.任何宏任务的阻塞都会影响后续宏任务的执行（毕竟不是真正的多线程)

综上所述，理解了Web前端的线程和任务机制，也就能理解异步处理的逻辑。希望这篇文章能对你有所帮助。