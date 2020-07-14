# 单页建站管理后台Vue+ElementUI开发文档

## 开发环境：
### 操作系统
* windows、mac、linux均可，建议使用centos7，因为要使用docker
### nodejs环境
* nodejs版本在10.10.16以上即可

## 项目安装
将项目代码克隆到本地之后，在项目根目录执行一下命令（确保已经安装nodejs）
```
npm install
```
以上命令会根据目录下的package.json中的描述，安装依赖

### 编译+热启动开发服务器
```
npm run serve
```
以上命令执行后会在本地开启一个热更新的服务器，编辑代码后保存能自动更新页面内容

### 编译+压缩+生成发布文件
```
npm run build
```
以上命令执行完成后，会生成dist目录，可用于直接发布，站点可以部署在各类云厂商的对象服务上，省去服务器资源
在目录中有一个deploy.js文件，可以通过读取根目录的`local_env.json`文件中的oss配置，配合以下命令
```
node deploy.js 
```
直接将网站发布到阿里云的oss中

### local_env.json文件配置说明
**这个文件不要提交到git上，内有敏感信息**
```js
{
    // 上线后的接口地址，需要做好白名单设置跨域访问
    "api":"http://localhost:7001/api/v1/", 
    // 自定义请求头部信息
    "headers": { 
        "X-SG-ID": "111", // 根据数据库配置的应用id填写
        "X-SG-KEY": "222" // 根据数据库皮遏制的应用id填写
    },
    // 阿里云oss的配置信息
    "oss":{
        "region": "",       // 区域名称 
        "accessKeyId": "",  // 在阿里云中开通授权id
        "accessKeySecret": "", // 在阿里云中开通授权秘钥
        "bucket": "",       // oss的bucket名称
        "internal": false   // 是否内网链接，部署的时候，填写true，如果不是部署在阿里云上的话，填写false
    },
    "icp": "ICP信息" // 网站的ICP备案信息
}
```

## 目录说明
```
|-- App.vue             // 根组件
|-- main.js             // 程序入口
|-- api                 // 存放所有api的目录
|-- assets              // 资源目录，在打包的时候，会根据配置做一些调整，如编译或者压缩
|-- components          // 组件存放目录
|   |-- form            // 所有表单组件
|   |-- layout          // 布局组件
|   |-- other           // 其他业务组件
|   |-- page            // 页面级组件
|   |-- search          // 搜索条件组件
|   |-- table           // 表格组件
|-- libs                // 代码库
|   |-- moment.js       // 时间处理库
|-- plugins             // 插件目录
|   |-- api.js          // 接口插件，负责管理所有的接口包装，暴露接口给vue实例调用，通用错误处理等
|   |-- components.js   // 全局组件注册
|   |-- index.js        // 插件管理入口
|-- router              // 路由目录
|   |-- index.js        // 路由主文件
|-- store               // vuex数据仓库目录
|   |-- index.js        // 仓库主文件
|-- views               // 视图组件
    |-- index.vue       // 首页
    |-- login.vue       // 登录页
    |-- department      // 部门相关目录
    |-- my              // 个人信息相关目录
    |-- page            // 页面目录
    |-- product         // 产品相关目录
    |-- role            // 角色相关目录
    |-- users           // 用户相关目录
```
