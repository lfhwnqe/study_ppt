# 分享

## vue 分享

### vue 的双向绑定到单向绑定

面试题： vue 和 react 的区别

普遍回答会包含这个答案：vue 是双向绑定，react 是单向绑定

从 Vue 0.x 开始，Vue 就用 v-model 来实现「双向绑定」。

```
data: {
    user: { name: 'liLei'}
}
<input v-model="user.name">
```

### v-model 

v-model 实际做了两个操作

1. user.name 的变化自动同步到 input.value
2. input.value 的变化自动同步到 user.name

flux 兴起过后，vue 的作者重新审视了双向绑定，发现双向绑定的一些问题后，更倾向于单向绑定，v-model 被拆成了两个部分

```
data: {
    user: { name: 'liLei'}
},
<input :value="user.name" @input="user.name = $event">
```

### 考虑如下场景

假设有两个地方都在使用同一个数据，如果两个地方同时改了这个数据，那这个数据最后的值是什么呢？

一旦出现了多个更改这个数据的地方，数据就变得不可控了，出现 bug 也很难定位

### 单向绑定的好处

> 一个数据只能一个人改，只有拥有这个数据的人才能更改数据

例如 app 组件把 user.name 同时传递给了两个组件 a 和 b，两个组件都不能更改数据，如果需要更改数据，需要向 app 组件提出申请，app 组件接到了申请，统一把数据的改变通知给使用 user.name 的组件 a 和 b。这样的好处

1.  数据拥有者清楚地知道数据变化的原因和时机（因为是它自己操作数据的）
2. 数据拥有者可以阻止数据变化（app 组件可以轻松的控制 user.name 的变化，让程序更可控）

### 举例

某双向绑定物流公司有 500 万流动资金，有两个部门手机和电脑，手机部门进货需要 400 万，电脑部门进货需要 200 万，他们知道公司有 500 万流动资金，足够进货，所以按照正常协议向销售公司进货，共借款 600 万，账期为一个月。一个月后，公司需要付款时，发现还差 100 万付不出来，于是只能向中智诚借款 100 万作为经营费用。结果因为借款流程太长，资金链断裂，破产了

### 语法糖

 现在 vue 更倾向于单向绑定
但是也提供了  双向绑定语法糖。如果一定想在子组件内更改父组件的值也是可以的(贴物流项目内的代码)

```
.sync语法糖
```

### vue 的渐进式

> 个人认为 vue 的渐进式是它更优于 react 的地方

所以很多原 jquery 项目可以很轻松的切换到 vue 来写。

```
var data = {
    el:"#app"
    data:{
        hello:'hello world'
    }
}
var app = new Vue(data)

$('#box').on('click', '.btn', function(e){
    data.data.hello = e.target.innerText
})
```

对于新手来说非常的友好

## VirtualDOM



### vue 与 jquery 的区别

```
// jquery
$('.box')
  .append('<p>Test</p>')
  .xxx()
  .yyy()
  .jjj()
  ...

// vue
<div v-if="show">{{msg}}</div>
```

jquery 有两种行为，更改状态+操作 DOM，vue 只需操作状态。
vue 的操作可以让我们只关心状态的改变，减少了对 DOM 的操作，大大降低了代码的复杂度

###  渲染

jquery 复杂的 DOM 操作，vue 是怎么帮我们实现的呢

 简单粗暴的方式

```
var msg = 'hello world'
```

当 msg 改变， 用 msg 生成新的 DOM 然后用 innerHTML 把就的 DOM 替换掉

写小功能时，用这种方法没问题，但是框架不行，框架需要做到局部渲染

 解决方案：
VirtualDOM、脏检测、细粒度绑定

vue 早期使用过细粒度方案，如果模版中数据有是 10 个标签使用它，那么变量绑定的就是 10 个具体的标签。数据改变的一瞬间，vue 就知道状态的变化，直接把与这个状态绑定的标签进行更新，达到局部更新的目的

 这个方案有一定的  代价，粒度太小，有一定的追踪开销。

### vue 的 VirtualDOM

> 一个状态对应某个组件，而不是具体标签，状态发生变化时，通知组件，组件再使用 VirtualDOM 来更新具体的 DOM

```
var mydiv = document.createElement('div');
var str = ''
for(var k in mydiv ){
  str += k
}
console.log(str)
```

运行上面代码可以看出，创建一个 DOM 节点，会生成很多不必要的属性，如果每次都去新建销毁，会浪费大量资源

vue 在编译会把 vue 模版编译成一个渲染函数，函数被调用后会返回一个虚拟的 DOM 树，这个 DOM 树用来描述当前页面所处的状态。vue 的 patch 方法负责把 virtulaDOM 渲染到真实 DOM 上

上面讲到了 vue 在数据改变时通知组件，组件内部执行渲染函数生成新的树，然后与旧的树进行对比，得到最终需要更改的真实 DOM 上的变化，然后调用 patch 方法渲染 DOM

### virtulaDOM 原理

```
msg = 'hello world'
var element = {
  tagName: 'ul', // 节点标签名
  props: { // DOM的属性，用一个对象存储键值对
    id: 'list'，
    text: msg
  },
  children: [ // 该节点的子节点
    {tagName: 'li', props: {class: 'item'}, children: ["Item 1"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 2"]},
    {tagName: 'li', props: {class: 'item'}, children: ["Item 3"]},
  ]
}
```

对应的 HTML

```
<ul id='list'>
  <li class='item'>Item 1</li>
  <li class='item'>Item 2</li>
  <li class='item'>Item 3</li>
</ul>
```



### 实现 virtulaDOM 的几个步骤

### 第一步

用 javascript 对象结构表示 DOM 树结构，用这个结构构建真实的 DOM 树并渲染到页面

### 第二步

状态改变时，重新构造 DOM 树，新旧树进行对比纪录差异

- 差异分类
  ```
  const REPLACE = 0   // 更换
  const TEXT = 1  // text变化
  ```
- 差异进行对比
  ```
  dif = [{type:REPLACE,newNode:{tagName:'ul',props:{}]
  ```

### 第三步

把步骤 2 记录的差异应用到步骤 1 创建的真实 DOM 树。

- 遍历虚拟 DOM 树，从步骤 2 生成 dif 中找到差异，根据差异不同进行操作
  ```
  applyPatches(node, currentPatches){
      currentPatches.forEach((currentPatch)=>{
          switch(currentPatch.type){
              case REPLACE:
                  // 做replace操作
              break
              case TEXT:
                  // 做更改text操作
              break
          }
      })
  }
  ```

virtulaDOM 本质是在 js 与 DOM 之间做了一个缓存，最大化的利用高效的 js 来处理 DOM 操作

### 优势

js 的运行速度非常快，相反 DOM 本身很慢，调用原  生 DOM API，浏览器需要在 javascript 引擎的语境下接触原生 DOM 的实现，过程有性能损耗，virtulaDOM 本质上是把浩时间的操作放到存粹的计算来做。把最后的结果一次性渲染的页面上。减少了 repaint 和 reflow。提高了性能

## 表编程

> 编写程序过程中，常遇到很多的 if else，可以用表来代替复杂的分支

例:
假设用户有三个特征 a,b,c

```
if(a&&b&&c){
    return 1
}else if(a&&!b&&!c){
    return 2
}else if(
    // 如何表示下图属于a，b，c中不属于b的部分呢
){}
```

### 图

![123](https://ws3.sinaimg.cn/large/0069RVTdgy1fv89xqivzbj30qy0t643s.jpg)

### 使用表来处理

```
var table = {
    true: { // 第一层true、false代表在不在a内部
        true: { // 第二层代表在b内部
            true: 1 // 第三层代表在c内部
            false: 2
        },
        false: {
            true: 1
            false: 3
        }
    },
    false: {
        true: { // 第二层代表在b内部
            true: 1
            false: 2
        },
        false: {
            true: 1
            false: 3
        }
    }
}
```

## 非技术分享

### teahour

[链接](http://teahour.fm/)

### RXjs

[视频](https://www.youtube.com/watch?v=XRYN2xt11Ek)

## 问题

### 同一个表单字段，不同情况下对应不同的校验规则

 一个证件号对应多个产品类型，每个类型都需要填写
借款额度
产品类型一 借款额度 限制 3-8
产品类型二 借款额度 限制 1-12
产品类型三 借款额度 限制 5-10

实际场景中产品类型会不停增加。
目前的解决方案是把 v-model 绑定到一个字段 表单验证 prop 规则绑定到对应每一个产品的字段。然后 watch 主要字段，再根据主要字段的变更把值赋给当前相应产品类型的字段、清空其他字段的值

另外还有 computed 动态计算验证规则

###  前端的学习路径

>  对前端成长路径有疑问

1.  做一个 T 型前端，技术的宽度如何定义(或应该专哪些，宽泛了解)
2.  一个专业的前端对后端的知识掌握应该到程度（或者最终都要走向全栈的道路？）
