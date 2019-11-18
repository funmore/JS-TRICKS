# 什么Vue 双向绑定

## V-MODEL

v-model是vue双向绑定的表现形式

## 哪些数据是reactive的

对于一个vue 实例设定在

```text
data(){
return{
    state0:'',
    state1:''
}
}
```

是被认为是reactive的

## 实现reactive的原理

### 原理图

![](../.gitbook/assets/image%20%286%29.png)

### 组件1: Obeserver-------Object.DefineProperty

对于每个data function 返回的属性, Vue会对属性做Object.DefineProperty 如下

```text
function observe(value,vm){
    if(!value||typeof value !='object'){
        return;
    }
    return new Observer(value)
}
function Observer(value){
    this.data=value;
    this.walk(value);
}
Observer.prototype={
    walk:function(data){
        var self=this;
        Object.keys(data).forEach(key=>{
            this.defineReactive(data,key,data[key])
        })
    },
    defineReactive:function(data,key,val){
        var dep=new Dep();
        var childobj=observe(val)  //实现递归调用，对data上所有的属性配置get/方法
        Object.defineProperty(data,key,{
            enumerable: true,
            configurable: true,
            get: function() {
                if (Dep.target) {
                    dep.addSub(Dep.target);
                }
                return val;
            },
            set: function(newVal) {
                if (newVal === val) {
                    return;
                }
                val = newVal;
                dep.notify();
            }
        })
    }
    
}
```

### 组件2: 订阅管理器Dep

```text
function Dep () {
    this.subs = [];
}
Dep.prototype = {
    addSub: function(sub) {
        this.subs.push(sub);
    },
    notify: function() {
        this.subs.forEach(function(sub) {
            sub.update();
        });
    }
};
Dep.target = null;
```

### 组件3: watcher

```text
//watcher 注册在Dep上当发生对数据的更改，触发set方法-->触发Dep.notify->触发watcher
function Watcher(vm, exp, cb) {
    this.cb = cb;
    this.vm = vm;
    this.exp = exp;
    this.value = this.get();  // 将自己添加到订阅器的操作
}

Watcher.prototype = {
    update: function() {
        this.run();
    },
    run: function() {
        var newValue = this.vm.data[this.exp];
        var oldVal = this.value;
        if (newValue !== oldVal) {
            this.value = newValue;
            this.cb.call(this.vm, newValue, oldVal);
        }
    },
    get: function() {
        Dep.target = this;  // 缓存自己
        var value = this.vm.data[this.exp]  // 强制执行监听器里的get函数
        Dep.target = null;  // 释放自己
        return value;
    }
};
```

### 组件4: 代理proxy

```text
SelfVue.prototype = {
    proxyKeys:function(key){
        var self=this
        Object.defineProperty(this,key, {
            enumerable: true,
            configurable: true,
            get: function proxyGetter(){
                return self.data[key]
            },
            set:function proxySetter(newVal){
                self.data[key]=newVal
            }
        })
    }
}
```

### 组件5: 实例化

```text
function SelfVue(data,el,exp){
    var self=this;
    this.data=data;
    Object.keys(data).forEach(key=>{
        this.proxyKeys(key)
    })
    observe(data)
    el.innerHTML = this.data[exp]
    new Watcher(this,exp,(value)=>el.innerHTML=value)
    return this
}
```



