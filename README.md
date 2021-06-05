# vue-app-base

1. 这是一个使用 Vue CLI 创建出来的 Vue 项目基础结构
2. 有所不同的是这里我移除掉了 vue-cli-service（包含 webpack 等工具的黑盒工具）
3. 这里的要求就是直接使用 webpack 以及你所了解的周边工具、Loader、Plugin 还原这个项目的打包任务
4. 尽可能的使用上所有你了解到的功能和特性

# 项目说明文档----2021-06-05

# 项目结构

```
├── node_modules................................// 项目依赖包
├── public..............................................//无需编译的静态资源目录
│ └── index.html
├── src...................................................//src 目录
│ ├── assets.....................................//资源目录
│ ├── components..........................//组件目录
│ ├── App.vue
│ └── main.js
├── .eslintrc.js......................................//eslint 配置文件
├── babel.config.js..............................//babel 配置文件
├── package.json................................//模块包配置文件
├── webpack.common.js...................//webpack 配置文件(公共部分)
├── webpack.dev.js............................//webpack 配置文件(测试环境部分)
├── webpack.prod.js..........................//webpack 配置文件(生产环境部分)
└── webpack.config.js.......................//webpack 配置文件(未拆分)
```

# 安装 webpack 和 webpack-cli

```
npm i webpack webpack-cli -D
```

# 配置入口起点

创建一个 webpack.config.js 文件

```
module.exports = {
  entry: './src/main.js',
};
```
