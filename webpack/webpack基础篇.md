- [01｜为什么需要构建工具](#01为什么需要构建工具)
- [02｜前端构建的演变过程](#02前端构建的演变过程)
- [03｜为什么选择webpack](#03为什么选择webpack)
- [04｜webpack 配置文件](#04webpack-配置文件)
- [05｜webpack 安装](#05webpack-安装)
- [06｜webpack 打包方式](#06webpack-打包方式)
- [07｜webpack核心概念之 entry：制定打包的入口 生成依赖图](#07webpack核心概念之-entry制定打包的入口-生成依赖图)
- [08｜webpack核心概念之 output：告诉 webpack 如何将编译后的文件输出到磁盘](#08webpack核心概念之-output告诉-webpack-如何将编译后的文件输出到磁盘)
- [09｜webpack核心概念之 loaders](#09webpack核心概念之-loaders)
- [10｜webpack核心概念之 plugins：用于 bundle 文件的优化，资源盖里和环境变量注入 作用于整个构建过程](#10webpack核心概念之-plugins用于-bundle-文件的优化资源盖里和环境变量注入-作用于整个构建过程)
- [11｜webpack核心概念之 mode：用来指定当前的构建环境是：production、development还是none](#11webpack核心概念之-mode用来指定当前的构建环境是productiondevelopment还是none)
- [12｜解析ES6 和 React JSX](#12解析es6-和-react-jsx)
- [14｜解析图片和字体](#14解析图片和字体)
- [15｜webpack中的文件监听【缺点】每次需要手动刷新浏览器 在开发调试中很不方便](#15webpack中的文件监听缺点每次需要手动刷新浏览器-在开发调试中很不方便)
- [16｜webpack中的热更新及原理分析](#16webpack中的热更新及原理分析)
- [17｜文件指纹策略：chunkhash、contenthash和hash](#17文件指纹策略chunkhashcontenthash和hash)
- [18｜代码压缩 - HTML、CSS和JS代码](#18代码压缩---htmlcss和js代码)

# 01｜为什么需要构建工具
- 转换ES6语法$$
- 转换JSX
- CSS前缀补全/预处理器
- 压缩混淆
- 图片压缩
# 02｜前端构建的演变过程
- ant + YUI Tool
- grunt ： 文件流
- fis3 / gulp ： 没有在维护 fis3
- rollup / webpack / parcel
# 03｜为什么选择webpack
- 社区生态丰富
- 配置灵活和插件化扩展
- 官方更新迭代速度快
# 04｜webpack 配置文件
webpack默认配置文件： webpack.config.js  或者 webpack --config 制定配置文件
# 05｜webpack 安装

需要安装 npm 和 node环境
2021.1.20 
- node - v14.7.0 
- npm - v6.14.7

- 创建空目录 和 packgae.json
   - mkdir my-project
   - cd my-project
   - npm init -y
- 安装 webpack 和 webpack-cli
   - npm install webpack webpack-cli --save-dev
   - 检查是否安装成功：./mode_modules/.bin/webpack -v
# 06｜webpack 打包方式
- 第一种：./node_modules/.bin/webpack 指令打包
- 第二种：通过 npm script 运行 webpack
  - 通过 npm run build 运行构建
  - 原理：模块局部安装会在 node_modules/.bin 目录创建软链接

# 07｜webpack核心概念之 entry：制定打包的入口 生成依赖图
  - 单入口：是一个字符串    例如：'./src/index.js'
  - 多入口：是一个对象      例如：
  ```javascript
    entry: {
        app: './src/app.js',
        adminApp: './src/adminApp.js'
    }
  ```
# 08｜webpack核心概念之 output：告诉 webpack 如何将编译后的文件输出到磁盘
  - 多入口配置
    ```
    entry: {
        app: './src/app.js',
        adminApp: './src/adminApp.js'
    },
    output: {
        filename: '[name].js', // 通过占位符 确保文件名称的唯一
        path: __dirname + 'dist'
    }
    ```
# 09｜webpack核心概念之 loaders
  - 【概念】webpack 开箱即用之支持JS和JSON两种文件类型，通过Loaders去豉汁其他文件类型并且把它们转换成有效的模块，并且可以添加到依赖图中
  - 【本质】本质上是一个函数，接收源文件作为参数，返回转换的结果
  - 【常用】常见的Loaders 以及作用
    - babel-loader：转换ES6、ES7等JS新特性语法
    - css-loader：支持.css文件的加载和解析
    - less-loader：将less文件转换成CSS
    - ts-loader：将TS转换成JS
    - file-loader：进行图片、字体等的打包
    - raw-loader：将文件以字符串的形式倒入
    - thread-loader：多进程打包JS和CSS
  - 【使用】module.rules 数组中写入
  ```
    module.exports = {
      // ...
      module: {
        rules: [
          {
            test: /\.text$/ , // 
            use: 'raw-loader'
          }
        ]
      }
    }
  ```
# 10｜webpack核心概念之 plugins：用于 bundle 文件的优化，资源盖里和环境变量注入 作用于整个构建过程  
  - 【常用】常用插件以及作用
    - CommonsChunkPlugin：将 chunks 相同的模块代码提取成公共JS
    - CleanWebpackPlugin：清理构建目录
    - ExtractTextWebpackPlugin：将CSS从 bundle 文件里 提取成一个独立的CSS文件
    - CopyWebpackPlugin：将文件或者文件夹拷贝到构建的输出目录
    - HtmlWebpackPlugin：创建 html 文件去承载输出的 bundle
    - UglifyjsWebpackPlugin：压缩JS
    - ZipWebpackPlugin：将打包出的资源生成一个zip包
  - 【使用】放入到plugin数组即可
  ```
  module.exports = {
    // ...
    plugins: [
      new HtmlWebpackPlugin({
        template: './src/index.html'
      })
    ]
  }
  ``` 
# 11｜webpack核心概念之 mode：用来指定当前的构建环境是：production、development还是none
  - 【好处】设置 mode 可以使用webpack 内置的函数， 默认值为 production
  - 【功能】mode 的内置函数功能 设置 process.env.NODE_ENV 为如下值时的触发的内置函数
    - development：开启 NamedChunkPlugin 和 NamedModulesPlugin
    - production：开启 
      - FlagDependencyUsafePlugin
      - FlagIncludedChunksPlugin
      - ModuleConcatenationPlugin
      - NoEmitOnErrorPlugin
      - OccurrenceOrderPlugin
      - SideEffectsFlagPlugin
      - TerserPlugin
    - none：不开启任何优化选项

# 12｜解析ES6 和 React JSX
  - 【解析ES6】 需要安装 @babel/core、@babel/preset-env和babel-loader 看.babelrc里面配置了什么就需要什么
    - npm install @babel/core @babel/preset-env babel-loader --save-dev
    - 或简写 npm i @babel/core @babel/preset-env babel-loader -D
    ```webpack.config.js 中
    module: {
      rules: [
        {
          test: /\.js$/,
          use: 'babel-loader'
        }
      ]
    }
    ``` 
    ``` .babelrc babel的配置文件
    {
        "presets": [ // 预设 是一系列功能的集合
            "@babel/preset-env" // 增加ES6 的 babel preset 配置
        ],
        "plugins": [ // 每一个plugin 对应一个功能
            "@babel/proposal-class-properties"
        ]
    }
  ```
  - 【解析 React JSX】
  - 【安装】npm i react react-dom @babel/preset-react -D
   ```
  .babelrc 配置
  {
    "presets": [
      "@babel/preset-env",
      "@babel/preset-react" // 增加 React 的 babel preset配置
    ]
  }
  
  ```

# 13｜解析CSS、Less和Sass
 - 【原理】css-loader 用于加载.css文件 并且转换成 commonjs 对象； style-loader 将样式通过 style 标签插入到 head 中
 - 【安装】css-loader 和 style-loader 
 - 【安装】css-loader、style-loader和less-loader、less
```javascript webpack.config.js
module: {
  rules: [
      {
        test: /.css$/, // 解析css
        use: [
            // ⚠️ loader的调用是链式调用 他的调用是从右到左的 
            // 所以在这里需要先写 style-loader 再写 css-loader 实际调用的顺序才是我们期望的
            'style-loader', // 用于加载.css文件 并且转换成 commonjs 对象； 
            'css-loader' // 将样式通过 style 标签插入到 head 中
        ]
    },
    {
      test: /.less$/, // 解析less
      use: [
        'style-loader',
        'css-loader',
        'less-loader'
      ]
    }
  ]
}
```

# 14｜解析图片和字体
  - 【安装】file-loader（常用）可以用于处理文件 也可以处理字体
  - 【使用】
    ```javascript webpack.config.js文件中 module.rules配置如下    
    {
        test: /.(png|jpg|gif|jpeg|svg)$/,
        use: 'file-loader', // 可以用来 处理图片 也可以用来处理字体
       
    },
    {
        test: /.(woff|woff2|eot|ttf|otf)$/,
        use: 'file-loader'
    }
    ```
  - 【资源解析】使用 url-loader 也可以处理图片和字体 可以设置较小资源自动base64
  ```javascript
   {
      test: /.(png|jpg|gif|jpeg|svg)$/, //file-loader 可以用来 处理图片 也可以用来处理字体
      use: [
          {
              loader: 'url-loader',
              options: {
                  limit: 10240 // url-loader 内部也是使用了file-loader 单位是字节 10240 byte => 10 kb 如果图片小于10k的话 webpack会自动转为base64
              }
          }
      ]
  }
  ``` 
# 15｜webpack中的文件监听【缺点】每次需要手动刷新浏览器 在开发调试中很不方便
  - 【定义】文件监听是发现源码发生变化时，自动构建出新的输出文件
  - 【使用】webpack 开启文件监听模式，有两种方式：
    - 启动 webpack 命令时，带上 --watch 参数
      - 【缺点】每次需要手动刷新浏览器 在开发调试中很不方便
       ```javascript package.json    
        // ...
        "script": {
          "watch": "webpack --watch"
        }      
      ```  
    - 在配置 webpack.config.js 中 设置 watch: true
    ```javascript
    {
      // ...
      watch: true, // 默认false 也就是不开启
      watchOptions: { // 只有开启监听模式时 watchOptions才有意义
        ignored: /node_modules/, // 默认为空，不监听的文件或文件夹，支持正则匹配
        aggregateTimeout: 300, // 监听到变化后会等300ms再去执行，默认 300ms
        poll: 1000 // 判断文件是否发生变化是通过不停询问系统制定文件有木有变化实现的，默认每秒问1000次
      }
    }
    ``` 
  - 【原理】轮训判断文件的最后编辑时间是否变化，某个文件发生了变化，并不会立即告诉监听者，而是先缓存起来，等 aggregateTimeout
# 16｜webpack中的热更新及原理分析
  - 【定义】热更新 webpack-dev-server 简称WDS 
    - 1、不刷新浏览器 
    - 2、不输出文件，而是放在内存中 (构建速度更快)
    - 3、使用HotModuleReplacementPlugin插件
  - 【使用】
    - 1、配置：在 package.json中 设置 “dev”: "webpack-dev-server --open" 指令 --open 构建完成后自动开启浏览器
    - 2、安装：
  
  ```javascript package.json中
  {
    "script": {
      "dev": "webpack-dev-server --open" // webpack 4
      "new-dev": "webpack server" // webpack 5 还需要安装webpack-dev-server插件
    }
  }
  
  ``` 
  - 【原理】初试过程：1->2->A->B 更新过程：1->2->3->4->5
    - Webpack Compile：将 JS 编译成 Bundle
    - HMR Server ：将热更新的文件输出给HMR Runtime
    - Bundle Server：提供文件在浏览器的访问
    - HMR Runtime：会被注入到浏览器。更新文件的变化（websoket）
    - bundle.js：构建输出的文件
  - ![示意图](./src/images/热更新原理示意图.png)

# 17｜文件指纹策略：chunkhash、contenthash和hash
  - 【定义】打包后输出的文件名的后缀
  - 【作用】好处 
    - 1、做一些版本管理 比如：一些文件上个版本的一些文件没有更新 我们只需要更新变化的文件
    - 2、可以缓存在浏览器 加速页面的访问速度
  - 【注意】webpack chunkhash 是没办法在热更新中使用的
  - 【常用占位符与含义】
    - [ext]- 资源后缀名 
    - [name]- 文件名称
    - [path]- 文件的相对路径
    - [folder]- 文件所在的文件夹
    - [contenthash]- 文件的内容hash，默认是md5生成
    - [hash]- 文件内容的hash，默认是md5生成
    - [emoji]- 一个随机的指代文件内容emoj
  - 【分类】文件指纹如何生成 ？
    - Hash：和整个项目相关，只要项目文件有修改，整个项目构建的hash值就会更改
    - ChunkHash：和webpack打包的chunk有关，不同的entry会生成不同的chunkhash 值 比如：一般对于js文件 采用chunkhash 因为其他文件不需要改变
    - ContentHash：根据文件内容来定义hash，文件内容不变，则contenthash不变 比如：一般正对css文件 采用contenthash
  - 【用法】设置 output 的 filename 使用 [chunkhash]
    - 【JS文件】 在output中 设置filename 使用 [chunkhash]
    ```javascript
    output: {
        path: path.join(__dirname, 'dist'),
        filename: '[name][chunkhash:8].js' // [name] 占位符区分
    },
    ``` 
    - 【CSS文件】  设置 插件MiniCssExtractPlugin 的filename 使用 [contenthash] 注意：style-loader与 插件MiniCssExtractPlugin.loader 是互斥的 作用：提取一个独立的css文件出来
    ```javascript
     plugins: [ // Plugins 配置 HotModuleReplacementPlugin 是 webpack 内置插件
        new MiniCssExtractPlugin({
            filename: `[name][contenthash:8].css`
        })        
    ],
    ```
    - 【图片文件】设置 file-loader 的 name 使用 [hash]
    ```javascript
      {
          test: /.(png|jpg|gif|jpeg|svg)$/, //file-loader 可以用来 处理图片 也可以用来处理字体
          use: [{
              loader: 'url-loader',
              options: {
                  name: 'img/[name][hash:8].[ext]',
                  limit: 10240 // url-loader 内部也是使用了file-loader 单位是字节 10240 byte => 10 kb 如果图片小于10k的话 webpack会自动转为base64
              }
          }]
      }
    ``` 
# 18｜代码压缩 - HTML、CSS和JS代码
  - JS文件的压缩 -- 内置了 uglifyjs-webpack-plugin（默认打包的js文件就已经压缩了）也可以单独引入 可以设置更多参数
  - CSS文件的压缩 -- 使用 optimize-css-assets-webpack-plugin 同时使用 cssnano(css预处理器)
  ```javascript
  {
    plugins: [
      new OptimizeCSSAssetsPlugin({
        assetsNameRegExp: /.css$/g,
        cssProcessor: require('cssnano')
      })
    ]
  }
  ``` 
  - HTML文件的压缩 -- 修改 html-webpack-plugin，设置压缩参数
  ```javascript
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, 'src/search.html'),
      filename: 'search.html',
      chunks: ['search'],
      inject: true,
      minify: {
        html5: true,
        collapseWhitespace:true,
        preserveLineBreaks:false,
        minifyCSS:true,
        minifyJS:true,
        removeComments:false
      }
    })
  ]
  ``` 
