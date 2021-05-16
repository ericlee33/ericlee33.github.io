---
title: 超好用的React脚手架，快来试试吧
date: 2021-04-25 18:10:47
tags:
- node.js
- cli
- react.js

top: 100
---
> 自建脚手架地址，觉得好用的话欢迎点个✨：
> [https://github.com/ericlee33/create-compositive-react-app-cli](https://github.com/ericlee33/create-compositive-react-app-cli)




**效果图**
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bf4e4c991aad4ce099f5f492191d4796~tplv-k3u1fbpfcp-zoom-1.image)

## 背景

- 由于最近公司内，立项了多个不同业务，项目之间相互独立，但底层架构相似，从0搭建新项目变成了很寻常的事情，当你一个事情做多了之后，就感觉到很难受，作为一个工具人，需要想办法解决掉这个问题。



- 当通过`create-react-app`创建项目模板之后，还需要删除掉一堆不需要的东西，安装很多框架生态链，Linter等依赖，完成install后，还要配置项目底层各种东西，很耗费精力和时间。


## FB已经提供了create-react-app，为什么还要做react脚手架？


### 先谈谈vue-cli的优点

- 提供了可选配置项（由于`Vue`周边生态较为单一，官方提供的路由/状态管理方案只有一个，不像`React`生态光一个状态管理方案就有`MobX、Redux-Saga、Redux-Thunk`等等。。）
- 令人舒适的`Inquirer`界面（不像`create-react-app`，通过执行脚手架时，传入不同`configuration`来创建不同需求的模板）
- 提供保存配置的功能
    <img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/07defb28770c437c9e7f4be3f73ef1e9~tplv-k3u1fbpfcp-zoom-1.image" width="500" />
    <img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/391857c9c41a42f7857405a48ae1cb02~tplv-k3u1fbpfcp-zoom-1.image" width="500" />

### create-react-app的缺点

在我看来，从0快速搭建项目的角度看，主要有以下几个问题

- 未提供可选`react-router`路由配置的模板
- 未提供状态管理`Lib`
- 没有提供`Linter`，`Coding`过程很不爽
- 一些工具库`npm`包中没有`typing`文件，需要手动去安装`@types`无法享受`vscode`利用`@types`文件在`js`中也提供代码提示的快感。



下面是官方文案提供的Router说明：

> Create React App 并未规定特定的Router(路由)解决方案，但 React Router 是最受欢迎的 Router(路由) 解决方案。
> 要添加它，请运行：

> 或者你可以使用 `yarn`:
> 要尝试它，删除 `src/App.js` 中的所有代码，并将其替换为其网站上的任何示例。 [基本示例](https://reacttraining.com/react-router/web/example/basic) 是开始尝试的好地方。
> 请注意，在部署应用程序之前，[你可能需要配置生产服务器以支持客户端路由](http://www.html.cn/create-react-app/docs/deployment#serving-apps-with-client-side-routing)。

```
npm install --save react-router-dom
```

```
yarn add react-router-dom
```




### 社区有没有现成方案呢？


- 在 `github` 上搜索 `react-scaffold` 、`react脚手架` ，很难在社区中找到一个完整满足自己需求的脚手架

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4361d13c1f4549e79ffba727c509e8bc~tplv-k3u1fbpfcp-zoom-1.image)

- 可以看到搜索结果大多是`Archived`或无人维护的项目。



## 开始自制脚手架

为了解决以上`create-react-app`痛点，我决定仿照`vue-cli`，制作一个`react`的集成脚手架。

### 技术栈

为了更方便开发脚手架，我们需要以下node工具🔧

- **commander** 命令行
- **inquirer** 交互式命令
- **ejs** 模板渲染
- **execa** 子进程管理工具
- **chalk** 用于输出带颜色的log
- **ora** 可以在命令行展示spinning

### 项目目录
```
.
├── README.md
├── bin
│   └── index.js
├── commitlint.config.js
├── lib
│   ├── Creator.js
│   ├── Generator.js
│   ├── PromptModuleAPI.js
│   ├── config.js
│   ├── create.js
│   ├── defaultFeaturesPrompts.js
│   ├── generator
│   │   ├── linter
│   │   │   ├── index.js
│   │   │   └── template ...此处省略template文件
│   │   ├── react
│   │   │   ├── index.js
│   │   │   └── template ...此处省略template文件
│   │   ├── redux
│   │   │   ├── index.js
│   │   │   └── template ...此处省略template文件
│   │   └── router
│   │       ├── index.js
│   │       └── template ...此处省略template文件
│   ├── getPromptModules.js
│   ├── promptModules
│   │   ├── linter.js
│   │   ├── moduleConstantsName.js
│   │   ├── redux.js
│   │   └── router.js
│   ├── templates
│   │   ├── components
│   │   │   ├── index.jsx
│   │   │   └── index.module.scss
│   │   ├── config.js
│   │   ├── createTemplate.js
│   │   ├── templatePrompts.js
│   │   └── views
│   │       ├── index.jsx
│   │       └── index.module.scss
│   └── utils
│       ├── chalk.js
│       ├── copyDir.js
│       ├── executeCommand.js
│       └── isObject.js
└── package.json
```


### 项目总结

总结一下，主要是以下模块

- **/bin/index.js**

命令行调用脚手架的入口
主要放置了`commander`命令

- **/lib/create.js**

执行完在命令行输入的`commander`后，在这里执行变量注入与模板渲染等逻辑

>   1. 注入每种不同模板项，从其 `/lib/promptModules/${name}` 增加不同的`prompts`
>   2. 执行`inquirer`
>   3. 根据用户所选项，拼接`package.json`
>   4. 注入模板所需的`ejs`变量，通过`ejs`进行模板渲染，不同的配置项最终会生成不同的`react-app`项目模板



- **/lib/promptModules/**

为`inquirer`注入不同的`prompts`选项

- **/lib/generator**

`React`项目模板，每一种模板，要求了不同配置项，会根据用户选择，动态注入到最终生成的项目中

- **/utils**

工具

- **/templates**

`components`和`views`模板注入


### 未来优化项

- `ejs`注入可选项不友好，比如为了动态在`App.jsx`入口中，根据用户所选配置项判断是否`import React-Router、Redux`等，v1版本目前是通过根据`answers`，使用`Ejs`来渲染的。未来这一块可以优化为通过修改文件的`AST`，来实现配置的注入，对模板的侵入性会更低。
- 后续需要在项目中加入`Jest`单元测试，保证脚手架后续迭代，对主流程功能不会造成影响。

如果大家想了解项目细节，后续我可以再写一篇关于项目中细节的文章。



## 自制React集成脚手架 功能介绍

### npm地址
[npm源地址](https://www.npmjs.com/package/create-compositive-react-app-cli)

### Introduction 脚手架介绍

减少从0到1搭建项目的成本，快速开发项目
在`create-react-app` `v4.0.3`脚手架基础上，增加了如下项目配置可选项

- `React-Redux + Redux-Thunk + Redux-Logger`
- `React-Router` (可选择`History`, `Hash`模式)
- `Linter / Formatter` （目前提供了`Eslint + EditorConfig + Prettier + CommitLint`）

完成模板创建后，自动安装依赖。

### Getting started 快速使用

- 推荐使用 `npx create-compositive-react-app-cli init <your project name>`
- 也可使用 `npm i -g create-compositive-react-app-cli` `ccra init <your project name>`


### Usage 使用方法

##### 快速搭建项目

`ccra init <name>`
配置项有3种可选项:

- Redux
- React-Router (可选择History, Hash模式)
- Linter / Formatter （目前提供了Eslint + EditorConfig + Prettier + CommitLint）

##### 快速创建Page、Component模板

`ccra create`
可以在 **CLI** 中自行选择创建 **Page** 或 **Component** 输入 **name** 后即可完成模板自动创建


### Features 功能介绍

- 一键快速创建`Page`组件
- 一键快速创建`Components`组件
- 在`ccra init <name>`进行初始化项目时，通过`Inquirer`库的功能，提供给用户各类可选项，可以根据用户所需配置，进行项目自动化构建。
  - 注：状态管理暂时仅提供`Redux`模板
  - 路由管理提供`React-Router` `v5`模板
- 自动安装所需要的`@types`文件，即便用户使用`JavaScript`进行开发，也能在`vscode IDE`下得到函数提示支持