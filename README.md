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

# 配置 output

webpack.config.js

```
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
};
```

# 配置 mode

webpack.config.js

```
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
};
```

# 样式文件处理

### 安装

```
npm i style-loader less less-loader css-loader -D
```

```
- 将 less 转换为 css
- 解析 CSS 文件后，使用 import 加载，并且返回 CSS 代码
- 将模块的导出作为样式添加到 DOM 中
```

### 配置

webpack.config.js

```
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  module: {
    rules: [
      {
        test: /\.(less|css)$/,
        use: ['style-loader', 'css-loader', 'less-loader'],
      },
    ],
  },
};

```

# CSS 提取

### 安装

```
npm i mini-css-extract-plugin optimize-css-assets-webpack-plugin terser-webpack-plugin -D
```

### 配置

因为 webpack 的 production 只会压缩 JS 代码，所以我们这边需要自己配置 optimize-css-assets-webpack-plugin 插件来压缩 CSS

webpack 官方建议我们放在 optimization 里，当 optimization 开启时，才压缩。

因为我们在 optimization 使用数组配置了 optimize-css-assets-webpack-plugin 插件，webpack 认为我们需要自定义配置，所以导致 JS 压缩失效，相对的我们需要使用 terser-webpack-plugin 插件来压缩 JS 代码

webpack.config.js

```
const path = require('path');
const TerserJSPlugin = require('terser-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  optimization: {
    minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})],
  },
  module: {
    rules: [
      {
        test: /\.(less|css)$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader', 'less-loader'],
      },
    ],
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css',
      chunkFilename: '[id].css',
    }),
  ],
};
```

# 处理文件资源

### 安装

```
npm i url-loader file-loader -D
```

### 配置

当文件资源大于 10000byte 时，生成文件到指定目录

webpack.config.js

```
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|svg)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          esModule: false,
          name: 'imgs/[name].[hash:4].[ext]',
        },
      },
      {
        test: /.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: 'media/[name].[hash:4].[ext]',
        },
      },
      {
        test: /.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: 'fonts/[name].[hash:4].[ext]',
        },
      },
    ],
  },
};
```

# 处理 vue 文件

### 安装

```
npm i vue-loader vue-template-compiler -D
```

### 配置

指定加载 src 目录下的，忽略 node_modules 目录

webpack.config.js

```
const path = require('path');
const VueLoaderPlugin = require('vue-loader/lib/plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        exclude: /node_modules/,
        include: path.join(__dirname, 'src'),
      },
    ],
  },
  plugins: [
    new VueLoaderPlugin(), // vue loader 15 必须添加plugin
  ],
};
```

# 处理 js 文件

将 es6+ 转换为 es5

### 安装

```
npm i babel-loader @babel/core @babel/preset-env -D
```

### 配置

webpack.config.js

```
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
      },
    ],
  },
};
```

配置 babel 使用插件集合将 es6+ 转换为 es5

babel.config.js

```
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        modules: false,
        targets: {
          browsers: ['> 1%', 'last 2 versions', 'not ie <= 8'],
        },
      },
    ],
  ],
};
```

# 处理 js 文件

### 安装

```
npm i html-webpack-plugin -D
```

### 配置

使用模板创建 html，并注入 BASE_URL

我将原 html 文件中的<%= BASE_URL %>改为<%= htmlWebpackPlugin.options.BASE_URL %>

webpack.config.js

```
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  plugins: [
    new HtmlWebpackPlugin({
      BASE_URL: 'public/',
      inject: true,
      template: 'public/index.html',
    }),
  ],
};
```

# 将无需编译的文件复制到打包目录下

### 安装

```
npm i copy-webpack-plugin -D
```

### 配置

babel.config.js

```
const path = require('path');
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  plugins: [
    new CopyWebpackPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, 'public'),
          to: 'public',
        },
      ],
    }),
  ],
};
```

# 删除构建文件夹

### 安装

```
npm i clean-webpack-plugin -D
```

### 配置

babel.config.js

```
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  plugins: [new CleanWebpackPlugin()],
};
```

# 浏览器同步测试工具

### 安装

```
npm i webpack-dev-server -D
```

### 配置

这里我开启了模块热替换，对于样式更改，不会进行浏览器刷新
babel.config.js

```
const path = require('path');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  devServer: {
    // 当使用内联模式(inline mode)时，控制台(console)将显示消息，可能的值有 none, error, warning 或者 info（默认值）。
    clientLogLevel: 'none',
    //当使用 HTML5 History API 时，任意的 404 响应都可能需要被替代为 index.html
    historyApiFallback: {
      index: `index.html`,
    },
    // 启用 webpack 的模块热替换特性
    hot: true,
    // 告诉服务器从哪里提供内容。只有在你想要提供静态文件时才需要。我们这里直接禁用掉
    contentBase: false,
    // 一切服务都启用gzip 压缩：
    compress: true,
    // 指定使用一个 host。默认是 localhost
    host: 'localhost',
    // 指定要监听请求的端口号
    port: '8000',
    // local服务器自动打开浏览器。
    open: true,
    // 当出现编译器错误或警告时，在浏览器中显示全屏遮罩层。默认情况下禁用。
    overlay: false,
    // 浏览器中访问的相对路径
    publicPath: '',
    // 代理配置
    proxy: {
      '/api/': {
        target: 'https://github.com/',
        changeOrigin: true,
        logLevel: 'debug',
      },
    },
    // 除了初始启动信息之外的任何内容都不会被打印到控制台。这也意味着来自 webpack 的错误或警告在控制台不可见。
    // 我们配置 FriendlyErrorsPlugin 来显示错误信息到控制台
    quiet: true,
    // webpack 使用文件系统(file system)获取文件改动的通知。监视文件 https://webpack.js.org/configuration/dev-server/#devserver-watchoptions-
    watchOptions: {
      poll: false,
    },
    disableHostCheck: true,
  },
};
```

# Webpack 多环境多配置文件

开发环境(development)和生产环境(production)的构建目标差异很大。在开发环境中，我们需要具有强大的、具有实时重新加载(live reloading)或热模块替换(hot module replacement)能力的 source map 和 localhost server。而在生产环境中，我们的目标则转向于关注更小的 bundle，更轻量的 source map，以及更优化的资源，以改善加载时间。建议为每个环境编写彼此独立的 webpack 配置。

因为生产环境和开发环境的配置只有略微区别，所以将共用部分的配置作为一个通用配置。使用 webpack-merge 工具，将这些配置合并在一起。通过通用配置，我们不必在不同环境的配置中重复代码。

webpack.common.js

```
const path = require('path');
const VueLoaderPlugin = require('vue-loader/lib/plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'development',
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|svg)$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          esModule: false,
          name: 'imgs/[name].[hash:4].[ext]',
        },
      },
      {
        test: /.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: 'media/[name].[hash:4].[ext]',
        },
      },
      {
        test: /.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: 'fonts/[name].[hash:4].[ext]',
        },
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        exclude: /node_modules/,
        include: path.join(__dirname, 'src'),
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
      },
    ],
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      BASE_URL: 'public/',
      inject: true,
      template: 'public/index.html',
    }),
    new VueLoaderPlugin(), // vue loader 15 必须添加plugin
    new CopyWebpackPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, 'public'),
          to: 'public',
        },
      ],
    }),
  ],
};
```

webpack.dev.js

```
const path = require('path');
const { merge } = require('webpack-merge');
const commonConfig = require('./webpack.common');

module.exports = merge(commonConfig, {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'development',
  devtool: 'eval-source-map',
  devServer: {
    // 当使用内联模式(inline mode)时，控制台(console)将显示消息，可能的值有 none, error, warning 或者 info（默认值）。
    clientLogLevel: 'none',
    //当使用 HTML5 History API 时，任意的 404 响应都可能需要被替代为 index.html
    historyApiFallback: {
      index: `index.html`,
    },
    // 启用 webpack 的模块热替换特性
    hot: true,
    // 告诉服务器从哪里提供内容。只有在你想要提供静态文件时才需要。我们这里直接禁用掉
    contentBase: false,
    // 一切服务都启用gzip 压缩：
    compress: true,
    // 指定使用一个 host。默认是 localhost
    host: 'localhost',
    // 指定要监听请求的端口号
    port: '8000',
    // local服务器自动打开浏览器。
    open: true,
    // 当出现编译器错误或警告时，在浏览器中显示全屏遮罩层。默认情况下禁用。
    overlay: false,
    // 浏览器中访问的相对路径
    publicPath: '',
    // 代理配置
    proxy: {
      '/api/': {
        target: 'https://github.com/',
        changeOrigin: true,
        logLevel: 'debug',
      },
    },
    // 除了初始启动信息之外的任何内容都不会被打印到控制台。这也意味着来自 webpack 的错误或警告在控制台不可见。
    // 我们配置 FriendlyErrorsPlugin 来显示错误信息到控制台
    quiet: true,
    // webpack 使用文件系统(file system)获取文件改动的通知。监视文件 https://webpack.js.org/configuration/dev-server/#devserver-watchoptions-
    watchOptions: {
      poll: false,
    },
    disableHostCheck: true,
  },
  module: {
    rules: [
      {
        test: /\.(less|css)$/,
        use: ['style-loader', 'css-loader', 'less-loader'],
      },
    ],
  },
});
```

webpack.prod.js

```
const path = require('path');
const TerserJSPlugin = require('terser-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const { merge } = require('webpack-merge');
const commonConfig = require('./webpack.common');

module.exports = merge(commonConfig, {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  devtool: 'none',
  optimization: {
    minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})],
  },
  module: {
    rules: [
      {
        test: /\.(less|css)$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader', 'less-loader'],
      },
    ],
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css',
      chunkFilename: '[id].css',
    }),
  ],
});
```

配置 scripts
package.json

```
{
  "scripts": {
    "serve": "webpack-dev-server --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js"
  }
}
```

# ESLint

```
- 最为主流的 JavaScript Lint 工具 监测 JS 代码质量
- ESLint 很容易统一开发者的编码风格
- ESLint 可以帮助开发者提升编码能力
```

### 安装

```
npm i eslint eslint-loader babel-eslint eslint-plugin-vue eslint-plugin-vue-libs -D
```

# 配置

webpack.prod.js

```
const path = require('path');
const TerserJSPlugin = require('terser-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const { merge } = require('webpack-merge');
const commonConfig = require('./webpack.common');

module.exports = merge(commonConfig, {
  entry: './src/main.js',
  output: {
    filename: 'bundle.[hash:8].js',
    path: path.join(__dirname, 'dist'),
  },
  mode: 'production',
  devtool: 'none',
  optimization: {
    minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})],
  },
  module: {
    rules: [
      {
        test: /\.(less|css)$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader', 'less-loader'],
      },
      {
        test: /\.(js|vue)$/,
        use: { loader: 'eslint-loader' },
        enforce: 'pre',
      },
    ],
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css',
      chunkFilename: '[id].css',
    }),
  ],
});
```

package.json

```
{
  "scripts": {
    "lint": "eslint --ext .vue,.js src --fix"
  }
}
```
