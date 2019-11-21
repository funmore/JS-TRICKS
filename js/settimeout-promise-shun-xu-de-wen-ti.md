# settimeout promise 顺序的问题

## 代码0

```text
代码如下
setTimeout(()=>{
    console.log('settime out  happening')
})

var p1= new Promise((resovle,reject)=>{
    console.log('promise happening')
    resovle('then data')
})
p1.then((data)=>{
    console.log(data)
})

代码输出如下
1. promise happening 
2. then data
3. settime out happenng 
```

## 代码1

```text
代码如下
setTimeout(()=>{
    console.log('settime out  happening')
})

var p1= new Promise((resovle,reject)=>{
    setTimeout(()=>{
        console.log('promise happening')
        resovle('then data')
    })
})
p1.then((data)=>{
    console.log(data)
})

代码输出
1. settime out happening 
2. promise happening 
3. then data
```

## event loop 的概念

* Javascript是单线程的，所有的同步任务都会在主线程中执行。
* 当主线程中的任务，都执行完之后，系统会 “依次” 读取任务队列里的事件。与之相对应的异步任务进入主线程，开始执行。
* 异步任务之间，会存在差异，所以它们执行的优先级也会有区别。大致分为 微任务（micro task，如：Promise、MutaionObserver等）和宏任务（macro task，如：setTimeout、setInterval、I/O等）。
* Promise 执行器中的代码会被同步调用，但是回调是基于微任务的。
* 宏任务的优先级高于微任务
* 每一个宏任务执行完毕都必须将当前的微任务队列清空
* 第一个 script 标签的代码是第一个宏任务
* 主线程会不断重复上面的步骤，直到执行完所有任务

