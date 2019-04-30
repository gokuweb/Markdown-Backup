---
title: Vue 中 watch、compute 和 methods
categories:
  - Web前端
tags:
  - watch
  - compute
  - methods
abbrlink: 4f74cd29
date: 2019-04-30 02:56:08
---
&emsp;&emsp;Vue 中有几个容易混淆的概念：计算属性（computed）、方法（method）和侦听器（watch），watch 和 computed 相关的函数会自动调用，而 methods 里面定义的函数是需要主动调用的。
<!-- more -->

## 作用机制

1. `watch` 和 `computed` 都是以 Vue 的依赖追踪机制为基础的，它们都试图处理这样一件事情：当某一个数据（称它为依赖数据）发生变化的时候，所有依赖这个数据的“相关”数据“自动”发生变化，也就是自动调用相关的函数去实现数据的变动。

2. `methods` 里面是用来定义函数的，很显然它需要手动调用才能执行。而不像 `watch` 和 `computed` 那样“自动执行”预先定义的函数。

## 性质上
1. `methods` 里面定义的是函数，你显然需要像“fuc()”这样去调用它，`methods` 不处理数据逻辑关系，只提供可调用的函数，例如：
```
new Vue({
  el: '#app',
  template: '<div id="app"><p>{{ say() }}</p></div>',
  methods: {
    say: function () {
      return '我要成为海贼王'
    }
  }
})
```

2. `computed` 是计算属性，事实上和 `data` 对象里的数据属性是同一类的（使用上），例如：
```
computed:{
  fullName: function () { 
    return  this.firstName + lastName
  }
}
```
&#8195;&#8195;使用的时候 `this.fullName` 去取用，就和取 `data` 一样（不要当成函数调用）。

3. `watch` 类似于监听机制 + 事件机制，例如：
```
watch: {
  firstName: function (val) {  
    this.fullName = val + this.lastName
  }
}
```
&#8195;&#8195;firstName 的改变是这个特殊“事件”被触发的条件，而 firstName 对应的函数就相当于监听到事件发生后执行的方法。

## 实例
### watch
watch 和 computed 各自处理数据关系的场景不同：
> 1. `watch` 擅长处理一个数据 **影响多个** 数据。
> 2. `computed` 擅长处理一个数据 **受多个** 数据影响。

&#8195;&#8195;在《海贼王》里面，主角团队的名称叫做“草帽海贼团”，我们把船员依次称为：草帽海贼团索隆、草帽海贼团娜美等，我们希望当船团名称发生变更的时候，这艘船上所有船员的名字一起变更，例如有一天船长路飞为了加强团队建设，弘扬海贼文化，决定将“草帽海贼团”改名为“橡胶海贼团”（路飞是橡胶恶魔果实能力者），用 `watch` 实现如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  </head>

  <div id="app">
    <input type="text" v-model="haiZeiTuan_Name">
    <input type="text" v-model="suoLong">
    <input type="text" v-model="naMei">
    <input type="text" v-model="xiangJiShi">
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      /*
        data选项中的数据：
        1.haiZeiTuan_Name --> 海贼团名称
        2.船员的名称 = 海贼团名称（草帽海贼团） + 船员名称（例如索隆）
 
        这些数据里存在这种关系：
       （多个）船员名称数据 --> 依赖于 --> （1个)海贼团名称数据
        一个数据变化 --->  多个数据全部变化
      */
      data: {
      haiZeiTuan_Name: '草帽海贼团',
      suoLong: '草帽海贼团索隆',
      naMei: '草帽海贼团娜美',
      xiangJiShi: '草帽海贼团香吉士'
    },
      /*
        在watch中，一旦haiZeiTuan_Name（海贼团名称）发生改变
        data选项中的船员名称全部会自动改变
      */
      watch: {
        haiZeiTuan_Name: function (newName) {
          this.suoLong = newName + ' ' + '索隆'
          this.naMei = newName + ' ' + '娜美'
          this.xiangJiShi = newName + ' ' + '香吉士'
          console.log(this.suoLong)
          console.log(this.naMei)
          console.log(this.xiangJiShi)
        }
      }
    })
  </script>
</html>
```

### computed
&#8195;&#8195;路飞的全名叫做蒙奇-D-路飞，他想成为海贼王，但路飞的爷爷卡普（海军英雄）因此感到非常恼怒，于是把“路飞”改成了叫“海军王”，希望他能改变志向，用 `computed` 实现如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  </head>

  <div id="app">
    <input type="text" v-model="firstName">
    <input type="text" v-model="secName">
    <input type="text" v-model="thirdName">
    <input type="text" v-model="fullName">
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      /*
        data选项中的数据：firstName，secName,thirdName
        computed监控的数据：full_Name
        两者关系： fullName = firstName + secName + thirdName
        所以等式右边三个数据任一改变，都会直接修改 fullName
      */
      data: {
        // 路飞的全名：蒙奇·D·路飞
        firstName: '蒙奇',
        secName: 'D',
        thirdName: '路飞'
      },
      computed: {
        fullName: function () {
          return this.firstName + ' ' + this.secName + ' ' + this.thirdName
        }
      }
    })
 
    console.log(vm.fullName)
  </script>
</html>
```

&#8195;&#8195;但是用 `watch` 实现这个功能就比较复杂了：
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  </head>

  <div id="app">
    <input type="text" v-model="firstName">
    <input type="text" v-model="secName">
    <input type="text" v-model="thirdName">
    <input type="text" v-model="fullName">
  </div>

  <script>
    var vm = new Vue({
      el: '#app',
      data: {
        firstName: '蒙奇',
        secName: 'D',
        thirdName: '路飞',
	fullName: '蒙奇 D 路飞'
      },
      watch: {
        firstName: function (val) {
          this.fullName = val + ' ' + this.secName + ' ' + this.thirdName
        },
        secName: function (val) {
          this.fullName = this.firstName + ' '  + val + ' ' + this.thirdName
        },
        thirdName: function (val) {
          this.fullName = this.firstName + ' ' + this.secName + ' ' + val
        }
      }
    })

    console.log(vm.fullName)
  </script>
</html>
```
&#8195;&#8195;`computed` 是用来把多个基础的数据组合成一个复杂的数据；同时获得了 vue 提供的自动变更通知机制。

&#8195;&#8195;`watch` 是利用了 Vue 的自动变更通知机制，用于把这一变化扩散出去（实现相关的更新逻辑或者做和 `computed` 相反的事情）。

### methods
1. `methods`，如果有其他父函数调用它，它会每一次都“乖乖”地执行并返回结果，即使这些结果很可能是相同的，是不需要的。
2. `computed`，计算属性具有缓存的功能，是基于它们的依赖进行缓存的。只在相关依赖发生改变时它们才会重新求值，只要依赖数据没有发生变化 `computed` 就不会再度进行计算。

`methods` 示例如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  </head>

  <div id="app">
    <button @click="getMethodsDate">Methods Get Date</button>
    {{dataOne}}<br>
    <button @click="getComputedDate">Computed Get Date</button>
    {{dataTwo}}
  </div>

  <template id="myComponent">
    <div>
      <h2>{{ msg }}</h2>
      <button v-on:click="showMsg">Show Message</button>
    </div>
  </template>

  <script>
    let app = new Vue({
      el: '#app', 
      data: {
        dataOne: null,
        dataTwo: null
      },
      methods: { 
        getMethodsDate: function () { 
          console.log(new Date())
          this.dataOne = new Date()        
        }, 
        getComputedDate: function () { 
          console.log(this.computedDate) 
          this.dataTwo = this.computedDate
        } 
      }, 
      computed: { 
        computedDate: function () { 
          return new Date() 
        } 
      } 
    })
  </script>
</html>
```
&#8195;&#8195;【注意】为什么两次点击 `computed` 返回的时间是相同的呢？`new Date()` 不是依赖型数据（不是放在 `data` 等对象下的实例数据），所以 `computed` 只提供了缓存的值，而没有重新计算。而 `methods` 调用一次就会执行一次。
 
只有符合如下两个条件，`computed` 才会重新计算：
> 1. 存在依赖型数据 ；
> 2. 依赖型数据发生改变。
 
## 参考
[谈Vue的依赖追踪系统 ——搞懂methods watch和compute的区别和联系](https://www.cnblogs.com/penghuwan/p/7194133.html)
