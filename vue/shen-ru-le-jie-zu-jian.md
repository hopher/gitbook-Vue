# 深入了解组件

### [组件名](https://vuejs.bootcss.com/v2/guide/components-registration.html#组件名)  <a id="&#x7EC4;&#x4EF6;&#x540D;"></a>

#### [组件名大小写](https://vuejs.bootcss.com/v2/guide/components-registration.html#组件名大小写)  <a id="&#x7EC4;&#x4EF6;&#x540D;&#x5927;&#x5C0F;&#x5199;"></a>

定义组件名的方式有两种：

**使用 kebab-case**

```javascript
Vue.component('my-component-name', { /* ... */ })
```

当使用 kebab-case \(短横线分隔命名\) 定义一个组件时，你也必须在引用这个自定义元素时使用 kebab-case，例如 `<my-component-name>`。

**使用 PascalCase**

```javascript
Vue.component('MyComponentName', { /* ... */ })
```

当使用 PascalCase \(驼峰式命名\) 定义一个组件时，你在引用这个自定义元素时两种命名法都可以使用。也就是说 `<my-component-name>` 和 `<MyComponentName>` 都是可接受的。注意，尽管如此，直接在 DOM \(即非字符串的模板\) 中使用时只有 kebab-case 是有效的。

### [全局注册](https://vuejs.bootcss.com/v2/guide/components-registration.html#全局注册)  <a id="&#x5168;&#x5C40;&#x6CE8;&#x518C;"></a>

到目前为止，我们只用过 `Vue.component` 来创建组件：

```javascript
Vue.component('my-component-name', {
  // ... 选项 ...
})
```

这些组件是**全局注册的**。也就是说它们在注册之后可以用在任何新创建的 Vue 根实例 \(`new Vue`\) 的模板中。比如：

```javascript
Vue.component('component-a', { /* ... */ })
Vue.component('component-b', { /* ... */ })
Vue.component('component-c', { /* ... */ })

new Vue({ el: '#app' })
```

```javascript
<div id="app">
  <component-a></component-a>
  <component-b></component-b>
  <component-c></component-c>
</div>
```

在所有子组件中也是如此，也就是说这三个组件_在各自内部_也都可以相互使用。

### [局部注册](https://vuejs.bootcss.com/v2/guide/components-registration.html#局部注册)  <a id="&#x5C40;&#x90E8;&#x6CE8;&#x518C;"></a>

全局注册往往是不够理想的。比如，如果你使用一个像 webpack 这样的构建系统，全局注册所有的组件意味着即便你已经不再使用一个组件了，它仍然会被包含在你最终的构建结果中。这造成了用户下载的 JavaScript 的无谓的增加。

在这些情况下，你可以通过一个普通的 JavaScript 对象来定义组件：

```javascript
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
var ComponentC = { /* ... */ }
```

然后在 `components` 选项中定义你想要使用的组件：

```javascript
new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```

对于 `components` 对象中的每个属性来说，其属性名就是自定义元素的名字，其属性值就是这个组件的选项对象。

注意**局部注册的组件在其子组件中**_**不可用**_。例如，如果你希望 `ComponentA` 在 `ComponentB` 中可用，则你需要这样写：

```javascript
var ComponentA = { /* ... */ }

var ComponentB = {
  components: {
    'component-a': ComponentA
  },
  // ...
}
```

#### [基础组件的自动化全局注册](https://vuejs.bootcss.com/v2/guide/components-registration.html#基础组件的自动化全局注册)  <a id="&#x57FA;&#x7840;&#x7EC4;&#x4EF6;&#x7684;&#x81EA;&#x52A8;&#x5316;&#x5168;&#x5C40;&#x6CE8;&#x518C;"></a>

可能你的许多组件只是包裹了一个输入框或按钮之类的元素，是相对通用的。我们有时候会把它们称为[基础组件](https://vuejs.bootcss.com/v2/style-guide/#基础组件名-强烈推荐)，它们会在各个组件中被频繁的用到。

所以会导致很多组件里都会有一个包含基础组件的长列表：

```javascript
import BaseButton from './BaseButton.vue'
import BaseIcon from './BaseIcon.vue'
import BaseInput from './BaseInput.vue'

export default {
  components: {
    BaseButton,
    BaseIcon,
    BaseInput
  }
}
```

而只是用于模板中的一小部分：

```javascript
<BaseInput
  v-model="searchText"
  @keydown.enter="search"
/>
<BaseButton @click="search">
  <BaseIcon name="search"/>
</BaseButton>
```

幸好如果你使用了 webpack \(或在内部使用了 webpack 的 [Vue CLI 3+](https://github.com/vuejs/vue-cli)\)，那么就可以使用 `require.context` 只全局注册这些非常通用的基础组件。这里有一份可以让你在应用入口文件 \(比如 `src/main.js`\) 中全局导入基础组件的示例代码：

```javascript
import Vue from 'vue'
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'

const requireComponent = require.context(
  // 其组件目录的相对路径
  './components',
  // 是否查询其子目录
  false,
  // 匹配基础组件文件名的正则表达式
  /Base[A-Z]\w+\.(vue|js)$/
)

requireComponent.keys().forEach(fileName => {
  // 获取组件配置
  const componentConfig = requireComponent(fileName)

  // 获取组件的 PascalCase 命名
  const componentName = upperFirst(
    camelCase(
      // 剥去文件名开头的 `./` 和结尾的扩展名
      fileName.replace(/^\.\/(.*)\.\w+$/, '$1')
    )
  )

  // 全局注册组件
  Vue.component(
    componentName,
    // 如果这个组件选项是通过 `export default` 导出的，
    // 那么就会优先使用 `.default`，
    // 否则回退到使用模块的根。
    componentConfig.default || componentConfig
  )
})
```

记住**全局注册的行为必须在根 Vue 实例 \(通过** `new Vue`**\) 创建之前发生**。[这里](https://github.com/chrisvfritz/vue-enterprise-boilerplate/blob/master/src/components/_globals.js)有一个真实项目情景下的示例。

## Props

### [Prop 的大小写 \(camelCase vs kebab-case\)](https://vuejs.bootcss.com/v2/guide/components-props.html#Prop-的大小写-camelCase-vs-kebab-case)  <a id="Prop-&#x7684;&#x5927;&#x5C0F;&#x5199;-camelCase-vs-kebab-case"></a>

HTML 中的特性名是大小写不敏感的，所以浏览器会把所有大写字符解释为小写字符。这意味着当你使用 DOM 中的模板时，camelCase \(驼峰命名法\) 的 prop 名需要使用其等价的 kebab-case \(短横线分隔命名\) 命名：

```javascript
Vue.component('blog-post', {
  // 在 JavaScript 中是 camelCase 的
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
```

```javascript
<!-- 在 HTML 中是 kebab-case 的 -->
<blog-post post-title="hello!"></blog-post>
```

### [Prop 类型](https://vuejs.bootcss.com/v2/guide/components-props.html#Prop-类型)  <a id="Prop-&#x7C7B;&#x578B;"></a>

到这里，我们只看到了以字符串数组形式列出的 prop：

```javascript
props: ['title', 'likes', 'isPublished', 'commentIds', 'author']
```

但是，通常你希望每个 prop 都有指定的值类型。这时，你可以以对象形式列出 prop，这些属性的名称和值分别是 prop 各自的名称和类型：

```javascript
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object
}
```

这不仅为你的组件提供了文档，还会在它们遇到错误的类型时从浏览器的 JavaScript 控制台提示用户。你会在这个页面接下来的部分看到[类型检查和其它 prop 验证](https://vuejs.bootcss.com/v2/guide/components-props.html#Prop-验证)。

### [传递静态或动态 Prop](https://vuejs.bootcss.com/v2/guide/components-props.html#传递静态或动态-Prop)  <a id="&#x4F20;&#x9012;&#x9759;&#x6001;&#x6216;&#x52A8;&#x6001;-Prop"></a>

像这样，你已经知道了可以像这样给 prop 传入一个静态的值：

```javascript
<blog-post title="My journey with Vue"></blog-post>
```

你也知道 prop 可以通过 `v-bind` 动态赋值，例如：

```javascript
<!-- 动态赋予一个变量的值 -->
<blog-post v-bind:title="post.title"></blog-post>

<!-- 动态赋予一个复杂表达式的值 -->
<blog-post v-bind:title="post.title + ' by ' + post.author.name"></blog-post>
```

在上述两个示例中，我们传入的值都是字符串类型的，但实际上_任何_类型的值都可以传给一个 prop。

#### [传入一个布尔值](https://vuejs.bootcss.com/v2/guide/components-props.html#传入一个布尔值)  <a id="&#x4F20;&#x5165;&#x4E00;&#x4E2A;&#x5E03;&#x5C14;&#x503C;"></a>

```javascript
<!-- 包含该 prop 没有值的情况在内，都意味着 `true`。-->
<blog-post is-published></blog-post>

<!-- 即便 `false` 是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post v-bind:is-published="false"></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:is-published="post.isPublished"></blog-post>
```

### [单向数据流](https://vuejs.bootcss.com/v2/guide/components-props.html#单向数据流)  <a id="&#x5355;&#x5411;&#x6570;&#x636E;&#x6D41;"></a>

所有的 prop 都使得其父子 prop 之间形成了一个**单向下行绑定**：父级 prop 的更新会向下流动到子组件中，但是反过来则不行

### [Prop 验证](https://vuejs.bootcss.com/v2/guide/components-props.html#Prop-验证)  <a id="Prop-&#x9A8C;&#x8BC1;"></a>

我们可以为组件的 prop 指定验证要求，例如你知道的这些类型。如果有一个需求没有被满足，则 Vue 会在浏览器控制台中警告你。这在开发一个会被别人用到的组件时尤其有帮助。

为了定制 prop 的验证方式，你可以为 `props` 中的值提供一个带有验证需求的对象，而不是一个字符串数组。例如：

```javascript
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 匹配任何类型)
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

当 prop 验证失败的时候，\(开发环境构建版本的\) Vue 将会产生一个控制台的警告

![](http://zhouxianfei.gitee.io/imgstore/front/vue/4.0.png)

#### [类型检查](https://vuejs.bootcss.com/v2/guide/components-props.html#类型检查)  <a id="&#x7C7B;&#x578B;&#x68C0;&#x67E5;"></a>

`type` 可以是下列原生构造函数中的一个：

* `String`
* `Number`
* `Boolean`
* `Array`
* `Object`
* `Date`
* `Function`
* `Symbol`

## 自定义事件（emit）

### [事件名](https://vuejs.bootcss.com/v2/guide/components-custom-events.html#事件名)  <a id="&#x4E8B;&#x4EF6;&#x540D;"></a>

不同于组件和 prop，事件名不存在任何自动化的大小写转换。而是触发的事件名需要完全匹配监听这个事件所用的名称。举个例子，如果触发一个 camelCase 名字的事件：

```javascript
this.$emit('myEvent')
```

则监听这个名字的 kebab-case 版本是不会有任何效果的：

```markup
<my-component v-on:my-event="doSomething"></my-component>
```

不同于组件和 prop，事件名不会被用作一个 JavaScript 变量名或属性名，所以就没有理由使用 camelCase 或 PascalCase 了。并且 `v-on` 事件监听器在 DOM 模板中会被自动转换为全小写 \(因为 HTML 是大小写不敏感的\)，所以 `v-on:myEvent` 将会变成 `v-on:myevent`——导致 `myEvent` 不可能被监听到。

因此，我们推荐你**始终使用 kebab-case 的事件名**。

### [自定义组件的 `v-model`](https://vuejs.bootcss.com/v2/guide/components-custom-events.html#自定义组件的-v-model)  <a id="&#x81EA;&#x5B9A;&#x4E49;&#x7EC4;&#x4EF6;&#x7684;-v-model"></a>

> 2.2.0+ 新增

一个组件上的 `v-model` 默认会利用名为 `value` 的 prop 和名为 `input` 的事件，但是像单选框、复选框等类型的输入控件可能会将 `value` 特性用于[不同的目的](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox#Value)。`model` 选项可以用来避免这样的冲突：

```javascript
Vue.component('base-checkbox', {
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

现在在这个组件上使用 `v-model` 的时候：

```markup
<base-checkbox v-model="lovingVue"></base-checkbox>
```

这里的 `lovingVue` 的值将会传入这个名为 `checked` 的 prop。同时当 `<base-checkbox>` 触发一个 `change` 事件并附带一个新的值的时候，这个 `lovingVue` 的属性将会被更新。

### [`.sync` 修饰符](https://vuejs.bootcss.com/v2/guide/components-custom-events.html#sync-修饰符)  <a id="sync-&#x4FEE;&#x9970;&#x7B26;"></a>

> 2.3.0+ 新增

在有些情况下，我们可能需要对一个 prop 进行“双向绑定”。不幸的是，真正的双向绑定会带来维护上的问题，因为子组件可以修改父组件，且在父组件和子组件都没有明显的改动来源。

这也是为什么我们推荐以 `update:myPropName` 的模式触发事件取而代之。举个例子，在一个包含 `title` prop 的假设的组件中，我们可以用以下方法表达对其赋新值的意图：

```javascript
this.$emit('update:title', newTitle)
```

然后父组件可以监听那个事件并根据需要更新一个本地的数据属性。例如：

```markup
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>
```

为了方便起见，我们为这种模式提供一个缩写，即 `.sync` 修饰符：

```markup
<text-document v-bind:title.sync="doc.title"></text-document>
```

![](http://zhouxianfei.gitee.io/imgstore/front/vue/4.1.png)

当我们用一个对象同时设置多个 prop 的时候，也可以将这个 `.sync` 修饰符和 `v-bind` 配合使用：

```markup
<text-document v-bind.sync="doc"></text-document>
```

这样会把 `doc` 对象中的每一个属性 \(如 `title`\) 都作为一个独立的 prop 传进去，然后各自添加用于更新的 `v-on` 监听器。

![](http://zhouxianfei.gitee.io/imgstore/front/vue/4.2.png)

## 插槽

\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

## 动态组件和异步组件

\#\#\#\#\#\#\#\#\#\#\#\#

## 处理边界情况

### [访问元素 & 组件](https://vuejs.bootcss.com/v2/guide/components-edge-cases.html#访问元素-amp-组件)  <a id="&#x8BBF;&#x95EE;&#x5143;&#x7D20;-amp-&#x7EC4;&#x4EF6;"></a>

在绝大多数情况下，我们最好不要触达另一个组件实例内部或手动操作 DOM 元素。不过也确实在一些情况下做这些事情是合适的。

#### [访问根实例](https://vuejs.bootcss.com/v2/guide/components-edge-cases.html#访问根实例)  <a id="&#x8BBF;&#x95EE;&#x6839;&#x5B9E;&#x4F8B;"></a>

在每个 `new Vue` 实例的子组件中，其根实例可以通过 `$root` 属性进行访问。例如，在这个根实例中：

```javascript
// Vue 根实例
new Vue({
  data: {
    foo: 1
  },
  computed: {
    bar: function () { /* ... */ }
  },
  methods: {
    baz: function () { /* ... */ }
  }
})
```

所有的子组件都可以将这个实例作为一个全局 store 来访问或使用。

```javascript
// 获取根组件的数据
this.$root.foo

// 写入根组件的数据
this.$root.foo = 2

// 访问根组件的计算属性
this.$root.bar

// 调用根组件的方法
this.$root.baz()
```

#### [访问父级组件实例](https://vuejs.bootcss.com/v2/guide/components-edge-cases.html#访问父级组件实例)  <a id="&#x8BBF;&#x95EE;&#x7236;&#x7EA7;&#x7EC4;&#x4EF6;&#x5B9E;&#x4F8B;"></a>

![](http://zhouxianfei.gitee.io/imgstore/front/vue/4.3.png)

![](http://zhouxianfei.gitee.io/imgstore/front/vue/4.4.png)

[访问子组件实例或子元素](https://vuejs.bootcss.com/v2/guide/components-edge-cases.html#访问子组件实例或子元素)

尽管存在 prop 和事件，有的时候你仍可能需要在 JavaScript 里直接访问一个子组件。为了达到这个目的，你可以通过 `ref` 特性为这个子组件赋予一个 ID 引用。例如：

```javascript
<base-input ref="usernameInput"></base-input>
```

现在在你已经定义了这个 `ref` 的组件里，你可以使用：

```javascript
this.$refs.usernameInput
```

来访问这个 `<base-input>` 实例，以便不时之需。比如程序化地从一个父级组件聚焦这个输入框。在刚才那个例子中，该 `<base-input>` 组件也可以使用一个类似的 `ref` 提供对内部这个指定元素的访问，例如：

```javascript
<input ref="input">
```

甚至可以通过其父级组件定义方法：

```javascript
methods: {
  // 用来从父级组件聚焦输入框
  focus: function () {
    this.$refs.input.focus()
  }
}
```

这样就允许父级组件通过下面的代码聚焦 `<base-input>` 里的输入框：

```javascript
this.$refs.usernameInput.focus()
```

当 `ref` 和 `v-for` 一起使用的时候，你得到的引用将会是一个包含了对应数据源的这些子组件的数组。

![](http://zhouxianfei.gitee.io/imgstore/front/vue/4.5.png)

### [程序化的事件侦听器](https://vuejs.bootcss.com/v2/guide/components-edge-cases.html#程序化的事件侦听器)  <a id="&#x7A0B;&#x5E8F;&#x5316;&#x7684;&#x4E8B;&#x4EF6;&#x4FA6;&#x542C;&#x5668;"></a>

现在，你已经知道了 `$emit` 的用法，它可以被 `v-on` 侦听，但是 Vue 实例同时在其事件接口中提供了其它的方法。我们可以：

* 通过 `$on(eventName, eventHandler)` 侦听一个事件
* 通过 `$once(eventName, eventHandler)` 一次性侦听一个事件
* 通过 `$off(eventName, eventHandler)` 停止侦听一个事件

你通常不会用到这些，但是当你需要在一个组件实例上手动侦听事件时，它们是派得上用场的。

![](http://zhouxianfei.gitee.io/imgstore/front/vue/4.6.png)

#### [通过 `v-once` 创建低开销的静态组件](https://vuejs.bootcss.com/v2/guide/components-edge-cases.html#通过-v-once-创建低开销的静态组件)  <a id="&#x901A;&#x8FC7;-v-once-&#x521B;&#x5EFA;&#x4F4E;&#x5F00;&#x9500;&#x7684;&#x9759;&#x6001;&#x7EC4;&#x4EF6;"></a>

渲染普通的 HTML 元素在 Vue 中是非常快速的，但有的时候你可能有一个组件，这个组件包含了**大量**静态内容。在这种情况下，你可以在根元素上添加 `v-once` 特性以确保这些内容只计算一次然后缓存起来，就像这样：

```javascript
Vue.component('terms-of-service', {
  template: `
    <div v-once>
      <h1>Terms of Service</h1>
      ... a lot of static content ...
    </div>
  `
})
```

![](http://zhouxianfei.gitee.io/imgstore/front/vue/4.7.png)

