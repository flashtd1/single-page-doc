# 单页建站后端接口介绍

## 快速开始

### 配置文件
由于敏感信息不能提交git，因此在项目中隐藏了两个文件
`config/config.mysql.ts`
文件格式如下：
```ts
import { EggAppConfig, PowerPartial } from 'egg'

export default () => {
  const config: PowerPartial<EggAppConfig> = {
      sequelize: {
        dialect: 'mysql',
        host: '数据库地址',
        port: 3306,
        username: '用户名',
        password: '数据库密码',
        database: '数据库名称',
      }
  }
  return config
}

```
`config/ocnfig.oss.ts`
文件内容如下：
```ts
import { EggAppConfig, PowerPartial } from 'egg'

export default () => {
  const config: PowerPartial<EggAppConfig> = {
    region: 'oss地区',
    accessKeyId: '阿里云授权id',
    accessKeySecret: '阿里云授权秘钥',
    bucket: "bucket名称",
    internal: false // 是否内网，部署在阿里云的时候，写true
  }
  return config
}

```

### 开发
在项目根目录,首先进行依赖安装,执行 `npm i`
接着执行 `npm dev` 会开启热更新服务器
```bash
npm i
npm run dev
```
服务器默认会在**7001**端口

在开发模式下不要用`tsc`命令编译ts，如果已经执行过，那就需要在执行`npm run dev`之前执行`npm run clean` 清理掉tsc编译出来的内容

### 发布
先执行`npm run tsc`进行编译
再执行`npm start`开启服务器
```bash
npm run tsc
npm start
```

### 环境要求
- Node.js 10.10.0+
- Typescript 2.8+


## 一些目录说明
```
|-- app
    |-- router.ts               // 路由配置，所有的接口都需要这个文件来暴露
    |-- controller              // 控制器
    |   |-- classes.ts          // 万能接口控制器
    |   |-- home.ts             // 默认控制器，可用来查看服务器是否正常
    |   |-- power.ts            // 权限控制器
    |   |-- product.ts          // 产品控制器
    |   |-- role.ts             // 角色控制器
    |   |-- upload.ts           // 上传文件控制器
    |   |-- user.ts             // 用户控制器
    |-- middleware              // 中间件目录
    |   |-- auth.ts             // 权限验证中间件
    |-- model                   // 模型文件，在这里定义过的模型，都可以用万能接口获取到
    |   |-- attributeAlias.ts   
    |   |-- attributeAliasGroup.ts
    |   |-- attrs.ts
    |   |-- department.ts
    |   |-- group.ts
    |   |-- language.ts
    |   |-- power.ts
    |   |-- powergroups.ts
    |   |-- powertype.ts
    |   |-- powertypegroupsrelation.ts
    |   |-- product.ts
    |   |-- role.ts
    |   |-- rolePower.ts
    |   |-- sku.ts
    |   |-- skuAttrRelation.ts
    |   |-- terminal.ts
    |   |-- terminalPower.ts
    |   |-- token.ts
    |   |-- user.ts
    |-- public                          // 静态资源目录，存放了excel模板，输出目录占位等
    |   |-- excels
    |   |   |-- sku-template.xlsx
    |   |-- images
    |   |   |-- changchuan.png
    |   |-- output
    |       |-- .keep
    |-- service                         // 服务，在控制器和模型中间，最好用服务来完成业务逻辑编写
        |-- Power.ts
        |-- Product.ts
        |-- Role.ts
        |-- Sku.ts
        |-- Terminal.ts
        |-- Test.ts
        |-- User.ts
```