# Fenix's BookStore前端工程

如果你此时并不曾了解过什么是“The Fenix Project”，建议先阅读[这部分内容]()。

Fenix Project的主要目的是展示不同的后端技术架构，相对而言，前端并非其重点。不过，前端的页面是比起后端各种服务来要直观得多，能让使用者更容易理解我们将要做的是一件什么事情。假设你是一名驾驶初学者，合理的学习路径肯定应该是把汽车发动，然后慢慢行驶起来，而不是马上从“引擎动力原理”、“变速箱构造”入手去设法深刻地了解一台汽车。所以，先来运行程序，看看最终的效果是什么样子吧。

## 运行程序

以下几种途径，可以马上浏览最终的效果：

- 从互联网已部署的网站访问：

> 直接在浏览器访问：http://icyfenix.net/

- 通过Docker容器方式运行：

> ```bash
> $ docker run -p 80:80 --name bookstore icyfenix/bookstore:frontend 
> ```
>
> 然后在浏览器访问：http://localhost

- 通过Git上的源码，以开发模式运行：
>```bash
># 克隆获取源码
> $ git clone https://github.com/fenixsoft/fenix-bookstore-frontend.git
> 
> # 进入工程根目录
> $ cd fenix-bookstore-frontend
> 
> # 安装工程依赖
> $ npm install
> 
> # 以开发模式运行，地址为localhost:8080
> $ npm run dev
> ```
> 
> 然后在浏览器访问：http://localhost:8080
>

![](markdown/sshot.jpg)

也许你已注意到，以上这些运行方式，均没有涉及到任何的服务端、数据库的部署。现代软件工程里，前、后端开发是可以完全分离的，Fenix's BookStore以开发模式运行时，会自动使用Mock.js拦截住所有的远程服务请求，并以事项准备好的数据来完成这些请求的响应。

同时，你也应当注意到，以纯前端方式运行的时候，所有对数据的修改请求实际都是无效的。譬如用户注册，无论你输入何种用户名、密码，由于请求的响应是静态预置的，所以最终都会以同一个预设的用户登陆。也是因此，我并没有提供”默认用户“、”默认密码“一类的信息供用户使用，你可以随意输入即可登陆。

不过，那些只维护在前端的状态依然是可以变动的，典型的如对购物车、收藏夹的增删改。让后端服务保持无状态，而把状态维持在前端中的设计，对服务的伸缩性和系统的鲁棒性都有着极大的益处，多数情况下都是值得倡导的良好设计。而其伴随而来的状态数据导致请求头变大、链路安全性等问题，都会在服务端部分专门讨论和解决。

## 构建产品

当你将程序用于正式部署时，一般不应部署开发阶段的程序，而是要进行产品化（production）与精简化（minification），你可以通过以下命令，由node.js驱动webpack来自动完成：

```bash
# 编译前端代码
$ npm run build
```

或者使用--report参数，同时输出依赖分析报告：

```bash
# 编译前端代码并生成报告
$ npm run build
```

编译结果存放在/dist目录中，应将其拷贝至Web服务器的根目录使用。对于Fenix Project的服务端而言，则是拷贝到Gateway工程中资源目录下。

## 与后端联调

同样出于前后端分离的目的，理论上后端通常只应当依据约定的接口（名称、参数结构、返回值结构、服务水平协议等）提供服务，并以此为依据进行不依赖前端的独立测试，最终集成时使用的是编译后的前端产品。

不过，在开发期就进行的前后端联合在现今许多企业之中仍是主流形式，由一个人“全栈式”地开发某个功能时更是如此，因此，当要在开发模式中进行联调时，需要修改项目根目录下的main.js文件，使其**不**导入Mock.js，即如下代码所示的条件语句判断为假

```javascript
/**
 * 默认在开发模式中启用mock.js代替服务端请求
 * 如需要同时调试服务端，请修改此处判断条件
 */
// eslint-disable-next-line no-constant-condition
if (process.env.NODE_ENV === 'development') {
  require('./api/mock')
}
```

也有其他一些相反的情况，需要在生产包中仍然继续使用Mock.js提供服务时（譬如Docker镜像icyfenix/bookstore:frontend就是如此），同样应修改该条件，使其结果为真，在开发模式依然导入了Mock.js即可。

## 工程结构

Fenix's BookStore的工程结构完全符合vue.js工程的典型习惯，事实上它在建立时就是通过vue-cli初始化的。此工程的结构与其中各个目录的作用主要如下所示：

```
+---build							webpack编译配置，该目录中的内容一般不做改动
+---config							webpack编译配置，将用户需要改动的内容提取至此
+---dist							编译输出结果存放的位置
+---markdown						与项目无关，用于支持markdown的资源（如图片）
+---src
|   +---api                         本地与远程的API接口
|   |   +---local					本地服务，如localStorage、加密等
|   |   +---mock					远程API接口的Mock
|   |   |   \---json				Mock返回的数据
|   |   \---remote					远程服务
|   +---assets						资源文件，会被webpack哈希和压缩
|   +---components					vue.js的组件目录，按照使用页面的结构放置
|   |   +---home
|   |   |   +---cart
|   |   |   +---detail
|   |   |   \---main
|   |   \---login
|   +---pages						vue.js的视图目录，存放页面级组件
|   |   \---home
|   +---plugins						vue.js的插件，如全局异常处理器
|   +---router						vue-router路由配置
|   \---store						vuex状态配置
|       \---modules					vuex状态按名空间分隔存放
\---static							静态资源，编译时原样打包，不会做哈希和压缩
```

## 组件
Fenix's BookStore前端部分基于以下开源组件和免费资源构建：

- [Vue.js](https://cn.vuejs.org/)
  渐进式JavaScript框架
- [Element](https://element.eleme.cn/#/zh-CN)
  一套为开发者、设计师和产品经理准备的基于Vue 2.0的桌面端组件库
- [Axios](https://github.com/axios/axios)
  Promise based HTTP client for the browser and node.js
- [Mock.js](http://mockjs.com/)
  生成随机数据，拦截 Ajax 请求
- [DesignEvo](https://www.designevo.com/cn)
  一款由PearlMountain有限公司设计研发的logo设计软件

## 协议

本作品采用[知识共享署名 4.0 国际许可协议](http://creativecommons.org/licenses/by/4.0/)进行许可。 

您可以自由地：

- 共享 — 在任何媒介上以任何形式复制、发行本作品
- 演绎 — 修改、转换或以本作品为基础进行二次创作

只要您遵守许可协议条款中署名、非商业性使用、相同方式共享的条件，许可人就无法收回您的这些权利。