---
title: Vue2.x 自定义事件
categories:
  - Web前端
tags:
  - v-model
  - emit
  - syn
abbrlink: f43d5e8a
date: 2019-04-26 02:37:10
---

&#8195;&#8195;点击鼠标是一个事件，按下键盘是一个事件，这都是我们熟知的原生事件。那么我们能不能人为定义一个事件呢？原理其实也简单，就是 `$on()` 监听事件，`$emit` 触发事件。prop 主要用于父组件给子组件传递数据，那自定义事件主要是为了子组件跟父组件通信。
<!-- more -->

## 事件名
&#8195;&#8195;不同于组件和 prop，事件名不会被用作一个 JavaScript 变量名或属性名，所以就没有理由使用 camelCase（小驼峰拼写法） 或 PascalCase（大驼峰拼写法）了。并且 v-on 事件监听器在 DOM 模板中会被自动转换为全小写（因为 HTML 是大小写不敏感的），所以 `v-on:myEvent` 将会变成 `v-on:myevent`，这导致 `myEvent` 不可能被监听到，所以推荐使用 **kebab-case 格式的事件名**。

## v-model 介绍
&#8195;&#8195;`v-model` 是 2.2.0+ 版本新增的自定义事件，可以用 `v-model` 指令在表单 `<input>`、`<textarea>` 及 `<select>` 元素上创建 **<font color="Red">双向数据绑定</font>** ，它会根据控件类型自动选取正确的方法来更新元素。`v-model` 本质上只是一个语法糖。

&#8195;&#8195;`v-model` 忽略所有表单元素的 `value`、`checked`、`selected` 特性的初始值而 **总是将 Vue 实例的数据作为数据来源**。我们应该通过 JavaScript 在组件的 `data` 选项中声明初始值。**原生标签中的 v-model 会根据不同的属性抛出不同的事件：**
> 1. `text` 和 `textarea` 元素使用 `value` 属性和 `input` 事件；
> 2. `checkbox` 和 `radio` 使用 `checked` 属性和 `change` 事件；
> 3. `select` 字段将 `value` 作为 `prop` 并将 `change` 作为事件。

&#8195;&#8195;**但是组件上的 v-model 默认会利用名为 value 的 prop 和名为 input 的事件**，

### 标签中的 v-model

1. `<input type="text" v-model="price" />` 等同于如下：
```html
<input type="text"
       v-bind:value="price" 
       v-on:input="price = $event.target.value" />

<!--可简写成如下-->
<input type="text" 
       :value="price" 
       @input="price = $event.target.value" />
```

2. `<input type="checkbox" v-model="status" />` 等同于如下：
```html
<input type="checkbox" 
       :checked="status" 
       @change="status = $event.target.checked" />
```

### 组件中的 v-model

1. `<my-input v-model="price"></my-input>` 等同于如下：
```html
<my-input
    v-bind:value="price"
    v-on:input="price = arguments[0]">
</my-input>

<!--还可写出如下形式-->
<my-input
    :value="price"
    @input="price = $event">
</my-input>
```
&#8195;&#8195;`arguments[0]` 也可以替换成 `$event`，自定义事件中 `$event` 指的就是 `$emit` 传出来的第一个参数，**组件中的 v-model 需要使用 $emit 手动触发事件才能实现双绑定** 。

2. `<my-checkbox v-model="status"></my-checkbox>` 等同于如下：
```html
<my-checkbox
    :value="status"
    @input="status = arguments[0]">
</my-checkbox>
```
这里 `arguments[0]` 也可以替换成 `$event`。

## v-model 实例
### 输入框中 v-model
&#8195;&#8195;示例如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
	
  <body>
    <div id="app">
      <input type="text" v-model="price" />
      <input type="text" 
             v-bind:value="price" 
             v-on:input="price = $event.target.value" />
      <input type="text" 
             :value="price" 
             @input="price = $event.target.value" />
      <p>{{price}}</p>
    </div>
  </body>

  <script>
    var app = new Vue({
      el: '#app',
      data: {
        price: '100'
      },
    });
  </script>
</html>
```
具体如下：
> 1. 将输入框的值绑定到 price 变量上，这个是单向绑定，意味着改变 price 变量的值可以改变 input 的 value，但是改变 value 不能改变 price；
> 2. 监听 input 事件（input 输入框都有该事件，当输入内容时自动触发该事件），当触发该事件就会执行 `price = $event.target.value` ，把输入的值赋值给 price。

这就实现了表单数据的双向绑定。

### 组件输入框中 v-model 
&#8195;&#8195;示例如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
	
  <body>
    <div id="app">
      <my-input v-model="price" ></my-input> 
      <my-input v-bind:value="price" 
                v-on:input="price = arguments[0]" ></my-input>
      <my-input :value="price" 
                @input="price = $event" ></my-input>
      <p>{{price}}</p>
    </div>
  </body>
  
  <template id="inputPrice">
    <input :value="value" 
           @input="$emit('input',$event.target.value)" 
           type="text">
  </template>
  
  <script>
    Vue.component('my-input', {
      template: '#inputPrice',
      props: ["value"],
    });

    var app = new Vue({
      el: '#app',
      data: {
        price: '100',
      }
    });
  </script>
</html>
```
数据从子组件到父组件的流程如下图：
![v-model1](/imgs/v-model1.png)

> 1. `my-input` 组件调用模板 `inputPrice`；
> 2. 模板中 `@input` 表示监听是否有数据输入，`$emit` 表示触发父组件 `input` 事件，并把参数 `$event.target.value` 传递过去；
> 3. 我们已经知道 `v-model` 是个语法糖，其功能是把刚才传递过来的参数赋值给 `price`（当然我们也可以自定义一个其他功能），并把 `price` 绑定到 value。
> 4. 综上，在输入框输入的数据最后都会赋值到 `price`，实现子组件数据传递到父组件的功能。


&#8195;&#8195;上面我们都是用 `$emit` 直接触发父组件的事件，然后传入参数完成赋值，但是如果要实现一些复杂的操作，就需要事件来调用不同的函数执行一些操作，如下： 
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
	
  <body>
    <div id="app">
      <my-input :the-num="price" 
                @cu-input="printValue" ></my-input> 
      <p>{{price}}</p>
    </div>
  </body>
  
  <template id="inputPrice">
    <input :value="theNum" 
           @input="updateVal($event.target.value)" 
           type="text">
  </template>
  
  <script>
    Vue.component('my-input', {
      template: '#inputPrice',
      props: ["theNum"],
      methods: {
        updateVal: function(val) {
        this.$emit('cu-input', val);  
        }
      }
    });

    var app = new Vue({
      el: '#app',
      data: {
        price: '100',
      },
	  methods: {
        printValue: function(val) {
          this.price = val;
        }
      }
    });
  </script>
</html>
```
事件触发和函数调用具体如下图：
![v-model2](/imgs/v-model2.png)

> 1. `v-on` 用于监听 input 事件，当有数据输入时时候触发监听器，调用子组件中 `updateVal()` 函数，并把当前输入数据作为参数；
> 2. `updateVal()` 函数中 `$emit` 用于触发 `cu-input` 事件；
> 3. `cu-input` 事件也处于监听状态，当它被触发后调用父组件 `printValue` 函数；
> 4. `printValue` 函数执行的操作是把参数赋值给了 `price` ，此时我们输入的数据已经传递到了 `price`，直观的体现就是输入框输入数据，下面显示的数据会同时变化；
> 5. 子组件中 `:value="theNum"` 表示输入框的初始值默认是 `the-num`，而第五步中把 `price` 绑定到了 props 的 `the-num`，所以输入框中初始值是 100 。

### 单选框中 v-model
&#8195;&#8195;在创建类似复选框或者单选框的常见组件时，`v-model` 就不好用了：
```
<input type="checkbox" v-model="status" />
```
&#8195;&#8195;`v-model` 给我们提供好了 `value` 属性和 `@input` 事件，但是这里我们需要的不是 `value` 属性，而是 `checked` 属性，并且当你点击这个单选框的时候不会触发 `input` 事件，它只会触发 `change` 事件。

因为 v-model 只是用到了 input 元素上，所以这种情况好解决：
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
	
  <body>
    <div id="app">
      <input type="checkbox" v-model="status" />
      <input type="checkbox" 
             :value="status" 
             @input="status = $event.target.checked" />
      <p>{{status}}</p>
    </div>
  </body>

  <script>
    var app = new Vue({
      el: '#app',
      data: {
        status: false
      },
    });
  </script>
</html>
```

### 组件单选框中 v-model
&#8195;&#8195;示例如下：
```
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
	
  <body>
    <div id="app">
      <my-checkbox v-model="status" ></my-checkbox>
      <my-checkbox :value="status" 
                   @input="status = arguments[0]"></my-checkbox>
      <p>{{status}}</p>
    </div>
  </body>

  <script>
    Vue.component('my-checkbox', {
      template: `
        <input 
          type="checkbox"
          :checked="value"			   
          @change="$emit('input', $event.target.checked)"
        />
      `,
      props: ['value'],
    });

    var app = new Vue({
      el: '#app',
      data: {
	status:true
      },
    });
  </script>
</html>
```

&#8195;&#8195;一个组件上的 `v-model` 默认会利用名为 `value` 的 prop 和名为 `input` 的事件，但是像单选框、复选框等类型的输入控件可能会将 `value` 特性用于不同的目的。`model` 选项可以用来避免这样的冲突：
```js
Vue.component('my-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})
```
&#8195;&#8195;现在在这个组件上使用 `v-model` 的时候：
```
<my-checkbox v-model="status"></base-checkbox>
```
&#8195;&#8195;这里的 `status` 的值将会传入这个名为 `checked` 的 prop。同时当 `<my-checkbox>` 触发一个 `change` 事件并附带一个新的值的时候，这个 `status` 的属性将会被更新。**组件的 props 选项里声明 checked 这个 prop 是必要的**。
&#8195;&#8195;示例如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
	
  <body>
    <div id="app">
      <my-checkbox v-model="status" ></my-checkbox>
      <my-checkbox :checked="status" 
                   @change="status = arguments[0]"></my-checkbox>
      <p>{{status}}</p>
    </div>
  </body>

  <script>
    Vue.component('my-checkbox', {
      template: `
        <input
          type="checkbox"
          :checked="checked"
          @change="$emit('change', $event.target.checked)"
        >
      `,
      model: {
        prop: 'checked',
        event: 'change'
      },
      props: {
        checked: Boolean
      }
    })
    var app = new Vue({
      el: '#app',
      data: {
	    status:true
      },
    });
  </script>
</html>
```

## syn 修饰符
&#8195;&#8195;在有些情况下，我们可能需要对一个 prop 进行“双向绑定”。不幸的是，真正的双向绑定会带来维护上的问题，因为子组件可以修改父组件，且在父组件和子组件都没有明显的改动来源。所以在 2.3.0+ 版本又新增了 `.sync` 修饰符。
&#8195;&#8195;`.sync` 也是一个语法糖，`<my-input :value.sync="price" />` 等同于如下：
```
<my-input 
  :value="price"
  @update:value="price = $event">
</my-input>
```
&#8195;&#8195;`$event` 也可以换成 `arguments[0]`。双向绑定需要 `.sync` 和 `$emit` 的配合才能实现，`$emit` 调用 `update:value` 的时候，把 `$event.target.value` 作为参数传了过去，完成对 `price` 的赋值，我感觉和 `v-model` 大同小异：
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
	
  <body>
    <div id="app">
      <my-input :value.sync="price" ></my-input> 
      <my-input :value="price" 
                @update:value="price = $event" ></my-input>
      <p>{{price}}</p>
    </div>
  </body>
  
  <template id="inputPrice">
    <input :value="value" 
           @input="$emit('update:value',$event.target.value)" 
           type="text">
  </template>
  
  <script>
    Vue.component('my-input', {
      template: '#inputPrice',
      props: ["value"],
    });

    var app = new Vue({
      el: '#app',
      data: {
        price: '100',
      }
    });
  </script>
</html>
```
注意：
> 带有 `.sync` 修饰符的 v-bind 不能和表达式一起使用（例如 v-bind:title.sync=”doc.title + ‘!’” 是无效的）。取而代之的是，你只能提供你想要绑定的属性名，类似 v-model。

&#8195;&#8195;再看一个例子，是参考[【该博客】](http://www.cnblogs.com/keepfool/p/5625583.html)双向绑定一节的内容，原文使用早版本的 `.sync` 进行双向绑定，现已失效，这里提供两种子组件修改父组件数据的方法：
```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  </head>
	
  <body>
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
	  <my-component :my-name="name" 
                    @input="onInput" 
                    :my-age.sync="age">
      </my-component>
    </div>
  </body>
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
        <td>my name</td>
        <td>{{ myName }}</td>
        <td><input type="text" 
                   :value="myName" 
                   @input="updateVal($event.target.value)" /></td>
      </tr>
      <tr>
        <td>my age</td>
        <td>{{ myAge }}</td>
        <td><input type="text" 
                   :value="myAge" 
		   @input="$emit('update:my-age',$event.target.value)"/></td>
      </tr>
	</table>
  </template>
  
  <script>
    new Vue({
      el: '#app',
      data: {
        name: 'keepfool',
        age: 28
      },
      components: {
        'my-component': {
          template: '#myComponent',
          props: ['myName', 'myAge'],
          methods: {
            updateVal: function(val) {
              this.$emit('input', val);  
            }
          }
	    }
	  },
      methods: {
        onInput: function(val) {
          this.name = val;
        }
      }
    })
  </script>
</html>
```
&#8195;&#8195;上面例子中子组件数据中的第一行 `name` 用作参考，没有做双向绑定，附上原文动图，和我上面例子比至少了子组件数据中第一行内容：
![twoway-props](/imgs/twoway-props.gif)

##
## 过滤器
&#8195;&#8195;过滤器可被用于一些常见的文本格式化。过滤器可以用在两个地方：双花括号插值和 `v-bind` 表达式（后者从 2.1.0+ 开始支持）。过滤器应该被添加在 JavaScript 表达式的尾部，由“管道”符号指示：
```html
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>
```
也可以在一个组件的选项中定义本地的过滤器：
```js
filters: {
  capitalize: function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1)
  }
}
```
或者在创建 Vue 实例之前全局定义过滤器：
```js
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0).toUpperCase() + value.slice(1)
})

new Vue({
  // ...
})
```

&#8195;&#8195;这个例子是参考[【该博客】](http://www.cnblogs.com/keepfool/p/5625583.html)最后一个 demo 的内容，原文使用早版本的 `.filter` 进行过滤，现已失效，这里完善一下：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title></title>
    <link rel="stylesheet" href="" />
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
  </head>
	
  <body>
    <div id="app">
      <div id="searchBar">
        Search <input type="text" v-model="searchQuery" />
      </div>
      <simple-grid :data="gridData" 
                   :columns="gridColumns" 
                   :filter-key="searchQuery">
      </simple-grid>
    </div>
    <template id="grid-template">
      <table>
        <thead>
          <tr>
            <th v-for="col in columns">
              {{col}}
            </th>
          </tr>
	</thead>
        <tbody>
          <tr v-for="entry in filteredData">
            <td v-for="col in columns">
              {{entry[col]}}
            </td>
	  </tr>
	</tbody>
      </table>
    </template>
  </body>

  <script>
    Vue.component('simple-grid',{
      template: '#grid-template',
      props: {
        data: Array,
	columns: Array,
        filterKey: String
      },
      computed: {
        filteredData: function(){
          var self = this;
          return self.data.filter(function (item) {
            return item.sex.indexOf(self.filterKey) !== -1;
          })
        }		
      }	
    });
	
    var vm = new Vue({
      el: '#app',
      data: {
        searchQuery: '',
        gridColumns: ['name', 'age', 'sex'],
	gridData: [{
          name: 'Jack',
          age: 30,
          sex: 'Male'
        }, {
	  name: 'Bill',
	  age: 26,
	  sex: 'Male'
	}, {
	  name: 'Tracy',
	  age: 22,
	  sex: 'Female'
	}, {
	  name: 'TraBN',
	  age: 26,
	  sex: 'Female'
	}, {
	  name: 'Chris',
	  age: 36,
	  sex: 'Male'
	}]
      }
    })
  </script>
</html>
```
&#8195;&#8195;如上实现了根据性别筛选数据的功能。

## 参考
[Vue2.x 教程](https://cn.vuejs.org/v2/guide/)
[VUE 进阶教程之：详解 V-MODEL](https://www.cnblogs.com/jiaoyu121/p/7078445.html)
[Vue之自定义组件的v-model](https://www.cnblogs.com/wind-lanyan/p/7899428.html)
[vue 使用自定义事件的表单输入组件](https://segmentfault.com/q/1010000012596246)
[vue2.0自定义事件](https://www.cnblogs.com/lhl66/p/7494770.html)

