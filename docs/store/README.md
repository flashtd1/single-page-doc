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
|-- single-page-store
    |-- .gitignore
    |-- babel.config.js
    |-- deploy.js
    |-- Dockerfile
    |-- local_env.json
    |-- package-lock.json
    |-- package.json
    |-- README.md
    |-- vue.config.js
    |-- public
    |   |-- favicon.ico
    |   |-- index.html
    |   |-- image               // 图片资源
    |       |-- 7day.png
    |       |-- down14px.png
    |       |-- huodao.png
    |-- src
        |-- App.vue
        |-- main.js
        |-- assets              // js资源目录
        |   |-- area_711.js     // 711地址配置
        |   |-- area_family.js  // 全家地址配置
        |   |-- locale.js       // 语言包
        |-- components
        |   |-- AreaSelect.vue  // 地区选择器组件
        |   |-- note.vue        // 说明组件（这里没有和语言包配合使用，直接将不同语言的提示写在这里了
        |   |-- swiper.vue      // 轮播图组件
        |-- routes
        |   |-- index.js        // 路由配置
        |-- style
        |   |-- index           // 电商页的样式
        |   |   |-- index.css
        |   |   |-- reset.css
        |   |-- result          // 订单提交后的样式
        |       |-- page-success.css
        |       |-- reset.css
        |-- views
            |-- index.vue       // 电商页
            |-- OrderNote.vue   // 订单提交后的提示组件
            |-- result.vue      // 订单提交页
```