##相应系统的设计与实现
####响应式数据与副作用函数
副作用函数就是会产生副作用的函数
```
function effect(){
    document.body.innerText="ddde"
}//这里effect就是一个副作用函数，他的作用是设置body的值，
但除他之外别的函数也有这个功能，
effect函数的执行会直接或间接的影响别的函数的执行，
就说这个是个副作用函数，
比如一个函数修改了全局变量也是一个副作用函数
```
响应式数据就是当某个值比如obj发生了变化了之后，副作用函数effet会自动执行，如果能实现这个目标，那么obj就是响应式数据

####响应式数据的基本实现
★当effct执行时会触发obj的读取操作
★当修改obj的值时，会触发obj的设置操作
假如可以拦截一个对象的设置和读取操作，就可以做出对应的修改
我们可以用Object.defineproperty来实现，这也是vue2的实现思路
在es2016之后可以采用Proxy来代理实现
```
const data={
    text:"12"
}
const bucket=new Set()
const obj=new Proxy(data,{
    get(target,key){
   bucket.add(effect)
   return target[key]
    },
    set(target,key,newVal){
        target[key]=newVal
        bucket.forEach(item=>item())
        return true
    }
})
function effect(){
    document.body.innerText=obj.text
}
effect()
setTimeout(()=>{
    obj.text="1233"
},1000)
```
上面的响应式处理有个缺点就是假如函数名称改变他就不能正常工作了，解决这个问题的办法如下
```
//定义一个全局的变量用来接收每一次放进桶里的函数上面的函数可修改为
let activeEffect
function effect(fn){
   activeEffect=fn
   fn()
}
//修改effect函数
effect(()=>{
    document.body.innerText=obj.text
})

const obj=new Proxy(data,{
    get(target,key){
        if(activeEffect){
          bucket.add(effect)
        }
   return target[key]
    },
    set(target,key,newVal){
        target[key]=newVal
        bucket.forEach(item=>item())
        return true
    }
})
```
但这里的响应系统依然存在问题，他没有做到一一对应，很多元素之间并没有直接关联但依然是每次修改数据都是将桶中的所有的元素拿出来执行。所以需要修改桶的数据类型为weakmap
```
const obj=new Proxy(data,{
    get(target,key){
        //没有activeEffect直接return
    if(!activeEffect) return target[key]
    //根据target从桶中取得depsMap，他是一个map类型
    let depsMap=bucket.get(target)
    //如果不存在depsmap,那么就新建一个map与之关联
    if(!depsMap){
        bucket.set(target,(deps=new Set()))
    }
    //再根据key从depsmap中取得deps，它是一个set类型
    //里面存储着所有与当前key相关联的副作用函数：effects
    let deps=depsMap.get(key)
    if(!deps){
        depsMap.set(key,(deps=new Set()))
    }
    //最后将当前激活的副作用函数添加到桶里
    deps.add(activeEffect)
    return target[key]
    }
    set(target,key,newVal){
  target[key]=newVal
  //根据target从桶中取得depsMap
  const depsMap=bucket.get(target)
  if(!depsMap)return
  const effects=depsMap.get(key)
  effects && effects.forEach(fn=>fn())
    }
})

```
数据结构关系
![图片](E:/vue/vue_next/src/assets/IMG_6336.JPG)
之所以用weakmap是因为weakmap是弱引用，可以让垃圾回收机制参与进来
