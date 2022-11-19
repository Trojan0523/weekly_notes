# Personal. 记一次esbuild 多平台编译差异导致的编译失败的排查思路

> 来源于本周二一次发版失败后的问题排查。 2022-11-19 16:47

## 上图

- ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/build-failed-jenkins-1.png)

## 时间线

- 2022.11.15  17.22 - 2022. 11.18 14.21
  
  - 第一次发版失败到第二次发版失败期间，服务端同学提醒测试环境没有改版后的代码，第一时间条件反射到测试环境测试究竟是代码写的不对还是代码没有发上线。
  
  - 使用同一页面本地和测试环境交互体验做对比，可能是代码没有发上线，于是才上去 jenkins 查看代码是否上线 (由于封板后需求不着急上，所以也没有去看上一次发版是否成功，隔了几分钟就在 15 号当天告诉服务端同学能测试了，自己要检讨一下)
  
  - 15 号当天的代码没有发版成功，觉得可能是 jenkins 抽风了，没发成功。于是 18 号第二次发版就再点了一次发版按钮，没有去看构建时报错信息
  
  - 第二次还是发版失败，就很奇怪了，发的版本没有修改过任何配置信息，这时候才去看看具体的报错信息
  
  - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/jenkins-output-failed-2.png)
  
  - 突然的发版失败还是先不麻烦运维大哥去看问题了，于是看到这个Error的信息，就google了一下具体的报错信息，`type Cannot find module 'node:module' in vite config`
  
  - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/vite-google-search.png)
  
  - 不出意外，搜不到精准的报错信息，浪费了点时间~~
  
  - 然后用本地在用的node版本`v14.6.0` 跑了一次编译build，成功了~~
  
  - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/terminal-output-success.png)
  
  - 这时候就纳闷了，问了一下运维大哥，看看最近有没有修改过`jenkins File`的配置，得知没有改动过后，尝试在发版的`dev`分支把新提交上去的`commit` `reset`  掉，再发一次，果不出奇然，还是挂了~~

- 2022.11.18 15.25- 2022.11.18 15.51
  
  - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/jenkins-env.png)
  
  - 那就初步确认了不是配置的问题，可能是jenkins的问题，于是尝试修改jenkins配置，此时使用的是从`dev` `checkout` 出来的另外一个分支，发现改来改去代码都不生效 (这里应该是小坑，无伤大雅)

- 2022.11.19 10.17 - 2022.11.19 11.33
  
  - 在此时间段之前，也进行了小版本的依赖锁定，不断地去看是否是小版本依赖升级导致的编译失败问题 (存疑：记录写下时仍不能确定，因为两周前的版本是没有任何问题的)
  
  - 次日 10.17，继续发了一版，然后找报错信息，`Console Output` 中找到了整个编译构建流水线的日志，确认代码分支修改失败后，发现可以进行回放，于是在回放中直接修改 `Push Docker Image` 构建指令，并重新执行构建操作:
  
  - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/jenkins-replay.png)
  
  - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/jenkins-node-version.png)
  
  - 这次构建把对应的node版本号进行输出，发现与本地Node版本`v14.6.0`和 docker Node版本 `v14.16.0` 和 jenkins Node版本 `v14.15.4`  存在差异，于是在本地通过`nvm install v14.15.4` 将node本地版本与Jenkins node版本对齐, 然后再进行编译。编译不过，但不是同样的报错信息。
  
  - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/terminal-error-output.png)

    > 由于个人使用习惯，本地经常使用高版本pnpm进行依赖安装，同时跑vite的时候也是使用较高node版本，所以对于报错复现有了一定的阻碍，这里还是得告诫自己尽量在复现问题的时候把所有的开发构建环境进行对齐。
  
  - 这里仍然没有复现到一模一样的报错信息，想了一下，平时开发时喜欢使用`pnpm` 和高版本的node，所以赶紧将本地项目的`node_modules` 删掉，用 `npm install` 对齐构建时依赖安装指令，然后再进行build，果不其然，一模一样的报错信息出来了:
  
  - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/terminal-error-output-2.png)
  
  - 既然是vite.config 报的esbuild的错，当然直接去vite的issue区找问题看看有没有开发者同样遇到的，赶巧看完整个报错信息后，抓了一个 `esbuild-darwin-64` package的关键字，就找到部分问题所在点了：
  
  - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/vite-issue.png)
  
  - vite 团队的老哥给出了部分的解释，解决的方法是要确保对应容器依赖安装正确(虽然是废话，但是也指向了可能是 darwin-64 和 linux-64 平台系统差异的问题)

    > 记录下 M1 Pro 的系统环境 {"os":"darwin","arch":"arm64"}
  
  - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/esbuild-issue.png)
  
  - 虽然在这个issue中找不到相同的报错信息，但是种种问题都是指向esbuild，那不妨去esbuild issue区转转吧，作者`evanw` 在此[Issue](![loading-ag-352]())中说出了部分的构成报错原因:
    ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/esbuild-issue-npm.png)
  
  - 构成错误的原因可能在npm而不是esbuild，需要做的就是升级你的npm版本，所以直观升级npm的方式，一是升级node版本(自带npm版本升级)，二是升级npm
  
  - 在 esbuild 官方文档中，作者也解释了对于不同平台的差异性 ：

    - ![](https://raw.githubusercontent.com/Trojan0523/weekly_notes/main/image/esbuild-doc.png)

    - 由于在不同OS跑的时候，会编译生成本地代码，所以需要安装具有本平台特性的二进制可执行文件(window平台生成的是PE文件格式， linux平台生成的是可执行的elf文件，mac上则是 Mac-O文件 )

    > - 插播一个补充，为什么由go编写的esbuild能在node里面跑呢 ?
    >
    > - 一是 go编写的代码可以编译成 `native code (本地代码)`，天然具有跨平台的优势,只要编译生成了本地可执行的二进制文件，本平台就能执行。
    >
    > - 我们知道，在node上可以写c++ 的 `addon（插件）` 的。遇到很特定极端的情况时 (比如ARM架构下的 M1) ，一是通过换掉此架构下的node，二是 使用`esbuild-wasm` 作为代替，此包原理大概是将go编译成wasm，然后通过node执行wasm，此方法带来非常多的隐式开销，会比esbuild本身慢十倍不止，所以没人用的

## 解决方案
  
- 问题既然能复现了，那就好办了, 两种初步解决的方案

  - 是跟运维大哥将jenkins的node版本对齐，然后再重新编译发布上线即可，但是为啥会导致编译失败的情况出现呢，看回上面的报错信息。

  - 换掉npm，直接使用pnpm

## summary

- 虽然解决这个问题花了不少时间，但是不少排查问题的思路和方法是值得自己去复盘的，查了不少资料，学到了不少东西。

- 完。
