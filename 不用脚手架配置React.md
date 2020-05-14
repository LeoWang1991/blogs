不用脚手架（create-react-app, umi, antd pro）等配置React开发环境。

主要包括：

1. 安装依赖项
2. 安装启动项 webpack



### 安装相关包

```javascript
// 安装react
yarn add react and react-dom

// 安装webpack
yarn add -D webpack webpack-cli webpack-dev-server html-webpack-plugin

// 安装babel
yarn add -D @babel/core @babel/preset-env @babel/preset-react babel-loader
```



### 配置webpack

### 配置Babel

新建`.babelrc` 当然可以在其他配置文件去配置，推荐这种方式吧 

```json
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react"
  ]
}
```



### 一些点

#### 1 webpack更新配置后自动生效？

目前来说，如果更改了配置需要自己手动重新启动才能生效，不是很方便。



