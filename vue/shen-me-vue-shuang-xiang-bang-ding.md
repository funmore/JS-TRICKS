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

## 实现reactive的关键----Object.DefineProperty

对于每个data function 返回的属性, Vue会对属性做Object.DefineProperty 如下

```text
function defineReactive(data, key, val) {
    observe(val); // 递归遍历所有子属性
    Object.defineProperty(data, key, {
        enumerable: true,
        configurable: true,
        get: function() {
            return val;
        },
        set: function(newVal) {
            val = newVal;
            console.log('属性' + key + '已经被监听了，现在值为：“' + newVal.toString() + '”');
        }
    });
}
function observe(data) {
    if (!data || typeof data !== 'object') {
        return;
    }
    Object.keys(data).forEach(function(key) {
        defineReactive(data, key, data[key]);
    });
};
```

## Async 



