##第一章
声明式框架比命令式的消耗的性能更多，声明式框架在意的是结果
虚拟DOM的作用就是声明式框架逼近的命令式框架的性能，虚拟DOM是由两部分组成的，一部分是创建JavaScript对象即Vnode，第二部分是创建真实DOM的计算量。
在进行DOM替换的时候，过程是创建新的JS对象即Vnode，在和老的Vnode进行比较，在有必要时进行替换。而innerHTML是销毁以前的DOM，直接创建新的DOM。虚拟DOM的计算量跟数据规模有关，而使用虚拟DOM的原因是让找出差异这个步骤的消耗最少。
