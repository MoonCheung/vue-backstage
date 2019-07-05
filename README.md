# Vue-Admin

[![GitHub stars](https://img.shields.io/github/stars/MoonCheung/vue-backstage.svg?style=flat-square)](https://github.com/MoonCheung/vue-backstage/stargazers)
[![GitHub issues](https://img.shields.io/github/issues/MoonCheung/vue-backstage.svg?style=flat-square)](https://github.com/MoonCheung/vue-backstage/issues)
[![GitHub forks](https://img.shields.io/github/forks/MoonCheung/vue-backstage.svg?style=flat-square)](https://github.com/MoonCheung/vue-backstage/network)
[![GitHub license](https://img.shields.io/github/license/MoonCheung/vue-backstage.svg?style=flat-square)](https://github.com/MoonCheung/vue-backstage/blob/master/LICENSE)

Blog backend management comes from the [vue-admin](https://github.com/PanJiaChen/vue-admin-template) project template and is built and developed by the vue + ElementUI application <br/>
The background management is based on the node server can run, need to install the node server in advance

- Maintained by [MoonCheung](salvador23@163.com)
- Blog applet: [mpvue-applet](https://github.com/MoonCheung/mpvue-applet) is provided run by mpvue + Vant + ColorUI
- admin server: [node-server](https://github.com/MoonCheung/node-server) is provided run by koa

## 程序结构

### auth 验证

- 从 api 服务器获取授权 Token,让每个请求携带 token 发送得到验证
- 用于权限控制,请求拦截的职责

### 引入 quill 富文本组件使用

- 想发表文章提前安装富文本或者 Markdown 编辑器组件,根据看用户的业务需求
- 安装`quill`增强模块,想实现拖放图像,调整图像大小需引入两个`quill`相关模块

### 七牛云上传配置

- token 从服务器 SDK 生成获取,通过 axios 封装调用接口,自行选好图像上传时候会携带 token 发送到七牛云
- 上传成功之后的后续处理,父组件通过 emit 事件发出子组件实现组件通信更新数据,比如说更换图像,调整图像大小

### 性能优化

## 相关截图

## 执行命令

### Build Setup

```bash
# clone the project
git clone https://github.com/MoonCheung/vue-backstage.git

# enter the project directory
cd vue-backstage

# install dependency
npm install

# develop
npm run dev
```

### Build

```bash
# build for test environment
npm run build:stage

# build for production environment
npm run build:prod
```

### Advanced

```bash
# preview the release environment effect
npm run preview

# preview the release environment effect + static resource analysis
npm run preview -- --report

# code format check
npm run lint

# code format check and auto fix
npm run lint -- --fix
```
