# 开发指南

## 数据库设计规范
### 数据库中表的定义中，必须要含有的字段
* id(主键)
* created_at（数据创建时间字段，默认值为NULL）
* updated_at（数据更新时间字段，默认值为NULL）
* deleted_at（数据删除时间字段，默认值为NULL）

### 分隔符使用
由于使用sequelize来做orm，数据库中的字段，分割用`_`分割，不要使用驼峰方式分割

### 外键的命名
为了方便查找关联字段与表的关系，外键的命名一般以`语义_poi_关联表名`来命名，例如产品表中需要关联用户id，则字段命名如下：
`id_poi_users`


## 接口开发

### 接口路由定义
在项目的`app/router.ts`中定义接口的路由，详细配置可以参考egg.js的官方文档，示例如下
```ts
import { Application } from 'egg';

export default (app: Application) => {
  const { controller, router } = app;
  // 特殊的RESTful请求，
  router.get('/', controller.home.index)

  // 通用RESTful请求，对应万能接口
  router.resources('classes', `/api/v1/classes/:classname`, controller.classes)
};

```
### 万能接口
万能接口用于一般的数据获取场景中，例如获取列表数据和单条数据的详情
得益于sequelize的查询配置，前端可以发送配置，后端解析配置生成sequelize的查询格式，完成一般请求。具体sequelize查询可以查看sequelize官方文档。

新建万能接口也比较简单，只需新建模型与数据库形成映射，并且定义模型之间的关系，就完成了
在`app/model`目录下建立与数据库对应的ORM模型并且定义好与关联表的关系，以产品表为例
```ts
/**
 * 产品模型
 */
export default app => {
    const { STRING, INTEGER, DATE,DECIMAL } = app.Sequelize
    const Product = app.model.define('sg_product', {
        id: {type: INTEGER, primaryKey: true, autoIncrement: true},
        chinese_name: STRING(100),
        foreign_name: STRING(100),
        purchase_name: STRING(100),
        purchase_url: STRING(100),
        purchase_price: STRING(100),
        heavy: DECIMAL(10,2),
        volume: STRING(100),
        type: STRING(100),
        id_poi_users: INTEGER,
        image: STRING(100),
        describe: STRING(100),
        created_at: DATE,
        updated_at: DATE,
    }, {
        paranoid: true,
        freezeTableName: true  //禁止表名复数
    })

    Product.associate = function() {
        // 产品和用户是多对一关系
        app.model.Product.belongsTo(app.model.User, {foreignKey: 'id_poi_users', targetKey: 'id'})
        // 产品和sku是一对多关系
        app.model.Product.hasMany(app.model.Sku, {foreignKey: 'id_poi_product', targetKey: 'id'});

        app.model.Product.hasMany(app.model.PackageProduct, {foreignKey: 'id_poi_product', targetKey: 'id'});
    }
    
    return Product
} 
```
然后就可以在接口中通过`/api/v1/classes/product`来查询产品了

### 自定义接口
万能接口虽然好用，但是处理需要表单验证的场景以及获取的数据中包含敏感信息，或者是接口中要完成复杂逻辑时，避免使用万能接口，应该自己封装接口

步骤如下：
1. 在`app/service`目录下新建业务逻辑处理的代码
2. 在`app/controller`目录下新建业务逻辑的控制器入口
3. 在`app/router.ts`中定义路由与控制器的绑定

以下以获取首页统计为例
在`app/service`目录下新建service代码`Index.ts`
```typescript
import { Service } from 'egg';
import * as moment from 'moment'

export default class Index extends Service {

    public async index() {
        const {ctx} = this
        const data = ctx.header
        // 这里不通过用户传递id了，直接从token信息，从token表中获取用户信息，所以需要验证token是否存在
        if(data['x-sg-token']){
            const token = data['x-sg-token']
            let id = await ctx.service.user.getUserId(token)
            if(!id){
               return false
            }
            
            // 获取用户信息
            let userinfo =  await ctx.service.user.getUser(id)
            // 获取角色id
            let roleId = userinfo.dataValues.sg_role.dataValues.id
            
            let time = moment(Date.now()).format('YYYY-MM-DD')
            // 获取今天的开始时间（数据库存的时间和本地时间差8小时时差）
            let start_time = moment(time + " 00:00:00").subtract(8, 'hours').format('YYYY-MM-DD HH:mm:ss')
            // 获取今天的结束时间（用开始时间直接+24小时得到结束时间
            let over_time = moment(start_time).add(24, 'hours').format('YYYY-MM-DD HH:mm:ss')

            let ordersql = ''
            let productsql = ''
            let sg_commoditysql = '';
            
            let order_count = 0
            let product_count = 0
            let commodity_count = 0
            
            // 超级管理员
            if(roleId === 1){
                ordersql = `SELECT COUNT(*) as count FROM sg_orders WHERE created_at >= '${start_time}' and created_at <= '${over_time}' and status in(1,2) and deleted_at is NULL`
                productsql = `SELECT COUNT(*) as count FROM sg_product WHERE created_at >= '${start_time}' and created_at <= '${over_time}' and deleted_at is NULL`
                sg_commoditysql = `SELECT COUNT(*) as count FROM sg_commodity WHERE created_at >= '${start_time}' and created_at <= '${over_time}' and deleted_at is NULL`
                let orders = await ctx.model.Order.findAll(
                    {
                        where:{
                            status: {
                                $in: [1,2]
                            }
                        }
                    }
                )
                let product = await ctx.model.Product.findAll()
                let commodity = await ctx.model.Commodity.findAll()
                order_count = orders.length
                product_count = product.length
                commodity_count = commodity.length
            } else { 

                // 部门组长
                console.log('------------用户信息', userinfo)
                let ids: Number[] = []
                if (userinfo.dataValues.sg_department.dataValues.id_poi_users == userinfo.dataValues.id) {
                    // 查询用户列表
                    let users = await ctx.model.User.findAll({
                        where: {
                            id_poi_department: userinfo.dataValues.id_poi_department
                        }
                    })
                    ids = users.map((user) => {
                        return user.dataValues.id
                    })

                } else {
                    // 普通用户
                    ids.push(id) 
                }
                
                ordersql = `SELECT COUNT(*) as count FROM sg_orders WHERE id_poi_users IN(${ids.join(',')}) and created_at >= '${start_time}' and created_at <= '${over_time}' and status in(1,2) and deleted_at is NULL`
                productsql = `SELECT COUNT(*) as count FROM sg_product WHERE id_poi_users IN(${ids.join(',')}) and created_at >= '${start_time}' and created_at <= '${over_time}' and deleted_at is NULL`
                sg_commoditysql = `SELECT COUNT(*) as count FROM sg_commodity WHERE id_poi_users IN(${ids.join(',')}) and created_at >= '${start_time}' and created_at <= '${over_time}' and deleted_at is NULL`
                let orders = await ctx.model.Order.findAll(
                    {
                        where:{
                            id_poi_users: {
                                $in: ids
                            }, 
                            status: {
                                $in: [1,2]
                            }
                        }
                    }
                )
                order_count = orders.length
                let product = await ctx.model.Product.findAll(
                    {
                        where: {
                            id_poi_users: {
                                $in: ids
                            }
                        }
                    }
                )
                product_count = product.length
                let commodity = await ctx.model.Commodity.findAll(
                    {
                        where:{
                            id_poi_users: {
                                $in: ids
                            }
                        }
                    }
                )
                commodity_count = commodity.length
            }
            interface IOrder {
                count:Number
            }
            let orderResult: IOrder[] = <IOrder[]>(await ctx.model.query(ordersql,{type: 'SELECT'}));
            interface IProduct {
                count:Number
            }
            let productReslt: IProduct[] = <IProduct[]>(await ctx.model.query(productsql,{type: 'SELECT'}))
            let commodityReslt: IProduct[] = <IProduct[]>(await ctx.model.query(sg_commoditysql,{type: 'SELECT'}))
            let result = {
                order_day: orderResult[0].count,
                product_day: productReslt[0].count,
                commodity_day:commodityReslt[0].count,
                order_count:order_count,
                product_count:product_count,
                commodity_count:commodity_count
            }
            return result
        }else{
            return false
        }
    }
}
```

在`app/controller`目录下新建index.ts
```typescript
import { Controller } from 'egg';

export default class IndexController extends Controller {
  public async index() {
    const { ctx } = this;
    let res = await ctx.service.index.index()
    if(!res){
        ctx.status = 400
        return ctx.body = {msg:"token 不存在或已过期 ? "}
    }
    ctx.status = 200
    ctx.body = res
  }
}
```

在`app/router.ts`中添加路由与接口绑定关系
```typescript
import { Application } from 'egg';

export default (app: Application) => {
  const { controller, router } = app;
  
  // 其他接口略
  router.get('/api/v1/index', controller.index.index)
  
  // 通用RESTful请求，直接请求数据库表，根据配置及权限控制是否通过
  router.resources('classes', `/api/v1/classes/:classname`, controller.classes)
};

```
