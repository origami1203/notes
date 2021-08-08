# Vue

```shell
npm install --registry=http://registry.npm.taobao.org
```

不使用cnpm，可使用以上方式淘宝源下载

### Vue实例

```javascript
var vm = new Vue({
  // 选项
})
```

常用选项

* `el`

  * 绑定标签

  ```html
  <div class=app>
  </div>
  
  <script>
      var vm = new Vue({
         el: '#app'
      })
  </script>
  ```

* `data`

  * 数据，以键值对形式
  * 可以是对象，数组，集合等，格式类似json

  ```html
  <div class=app>
  </div>
  <script>
      var vm = new Vue({
         data: {
             [name: zhangsan]
         }
      })
  </script>
  ```

* `methods` 定义方法

  ```javascript
  var vm = new Vue({
    data: { a: 1 },
    methods: {
      plus: function () {
        this.a++
      }
    }
  })
  vm.plus()
  vm.a // 2
  ```

* `computed` 计算属性

  ```html
  <div id="example">
    <p>Original message: "{{ message }}"</p>
    <p>Computed reversed message: "{{ reversedMessage }}"</p>
  </div>
  
  <script>
      var vm = new Vue({
          el: '#example',
          data: {
              message: 'Hello'
          },
          computed: {
              // 计算属性的 getter
              reversedMessage: function () {
                  // `this` 指向 vm 实例
                  return this.message.split('').reverse().join('')
              }
          }
      })
  </script>
  ```

  > ​	与methods类似，但如果上面例子使用methods，每次调用都会执行方法，而computed具有缓存作用，只有当message内容发生变化时才会再次执行。
  >
  > ​	computed内部其实有两个方法，getter和setter，默认只有getter方法。

* `watch` 监听器，当data某个属性发生变化时，执行某些操作

* ==**component（重要）**== 组件，用于代码复用

* 各种生命周期钩子，比如在加载页面后使用`created`钩子，向服务器请求json数据。

### 模版语法

##### 插值

* 文本

  ```html
  <span>Message: {{ msg }}</span>
  ```

* innerHtml `v-html`

  ```html
  <span v-html="rawHtml"></span>
  ```

* 属性绑定 `v-bind`

  ```html
  <span v-bind:title="message">233</span>
<!--v-bind语法糖-->
  <span :title="message">233</span>
  
  <button v-bind:disabled="isButtonDisabled">Button</button>
  ```

##### 指令

* `v-if` `v-else` `v-else-if` 条件判断

  ```html
  <div id="app-3">
    <p v-if="seen">现在你看到我了</p>
  </div>
  
  <script>
      var app3 = new Vue({
          el: '#app-3',
          data: {
              seen: true
          }
      })
  </script>
  ```

* `v-for` 循环

  ```html
  <div id="app-4">
    <ol>
      <li v-for="todo in todos" :key="todo">
        {{ todo.text }}
      </li>
    </ol>
  </div>
  
  <script>
      var app4 = new Vue({
          el: '#app-4',
          data: {
              todos: [
                  { text: '学习 JavaScript' },
                  { text: '学习 Vue' },
                  { text: '整个牛项目' }
              ]
          }
      })
  </script>
  ```

  > 一般会在`v-for`上在加上一个`:key`，正常情况下在前面插入一列，效果类似`ArrayList`的插入，会把后面的元素索引全部更改，假如`:key`以后类似`linkedList`，效率更高。

* `v-on` 事件监听

  ```html
  <div id="app-5">
      <p>{{ message }}</p>
      <button v-on:click="reverseMessage">反转消息</button>
      <!--v-on语法糖-->
      <button @click="reverseMessage">反转消息</button>
  </div>
  
  <script>
      var app5 = new Vue({
          el: '#app-5',
          data: {
              message: 'Hello Vue.js!'
          },
          methods: {
              reverseMessage: function () {
                  this.message = this.message.split('').reverse().join('')
              }
          }
      })
  </script>
  ```

* `v-model` 双向绑定，改变表单也会改变data中的数据

  ```html
  <div id="app-6">
    <!--此时input值改变，vue的data的message也会改变-->
    <p>{{ message }}</p>
    <input v-model="message">
  </div>
  
  <script>
      var app6 = new Vue({
          el: '#app-6',
          data: {
              message: 'Hello Vue!'
          }
      })
  </script>
  ```

* `v-show` 指定标签是否展示，类似disable的作用

### 组件

##### 基本示例

```html
<div id="components-demo">
    <!--调用组件-->
    <button-counter></button-counter>
</div>

<script>
    // 注册一个名为 button-counter 的新组件
    Vue.component('button-counter', {
        data: function () {
            return {
                count: 0
            }
        },
        // 组件模版代码
        template: 	`<button v-on:click="count++">
						You clicked me {{ count }} times.
    				</button>`
    })
    
    //vue实例，此时定义的组件在实例外部
    var vm = new Vue({
        el: components-demo
    })


</script>
```

component内部的选项与Vue实例类似，但此时，组件中的data不再是直接提供一个对象

```js
data: {
  count: 0
}
```

而是一个函数，这是因为组件可以被复用，若定义为对象，则多个组件共享同一个data数据，上面例子是一个按钮点击计数器，若共享同一个data，则点击一个组件，其他组件计数也会增加，类似Java中的多线程并发访问，定义为方法，可以每个组件实例拥有自己的data数据。

##### 全局组件与局部组件

组件可分为`全局组件`和`局部组件`，全局组件就是上面的例子，该组件可以在所有的Vue实例中使用，而局部组件定义与Vue实例内部，只有在该Vue实例中使用。

```html
<div id="components-demo">
    <!--调用组件-->
    <button-counter></button-counter>
</div>

<script>
    // 定义两个组件
    var ComponentA = { /*...*/ }
    var ComponentB' = { /*...*/ }

    // vue实例，此时定义的组件在实例内部
    var vm = new Vue({
        el: components-demo,

        // 定义两个局部组件
        components: {
        	conponent-a: ComponentA,
        	conponent-b: ComponentA
    	}
    })

</script>
```

##### 父子组件

组件之间可以嵌套，父组件可以包含子组件。

```html
<div class="app">
    <cnt-b></cnt-b>
</div>

<script>
    // 定义组件A
    var ComponentA = { /*...*/ }
    
    // 定义组件B
    var ComponentB = {
        template: `
			<div>lalala</div>
			<cnt-a></cnt-a>
			`
        ,
        // 在组件b中使用组件a
        components: {
        	cnt-a: ComponentA
    	}
    }

    // vue实例，此时定义的组件在实例内部
    var vm = new Vue({
        el: '#app',

        // 定义组件b
        components: {
        	cnt-b: ComponentB
    	}
    })

</script>
```

##### 组件通信

* 父传子

  将父组件中得到的信息等传递给子组件。

  父传子使用props属性进行传递，vue实例可以看作是一个根组件。

  ```html
  // 定义组件模版
  <template id="cnp">
      <div v:for="movie in movies">
          {{movis}
      </div>
  </template>
  
  
  <div id="app">
    //使用组件,将父组件data中的movies的值传递给子组件的props的cmovies中
      <cpn :cmovies="movies"></cpn>
  </div>
  
  <script>
      var cnp = {
          template: '#cnp',
          /*
          props一般这样写，可以定义一些属性
          props: {
          	cmovies: {
          		//必须是的对象
          		type: Object,
          		//必须有此属性
          		requied: true
          		//默认值
          		default: function(){
          			return {};
          		}
          	}
              
          }
          */
          props: ['cmovies']       
      }
      
      var app = new Vue({
          el: '#app',
          data: {
              movies: ['冰雪奇缘','花木兰','飞屋环游记']
          },
          components: {
              cpn: cnp
          }
      })
  </script>
  ```

  > ​	在props中可以使用驼峰命名法，但是在绑定时，`v-bind:`不能使用驼峰，而要使用连接线。
  >
  > 如props中定义userInfo，在父传子绑定时要使用`v-bind:user-info="xxx"`。

* 子传父

  子传父通过自定义事件方式。

  子组件点击时，子组件调用子组件的`methods`内的某一方法，其方法内调用`this.$emit('childBtnClick')`方法，将事件及数据传递给父组件。

  父组件通过`v-on`监听子组件传递的自定义事件`childBtnClick`，然后调用父组件的`methods`的方法来执行操作。

##### 组件调用

父组件直接调用子组件的属性和方法，或是子组件调用父组件的方法等。

父组件调用子组件，使用`$children`或`$refs` (推荐)。

```html
<div id="app">
    <cnp></cnp>
    <!--子组件的引用为aaa，以便使用$refs调用-->
    <cnp ref="aaa"></cnp>
</div>

<script>
    // 注册一个名为 cnp 的新组件
    Vue.component('cnp', {
        data: function () {
            return {
                count: 0
            }
        },
        
        template: 	`<button v-on:click="count++">
						You clicked me {{ count }} times.
    				</button>`
    })
    
    var vm = new Vue({
        el: '#app',
        componends: {
        	cnp: 'cnp'
    	},
        methods: {
            func(){
                /*
                调用第一个cnp子组件的count属性，
                若此时在前面添加一个<cnp>，则会调用添加为第一的那个，因此不推荐使用
                一般在获取所有子组件的时候才会使用
                */
                this.$children[0].count
                //调用指定ref名字为aaa的子组件
                this.$refs.aaa.count
            }
        }
                     
    })


</script>
```

子组件调用父组件使用`$parent`(使用较少)。

`$root`用于获取Vue实例。

### 插槽

插槽`<slot>`即是在组件中预留一个位置，使用`<slot>`占位，然后在不同的组件使用场景中，自定义插槽中的内容，从而实现很好的扩展性。

```html
<template>
    <div>
        <div>aaa</div>
        <div>bbb</div>
        <slot></slot>
    </div>
</template>
```

使用组件后，默认情况下会显示：

```
aaa
bbb
```

给组件的插槽设置内容

```html
<cnp>
	<button>按钮</button>
</cnp>
```

aaa

bbb

<button>按钮</button>

可以替换任意内容到插槽中，可以设置插槽显示默认内容，当想要替换为其他内容时，在组件内添加，则会代替默认插槽内容。

### webpack

Webpack 可以将多种静态资源 js、css、less 转换成一个静态文件，减少了页面的请求。

webpack是js的打包工具，使用需要先安装node.js，然后安装webpack包。

用于模块式开发时的代码转换，导入等。

```powershell
webpack ./src/main.js ./dist/bundle.js
```

将src目录下的main主文件，打包到dist下的命名为bundle.js的文件。

Webpack 本身只能处理 JavaScript 模块，如果要处理其他类型的文件，就需要使用 loader 进行转换。所以如果我们需要在应用中添加 css 文件，就需要使用到 css-loader 和 style-loader，他们做两件不同的事情，css-loader 会遍历 CSS 文件，然后找到 url() 表达式然后处理他们，style-loader 会把原来的 CSS 代码插入页面中的一个 style 标签中。可以安装loader打包css，转换less，es6转es5等

```shell
npm install css-loader style-loader
```

关于vue

```shell
npm install vue --save
```

vue加载器

```shell
npm install vue-loader vue-template-compiler--save-dev
```

脚手架3以后内置。

### Vue CLI

需要先安装node和webpack

```sh
# 安装脚手架
npm install -g @vue/cli
```

```sh
# 会进行一些配置
vue create 项目名
```

脚手架3以后可以使用gui页面

```sh
vue ui
```
完成配置后即可关闭

```sh
# 在指定项目下使用，在src下添加pligins/axios.js插件
vue add axios
```

package.json

```json
{
    # 项目信息
  "name": "vuetest",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build"
  },
	# 运行依赖
  "dependencies": {
    "axios": "^0.19.2",
    "core-js": "^3.6.4",
    "vue": "^2.6.11",
    "vue-router": "^3.1.6",
    "vuex": "^3.1.3"
  },
	# 开发过程的依赖
  "devDependencies": {
    "@vue/cli-plugin-babel": "~4.3.0",
    "@vue/cli-plugin-router": "~4.3.0",
    "@vue/cli-plugin-vuex": "~4.3.0",
    "@vue/cli-service": "~4.3.0",
    "axios": "^0.18.0",
    "vue-cli-plugin-axios": "0.0.4",
    "vue-template-compiler": "^2.6.11"
  },
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not dead"
  ]
}

```

##### 配置

cli3.0以后没有config文件夹即配置文件，根目录创建`vue.config.js`文件来自定义配置。如进行跨域处理，端口号等。

```js
module.exprots={
    devServer:{
        port:8080,
        host: '1ocalhost',
        //浏览器自动打开
        open: true 
    }
}
```

### 路由

router用于页面之间的跳转，类似SpringMVC中，将指定路径与指定的页面进行匹配的路径匹配。

##### 配置

在创建项目时选择路由，那么脚手架会创建路由，位于router/index.js下。

下面是它的一些配置

```js
//导入要使用的组件
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '../views/Home.vue'
import Book from '../views/Book.vue'

Vue.use(VueRouter)

const routes = [
  {
	path: '/',
    redirect: '/home'	// 将/重定向到/home
  },
  {
    path: '/home',
    name: 'Home',
    component: Home	// /home匹配Home组件
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('../views/About.vue')
  }，
  {
    //添加其他组件及配置路径
    }


const router = new VueRouter({
  routes,
  //使用history模式，即#号跳转模式一般在创建vue项目时选择为history模式
  mode: 'history'
})
```

##### 路由的跳转

在组件中使用`router-link`和`router-view`进行路由跳转与显示。

```html
<router-link to="/">首页</router-link>
<router-link to="/about">关于</router-link>

<!-- router-view为点击上面路由后对应页面要显示的内容 -->
<div>
    <div>
        <router-view></router-view>
    </div>
</div>
```

`router-link`的其他属性

```html
<router-link to="/home" tag="botton" repalce >
    <!-- 
	to为想要跳转的页面，类似href
	tag为此标签想要解析为的标签，默认解析为a标签
	replace表示此标签禁止后退
	-->
</router-link>
```

`$router`是一个全局参数，代表路由，可以用`this.$router.push/replace('/xxx')`等进行路由操作设置

##### 动态路由

一般开发中我们的路由不是固定的，而是动态改变的，如`/user/id`，每个用户的id不同，我们需要设置动态路由。

```js
// 路由index.js的配置
{
    path: '/user/:uid',
    name: 'Home',
    component: Home
}
```

```html
//绑定到指定data上，此时uid为data返回的数据，拼接形成类似/user/233
<router-link v-bind:to=" '/user' + uid">用户</router-link>

<script>
	export default {
        data() {
            return {
                uid: 233
            }
        }
    }
</script>
```

`$route`可用于获取指定路由内的数据信息，如获取上面的uid到页面，`this.$route.param.uid`被点击路由上的数据。

> 注意：区分`$router`和上面`$route`的区别，`$router`是进行一些路由操作，而`$route`是用于获取路由中的数据。

##### 路由传递数据

除了上面的`$route.param.xxx`,还可以使用`$route.query.xxx`获取。

```html
<!-- 地址栏 http://localhost:8080/xxx?name=zhangsan&age=14 -->
<router-link v-bind:to="{path: '/xxx',query: {name: 'zhangsan',age: 14}}">用户</router-link>
```

使用`this.$route.query.age`获取指定数据

##### 懒加载

项目完成后，会把所有js文件打包，此时js文件会非常大，我们可以给应用分包，应用不会在一开始访问时加载所有的文件，而是只加载目前需要的，只有在使用时才加载指定的文件，这就是懒加载。

```js
//懒加载方式不要使用使用import...from...,使用lamuda表达式

const Home = () => import('../views/Home.vue')
const Book = () => import('../views/Book.vue')

Vue.use(VueRouter)

const routes = [
  {
	path: '',
    //用于重定向
    redirect: '/home'
  },
  {
    path: '/home',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('../views/About.vue')
  }，
  {
    //添加其他组件及配置路径
    }


const router = new VueRouter({
  routes,
  //使用history模式，一般在创建vue项目时选择为history模式
  mode: 'history'
})
```

##### 路由嵌套

类似`/home/news`和`/home/message`的效果

先在路由中配置嵌套路由

```js
const routes = [
    {
        path: '/home',
        component: Home,
        //添加嵌套子组件的路由
        childern: [
            {
                //不用加/
                path: 'news',
                component: News
            },
            {
                path: 'message',
                component: Message
            }
        ]
    }
]
```

在父路由的组件中使用`<router-link>`和`<router-view>`

```vue
<template>
<div>
    <div>
        <!-- 此处填写嵌套全路径，而不要只写子嵌套/news -->
        <router-link to="/home/news">新闻</router-link>
        <router-link to="/home/message">消息</router-link>
    </div>
    <routrt-view></routrt-view>
</div>
</template>
```

##### 跨域问题

在前后端分离时，前后端分别有自己的端口，在前端向后端发送ajax请求接受后端数据时，会出现跨域问题，阻止一个域内的js脚本与另一个域中的数据交互。可在前端中解决，也可在后端中解决。

sprintboot解决跨域问题

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override 
    public void addCorsMappings(CorsRegistry registry){
        registry.addMapping("/**")
            .allowedOrigins("*")
            .allowedMethods("GET","HEAD","POST","PUT","DELETE")
            .allowCredentials(true)
            .maxAge(3600)
            .allowedHeaders("*");
    }
}
```

也可以在`Controller`类上或方法上使用`@CrossOrigin`注解来解决。(**开发中常用**)

```java
//	origins:允许可访问的域列表
//	maxAge:准备响应前的缓存持续的最大时间（以秒为单位）
@CrossOrigin(origins = "http://baidu.com", maxAge = 3600)
```

### vuex

vue的状态管理，类似Spring的ioc容器，用于组件之间共享同一个变量。 

### axios

发送网络请求，代替原生ajax

```sh
npm install axios --save
```

导入axios

```js
import axios from 'axios'
```

使用

```js
axios({
    /*
    baseUrl: 'http://localhost'
    url: 'index'
    */
    // 请求路径，与上面等价
    url: 'http://localhost/index',
    
    // get请求要携带的参数
    params: {
        page: 1
    },
    
    //请求方式，默认为get
    method: 'post',
    
    // 请求体数据，支持'PUT', 'POST', 和 'PATCH'
    data: {
    	username: 'zhangsan',
    	gender: '1'
	},
    
    //响应类型
    responseType: 'json',
})	//对响应数据的处理
.then(function (data) {
    console.log(data)
})
```

也可以使用

```js
axios.get/post/delete()
```

根据多个异步请求的结果执行一些操作(例如当用户名和密码同时符合要求时，才能注册)

```js
axios.all([axios(),axios()]
).then(axios.spread( (result1, result2) => {
    if (restlt1&&result2) {
        console.log(xxx)
    }
}))
```

### element-ui

使用脚手架创建项目

npm安装element

```sh
npm install element-ui --save
```

在main.js中引入element

```js
import Vue from 'vue';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
import App from './App.vue';

Vue.use(ElementUI);

new Vue({
  el: '#app',
  render: h => h(App)
});
```

element标签

```
el-container : 整个页面布局
el-aside : 左侧侧边栏
el-menu : 侧边栏中菜单选项
	default-openeds : 默认展开的菜单，通过index设置
	default-openeds : 默认选中的菜单，通过index设置
el-submenu : 可展开的菜单
	index ：对应的索引值
template : 展开菜单的名字
i : 导航的图标
	el-icon-messae	信息类型图标
    el-icon-menu	菜单类型图标
    el-icon-setting	设置类型图标
el-menu-item : 菜单对应的子选项，可以进行嵌套生成多级菜单
```

