# Personal. 论`delete` 和 `Reflect.deleteProperty` 

> 昨天跟老大讨论了一下一年前遇到的一次 Code review 下的优化，当时为了团队的风格选择了使用 Reflect.deleteProperty 作为删除某个对象下的属性键值，到了一年后的现在，团队不断加入新同学，delete 语法逐渐使用的多了起来，究竟这两个 API 有什么区别和好处？本文会先贴出一个链接 [讨论帖](https://github.com/xojs/xo/issues/14)，需不需要追求语言下的极致性能，暂时认为，团队不需要，但是对于语法，我想有一个深入的认识。



### 前言

- 开源 NPM 包大佬 sindresorhus 在自己的 xo 包中开了一个 issue 对比两个 API 的倾向性，ESLint 1.2.0 新增的 ES2015 规则提出，reflect(反射)更适用于删除的语法，作者对于 `Reflect.deleteProperty`和`delete`两个关键字也不太确定孰胜孰略，同样具有疑问的是 `Function.prototype.call()`和`Function.prototype.apply()`

### 语法比较

```javascript
// 先来看一下两个API的表现形式：
Object.defineProperty(a, 'b', { value: 2 })
const eg1 = Reflect.deleteProperty(a,  b) // eg1: true a: {}
const eg2 = delete a.b // eg2: true a: {} 
/**
 * 输出结果表现一致，同时也不能对不可配置的对象属性进行删除
 **/
Object.defineProperty(c, 'd', {
  configurable: false,
  value: 2
})
const eg3 = Releft.deleteProperty(c, d) // eg3: false, c: { d: 2 }
const eg4 = delete c.d // eg4: false, c: { d: 2 }
```

- MDN：`Reflect.deleteProperty`
  - 返回值为布尔类型，表示属性是否被成功删除
- MDN: `delete`
  - 在严格模式下，如果属性为不可配置属性(即不能覆盖初始化类型即值表现形式)时，删除会抛出异常，反之则为布尔类型去否属性返回 (false)，其他一切情景则返回 true

### 论点 1: `Reflect.deleteProperty`

- 虽然十分难以选择，也没有很强力的理由坚定自己的观点，但是更喜欢 `Reflect.deleteProperty` 

- 假设我想在代码某处使用删除这一功能，会更倾向于使用 `Reflect.deleteProperty()` , 因为 `delete foo.bar` 的 可读性比 `Reflect.deleteProperty(foo, bar)`稍差一点，后者比前者更为清晰的表明其功能是删除一个属性选项，同时也有一些其他语言(比如 Java)的因素存在，所以答案显而易见。
- 选择`Reflect.deleteProperty`的原因是对于调用形式是更为清晰可见的，需要删除任何标识的时候，`Reflect.deleteProperty`需要传入两个参数，一个是对象本身，一个是对象映射的属性名(key)，而`delete`则只抛出一个`ReferenceError`的错误，部分评论只给出的评论是抛出异常是更佳的处理方式，可能 delete 有多重重载的方式，出现多重情景，同时表现出不同的错误异常提示(后面这位同学也表明了自己查阅文档后这种观点是错误的，表现形式只有 MDN 提供的几种方案)
- 使用`deleteProperty`更佳，因为抛出异常个人认为这是我的代码在某处出现了 Bug，而通常个人认为抛出错误更倾向认为是逻辑错误或者 Bug，对于这种可能出现正常的处理方式，这位同学认为`delete`是一种在设计上不太合理的方案。



### 论点 2：`delete` 

- 由于讨论帖是 15 年八月的时候展开的，虽然认为`Reflect.deleteProperty` 表达的功能性更强，但是由于`Reflect`的支持基本为 0，所以选择`delete`是无奈之举。(2022 年的现在，无论是浏览器的支持还是 polyfill，这一个观点不复存在)
- 对于两个语法的理解是混乱的。不过我更为认同 `Reflect.deleteProperty`是更为容易理解。由于当时支持很弱，通过编写 benchmark 去对比他们的性能表现，v8 可选项 `--harmony-reflect`不能使用，所以选择使用降级的 polyfill，benchmark 标识，`Reflect.deleteProperty`比`delete`会慢上不少，于此不推荐 xo 包的用户在使用的同时还需要引入 polyfill 运行，所以不得不使用`delete`。
- 毫不犹豫选择 `delete`,  删除语法和其他语言类似，并且不需要额外去查阅文档，在使用上十分便捷迅速，同时`Reflect.deleteProperty`更像是一个方法而不是操作符，这意味着 delete 性能上会更加快。除非`Reflect.deleteProperty` 能强制删除非可配置的属性，否则我会坚定的站在`delete`这一边。（看来这一点以后都不会实现了，同时两个语法也同样不可能实现）
- 倾向于选择`delete`, 因为当在做一些错误操作尝试删除某些属性时，除了不能删除以外，还会抛出异常提示，在严格模式下能清楚的表明项目中是否出现错误操作，而`Reflect.deleteProperty`没有任何提示出现，从这一点表明`delete`完胜。



### 个人观点

- 上面的论点其实已经给出了答案，其实两个 API 各有各的好处，在实现上和表现形式都是十分类似的，由于 ES 在之前的设计中，更倾向于认为删除这一行为是一个操作符。借用一个老哥给出来的偏技术下的指导:

  >从技术上而言，ES 是没有办法在不适用`delete`或者`Reflect.deleteProperty`的情况下，严格地去删除自身非继承的属性。
  >
  >所以我们的目标是达成一种简单的方法不使用操作符的同时在 V8 中能利用到隐藏类的好处(考虑到 C++的特性,在运行时对象就已经被冻结，而且通常已经在内部映射到类实例上) ,这个是 V8 的一个重要的设计性能特性。 将成员属性移除为`undefined`，在确定的情景上其实已经足够了，在此作用域下其实也能忽略到此成员，并且可以认为与`undefined`严格相等，此行为可以看作模拟和模仿一个不存在的属性。此独特指出在于选项属性仍然是属性，举个🌰，通过`Object.keys()`方法映射的类型
  >
  >其他类似的方法是通过克隆的形式移除属性，通过省略这些字段再拷贝的过程中进行删除，但多数情况下，拷贝一整个对象带来的性能开销是非常巨大的，不建议这么做 。
  >
  >显而易见的是，我们能对非常特定的优化细节做覆盖，不过只有在比较特定的情境下才合理，所以如果你要对每个方法做权衡，同时基于特定的使用情景，以我的个人经验来说，绝大部分情境下，对象中最重要的部分是他的 value 值，不是他的 key 键。所以其键可以保留，但是键不能指向任意的内存地址。

- Sindresorhus 老哥在贴子结尾处留下的结论是会继续使用`delete`关键字作为删除属性的使用方法，不过个人通过以上的讨论中觉得，二者并没有很大的区别，delete 即使在当时性能比`Reflect.deleteProperty`要好，其可读性稍差和异常错误处理不是我特别喜欢的特点，同时团队中使用的规范下，可读性要比极致的性能要重要，所以我个人还是会偏向于使用`Reflect.deleteProperty` 保持整个团队的函数式风格。
