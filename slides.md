---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS (experimental)
css: unocss
---

# 有 pnpm 了还用 🅿️npm

panhailiang@thoughtworks.com

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    更好的包管理器<carbon:arrow-right class="inline"/>
  </span>
</div>

---
layout: image-left
image: https://static.npmjs.com/338e4905a2684ca96e08c7780fc68412.png
---

# npm

npm 还包括 服务、站点、CLI，这里讨论特指 CLI。

<v-click>

起初作为下载和管理 Node.js 包依赖的方式。

</v-click>

<v-click>

现在也成为前端 JavaScript 使用的工具。可以管理项目的依赖安装、更新、下载。

</v-click>

<br>

<v-click>

```sh
npm install

npm install --save <package-name>

npm install --save-dev <package-name>

npm update

npm update <package-name>

npm run <task-name>
```

</v-click>


<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #b13c36 10%, #da6b38 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: bold;
}
</style>

---

# 依赖黑洞

npm 早期存在依赖黑洞问题：即使是相同包的相同版本，作为不同包的依赖时也会被安装多次

- 大量重复文件浪费磁盘空间
- 并且无法实现共享实例（一些需要单例运行的包）。

<div class="flex justify-center">
  <img class="w-140 rounded" src="https://d1dwq032kyr03c.cloudfront.net/upload/images/20191010/20111380t9EkQTYpmS.jpg" />
</div>

<!-- 经常会听到议论依赖黑洞，依赖黑洞是一个问题吗？单论“黑洞”层级过深，其实不算是个问题，本质上对于一个一般规模的项目来说，其依赖结构树就是一个层级很深，只是直接将逻辑树映射到物理存储时发现有内存占用大和无法实现单例 -->

---
layout: image-left
image: https://engineering.fb.com/wp-content/uploads/2016/10/gozv3waqlu2rj6cdaaaaaabdxp9ebj0jaaab.jpg
---

# yarn

[yarn](https://classic.yarnpkg.com/en/) 主打确定性、扁平依赖和离线模式的快速、可靠、安全的依赖管理。

<br>

```sh
npm install --global yarn
```
<br>

```sh
yarn add <package-name>

yarn add <package-name> --dev

yarn add <package-name> --peer

yarn upgrade <package-name>

yarn install # or `yarn`

yarn run <task-name>
```

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #3e789b 10%, #4187b2 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: bold;
}
</style>

---

# 如何解决依赖黑洞的问题

Node.js 有一套模块的查找读取规则：[require.resolve](https://nodejs.org/docs/v0.4.2/api/modules.html#all_Together...)。

<v-click>

对于以“包名”的方式读取包时，会从当前文件所在文件夹下开始逐层向外寻找 node_modules 文件夹，并在其中尝试寻找目标包。

</v-click>

<v-click>

yarn 的解决思路就是基于此机制，将依赖提升至最外层（node_modules 扁平化）。

</v-click>

<v-click>

<div class="flex justify-center">
  <img class="w-140 rounded" src="https://s1.ax1x.com/2022/07/19/jozc6S.png" />
</div>

</v-click>

---

# 解决了但又没完全解决

npm 的 [SemVer](https://semver.org/lang/zh-CN/) 规范，对于大版本的更新意味着向下不兼容。

对于相同包的不同版本，也只能提升一个到最外层，另一个版本依然存在重复安装的问题。然而到底提升哪个版本，是存在不确定性的。

<v-click>

<div class="flex justify-center">
  <img class="w-90 rounded" src="https://s1.ax1x.com/2022/07/18/joJw1P.png" />
</div>

</v-click>

<v-click>

因此 yarn 增加了 `yarn.lock`，通过 `package.json` 和 `yarn.lock` 共同确定依赖树结构。

</v-click>

<v-click>

yarn 解决的问题只有避免大多数包重复安装的问题，

</v-click>

<!-- yarn 并没有直接去解决真正的问题，依赖层级不是问题本质。回顾一下真正的问题是啥 -->

---

# doppelganger

相同包的不同版本只会提升一个至最外层，另一个版本的包如果被其他多个包依赖，也会安装多次，就像“双胞胎陌生人”。

<div class="flex justify-center">
  <img class="w-40 rounded" src="https://wxsm.space/2021/npm-history/d51b29d640de4f6fa97858e68db9144a.png" />
</div>

<v-click>

无法实现单例、多次安装也浪费磁盘空间和安装速度。

</v-click>

---

# phantom

拍平后的 `node_modules` 里可以直接看到项目中所有的依赖，因此在项目中可以引用 `package.json` 中并未显示声明的包（幻影依赖）。

<div class="flex justify-center">
  <img class="w-30 rounded" src="https://wxsm.space/2021/npm-history/91ba2aeacbb640fe931c2b3f9e762b2b.png" />
</div>

<v-click>

这种依赖引用是不可靠的，比如后续显式引用了另一个版本的包存在不兼容的用法，就会出现非常难定位的 bug。

</v-click>

---
layout: image-left
image: https://engineering.fb.com/wp-content/uploads/2016/10/gozv3waqlu2rj6cdaaaaaabdxp9ebj0jaaab.jpg
---

# yarn2.0

新的解决思路：Plug'n'Play(PnP)。

因为在安装依赖时就已经知道整棵依赖树的结构，yarn2 直接放弃了 node_modules，重写 node 的 require 机制，在 require 时不需要之前逐层向上查找，而是直接告诉 node 包的路径。

但这种方式避免不了增加额外的运行时代码，项目根目录会新增一个 `.pnp.cjs` 的 setup 文件，需要在项目的初始脚本头部添加 setup 函数调用。

<v-click>

```js
require('./.pnp.cjs').setup();
```

</v-click>

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #3e789b 10%, #4187b2 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: bold;
}
</style>

---

# PnP 的问题

PnP 看似直接解决了 doppelganger 和 phantom 问题，但因为直接砍掉 node_modules，并重写了 node 的 require，意味着所有依赖传统 node_modules 目录结构和 require 机制的包都会不兼容。

<v-click>

PnP 也提出很多年了，所以像 webpack、babel 的新版本都支持了 PnP 模式，webpack4 也可以通过引入 plugin 解决。

但仍然存在不兼容的包：[Incompatible](https://yarnpkg.com/features/pnp#incompatible)。

</v-click>

---
layout: image-left
image: https://i.morioh.com/2021/10/14/a009eaf0.webp
---

# pnpm

Performant NPM

- 高效：节省磁盘空间
- 稳定：基于符号链接的 node_modules 结构

<v-click>

```sh
pnpm install

pnpm add <package-name>

pnpm up <package-name>

pnpm <task-name>
```

</v-click>

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #e8963f 10%, #eeaf3d 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
  font-weight: bold;
}
</style>

---

# 节省磁盘空间

硬连接：硬连接的作用是允许一个文件拥有多个有效路径名，这样用户就可以建立硬连接到重要文件，以防止“误删”的功能。

<v-click>

当项目中添加一个依赖包时，会将包的所有文件下载到系统全局的 pnpm store 中，并在项目的 node_modules 下通过硬链接的方式链到 pnpm store。

当有其他项目继续用到该包时，pnpm 仅通过创建硬链接就完成依赖安装了。

</v-click>

---

# 基于符号链接的 node_modules 结构

符号链接：也叫软链接，类似路径别名，删除软链接并不会影响被指向的文件，删除源文件，相关软链接就变成了死链接。

<v-click>

<div class="flex justify-center">
  <img class="rounded" src="https://pnpm.io/zh/assets/images/node-modules-structure-8ab301ddaed3b7530858b233f5b3be57.jpg" />
</div>

</v-click>

---
layout: image-left
image: https://github.com/pnpm/benchmarks-of-javascript-package-managers/raw/main/results/imgs/alotta-files.svg
---

# benchmark

绝大多数场景下，安装包的速度比 npm、yarn 快 2-3 倍。

---

# corepack

[Corepack](https://github.com/nodejs/corepack) 是 Node.js 16.9.0 新增的内置 CLI，用于管理 npm、yarn、pnpm。——包管理器的管理器

package.json 中通过 packageManager 声明项目指定的包管理器，比如

```json
{
  "name": "foo",
  "version": "1.0.0",
  "packageManager": "pnpm@7.4.1",
}
```

无需手动全局安装 pnpm，首次执行 `pnpm` 时会自动安装指定版本的二进制执行文件。

当使用 yarn 安装依赖时会提示：`Usage Error: This project is configured to use pnpm`。

---

# 总结

<br>

<v-click>

yarn 的思路就是很骚，先是把 node_modules 拍平了，后来又提出 PnP。对于现有方案来说改动都非常激进。不可避免带来兼容问题和改造成本。

</v-click>

<v-clicks>

pnpm 思路就很“本分”，也是因为其团队的底层技术过硬，才可以保留 node_modules 依赖层级结构同时通过软硬链接解决问题。

</v-clicks>

<!-- 通过底层技术，在中间层解决上层的应用的问题，应用层可以完全无感知。 -->

---

# learn more

- [Why should we use pnpm? -- Zoltan Kochan](https://www.kochan.io/nodejs/why-should-we-use-pnpm.html)
- [Flat node_modules is not the only way -- Zoltan Kochan](https://pnpm.io/blog/2020/05/27/flat-node-modules-is-not-the-only-way)
- [关于现代包管理器的深度思考——为什么现在我更推荐 pnpm 而不是 npm/yarn?](https://juejin.cn/post/6932046455733485575)
- [JavaScript 包管理器简史](https://zhuanlan.zhihu.com/p/451025256)

<v-click>

很有意思的一个 yarn 的 issue，讨论通过软链接链接模块
- [Looking for brilliant yarn member who has first-hand knowledge of prior issues with symlinking modules](https://github.com/yarnpkg/yarn/issues/1761)

</v-click>

<v-click>

阿里也提出了一个 tnpm，用了 FUSE 黑科技
- [深入浅出 tnpm rapid 模式 - 如何比 pnpm 快 10 秒](https://zhuanlan.zhihu.com/p/455809528)

</v-click>

---
layout: cover
---

# 谢谢
