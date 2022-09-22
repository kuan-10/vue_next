##第三章
---
声明式框架比命令式的消耗的性能更多，声明式框架在意的是结果
虚拟DOM的作用就是声明式框架逼近的命令式框架的性能，虚拟DOM是由两部分组成的，一部分是创建JavaScript对象即Vnode，第二部分是创建真实DOM的计算量。
在进行DOM替换的时候，过程是创建新的JS对象即Vnode，在和老的Vnode进行比较，在有必要时进行替换。而innerHTML是销毁以前的DOM，直接创建新的DOM。虚拟DOM的计算量跟数据规模有关，而使用虚拟DOM的原因是让找出差异这个步骤的消耗最少。
###渲染器
虚拟DOM到真实DOM的转化是通过渲染器完成的
```    
    const vnode={
        tag:"div",
        props:{
            onClick:()=>alert('ok!')
        },
        children:'click me'
    }
    //这段代码块表示的就是虚拟DOM，类似于这样的真实DOM
    <div @click=>"()=>alert('ok!')">click me</div>
```
接着需要使用渲染器把虚拟DOM转化为真实DOM
```
  function render(vnode,container){
    const el=document.createElement(vnode.tag)//创建元素
    for( const key in vnode.props){
          if(/^on/.test(key)){
            el.addEventListener(
                key.substring(2).toLowerCase(),
                vnode.props[key]
            )
          }
    }
    if(typeof(vnode.children)==='string'){
        el.appendChild(document.createTextNode(vnode.children))

    }
    else if(Array.isArray(vnode.children)){
        vnode.children.forEach(item=>{
            render(el,item)
        })
    }
    container.appendChild(el)
}
render(vnode,document.body)
```
分析器render所做的思路：
1.用tag标签名称来创建新的节点
2.为元素添加属性和事件
3.处理children

对于渲染器来说，假如vnode发生了修改，应该只更新发生了修改的内容

###组件的本质
组件并不是一个真正的DOM元素，而是一个虚拟的DOM元素，组件返回的也是虚拟的DOM，假如是文本节点，就调用mountElement函数，假如不是文本节点就调用mountComponent函数，其中mountElement函数跟上文的render函数一致，mountCoponent函数可以理解为：
```
function mountComponent(vnode,container){
    //调用组件函数，获取组件要渲染的内容(虚拟DOM)
    const subtree=vnode.tag()
    //tag是一个函数，需要加括号，递归的调用render来渲染
    render(subtree,container)
}
```
同理有其他的object对象
###模板的工作原理
vuejs中通过编译器来完成虚拟DOM转化为真实DOM的过程，编译器的作用就是将模板编译为渲染函数，
```
<div @click="handler">click me</div>
编译器认为这就是个普通的字符串，经编译器转化为渲染函数
render(){
    return h('div',{onClick:handler},'click me')
}

```
无论是使用模板还是直接使用渲染函数，对于一个组件来说，他要渲染的内容最终都是渲染函数产生的，然后渲染器再把渲染函数返回的虚拟DOM渲染为真实DOM，这就是模板的工作原理
###vuejs是各个模块组成的有机整体
编译器和渲染器两者相互作用，共同合作，组件的实现依赖于渲染器，模板的编译依赖于编译器，编译后的生成的代码都是根据渲染器和虚拟DOM的设计决定的。编译器和渲染器之间是存在信息交流的

##总结
声明式框架的好处就是，直接描述结果，用户不需要关注过程，
基本渲染器的实现，渲染器的作用是把虚拟的DOM对象渲染为真实的DOM元素，工作原理是递归的遍历虚拟DOM对象，并调用js的原生api来完成真实的DOM创建，精髓在于后续的更新，通过dif算法来找出变更点，并且只更新新的内容。
