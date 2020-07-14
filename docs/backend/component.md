

## 表单项组件开发
表单项组件是为了服务通用表单组件的，通用表单组件接收一个配置来生成各类复杂的表单，但是默认的表单项组件有限，无法满足复杂的业务需求

表单组件遵循一个简单的原则
在定义表单组件时，默认需要接受一个名为`value`的`props`，在“值”发生变化时，通过`input`事件将“值”通知到表单，代码如下，以用户选择组件为例
```vue
<template lang="pug">
el-select(v-model="v" @change="onChange")
  el-option(v-for="(user, index) of users" :key="user.id" :label="user.username" :value="user.id") {{user.username}}
</template>

<script>
import {mapState} from 'vuex'

export default {
  name: 'UserSelector',
  props: ['value', 'forsearch', 'queryKey', 'autoSelect'],
  data() {
    return {
      users: [],
      v: 0
    }
  },
  computed: {
    ...mapState(['userInfo', 'userDepartment']),
  },
  watch: {
    
    $route({query}) {
      if (!this.forsearch) return
      this.init()
    },
    value(newValue) {
      // console.log(newValue)
      this.v = parseInt(newValue || 0)
    },
    v(newV) {
      this.$emit('input', newV)
    }
  },
  methods: {
    init() {
      let query = this.$route.query
      if (query[this.queryKey]) {
        this.v = parseInt(query[this.queryKey])
      } else {
        this.v = ''
      }
    },
    onChange() {
      if (this.forsearch) {
        this.$emit('change', {
          key: this.queryKey,
          value: this.v
        })
      }
    }
  },
  async mounted() {
    let query = {}
    // console.log(this.userInfo)
    if (this.userInfo.id_poi_role != 1) {
      // 如果自己是部门组长
      if (this.userDepartment.id_poi_users == this.userInfo.id) {
        query = {
          where: {
            id_poi_department: this.userDepartment.id
          }
        }
      } else {
        query = {
          where: {
            id: this.userInfo.id
          }
        }
      }
      
    }
    let res = await this.$api.GetUsers(query)
    this.users = [{id: 0, username: '请选择'},...res.items]
    if (this.value) {
      this.v = this.value
    } else {
      if (this.autoSelect == undefined || this.autoSelect == true) {
        this.v = this.userInfo.id
      } else {
        this.v = null
      }
    }
    if (this.forsearch) {
      this.init()
    }
  }
}
</script>
```
用户选择组件会在挂载后，获取用户列表填充选项。如果表单中有用户id通过value传入，则用户选择组件会通过id获取对应的用户数据，并选中，如果选择了其他的用户，则组件会通过input事件将新的id通过input事件传出。

在使用时，只需将配置中的component字段填入UserSelector即可，例如
```js
{
    name: '部门组长',
    component: 'UserSelector',
    key: 'id_poi_users',
    default: null,
    attrs: {
      autoSelect: false,
      placeholder: '请选择部门组长',
    }
},
```

## 通用组件说明
### 通用表单组件
`/src/components/form/Commonform.vue`
通用表单组件。一般简单的表单都可以直接使用CommonForm来搭建，给该组件传入数据即可生成表单。具体使用案例可以参考views下目录中的form.js或者Form.vue中的用法。
配置文件结构。
```js
// 表单的数据是一个数组
[
  // 每个对象是一项
  { 
    name: '名称',             // 表单项名
    component: 'el-input',    // 使用的表单控件
    key: 'name',              // 与数据的key对应
    default: '',              // 默认值（要和component需要的数据类型一致）
    attrs: {                  // 组件可以传递的参数，
      placeholder: '请输入角色名',
    },
    rules: [                  // 表单项验证规则
      { required: true, message: '请输入角色名', trigger: 'blur' }
    ]
  }
]
```
该写法给我们一种思路，就是尽量利用vue的组件开发优势，将所有遇到的表单项，都封装成使用v-model进行双向绑定数据的组件，可大大简化编写页面的难度。
缺点，摒弃了一些vue的特性，在灵活性上不太够，做可变表单会比较麻烦。虽然可以通过组件去做可变表单，但是会造成很大的麻烦。

### 通用表单页组件
`/src/page/CommonFormPage.vue`
通用表单页组件。本组件集成了CommonForm组件，用户可通过多表单配置来拼接出稍微完整的表单页，提交时组件会将所有表单的信息汇总到一起发送给提交函数。
用法参考`页面开发`相关章节


### 通用表格组件
`/src/components/table/CommonTable.vue`
通用表格组件。该组件负责在使用表格时，通过配置文件将表头、表格数据组装在页面上。适合在列表页使用
用法参考`页面开发`相关章节


### 通用搜索组件
`/src/components/search/CommonSearchFilter.vue`
通用搜索过滤器组件。该组件负责在列表页配置搜索条件，该组件下的子组件都遵循将条件放置在url路由中。由具体的搜索方法，通过route获取query将搜索条件取出进行接口调用。
具体配置结构可以参考各个views目录中子目录中使用了TopOperator.js的文件
用法参考`页面开发`相关章节


### 通用表格页组件
`/src/page/CommonListPage.vue`
通用表格页组件。本组件集成了通用搜索过滤组件、表格组件、弹出窗口组件，用于没有特殊要求的列表页面展示，通过给定的配置，可调整搜索条件、顶部操作按钮、表格、分页等功能。
用法参考`页面开发`相关章节
