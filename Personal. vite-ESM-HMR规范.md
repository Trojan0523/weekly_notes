# Personal. (vite/snowpack 等工具链)共同约定的 ESM-HMR 规范

## `import.meta.hot`

- HMR 开启，`import.meta.hot` 会被定义  

- HMR 不开启，`import.meta.hot` 不会被定义  

- 生产构建时，可创建一个条件分支作为跳板，比如 `if (import.meta.hot) { /***/ }` 作为 dead code (死代码，会被 tree shaking 掉)  

- 注意： 需要自身充分扩展 `import.meta.hot` 定义，让服务能够进行静态检测和开启 HMR 用法  

## `accept()` `import.meta.accept()`

- 允许 HMR 更新对应模块  

- 模块更新时，浏览器会自动重新导入该模块  

- 注意：重导入此已更新模块实例并不会自动替换当前应用的当前模块实例，如果需要更新当前模块导入，需要一个回调(callback handler)  

## `accept(handler: ({module: any}) => void)`

![accept()](https://github.com/Trojan0523/weekly_notes/blob/main/image/accept().png?raw=true)

- 允许 HMR 更新当前模块
- 已更新模块实例执行 accept handler
- 用`Function.prototype.apply(this)` 给当前应用的模块实例做新模块的导入，当前模块能在运行时应用中进行更新

- 重要区别：ESM-HMR 从不给用户替换允许的模块，相反，当前模块其实会在 accept() 回调中提供了一个已更新的模块实例，这一变更也取决于 `accept()` 回调能否给与当前应用中的当前模块的更新。

- 用例：导入模块需要被更新

## `accpet(deps: string[], handler: ({ dep: any[]; module: any }) => void)`

![accept(handler)](https://github.com/Trojan0523/weekly_notes/blob/main/image/accept(handler).png?raw=true)

- 允许 HMR 更新当前模块

- 已更新模块实例执行 accept handler

- 用`Function.prototype.apply(this)` 给当前应用的模块实例做新模块的导入，当前模块能在运行时应用中进行更新

- 重要区别：ESM-HMR 从不给用户替换允许的模块，相反，当前模块其实会在 accept() 回调中提供了一个已更新的模块实例，这一变更也取决于 `accept()` 回调能否给与当前应用中的当前模块的更新。

- 用例：导入模块需要被更新

## `accpet(deps: string[], handler: ({ dep: any[]; module: any }) => void)`

- ![accept(deps, handler)](https://github.com/Trojan0523/weekly_notes/blob/main/image/accept(deps,%20handler).png?raw=true)

- 部分情景下，自身依赖没有引用的时候，大概率不会更新一个已经存在的模块，如果能传一个依赖导入标识符数组给到`accept`回调, 这些模块在回调中能通过依赖属性赋予有效状态，反之，`deps`属性应该为空。
- 用例：需要一个方法给依赖打引用标，更新当前模块

## `dispose(callback: () => void)`

- ![dispose()](https://github.com/Trojan0523/weekly_notes/blob/main/image/dispose.png?raw=true)

- `dispose()` 的回调会在新模块被加载、`accept()` 被调用前两个阶段走完后执行。用`this` 移除掉所有副作用，或者加载(二、三等等后续)副本模块前的扫除操作。

- 用例：模块存在副作用，需要做清理操作

## `decline()` `import.meta.hot.decline()`

- 此模块不是`HMR-compatible` (非 HMR-兼容)
- 下放所有更新，强制整个页面重新载入
- 用例：模块不接受 HMR 更新，例如导致永久副作用的模块。

## `invalidate()`

![invalidate()](https://github.com/Trojan0523/weekly_notes/blob/main/image/invalidate.png?raw=true)

- 被调用时，根据条件做当前模块校验
- 此作用会将进行中的更新做 block，同时强制页面重新载入
- 用例：符合某些条件下拒绝更新

## `import.meta.hot.data`

![hot.data()](https://github.com/Trojan0523/weekly_notes/blob/main/image/hot_data.png?raw=true)

- 可用`import.meta.hot.data` 把`dispose()`回调中的 data 传递到`accept()`回调中去。
- 任何时候更新开始时，此 data 默认为一个空数组({})
