# **Personal.谈谈 script setup 定稿**

> 7 月 29 日，vue3 setup 顶层提升已经定稿，RFC 处于最后一个升级的阶段。总结出来有以下几个变动，script setup 的出现总体来说让我们少写了很多代码，vue 3.2 会将这些新特性移除实验性状态

## 编译器的宏定义不再需要引入

- 以往 script setup 需要手动引入编译期的 emit 和 props 才能使用，现在默认情况下在`<script setup>` 隐式包含这几个方法的初始化，包括：`defineExpose withDefault defineProps defineEmits`

### defineExpose

- 此方法适用于显式定义组件需要保留出去的公共方法，(通过模板引用访问子组件实例时)， 同时也是一个编译宏，编译成等价暴露出去的运行时内容。当需要默认导出子组件和将子组件需要导出的变量导出的时候可以使用.

  ```vue
  <template setup>
   cosnt a = 1
  defineExpose({a})
  </template>
  ```

#### defineEmit -> defineEmits

- 用法上面没啥变化，同样是直接使用定义 emits 自定义事件即可

##### useContext -> useSlots() & useAttrs()

- 没什么好说的，将 slot 和 attrs 从 context 里面解构出来使用，因为使用 script setup 的时候上下文表示不清晰，所以更多的想显式暴露出`slots` 和`attrs` 这样两个方法在实际运行时也能在组合 api 函数 usexxx()里面能使用到 : (类似 setup(props, { slot, attr }) )

##### inherit-attrs 属性从 template 移除

- 为了避免给自己挖坑，就不选择将继承的属性暴露出来了，当然有必要的话，可以在分离的 script 块中自己定义额外的属性

##### variable -> 指令映射

- 模板指令映射会被回归作为局部变量(带上 v 前缀) e.g.指令变量 命名为 vMyDir 需要在模板中作为`v-my-dir`使用
- 对比组件，全局注册的指令和局部注册的变量有冲突，有 issue 复现，所以其实相对于偶发性的全局指令来说显得没那么重要

##### 类型声明改进

- `defineProps` `defineEmit` 现在可以在基础类型声明下分别定义类型和接口字面量了，喜大普奔：）

  ```typescript
  interface Props {
    msg?: string
  }
  const props = defineProps<Props>()
  ```

- defineEmits 可以定义向上抛出事件的类型字面量或带有回调签名的接口~~

  ```typescript
  const emit = defineEmit<{
    (e: 'change', id: number): void,
    (e: 'update', value: string): void,
  }>()
  ```

- 新的`withDefaults` 编译宏可以给定义的 props 赋初始值了，不过需要搭配 defineProps 声明使用

  ```typescript
  interface Props {
    msgs?: string
  }
  const prop = withDefaults(defineProps<Props>(), {
    msg: 'hello'
  })
  ```

- 在导入类型独享声明里面，vue 团队选择继续保留其能力，虽然技术上是可行的，但是并不像影响到 api 的设计风格

##### async/await 自动上下文恢复

- 只要使用了 `script setup` ，其里面所有函数如果识别到 await ，该 sfc 就会顶层注入 async，将其作为顶层 await 使用，返回的也是一个 async 包裹的异步函数式子。

## summary

- 相比于之前的 setup defineComponent，script setup 确实是方便了很多，不需要多写很多导出的代码，代码结构可以清晰一点，不过也得看个人选择，不一定升，不过 vue 现在手脚也放开了不少，用户玩法多了很多，本质上框架的功能不变，TS 的支持更高，这次定稿还是十分让人兴奋的~
- 个人使用的时候已经体验到好处了，不过推荐他人强烈升级的意愿不是很大，有兴趣可以尝鲜。
