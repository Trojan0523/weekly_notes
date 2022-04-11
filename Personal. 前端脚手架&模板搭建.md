# Personal: 前端脚手架&模板搭建

> 内部搭建脚手架和模板的事情暂时先放下来先迭代新项目了，不过边写的过程也会不断更新本文，自己内部分享的文章整理一下，简单说一下近期优化的前端脚手架和基于 Vite 的前端模板搭建流程



## 前言(问题)

- 问题 1: 社区那么多模板，为什么不用社区的？
  - 答： 社区的模板能满足一定用户的需求，未必能满足公司内部的一些定制化的需求，所以会选择自己去搭建公司内部的模板以便下次新项目的复用。
- 问题 2: 怎么看待社区的技术？
  - 答：用就完事了，提效的工具不一定都要自己去写，很多优秀的工具学会使用和分辨使用场景就很好了。



## 脚手架

- 对于公司内部的脚手架，其实做的事情比较简单，一是根据用户选择拉取对应的项目模板，二是根据用户输入的信息在拉取模板后进行指定映射路径及文件关键字进行模板字符串替换，达到每次不需要去手动修改对应项目名称和描述的效果。
- 脚手架用到的技术栈
  - Shell.js  执行 shell 命令
  - fs-extra fs 模块的拓展库，用于更好的读取文件系统的信息，并支持 promise
  - Download-git-repo 从 git 仓库中下载模板
  - consola.js 为控制台提供更好的信息展示标识
  - chalk.js 控制台交互信息高亮，提供不同的二进制颜色展示
  - Inquirer.js nodejs 交互式命令行工具，用于用户选择或输入信息
- ![cli-1](https://github.com/Trojan0523/weekly_notes/blob/main/image/cli-create-1.png?raw=true)

- ![cli-2](https://github.com/Trojan0523/weekly_notes/blob/main/image/cli-create-2.png?raw=true)

## Vite 前端项目模板

- 内部 template

  - 公司一共有三套基础模板，对应的是 B 端 Webpack 模板、B 端 Vite 模板、C 端 Webpack 模板，这里只对 Vite 部分进行介绍

- 前置知识

  - Koa

    - Vite 利用 Koa 启动了一个 http 服务器，并且通过 Koa middlewares 按序加载插件

    - Koa 会读取 vite 的部分属性进行服务器的配置

    - ```typescript
      // src/index.ts 省略部分代码,ViteDevServer源码实现缩略版
      const app = new Koa<State, Context>()
      const server = resolveServer(config, app.callback())
      const watcher = chokidar.watch(root, {
        ignored: [/\bnode_modules\b/, /\b\.git\b/]
      }) as HMRWatcher
      const resolver = createResolver(root, resolvers, alias) // 解析模块逻辑
      
      const context: ServerPluginContext = {
        root, // 项目根目录
        app, // Koa 实例
        server, // 自定义的 server 主要服务 https/http2.0 的情况
        watcher, // 本地文件的watcher
        resolver, // 定义模块的解析逻辑
        config,
        port: config.port || 3000
      }
      const resolvedPlugins = [
        // 这里加载一堆插件，本质是 Koa 中间件增强服务端能力
        moduleRewritePlugin,
        moduleResolvePlugin,
        ...
      ]
      resolvedPlugins.forEach((m) => m && m(context))
      ```

  - esBuild

    - Esbuild 是 figma CTO 用 GO 写的打包工具 
    - 优点如下：
      - 利用 GO 编译原生程序，进行多线程打包，其内部打包算法充分利用多核 cpu 优势，让所有的步骤尽可能并行执行
      - 没有第三方依赖，不存在第三方库的黑盒逻辑
      - 内存利用效率高，webpack 等等的打包会频繁的进行解析和传递 AST，   eg: string -> TS -> JS -> string, 涉及的工具链比如： webpack -> babel -> terser, 每经过一个工具链都要重新解析 AST，内存占用大，使用一个工具链能尽可能复用一份数据，提升编译性能~
    - 缺点：
      - 没有 TS 类型检查

    - 编译出来的 target 模块 无法降级到 ES5 及以下

  - Rollup 

- 搭建基于 Vite 的基础模板 vite-business

  1. 技术栈

     - 包管理工具: npm/pnpm(推荐)
     - 构建工具 Vite/Rollup
     - 框架 Vue3
     - 组件库 Element Plus
     - 状态管理 Vuex4
     - 路由 Vue-router 4
     - css 预处理器 scss
     - 拓展能力 TypeScript TSX

  2. 插件系统

     1. ### [vitejs/plugin-vue-jsx](https://github.com/vitejs/vite/tree/main/packages/plugin-vue-jsx)  为 Vue3 提供 jsx 支持

     2. ### [vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy)  为打包后的文件提供传统浏览器兼容性支持

     3. ### [unplugin-vue-components](https://github.com/antfu/unplugin-vue-components) 组件按需自动导入

     4. ### [vite-plugin-svg-icons](https://github.com/vbenjs/vite-plugin-svg-icons) 生成 svg 雪碧图

     5. ### [unplugin-auto-import](https://github.com/antfu/unplugin-auto-import) 依赖自动引入

     6. ### [vite-plugin-style-import](https://github.com/vbenjs/vite-plugin-style-import) 样式自动引入

     7. ### [vite-plugin-optimize-persist](https://github.com/antfu/vite-plugin-optimize-persist) 优化预构建自动写入

     8. ### [vite-plugin-package-config](https://github.com/antfu/vite-plugin-package-config) package.json 自动写入

  3. 项目目录

     1. ```bash
        ├── README.md  # 项目介绍
        ├── app
        │   ├── App.vue # 根组件
        │   ├── assets # 静态资源    
        │   ├── components # 组件
        │   ├── defineds # 类型声明文件
        │   ├── directive # 自定义指令
        │   ├── env.d.ts # 模块类型声明文件
        │   ├── icons # svg文件夹
        │   ├── lang # i18n
        │   ├── layout # 布局
        │   ├── main.ts # 入口文件
        │   ├── permission.ts # 权限控制
        │   ├── public # 客户端项目资源目录
        │   ├── router # 路由
        │   ├── store # 状态管理
        │   ├── styles # 样式文件
        │   ├── types # 共用类型
        │   ├── utils # 工具函数
        │   └── views # 页面文件
        ├── auto-imports.d.ts # unplugin-auto-import插件自动生成的定义文件
        ├── components.d.ts # unplugin-vue-components/vite 插件生成的定义文件
        ├── env.d.ts
        ├── index.html # html入口
        ├── jsconfig.json 
        ├── server # 启用express作为http服务器，加载Vite中间件，部分Restful API服务也在本文件夹编写
        ├── lib
        │   └── env-config
        ├── nodemon.json
        ├── package.json
        ├── pnpm-lock.yaml
        ├── public
        │   └── favicon.ico
        ├── test
        │   └── basic.test.ts
        ├── tsconfig.json
        └── vite.config.ts # vite配置文件
        ```

  4. 项目配置

     1. TS Vite 天然支持 .ts 文件，所以简单的配置下 tsconfig.json 就可以了~,跟之前模板项目差异不大

        1. ```json
           {
             "compilerOptions": {
               "target": "esnext",
               "useDefineForClassFields": true,
               "module": "esnext",
               "moduleResolution": "node",
               "strict": true,
               "jsx": "preserve",
               "sourceMap": true,
               "resolveJsonModule": true,
           		"importHelpers": true,
               "esModuleInterop": true,
               "lib": ["esnext", "dom", "DOM.Iterable", "ScriptHost"],
               "isolatedModules": true,
           		"allowSyntheticDefaultImports": true,
               "skipLibCheck": true, // 跳过lib.d.ts检查
               "allowJs": true,
               "paths": {
                 "@/*": ["./app/*"],
                 "@assets/*": ["./app/assets/*"],
                 "@components/*": ["./app/components/*"],
                 "@defineds/*": ["./app/defineds/*"],
                 "@directive/*": ["./app/directive/*"],
                 "@icons/*": ["./app/icons/*"],
                 "@lang/*": ["./app/lang/*"],
                 "@layout/*": ["./app/layout/*"],
                 "@router/*": ["./app/router/*"],
                 "@store/*": ["./app/store/*"],
                 "@styles/*": ["./app/styles/*"],
                 "@utils/*": ["./app/utils/*"],
                 "@views/*": ["./app/views/*"],
             }
             },
             "include": ["app/**/*.ts", "app/**/*.d.ts", "app/**/*.tsx", "app/**/*.vue"],
             "exclude": ["node_modules", "dist"],
             "types": ["vite/client", "node"],
           }
           ```

     2. 环境变量

        1. Vite 提供了两种模式： 开发模式和生产模式

        2. 官网的推荐是可以创建 4 个.env 文件，一个通用配置和三种环境： 开发、测试、生产

        3. ```bash
           # .env .env.development
           NODE_ENV = development
           
           prefix = 'xxx'
           
           api_prefix = '/xxx/xxx.com'
           ```

     3. 路径别名

        1. Vite 提供了内置的路径配置方法，同时可以使用 rollup 方式进行配置 (注意： 上文的 tsconfig.json 是给 IDE 和 ts 编译器识别模块使用的，这里的 Vite 路径别名是给 Vite 构建的时候使用的，用途不一样，在 Webpack 中通过配置 alias 实现)

        2. ```typescript
           // vite.config.ts
           resolve: {
                 alias: {
                   '~': _resolve('./'),
                   '@': _resolve('app'),
                   '@assets': _resolve('./app/assets'),
                   '@components': _resolve('./app/components'),
                   '@defineds': _resolve('./app/defineds'),
                   '@directive': _resolve('./app/directive'),
                   '@icons': _resolve('./app/icons'),
                   '@lang': _resolve('./app/lang'),
                   '@layout': _resolve('./app/layout'),
                   '@router': _resolve('./app/router'),
                   '@store': _resolve('app/store'),
                   '@styles': _resolve('./app/styles'),
                   '@utils': _resolve('./app/utils'),
                   '@views': _resolve('./app/views'),
                 },
                 extensions: ['.js', '.ts', '.jsx', '.tsx', '.json'],
               },
           ```

     4. 封装 SVG 图标组件

        1. 通过 `vite-plugin-svg-icons`插件，实现自动引入 svg 图标

        2. ```typescript
           viteSvgIcons({
              // config svg dir that can config multi
              iconDirs: [_resolve('app/icons/svg')], // icon路径
              // appoint svg icon using mode
              symbolId: 'icon-[dir]-[name]', // icon使用规则
           }),
           ```

     5. 按需自动引入

        1. `unplugin-vue-components` 可以直接帮我们自动按需引入组件，只会注册到使用的组件，通过在 vite.config.ts 文件中简单做下配置，省去了在入口文件(main.ts)注册的步骤，同时开发的时候使用过某些组件，都会记录 components.d.ts 中，记录下来以后可能会用于预构建优化。

        2. ```typescript
           import { defineConfig } from 'vite'
           import Components from 'unplugin-vue-components/vite'
           
           export default () => {
               return defineConfig({
                   Components({
                   dts: true, // 是否生成.d.ts文件
                   resolvers: [
                      // 第三方组件库解析器引入
                     ElementPlusResolver(),
                   ],
                 }),
               })
           }
           ```

        3. `unplugin-auto-import` 同样也能帮助我们自动导入第三方库，形成引入依赖，全局下可以不需要引入就可以直接使用某些三方库的函数:

        4. ```typescript
           import { defineConfig } from 'vite'
           import autoImport from 'unplugin-auto-import/vite'
           
           export default () => {
               return defineConfig({
                   imports: [
                     'vue',
                   ],
                   dts: true,
               })
           }
           ```

        5. ```typescript
           <script lang="tsx">
           onMounted(() => xxx())
           </script>
           <!-- 自动编译为 -->
           
           <script lang="tsx">
           import { onMounted } from 'vue'
           onMounted(() => xxx())
           </script>
           ```

     6. CSS 预处理器

        1. Vite 提供了对.scss .sass 等 css 预处理文件的内置支持，只需要装一下相对应的预处理依赖就可以使用了

        2. styles 文件夹下的全局 scss 变量要通过 preprocessorOptions 引入，不需要在入口文件中导入了~

        3. ```typescript
           // vite.config.ts
           css: {
                 preprocessorOptions: {
                   // define global scss variable
                   scss: {
                     additionalData: '@use "./app/styles/variables.scss" as *; @use "./app/styles/function.scss" as *;',
                   },
                 },
               },
           ```

        4. 组件库中的部分按需引入的样式可以通过 `vite-plugin-vitestyle-import` 进行引入

        5. ```typescript
           // 按需导入element-plus的css样式
                 styleImport({
                   libs: [
                     {
                       libraryName: 'element-plus',
                       esModule: true,
                       resolveStyle: name => {
                         return `element-plus/lib/theme-chalk/${name}.css`
                       },
                     },
                   ],
                 }),
           ```

     7. 项目细节和打包构建

        1. 依赖预构建

           1. 第一次启动 Vite 的时候，会进行动态依赖分析，分析出来的预构建的依赖会缓存到 node_modules/.vite 中，通过源来决定是否重新运行与构建步骤：
           2. ![](https://github.com/Trojan0523/weekly_notes/blob/main/image/pre-building.png?raw=true)
           3. vite.config.ts 中添加 optimizeDeps: { include: ['element-plus/lib/locale/lang/zh-cn'] }，添加到 include 中的包可以强制添加到预构建链接中, 依赖多了以后，可以使用插件(`vite-plugin-optimize-persist` `vite-plugin-package-config`  )帮助你自动找到依赖项，写入 package.json （注意：个人对此插件有争议，因为会直接写入 package.json, 侵入性比较强，同时破坏 package.json 结构的清晰完整性）

        2. 构建打包选项

           1. vite 配置中提供了 build 选项让我们进行项目打包构建,生产环境下，vite 走的还是 rollup 的打包构建方式 ,配置如下:

           2. ```typescript
              build: {
                    // 使用esbuild来压缩js和css代码，官网说它比 terser 快 20-40 倍，压缩率只差 1%-2%。
                    minify: 'esbuild',
                    sourcemap: process.env.NODE_ENV === 'development',
                    // gzip报告不显示，减少构建时间
                    reportCompressedSize: false,
                    // 消除打包大小超过500kb警告
                    chunkSizeWarningLimit: 2000,
                    // terserOptions: {
                    //   // detail to look https://terser.org/docs/api-reference#compress-options
                    //   compress: {
                    //     drop_console: false,
                    //     pure_funcs: ['console.log', 'console.info'],
                    //     drop_debugger: true,
                    //   },
                    // },
                    // build assets Separate
                    assetsDir: 'static/assets',
                    rollupOptions: {
                      output: {
                        chunkFileNames: 'static/js/[name]-[hash].js',
                        entryFileNames: 'static/js/[name]-[hash].js',
                        assetFileNames: 'static/[ext]/[name]-[hash].[ext]',
                      },
                    },
                  },
              ```

     8. 代码风格和流程规范

        1. 项目沿用了之前的 eslint-airbnb 规范，避免基本的语法错误，同时也能保证代码的可读性

        2. 对于 commit 提交的信息，做一个小的约定(不强制)，用 commitlint + husky 规范 git commit -m ""中的一些描述信息，在执行 git commit 指令的时候，会经过两个阶段： 1 是进行 eslint 的检查，2 是进行 commit 提交格式的检查

        3. ```bash
           # 提交格式(注意冒号后面有空格)
           <type>: <description>
           # type: 用户表明提交的改动类型，具体见项目中CONTRIBUTE.md 文件
           # description: 一句话描述需求
           git commit -m "feat：xxxx"
           ```

     9. 开发风格(3 套，不受限制)

> 抽离出这 tsx 一套范式的原因：hooks 抽离可以达到多端复用的效果，通过只替换 html 结构和样式达到不同端逻辑统一

- 体验 tsx 在 vue 中的开发可以按照项目中 views/template 的格式进行开发，目录如下:
  ![](https://github.com/Trojan0523/weekly_notes/blob/main/image/tsx-category.png?raw=true)
  ![](https://github.com/Trojan0523/weekly_notes/blob/main/image/tsx-template.png?raw=true)
  ![](https://github.com/Trojan0523/weekly_notes/blob/main/image/tsx-hook.png?raw=true)

- 顶层 Script setup 在 Vue 3.2 之后也已经从 rfc 正式释放了，感兴趣的同学可以看看雄宇同学的这篇博客： [谈谈script setup 定稿](https://github.com/Trojan0523/weekly_notes/blob/main/Personal.%E8%B0%88%E8%B0%88%20script%20setup%20%E5%AE%9A%E7%A8%BF.md)

- 普通的 script setup `<script>setup ()</script>`



### Summary

- 以后会接入的工作:
  - 与团队普及 TDD、BDD 等开发方式，接入 Vitest 作为测试框架
  - 基于 Pnpm 拓展 monorepo， 接入 Vitepress 作为独立文档维护站点，书写业务文档和开发文档
  - 使用 Pinia 代替 Vuex4 ,去除繁琐的类型定义 （感兴趣的同学可以看看同样是雄宇同学翻译的中文博客文档 [Why Pinia](https://github.com/Trojan0523/weekly_notes/blob/main/Personal.%20Why%20Pinia.md#personal--why-pinia)）
  - 脚手架集成 EJS 做文件模板，直接写入文件代替模板字符串替换
  - 微前端
  - C 端 webpack 项目改造， Vite SSR 调研
  - BUG 解决：依赖预构建反复执行，页面卡住 10-20s(预构建缓存目录 .Vite 中会不断产生新的依赖缓存文件，随着服务频繁 reload，不断清空所有缓存文件，再次产生更多的新依赖，也就是不断地`重刷`缓存目录，过大概 20 多秒才稳定下来。) (抖音技术团队三元给出了解决方案，待解决问题)



###  附录

- 引用一个比较认同的观点

  - ```markdown
    1. 个人认为 Webpack 最大的问题从来都不是它的性能，而是配置的复杂度过高。当然过于复杂的配置导致绝大多数人并不能正确的配置导致性能十分低下又绕回去了。但是如果正确的使用了 Webpack 的配置，在绝大多数场景下 Webpack > 4.0 的性能我认为是足够用的，大部分应用的启动构建速度都能够控制在 10s 以内，虽然仍然跟 Vite 的秒级启动无法相比，但也足够够用，如果你的 Webpack 项目动辄需要构建几十秒甚至一分钟以上，那么你应该思考一下是不是你的使用姿势或者是框架本身的问题。SSR 框架使用 Webpack 也用来开发过十几个页面几百个组件的应用，并没有感受到速度的明显下降。绝大多数前端框架(这里指的是在 React/Vue 上面包一层或者包很多层的那种框架比如 Next, Nuxt 等等)速度慢的原因无外乎是使用了错误的构建配置，或者是包了太多层导致自己都不知道哪一层导致性能出问题了。
    2. 旧应用迁移到 Vite 有一定成本，且无法保证稳定性。了解 Vite 的同学都知道，它是基于浏览器 ESM 特性的，虽然它提供了预优化这一步骤来将不支持 ESM 格式的第三方模块转换为 ESM 格式，但对于业务代码仍需要我们手动修改。有些几万行十几万行代码的旧应用实在是改不动，且生产环境它采用 Rollup 进行打包构建，并不能保证构建出来的行为与原 Webpack 构建出的文件行为一致。目前我的部分同事反馈会出现一些非预期的情况。这一点有待商榷需要收集整理问题。不过如果是新的 SPA 应用，倒是可以放心大胆的使用
    ```

- 实在话，新技术都是有风险的，上生产之后出现什么状况现在还不清楚，主要的问题应该是 rollup 和 webpack 打包出来的产物不一致的问题。目前 webpack 在 b 端 c 端的项目里面都能做到比较好的优化，一个初期的小项目能给与我机会去实验性落地 Vite，即使挂了也不会有很大的影响，冲就完事了。