
- [01｜商城技术栈选型和整体架构](#01商城技术栈选型和整体架构)
  - [商城技术选型](#商城技术选型)
  - [商城架构设计](#商城架构设计)
- [02｜商城界面UI设计与模块划分](#02商城界面ui设计与模块划分)
  - [商城界面UI](#商城界面ui)
  - [前台模块拆分](#前台模块拆分)
  - [后台模块拆分](#后台模块拆分)
- [03｜React全家桶环境搭建](#03react全家桶环境搭建)
  - [React 全家桶环境搭建](#react-全家桶环境搭建)
  - [创建 actions、reducers、store](#创建-actionsreducersstore)
- [04｜数据库实体和表结构设计](#04数据库实体和表结构设计)
- [05｜登录注册模块开发](#05登录注册模块开发)
- [06｜商品模块开发](#06商品模块开发)
- [07｜订单模块开发](#07订单模块开发)
- [08｜谈谈Web商城的性能优化策略型（重点）](#08谈谈web商城的性能优化策略型重点)
  - [Web商城的性能优化策略](#web商城的性能优化策略)
- [09｜功能开发总结](#09功能开发总结)
- [10｜玩转webpack & 结业测试](#10玩转webpack--结业测试)

# 01｜商城技术栈选型和整体架构
## 商城技术选型
- 前端
  - React / Vue 全家桶
  - webpack
- 后端
  - koa
  - MySql
## 商城架构设计
- 平台层
  - 用户端（PC｜H5｜小程序｜App内嵌H5）
    - 首页｜列表页｜购物车页｜订单页｜详情页｜登录/注册｜...
  - 管理后台
    - 商品管理｜订单管理｜用户信息管理｜...
- 服务层
  - 商品服务｜订单服务｜购物车服务｜搜索服务｜支付服务｜评论服务｜...
- 基础设施
  - mysql｜docker｜k8s（集群化部署）｜...
![商城架构设计](public/商城架构设计.png)

# 02｜商城界面UI设计与模块划分
## 商城界面UI 
- [极客时间商城](https://shop18793264.m.youzan.com/v2/showcase/homepage?alias=11bl3ra5z&reft=1611820539770_1611820587883&spm=f.69794282_f.83045217)
## 前台模块拆分
- web商城前台功能模块设计
  - 首页
    - bannner轮播
    - 公告
    - 商品列表
    - 底部bar
    - 搜索框
    - 新品列表
  - 列表页
    - 搜索框
    - 左侧bar
    - 右侧列表
  - 详情页
    - 图片轮播
    - 商品基本信息
    - 品论
    - 购买操作模块
  - 购物车
    - 已购商品列表
      - 删除
      - 修改
    - 金额计算
  - 订单页
    - 送货地址
    - 商品列表
  - 搜索页
    - 搜索框
    - 搜索结果展示
  - 登录/注册
    - 表单输入
    - 许可协议
![前台模块拆分](public/前台模块拆分.png)
## 后台模块拆分
- web商城后台功能模块设计（通过http 或 websocket 协议提供服务）
  - 首页推荐服务
  - 搜索服务
  - 类目服务
  - 商品服务
    - 商品查询服务
    - 商品录入
    - 商品信息修改
  - 评论服务
    - 添加评论
    - 删除评论
  - 购物车服务
  - 订单服务
  - 用户服务
    - 登录
    - 注册
    - 个人信息查询
    - 个人信息修改
- ![](public/后台模块划分.png)

# 03｜React全家桶环境搭建
## React 全家桶环境搭建
- 初始化项目
  - npm init -y
- 创建项目目录
  - src
    - cart
    - detail
    - index
      - actions 存放 redux里的action
      - assets  存放 多个组件的共同资源
      - components 业务组件
      - reducers 存放 redux的reducer
    - app.js 容器组件
    - app.less 容器组件样式
    - index.html 入口模板
    - index.js 入口
    - store.js redux 中的 store
  - .babelrc
  - webpack.config.js
## 创建 actions、reducers、store
- actions 和  reducers
  - src/actions/ 防止所有的actions、src/reducers 放置所有的 reducers
- rootReducer
  - src/reducers/rootReducer.js 将所有 reducers 进行 Combine
- 使用Provider 传递 store
  - store 通过 provider 传递给容器组件
## 
# 04｜数据库实体和表结构设计

# 05｜登录注册模块开发

# 06｜商品模块开发

# 07｜订单模块开发

# 08｜谈谈Web商城的性能优化策略型（重点）
## Web商城的性能优化策略
- 渲染优化
  - 首页、列表页、详情页采用SSR 或者 Native（weex、rn、flutter） 渲染
  - 个人中心页预渲染
- 弱网优化
  - 使用离线包、PWA等离线缓存技术（放到客户端缓存里面）
- Webview 优化
  - 打开Webview的同时并行地加载页面数据（二级、三级入口做优化）
  

# 09｜功能开发总结
- 功能开发要点
  - 浏览器端：
    - 组件化，组件颗粒尽可能小
    - 直接复用 builder-webpack-geektime 的构建配置，无需关注构建脚本
  - 服务端：
    - MVC开发方式，数据库基于 Sequelize
    - Restful API风格
    - 采用 JWT 进行鉴权

# 10｜玩转webpack & 结业测试
- 关注社区、构建工具发展 因为更新很快