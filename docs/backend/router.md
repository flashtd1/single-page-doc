## 路由管理
### 配置文件概览
所有的页面都定义在``src/router/index.js``中，文件结构说明如下
``route`` 是所有路由的配置，目前路由的配置总共有3层
* 第一层为独立的路由，用于跳转到一些结构不一样的页面，例如登录页和首页
* 第二层为侧边栏关联的路由，这一层的配置会显示在登录之后的侧边栏，例如`产品中心`、`建站中心`等，这一层一般没有实体页面，组件使用空的路由容器组件(`EmptyRouterView`)
* 第三层为详情页，一般用于存放`列表`、`新增`、`编辑`相关的页面

更多关于路由的知识，可以移步到vue-router的文档查看

**注意**
因为权限只发生在登录后的页面，因此权限的路由守卫放在了`route`的第二层`beforeEnter`中

```js
const routes = [
  {
    path: '/login',
    name: 'login',
    meta: {
      title: '登录'
    },
    component: () => import('@/views/login.vue')
  },
  {
    path: '/',
    name: 'home',
    meta: {
      title: '首页'
    },
    component: () => import('@/views/index.vue'),
    /**
     * 路由守卫，用于从cookie中取出token，还原登录身份，如果没有token则返回登录页，如果有，就设置自定义头部将token带入请求
     * @param {Route} from 从哪来
     * @param {Route} to 到哪去
     * @param {Function} next 执行下一步的函数
     */
    beforeEnter: async (to, from, next) => {
      let token = Cookies.get('token')
      if (!token) {
        next({name: 'login'})
      } else {
        if(!store.state.token) {
          store.commit('setToken', token)
          api._RefreshInstance()
        }
      }
      let userInfo = await api.GetInfo()
      if(!userInfo) {
        next({name: 'login'})
      } else {
        store.commit('setUserInfo', userInfo)
      }
      next()
    },
    children: [
      {
        path: '',
        name: 'index',
        meta: {
          title: '首页'
        },
        component: () => import('@/components/layout/EmptyRouterView.vue'), // 空路由组件容器，用于显示下级路由
        children: [
          {
            path: '/',
            name: 'index',
            meta: {
              title: '首页'
            },
            component: () => import('@/views/page/index.vue')
          },
          {
            path: 'my',
            name: 'my',
            meta: {
              title: '修改密码'
            },
            component: () => import('@/views/my/index.vue')
          }
        ],
      },
      {
        path: 'product',
        name: 'product',
        meta: {
          title: '产品中心',
          power: 'product'
        },
        component: () => import('@/components/layout/EmptyRouterView.vue'),
        children: [
          {
            path: '/',
            name: 'product',
            meta: {
              title: '产品列表',
              topConfig: require('../views/product/topOperator').default, // 加载页面的顶部操作配置
              viewConfig: require('../views/product/list').default,       // 加载页面配置
            },
            component: () => import('@/views/product/List.vue')           // 使用以上配置填入List.vue生成页面
          },
          {
            path: 'add',
            name: 'addProduct',
            meta: {
              title: '添加产品',
              viewType: 'add',                                            // 页面类型，add添加，edit编辑，preview查看，使用通用表单页组件可以直接区分新增、编辑和查看页
              formConfig: require('../views/product/form').default,
            },
            component: () => import('@/views/product/Form.vue')
          },
          {
            path: 'edit',
            name: 'editProduct',
            meta: {
              title: '编辑产品',
              viewType: 'edit',
              formConfig: require('../views/product/form').default
            },
            component: () => import('@/views/product/Form.vue')
          }
        ]
      }
    ]
  }
]

Vue.use(VueRouter)  // 在Vue上注册VueRouter插件

// 通过routes配置生成路由实例
const router = new VueRouter({
  mode: 'history',  // 历史记录模式下，路由直接以/分割，如果mode是hash，则路由为#/../../..
  routes
})

// 默认导出路由实例
export default router

// 单独导出路由配置，侧边栏可用
export { routes }
```