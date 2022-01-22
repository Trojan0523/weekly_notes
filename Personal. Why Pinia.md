# Personal:  Why Pinia

> 本文为个人译文，翻译原文为Pinia官网的介绍，同时文章末尾会参杂着一些个人见解

- 想看英文原文的同学可以点这里: [Pinia Introduction](https://pinia.vuejs.org/introduction.html)

## 介绍

- Pinia的故事始于2019年的11月，那时核心成员们正在讨论怎么给基于Composition API的Vue重新设计一个全局状态管理的工具，最初的想法挺类似，不过Pinia可以在Vue2、Vue3上跑，不允许用户使用组合式API。API风格除了安装和SSR的使用方式外，其他一模一样，文档是针对Vue3设计的，对于Vue2也提供了足够的支持，以便Vue2、Vue3的用户能畅读无阻。



### 为什么我应该使用Pinia

- Pinia是Vue的状态管理库，其允许你跨越组件或页面共享一个状态。对于比较熟悉组合式API的选手而言，你可能会想：我完全可以使用一个简单全局的状态 `export const state = rective({})` . 确实对于简单的单页应用完全没问题，不过对于服务端侧渲染的情景，可能你的应用会暴露出一定的安全问题(漏洞)。即便如此，很小的单页应用，也能利用到不少Pinia的优势：
  - 开发者工具支持(DevToos)
    1. 同步、异步行为(actions,mutaions)的时间流水线
    2. 状态在组件使用的时候展示出来
    3. 时光机, 更简单的bubugger行为
  - 热模块替换(HMR)
    1. 无需刷新页面即可修改状态
    2. 开发时留存状态持久化
  - 插件：插件使用时集成Pinia的特性
  - 更准确的TS类型提示，同时支持JS用户更智能贴心的IDE API提示
  - 服务端侧渲染支持



### 简单的实例

- 使用Pinia创建的store实例展示(起步页还是得仔细看看的，确保装了Pinia和全局创建一个Pinia实例，丢个[链接](https://pinia.vuejs.org/getting-started.html))

```js
// stores/counter.js
// 创建counter实例，
export const useCounterStore = defineStore('counter', {
  state: () => {
    // 实例中的属性，类似data
    return { count: 0 }
  },
  // 同样可以简单定义:
  // state: () => ({count: 0})
  // 执行的操作
  actions: {
    increment() {
      this.count++
    }
  }
})
```

- 在组件中使用示例

```js
import { useCounterStore } from '@/stores/counter'

export default {
  setup() {
    const counter = useCounterStore()
    
    counter.count++
    // IDE智能提示
    count.$patch({ count: counter.counter + 1 })
    // 以下action写法等同于 上方$patch
    counter.increment()
  }
}
```

- 甚至可以使用一个hooks(类似setup() 组件)去定义一个实例来支持更优的写法

```typescript
export const useCounterStore = defineStore('counter', () => {
  const count = ref<number>(0)
  function increment() {
    count.value++
  }
  return {
    count, 
    increment
  }
})
```

- 不用setup() 或者组合式API也不需要别担心，Pinia也提供了类似Vuex的一系列map辅助函数，你也可以使用往常的`mapStores()` `mapState()` `mapActions()` 定义状态实例：

```js
const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  getters: {
    double: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++
    }
  }
})
const useUserStore = defineStore('user', {
  // ...
})

export default {
  computed: {
    // 其他计算属性
    // 可访问counterStore和userStore
    ...mapStores(useCounterStore,useUserStore),
    // 可访问状态实例中的属性
    ...mapState(useCounterStore, ['count', 'double']),
  },
  methods: {
    // 可访问状态实例中的方法
    ...mapActions(useCounterStore, ['increment']),
  }
}
```

- 核心理论讲解里面有对于每个map辅助函数的详细介绍，这里就不展开了



### 为什么叫Pinia

- Pinia(发音读法我不会嘿嘿，不过类似菠萝), 包名就叫菠萝吧~, 菠萝在植物科里面也能归属到花科中。菠萝是由一个个花朵组成的，表面花朵成果球状，每个花序有百余朵小花，果实是复果，所以其实一颗菠萝是有百余朵小花中的果实组成的，同时在南美的也是很好吃的热带地区水果。(参考了[维基百科](https://zh.wikipedia.org/wiki/%E8%8F%A0%E8%98%BF)~)跟我们做的状态管理十分类似，每一个都是独立生长的个体， 但是他们最终会联结在一起(名字看来也有哲学)。



###  更贴近实际开发的实例

- 日常开发都是比较复杂的场景才会用上状态管理了~ 这里展示了一个比较复杂的API实例给即将使用pinia的同学(TS/JS都可)，资深用户可能看一下起步就足够了，不过还是推荐将剩下的文档看完或者跳过不看这个实例，回头再来阅读所有的核心概念。

```typescript
import { defineStore } from 'pinia'

export const todos = defineStore('todos', {
	state: () => ({
		/** @type {{ text: string, id: number, isFinished: boolean }[]} */
		todos: [],
    /** @type {'all' | 'finished' | 'unfinished'} */
    filter: 'all',
    // 类型自动推断成number
    nextId: 0,
	}),
  getters: {
    finishedTodos(state) {
      // 智能IDE一路Tab
      return state.todos.filters((todo) => todo.isFinished)
    },
    unfinishedTodos(state) {
      return state.todos.filters((todo) => !todo.isFinished)
    },
    filterTodos(state) {
      if (this.filter === 'finished') {
        // IDE自动推断调用其他getters
        return this.finishedTodos
      } else if (this.filter === 'unfinished') {
        return this.unfinishedTodos
      }
      return this.todos
    },
  },
  actions: {
    addTodo(text) {
      this.todos.push({text, id: this.nextId++, isFinished: false}),
    }
  }
})
```



### 对比Vuex

- Pinia尽可能贴近Vuex的哲学设计思想，一开始设计出来就是为了测试下一代Vuex提案的，其非常成功，现在作者们已经开启了非常接近`Pinia` `API`设计Vuex5的RFC提案。Vue核心贡献团队成员 ,致力于路由和Vuex的API设计的Pinia作者Eduardo 记道: "对于整个项目，我个人意愿是保持Vue平易近人且简单的设计哲学的同时，重新设计使用全局状态管理的体验，我将Pinia的API设计尽量接近vuex，这样用户在未来就能更简单的去做技术迁移(或者和其他基于Vuex的项目做交融)"



### RFCs (提案)

- Vuex尽可能的从社区中获取反馈，Pinia相反，作者是通过自己的实际开发应用去实践自己的想法，阅读别人写的代码，为使用Pinia的用户买单，在Discord(一个类似Reddit的社区)中解答问题，这种做法允许我根据不同的用例和应用的大小来制定不同的解决方案，作者在频繁发版的同时既保证了开源库的发展, 并且保证与核心API对齐。



### 对比Vuex3/Vuex4

> Vuex3 适配的是Vue2, Vuex4 适配的是Vue3

- 从字面意义上，Pinia的API风格和小于等于4的Vuex有很大的不同：
  - mutations不复存在，通常我们会觉得写mutations的实现会非常冗余，最初他的出现带来了devtools的集成，目前来看这一层面不再是问题。
  - 不需要为了支持TS而写一大堆冗长复杂的类型，Pinia的世界里，所有API、参数、属性等等都是有类型的，API设计的定位就是要平衡TS的类型推断。
  - 不需要魔法字符串(魔法值)进行注入，引入大量的函数，能充分享受IDE带来的智能提示~
  - 不需要动态添加Stores实例了，实例会默认动态引入，用户无需关心。你仍然可以像往常一样随心所欲的使用store实例注册，流程上是全自动化的了，不需要关心这个。
  - 嵌套模块不复存在，你还是可以通过在另一个模块使用一个store实例来达到隐式嵌套store实例的目的，不过Pinia从设计层面上已经提供了一种扁平化结构的模块，允许在store实例间交叉组合。甚至stores实例可以存在循环依赖引用的关系。
  - 取消具名模块，由于stores的结构是扁平化的，"具名"store实例已经在他们定义的时候被自动继承了，你也可以认为所有的store实例是具名的。



### Summary

- 如何从已有的版本4及以下的Vuex迁移到Pinia，更详细的介绍可以看此链接，这里就不详细翻译了~，有兴趣的同学接着看[此链接](https://pinia.vuejs.org/cookbook/migration-vuex.html)

- 个人观点: 以往我们在写Vuex4的时候，总是要写一大堆类型，确实不方便，贴个图出来自行体会以下(狗头.jpg)
- ![vuex-actions](https://github.com/Trojan0523/weekly_notes/blob/main/image/vuex-actions.png?raw=true)

![vuex-mutations](https://github.com/Trojan0523/weekly_notes/blob/main/image/vuex-mutations.png?raw=true)

- Vuex也是真够可以的，TS支持度不够，为了类型推断强行提升TS的体操水平,  Pinia对TS的支持比较好，改写的实例往后稍稍，之后会迭代出来，

