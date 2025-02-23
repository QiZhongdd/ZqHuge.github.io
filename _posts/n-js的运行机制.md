**变量提升以及块级作用域**

- js在执行的时候首先会进行编译，编译生成执行上下文和可执行代码
- 执行上下文包括作用域、this、词法环境、变量环境、外部环境等
- 在变量环境中会进行变量提升，变量提升带来的弊端是可能会覆盖变量，或者运行完后变量不会回收
- 为了解决变量提升的弊端就出现了let和const块级作用域
- 在执行代码时，如果遇到块级作用域，那么会单独包裹在一个单独的区域中添加到词法环境中。如果这个时候还有快，那么也会在将快中的let和const单独包裹添加到词法环境。此法环境类似一个栈，当块执行完毕会从词法环境抛出。
- 在此法环境中查找一个变量会从词法环境的栈底一直往上找，此法环境中没有那么就会从变量环境中查找，然后去外部环境中查找。

**暂时性死区**

所谓的暂时性死区是指在作用域实例化时let和const会在作用域中创建，但此时还没有与此法环境绑定，如果这个时候访问会抛出异常。因此在创建作用域到运行变量这一段时间被称为暂时性死区。


**作用域链**

js的执行上下文包括this、变量环境、词法环境、外部环境，外部环境是用来指向外部的执行上下文。当一段代码查找一个变量时，通常会在当前的变量环境中查找，然后js引擎会通过outer指向的外部的执行上下文中查找，这里的外部的执行上下文是指函数声明所在的上下文，这就是作用域链。

**闭包**

所谓的闭包是指在执行一个函数时会返回他内部的函数，同时返回的内部函数被其他的变量引用，并且内部的函数中引用了当前执行函数中的变量，闭包的本质是作用域链的查找。 在执行的时候首先会将外部函数的执行上下文添加到栈中，当执行完毕后会出栈，但由于这时内部的变量被引用了，会将变量进行拷贝保存在堆内存中。所以内部函数查找变量时会内部的变量环境中查找，然后再去闭包查找，最后去外部的的上下文变量环境中查找。
在v8中执行一段代码会先编译后执行，编译采用的策略是惰性分析和预解析的策略，所谓的惰性分析是指编译的时候只编译顶端代码，不编译函数，只是将函数包裹成一个对象保存在内存中，待执行的时候才进行编译。但堕性分析有个弊端就是像遇到必包这种它无法识别到内部函数是否引用了外部变量，为了解决这个问题，他采用了预解析器，预解析的目的是为了识别函数是否有引用外部的变量同时检查函数是否有错误

**js为什么要区分栈和堆的空间，为什么不能把数据都存到栈中**

这是因为js引擎需要用栈来维护上下文的状态，如果栈空间大了，所有数据都存放到栈中，那么就会影响到上下文的切换效率，进而影响到程序的执行效率

所以通常情况下，栈空间都不会设置太大，主要用来存放一些原始类型的小数据。而引用类型的数据占用的空间都比较大，所以这一类数据会被存放到堆中，堆空间很大，能存放很多大的数据，不过缺点是分配内存和回收内存都会占用一定的时间。






**settimeout应该注意的**

- settimeout嵌套超过5次后，后面的settimeout使用setimout会间隔4ms
- 如果定时器之前执行的任务太长了,那么会影响当前定时器的执行。比如设置了500，可能会超过500ms执行。
- 使用setTimeout执行对象中的方法，对象方法中的this指向全局
- 定时器是有最大时间的，24.8天。超过会变成0
- 未激活的tab的最小时间为1s，目的是为了优化后台的价值资源和降低耗电量

实现一个没有延时的定时器
postmessage类似于settimeout，是宏任务

```
(function(){
 let fns=[];
 let messageName='abc'
 function setZeroTimeout(fn){
   fns.push(fn)
   window.postMessage(messageName,'*')
 }
 function handler(evt){
  if(evt.data===messageName){
    evt.stopPropagation();
    const fn=fns.shift();
    fn();
  }
 }

 window.addEventListener('message',handler,false);
 window.setZeroTimeout=setZeroTimeout

})()

```



其实在 React 的源码中，做时间切片的部分就用到了没有延时的定时器。React 把任务切分成很多片段，通过把任务交给postMessage的回调函数，来让浏览器主线程拿回控制权，进行一些更优先的渲染任务（比如用户输入）。为什么不用执行时机更靠前的微任务呢？关键的原因在于微任务会在渲染之前执行，这样就算浏览器有紧急的渲染任务，也得等微任务执行完才能渲染。


**页面为什么会造成卡顿**

首先显示器会以60hz的频率从显卡的前缓存区读取图像进行显示，浏览器会将生产的图像提交给显卡的后缓存区，待显示器显示完一睁后，会通知显卡发送一个vsync信号给gpu，同时会显卡的前缓存区变成后缓存区。造成掉帧和卡顿、不流畅的原因可能有以下：
渲染进生成图像的速率快于显示器显示的速率，造成gpu的图像不会显示出来，就造成丢正的现象
渲染进程的速率鳗鱼显示器的速率，就会造成卡顿的现象
就算gpu生成图像的速率与显示器显示图片的速率一样，但由于他们是不同系统，生成图像的正律鱼发送vsync的速率也可能不一样



# window.requestIdleCallback()

浏览器渲染每帧的速率为16.6ms每帧，加入渲染进程生存图像速率小于16.6.ms，这个时候渲染主进程就会有一段时间空闲，就可以去执行没有那么紧急的任务，比如垃圾回收，或者requestIdleCallback设置的回调

# requestAnimationFrame

首先css的动画是跟随渲染进程渲染图像处理的，所以渲染进程只要让生成css的动画的过程与vsync时钟保持一致就能让css高效处理。如果用setTimeout去触发css动画的绘制是没有办法保证触发时机与vsync始终保持一致的，所以有了requestAnimation，requestAnimation的回调会在每一真开始执行。



```
利用requestAnimationFrame实现一个requestIdleCallback

// 计算出当前帧 结束时间点

var deadlineTime

// 保存任务

var callback

// 建立通信

var channel = new MessageChannel()

var port1 = channel.port1;

var port2 = channel.port2;



// 接收并执行宏任务

port2.onmessage = () => {

    // 判断当前帧是否还有空闲，即返回的是剩下的时间

    const timeRemaining = () => deadlineTime - performance.now();

    const _timeRemain = timeRemaining();

    // 有空闲时间 且 有回调任务

    if (_timeRemain > 0 && callback) {

        const deadline = {

            timeRemaining, // 计算剩余时间

            didTimeout: _timeRemain < 0 // 当前帧是否完成

        }

        // 执行回调

        callback(deadline)

    }

}


window.requestIdleCallback = function (cb) {

    requestAnimationFrame(rafTime => {

        // 结束时间点 = 开始时间点 + 一帧用时16.667ms

        deadlineTime = rafTime + 16.667

        // 保存任务

        callback = cb

        // 发送个宏任务

        port1.postMessage(null);

    })

}

```








```
const message=new MessageChannel();
const port1=message.port1;
const port2=message.port2;
let callback,deadline;
port2.onMessage=()=>{
  if(performance.now()-deadline){
    callback()
  }
}

window.requestIdleCallback=function(cb){
  window.requestAnimationFrame=(rafTime)=>{
    deadline= rafTime+16.67;
    callback=cb;
    port1.postMessage(null)
  }
}

```







