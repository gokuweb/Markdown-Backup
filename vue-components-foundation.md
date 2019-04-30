---
title: Vue2.x 组件基础
categories:
  - Web前端
tags:
  - vue
  - components
abbrlink: 5bb27d6a
date: 2019-04-26 02:35:58
---

&#8195;&#8195;组件就是可复用的 Vue 实例，所以它们与 `new Vue` 接收相同的选项，例如 `data`、`computed`、`watch`、`methods` 以及生命周期钩子等，仅有的例外是像 `el` 和 `data` 等根实例特有的选项。**<font color="Red">注册组件必须发生在根实例初始化前</font>**。
<!-- more -->

## 组件的使用
&#8195;&#8195;每个 Vue 应用都是通过用 Vue 函数创建一个新的 Vue 实例开始的:
```
var vm = new Vue({
    // 选项
})
```

&#8195;&#8195;Vue.js 的组件的使用有三个步骤： **创建组件构造器、注册组件和使用组件**  ，如下：

```html
<!DOCTYPE html>
<html>
    <head>
        <link rel="stylesheet" href="">
        <script src="https://cdn.jsdelivr.net/npm/vue"></script>
    </head>
    <body>
        <div id="app">
            <!-- 3. #app是Vue实例挂载的元素，应该在挂载元素范围内使用组件 -->
	    <my-component></my-component>
	</div>
    </body>
	
    <script>	
        // 1.创建一个组件构造器
        var myComponent = Vue.extend({
            template: '<div>This is my first component!</div>'
        })
		
        // 2.注册组件，并指定组件的标签，组件的HTML标签为 my-component
        Vue.component('my-component', myComponent)	

	new Vue({
	    el: '#app'
	});
    </script>
</html>
```
我们用以下几个步骤来理解组件的创建和注册：
> 1. **Vue.extend()** 是 Vue 构造器的扩展，调用 Vue.extend() 创建的是一个组件构造器。 
> 2. Vue.extend() 构造器有一个选项对象，选项对象的 **template** 属性用于定义组件要渲染的 HTML。 
> 3. 使用 **Vue.component()** 注册组件时，需要提供 2 个参数，第 1 个参数是组件的标签，第 2 个参数是组件构造器。 
> 4. 组件应该挂载到某个 Vue 实例下，否则它不会生效。

## 全局组件和局部组件
&#8195;&#8195;调用 Vue.component() 注册组件时，组件的注册是全局的，这意味着该组件可以在任意 Vue 示例下使用。如果不需要全局注册，或者是让组件使用在其它组件内，可以用选项对象的 components 属性实现局部注册，上面的示例可以改为局部注册的方式：

```html
<!DOCTYPE html>
<html>
    <head>
        <link rel="stylesheet" href="">
        <script src="https://cdn.jsdelivr.net/npm/vue"></script>
    </head>
    <body>
        <div id="app">
            <!-- 3. my-component只能在#app下使用-->
            <my-component></my-component>
        </div>
    </body>

    <script>
        // 1.创建一个组件构造器
        iivar myComponent = Vue.extend({
            template: '<div>This is my first component!</div>'
        })
		
        new Vue({
            el: '#app',
	    components: {
                // 2. 将myComponent组件注册到Vue实例下
	        'my-component' : myComponent
	    }
        });
    </script>
</html>
```
&#8195;&#8195;由于 my-component 组件是注册在 #app 元素对应的 Vue 实例下的，所以它不能在其它 Vue 实例下使用：
```js
<div id="app2">
    <!-- 不能使用my-component组件，因为my-component是一个局部组件，它属于#app-->
    <my-component></my-component>
</div>

<script>
    new Vue({
	el: '#app2'
    });
</script>
```

&#8195;&#8195;**大量使用全局组件容易产生组件命名冲突**，所以大部分时候我们应该使用局部注册组件，部分可全局复用的公共 UI 组件才会选择全局注册组件。

## 父组件和子组件
我们可以在组件中定义并使用其他组件，这就构成了父子组件的关系。
```html
<!DOCTYPE html>
<html>
    <head>
        <link rel="stylesheet" href="">
        <script src="https://cdn.jsdelivr.net/npm/vue"></script>
    </head>
    <body>
	<div id="app">
            <parent-component></parent-component>
	</div>
    </body>

    <script>	
	var Child = Vue.extend({
            template: '<p>This is a child component!</p>'
	})
		
	var Parent = Vue.extend({
	    // 在Parent组件内使用<child-component>标签
	    template :'<div> \
                      <p>This is a Parent component</p> \
                      <child-component></child-component> \
                      </div>',
	    components: {
		// 局部注册Child组件，该组件只能在Parent组件内使用
		'child-component': Child
	    }
	})
		
	// 全局注册Parent组件
	Vue.component('parent-component', Parent)	
        
        new Vue({
	    el: '#app'
	})
		
    </script>
</html>
```
我们分几个步骤来理解这段代码：
> 1. `var Child = Vue.extend(...)` 定义一了个 Child 组件构造器
> 2. `var Parent = Vue.extend(...)` 定义一个 Parent 组件构造器
> 3. `components: { 'child-component': Child }`，将 Child 组件注册到 Parent 组件，并将 Child 组件的标签设置为 child-component。
> 4. `template :'<div><p>This is a Parent component</p><child-component></child-component></div>`，在 Parent 组件内以标签的形式使用 Child 组件。
> 5. `Vue.component('parent-component', Parent)` 全局注册 Parent 组件。
> 6. 在页面中使用 `<parent-component>` 标签渲染 Parent 组件的内容，同时 Child 组件的内容也被渲染出来。

&#8195;&#8195;Child 组件是在 Parent 组件中注册的，它只能在 Parent 组件中使用，确切地说 **<font color="Red">子组件只能在父组件的 template 中使用</font>** ，如下以标签形式使用是 **<font color="Red">错误的</font>** ：
```html
<div id="app">
    <parent-component>
	<child-component></child-component>
    </parent-component>
</div>
```
&#8195;&#8195;当子组件注册到父组件时，Vue.js 会编译好父组件的模板，模板的内容已经决定了父组件将要渲染的 HTML。`<parent-component>…</parent-component>` 相当于运行时，它的一些子标签只会被当作普通的 HTML 来执行，`<child-component></child-component>` 不是标准的HTML标签，会被浏览器直接忽视掉。

## 组件注册语法糖
### 全局注册
&#8195;&#8195;每次使用组件还要经过创建组件构造器、注册组件、使用组件三个步骤，太繁琐了，语法糖可以简化这个过程，使用 Vue.component() 直接创建和注册组件：
```js
// 全局注册，my-component1是标签名称
Vue.component('my-component1',{
    template: '<div>This is the first component!</div>'
})

var vm1 = new Vue({
    el: '#app1'
})
```
&#8195;&#8195;Vue.component() 的第 1 个参数是标签名称，第 2 个参数是一个选项对象，使用选项对象的 template 属性定义组件模板。Vue 在背后会**自动地调用 Vue.extend()** 。

### 局部注册
在选项对象的 components 属性中实现局部注册：
```js
var vm2 = new Vue({
    el: '#app2',
    components: {
	// 局部注册，my-component2是标签名称
	'my-component2': {
            template: '<div>This is the second component!</div>'
	},
	
        // 局部注册，my-component3是标签名称
	'my-component3': {
	    template: '<div>This is the third component!</div>'
	}
    }
})
```

## <span id="tags">script 和 template 标签<span>
&#8195;&#8195;尽管语法糖简化了组件注册，但在 template 选项中拼接 HTML 元素比较麻烦，这也导致了HTML 和 JavaScript 的高耦合性。庆幸的是，Vue.js 提供了两种方式将定义在 JavaScript 中的 HTML 模板分离出来。

### script 标签
```html
<!DOCTYPE html>
<html>
    <head>
        <link rel="stylesheet" href="">
        <script src="https://cdn.jsdelivr.net/npm/vue"></script>
    </head>
    <body>
	<div id="app">
            <my-component></my-component>
	</div>
		
	<script type="text/x-template" id="myComponent">
	    <div>This is a component!</div>
        </script>
    </body>

    <script>		
	Vue.component('my-component',{
            template: '#myComponent'
	})
		
	new Vue({
	    el: '#app'
	})	
    </script>
</html>
```
&#8195;&#8195;template 选项现在不再是 HTML 元素，而是一个 id，Vue.js 根据这个 id 查找对应的元素，然后将这个元素内的 HTML 作为模板进行编译。
![script-tag](/imgs/script-tag.png)

> 注意：使用 `<script>` 标签时，type 指定为 `text/x-template`，意在告诉浏览器这不是一段 js 脚本，浏览器在解析 HTML 文档时会忽略 `<script>` 标签内定义的内容。

### template 标签

如果使用 `<template>` 标签，则不需要指定 type 属性。
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
	<title></title>
	<link rel="stylesheet" href="">
        <script src="https://cdn.jsdelivr.net/npm/vue"></script>
    </head>
    <body>
	<div id="app">
            <my-component></my-component>
	</div>
		
	<template id="myComponent">
	    <div>This is a component!</div>
	</template>
    </body>

    <script>		
        Vue.component('my-component',{
	    template: '#myComponent'
	})
		
	new Vue({
	    el: '#app'
	})
		
    </script>
</html>
```
&#8195;&#8195;在理解了组件的创建和注册过程后，建议使用 `<script>` 或 `<template>` 标签来定义组件的 HTML 模板。这使得 HTML 代码和 JavaScript 代码是分离的，便于阅读和维护。另外在 Vue.js 中，可创建 `.vue` 后缀的文件，在 `.vue` 文件中定义组件。

## 组件的 el 和 data 选项
&#8195;&#8195;传入 Vue 构造器的多数选项也可以用在 Vue.extend() 或 Vue.component() 中，不过有两个特例： `data` 和 `el`。Vue.js 规定：在定义组件的选项时，**<font color="Red">data 和 el 选项必须使用函数</font>** 。
```js
Vue.component('my-component', {
    data: function(){
	return {a : 1}
    }
})
```

## props 特性和使用
**父组件想要把数据传递给子组件**，怎么实现呢？答案就是 props 。
### 大小写
&#8195;&#8195;camelCase VS kebab-case，HTML 中的特性名是大小写不敏感的，所以浏览器会把所有大写字符解释为小写字符。这意味着使用 DOM 中的模板时，camelCase（驼峰命名法）的 prop 名需要使用其等价的 kebab-case（短横线命名法）命名：
```html
<!DOCTYPE html>
<html>
    <head>
	<link rel="stylesheet" href="">
        <script src="https://cdn.jsdelivr.net/npm/vue"></script>
    </head>
    <body>
        <!-- 在 HTML 中 post-title 是 kebab-case 的格式 -->
        <div id="app">
            <blog-post post-title="hello!"></blog-post>
        </div>
    </body>
    <script>
        Vue.component('blog-post', {
            // 在 JavaScript 中 postTitle 是 camelCase 的格式
            props: ['postTitle'],
            template: '<h3>{{ postTitle }}</h3>'
        })

	new Vue({
	    el: '#app'
	})
    </script>
</html>
```

### 类型和验证
&#8195;&#8195;我们可以为组件的 prop 指定验证要求，例如你知道的这些类型。如果有一个需求没有被满足，则 Vue 会在浏览器控制台中警告你。这在开发一个会被别人用到的组件时尤其有帮助。为了定制 prop 的验证方式，可以为 props 中的值提供一个带有验证需求的对象，而不是一个字符串数组。例如：
```js
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```

### 传递静态或动态 Prop

```html
<!-- 对 prop 进行静态赋值（一个字符串）。-->
<blog-post title="My journey with Vue"></blog-post>

<!-- 即便 `42` 是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post v-bind:likes="42"></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:likes="post.likes"></blog-post>
```

不使用 `v-bind` 情况下，所赋的值只能被作为字符串解析：
```js
Vue.component('my-component', {
  props: ['number'],
  template: '<p>检测number的类型</p>',
  created: function () {
    console.log(typeof this.number)
  }
})

new Vue({
  el: '#app',
  template: '<my-component number="1"></my-component>'
})
```
&#8195;&#8195;number 被检测为字符串，这表明在不加 `v-bind` 绑定的情况下，props 接受到的都是字符串，（注：如果被作为 javacript，”1“ 会被解析为 Number 的 1，而” ‘1’ “才会被解析为 String 的 1），当使用 `v-bind` 的时候，在模板中 props 将会被作为 javascript 解析：
```js
Vue.component('my-component', {
  props: ['number'],
  template: '<p>检测number的类型</p>',
  created: function () {
    console.log(typeof this.number)
  }
})

new Vue({
  el: '#app',
  template: '<my-component v-bind:number="1"></my-component>'
})
```

所以：
<font color="Red">1. 用 v-bind 一般是为了做数据的动态绑定。</font>
<font color="Red">2. 有时 v-bind 并不为了实现 1，只是纯粹为了让字符串内的内容被当作 JS 解析。</font>

详细参考 [官方文档](https://cn.vuejs.org/v2/guide/components-props.html#%E4%BC%A0%E9%80%92%E9%9D%99%E6%80%81%E6%88%96%E5%8A%A8%E6%80%81-Prop) 。

### 单向数据流
&#8195;&#8195;组件实例的作用域是孤立的。这意味着不能并且不应该在子组件的模板内直接引用父组件的数据。props 是单向数据流（[官方文档](https://cn.vuejs.org/v2/guide/components-props.html#%E5%8D%95%E5%90%91%E6%95%B0%E6%8D%AE%E6%B5%81)），所有的 prop 都使得其父子 prop 之间形成了一个单向下行绑定：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。下面的代码定义了一个子组件 `my-component`，在 Vue 实例中定义了 data 选项：
```js
var vm = new Vue({
    el: '#app',
    data: {
	name: 'keepfool',
	age: 28
    },
    components: {
	'my-component': {
            template: '#myComponent',
	    props: ['myName', 'myAge']
	}
    }
})
```
&#8195;&#8195;为了便于理解，可以将这个 Vue 实例看作 `my-component` 的父组件。如果我们想使父组件的数据，则必须先在子组件中定义 props 属性，也就是 `props: ['myName', 'myAge']` 这行代码。

下面使用 `<template>` 标签定义子组件的 HTML 模板（用法参考 [template 标签](#tags)）：
```html
<template id="myComponent">
    <table>
	<tr>
            <th colspan="3">子组件数据</td>
	</tr>
	<tr>
	    <td>my name</td>
	    <td>{{ myName }}</td>
	    <td><input type="text" v-model="myName" /></td>
	</tr>
	<tr>
	    <td>my age</td>
	    <td>{{ myAge }}</td>
	    <td><input type="text" v-model="myAge" /></td>
	</tr>
    </table>
</template>
```
将父组件数据通过已定义好的 props 属性传递给子组件：
```html
<div id="app">
    <table>
	<tr>
            <th colspan="3">父组件数据</td>
	</tr>
	<tr>
	    <td>name</td>
	    <td>{{ name }}</td>
	    <td><input type="text" v-model="name" /></td>
	</tr>
	<tr>
	    <td>age</td>
	    <td>{{ age }}</td>
	    <td><input type="text" v-model="age" /></td>
	</tr>
    </table>

    <my-component v-bind:my-name="name" v-bind:my-age="age"></my-component>
</div>
```
> 注意：在子组件中定义 prop 时，使用了 camelCase 命名法。由于 HTML 特性不区分大小写，camelCase 的 prop 用于特性时，需要转为  kebab-case（短横线命名法）。例如在 prop 中定义的 myName ，在HTML（模板）中，只能写短横线命名法 my-name，在 template 选项属性中时，两种都行。如果只是用了一个小写字母的单词命名，比如 name 则没有这么多麻烦，因为它既符合驼峰写法也符合短横线写法。

![oneway-props](/imgs/oneway-props.gif)

&#8195;&#8195;prop 默认是单向数据绑定：当父组件的属性变化时，将传导给子组件，但是反过来不会，这是为了防止子组件无意修改了父组件的状态。在 Vue1.x 中可以直接使用 `.sync` 显式地指定双向绑定，使组件的数据修改会回传给父组件：
```
<my-component v-bind:my-name.sync="name" v-bind:my-age.sync="age"></my-component>
```

&#8195;&#8195;但是 Vue2.x 移除了 props 的 `.sync` 修饰符，因为真正的双向绑定会带来维护上的问题，子组件可以修改父组件，且在父组件和子组件都没有明显的改动来源。不过在 2.3.0+ 版本又重新增加了完善后 `.sync` 修饰符，配合自定义事件来实现双向绑定。原文中作者写的过滤器和双向绑定在 Vue2.x 已失效，更新后的代码可以参考我下一篇文章 [Vue2.x 自定义事件](https://www.gokuweb.com/webfe/f43d5e8a.html) 。

## 插槽
待补充

## 动态组件 & 异步组件
待补充

## 参考
[Vue2.x 教程](https://cn.vuejs.org/v2/guide/)
[Vue.js——60分钟组件快速入门（上篇）](https://www.cnblogs.com/keepfool/p/5625583.html)
[【Vue】详解Vue组件系统](https://www.cnblogs.com/penghuwan/p/7222106.html)

