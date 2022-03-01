> * 原文地址：[Replacing Lerna + Yarn with PNPM Workspaces](https://www.raulmelo.dev/blog/replacing-lerna-and-yarn-with-pnpm-workspaces)
> * 原文作者：[Raul Melo](https://www.raulmelo.dev/)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/article/2022/replacing-lerna-and-yarn-with-pnpm-workspaces.md](https://github.com/xitu/gold-miner/blob/master/article/2022/replacing-lerna-and-yarn-with-pnpm-workspaces.md)
> * 译者：[CarlosChenN](https://github.com/CarlosChenN)
> * 校对者：[xyj1020](https://github.com/xyj1020)

# 用 PNPM Workspaces 替换 Lerna + Yarn

近年来，Monorepo 架构以越来越流行，鉴于它解决的问题，这是可以理解的。然而，最大的挑战是找到一个简单易用的工具去处理类似的架构。

如果你搜索 “monorepo tool javascript”，你会发现有很多文章都在向我们展示当下非常流行的一些工具，但是奇怪的是每一种工具都在用一种截然不同的方式去解决 monorepo 能解决的问题。

从我们现有的选项来看，一些像 Lerna 一样的工具已经存在一段时间，但已经不再持续维护了；一些类似于 Bolt 的工具，方案还没有被通过；还有一些工具可以运行，但只适用于某种特定的项目。

不幸的是，我们没有一个好的工具适用于所有 JavaScript/Typescript 类型的项目和所有团队规模，但这是可以理解的。

然而，现在有一个（“新”）选项，可能在大多数场景中，对我们有所帮助：**pnpm workspaces**。

但在谈到 pnpm 之前，让我告诉你，我的 monorepo/workspaces 用法，以及我是如何尝试解决使用 monorepo 中遇到的问题的。

## 我的博客

在我第一次创建我的博客，我自己做了一个 Next.js 应用，把他放到一个 git 仓库，并且同时把脚手架的代码提交（或者 push）进了同一个仓库。

在此之后，我需要建立一个 CMS（内容管理系统）存放我的内容。然后，我创建一个 Strapi 应用，把它放到另一个 git 仓库，然后将它推送到另一个 Github 仓库。

然后，我决定 fork 一个叫 `mdx-prism` 的库去修复它的一些小问题，并且让它能够自动化部署。同样的，我又新建了一个新的 git 仓库，里面放着 `mdx-prism` 的代码以及它的初始化配置。

我有 3 个 git 仓库，意味着我有了三个 eslint，prettier，jest，babel，和 typescript 配置，我有 3 个 git 仓库，意味着我有了三个 eslint，prettier，jest，babel，和 typescript 配置，但是我还是维护了一段时间。

很快，我对每次的依赖（像 TypeScript）更新感到困扰，因为每次我都需要更新三个仓库，也就意味着我需要 pull 三次。每当我学会了新的知识，比如说学习了 eslint 的新规则，我都不得不更改三个代码库中的代码等等。

我的第一反应是：

> 如果我把所有项目放到一个单独的文件夹和仓库里，创建我的基础配置，然后用它去扩展到每个项目配置里呢？

不幸的是，我无法简单放一些文件在那，扩展，希望它运行，因为实际比那复杂得多。工具需要 module/file 解决方案，我不想在我想要部署的时候，传送到所有的项目里。

在那瞬间，我意识到我需要一个 monorepo 工具去连接分散的各个项目，让我的体验更好。

我尝试一些解决方案，最简单构建和运行的方式是 Lerna + Yarn Workspaces。

当然，在构建流程中，我有一些感悟，比如理解为什么有些构建失败（不是所有的应用喜欢提升依赖关系），必须适配我的 pipelines，以及如何我如何部署每个项目。尽管如此，我还是管理所有东西，并有个合适的配置。

有了之前最简单的设置，我开始创建更小的独立模块/应用去复用，扩展，并且在不影响我现有的代码条件下，尝试新的工具。这是我亲眼见证它在一个 monorepo 运行得让人惊讶的时刻。

## 关于 Lerna + Yarn Workspaces

Lerna 是一个高级 monorepo 工具，它提供同时管理一个或多个应用/包的抽象。 

你可以运行命令（像 build, test, lint 等），用一条命令指示所有你控制的项目，如果你想，你甚至可以用标记 `--scope`过滤一个指定的项目。

Yarn Workspaces 是一个低级工具，它处理包的安装，在项目之间创建符号链接，和在根中分配模块，控制项目文件夹。

你能用整个 Lerna 或者 Yarn Workspaces 去管理你的仓库，但你可能注意到，相比于这两个工具的排他性来说，它们之间的互补性是更加突出的。换句话说，它们一起运行的很好。

甚至现在，这个组合仍然是好选择，但一些问题可能还是突出：

* Yarn Workspaces（至少对 v1）不再接收新特性和改进（上次更新是在 2018）；
* Lerna 文档是可以的，但你需要自己弄明白很多问题；
* Lerna 发布系统并不是它看到的那么简单，特别是用 commit lint 引发自动发布。
* 你可能会不太理解你需要运行的命令是什么意思或者你不知道你在运行其它命令的时候，还有什么命令正在运行中；
* Lerna CLI 有一些像[你无法在同一时间安装多个依赖](https://github.com/lerna/lerna/issues/2004) 等问题。
* Lerna CLI `--scope`并不可靠并且难以理解和使用；
* 有一个 [wizard](https://github.com/webuniverseio/lerna-wizard) 在常见任务中帮助我们，但它更像是在主仓库之外维护的。
* [Lerna 目前是没有维护的](https://github.com/lerna/lerna/issues/2703#issuecomment-744601134);

Lerna是于 2015 年创建的，这个工具的出现帮助我们解决了缺少工具管理 JS monorepos 的现状，并且它在管理 JS monorepos 上做得很好。

然而 Lerna 因为没有一个致力于它发展的团队或成员去更新和规划这个工具的未来，所以 Lerna 正在慢慢的退出管理 JS monorepos 的舞台。

我在这里不是抱怨创建者和维护者，开源的世界有许多的问题，但这是另一篇文章的主题。

你现在可能会这么想：

> 如果 Lerna 在这个时代，我们现在有什么选项？

## pnpm 简介

以防你不知道，像 **npm** 和 **Yarn**，**pnpm** 也是一个 JavaScript 项目的包管理工具。它用高效的方式做同样的工作。

使用 **pnpm** 最大的好处是 **pnpm** 解决了 npm 引入和 Yarn 复制的问题，即 npm 和 Yarn 安装依赖项的方式。

**pnpm** 旨在解决两个大问题：

### 磁盘空间

比如说你有 5 个包含 `react@17.0.2` 作为依赖的项目。

当你在所有项目里  **npm** or **Yarn** install 时，每个项目将会在  `node_modules` 中有它自己的 React 副本。考虑到一个 React 包的大小大概是 **6.9kB**， 那么在5个仓库中，我们的磁盘中就会有 **34.5kB** 相同的依赖。

这个例子看起来很小，但每个使用 JS 的 人知道，有时候 `node_modules` 很容易达到 GB 级别。

如果使用 **pnpm** 安装依赖的话，pnpm 会先把它下载的依赖存储在它自己的 store 中，也就是存在（`~/.pnpm-store`）中。依赖下载完成存储在（`~/.pnpm-store 之后，pnpm 会建立一个强链接用来链接 ~/.pnpm-store`) 中的模块和你项目中的 node-modules。

拿之前的例子举例，**pnpm** 将在它自身的存储中安装 `react@17.0.2`，当我们安装项目的依赖时，它会先检查 17.0.2 版本的 React 是否已经保存。如果是的话，它会在项目的 `node_modules` 创建一个硬链接（指向磁盘中的文件）。

相比于在磁盘中有 5 个 `react@17.0.2` 的副本 （**34.5kB**） 在我们的磁盘中，现在我们只有 1 个版本（**6.9kB**）存在 pnpm 的 store 和在每个项目中都有一个和 react 的副本文件有相同功能的强链接。

因此，我们节省了很多磁盘空间，并且如果我们的新项目使用了我们已经安装过的依赖，那么新项目安装依赖的速度会更加的快。

## Phantom 依赖

当我们用 **npm** 安装依赖时，他会把所有的依赖和依赖中的所有东西都打包到 `node_modules` 文件夹下。这种方式就是我们的说的扁平化方式。

让我们在实践中来看看。下面是 `package.json`： 

```json
{
  "dependencies": {
    "unified": "10.1.0"
  }
}
```

在运行 `npm install` 之后，`node_modules` 会变成下面这样： 

```text
node_modules
├── @types
├── bail
├── extend
├── is-buffer
├── is-plain-obj
├── trough
├── unified
├── unist-util-stringify-position
├── vfile
└── vfile-message
```

虽然这种方式已经工作了许多年，但是这种方式会导致一些问题，我们称这种问题叫做：“幽灵依赖”。

在我们项目中声明的唯一依赖是`统一的`，但在我们的项目中，仍然能在我们的代码中导入`is-plain-obj` (unified 依赖)：

```js
import ob from "is-plain-obj";

console.log(ob); // [Function: isPlainObject]
```

因为这是可能发生的，所以我们的依赖以及依赖的依赖也能出现这种在没有声明一个依赖作为依赖或同等依赖的前提下，可以从 `node_modules` 中引入某个依赖的问题。

现在，让我们看看 **pnpm** 是如何做到的。

用相同的 `package.json`，然后运行 `pnpm install`，我们将会有下面的 `node_modules`：

```text
node_modules
├── .pnpm
├── @types
├── unified -> .pnpm/unified@10.1.0/node_modules/unified
└── .modules.yaml
```

正如你所看到的，对于唯一的依赖，我们用的是 `unified`，这也是我们仅有的，但是……有一个箭头指明这个模块的是一个符号链接。

然后，观察 `.pnpm` 里面有什么：

```text
node_modules
├── .pnpm
│ ├── @types+unist@2.0.6
│ ├── bail@2.0.1
│ ├── extend@3.0.2
│ ├── is-buffer@2.0.5
│ ├── is-plain-obj@4.0.0
│ ├── node_modules
│ ├── trough@2.0.2
│ ├── unified@10.1.0
│ ├── unist-util-stringify-position@3.0.0
│ ├── vfile-message@3.0.2
│ ├── vfile@5.2.0
│ └── lock.yaml
├── @types
│ └── unist -> ../.pnpm/@types+unist@2.0.6/node_modules/@types/unist
├── unified -> .pnpm/unified@10.1.0/node_modules/unified
└── .modules.yaml
```

**pnpm** 将每个依赖项作为后缀安装在 `.pnpm` 文件夹中，然后，将它们移动到在你的 package.json 中准确定义的 `node_modules` 根目录。

现在，如果我们尝试像以前一样写相同的代码，我们将会得到一个错误，因为 `is-plain-obj` 没有安装在 `node_modules` 中：

```
internal/process/esm_loader.js:74
 internalBinding('errors').triggerUncaughtException(
 ^

Error [ERR_MODULE_NOT_FOUND]: Cannot find package 'is-plain-obj' imported from /Users/raulmelo/development/sandbox/test-pnpm-npm/pnpm/index.js
```

虽然用这种方式 安装 `node_modules` 更加可靠，但这可能破环一些已经用上面的扁平化 `node_modules` 构建的应用兼容性。

> 一个例子就是 Strapi v3。你可以在[这](https://github.com/strapi/strapi/issues/9604)看到, 他们也意识到了这一点，在将来某天也会解决的。

幸运的是，**pnpm** 考虑了这些问题，并且提供了个叫 [`shamefully-hoist`](https://pnpm.io/npmrc#shamefully-hoist) 的标记来解决这些问题。

当我们用这个标记时，依赖会用”扁平化的方式“安装依赖，应用会像 Strapi 运行。

## pnpm Workspaces

**pnpm** 在 v2 中引入的工作区特性。

它的目标是填补我们目前拥有的易于使用和维护良好的 monorepo 工具的差距。

由于它们已经有了低级的包管理工具了，无论何时你只需要在你项目的根目录下创建一个 `pnpm-workspace.yaml` 文件，他们就只需要创建一个新模块来处理工作区。

它与 Lerna + Yarn Workspaces 的配置几乎一样，有三个显著的优势：

1. 我们从 **pnpm** 控制磁盘空间修复;
2. 我们用它们很棒的 CLI （这是内置好的，并且有特别棒的 DX）；
3. 它解决了许多 Lerna CLI 像过滤，安装多版本的问题。

在（几乎）所有命令， **pnpm** 允许我们用一个叫 `--filter` 的标识运行。我认为这是不言自明的，但它会为过滤后的仓库运行该命令。

假设你有两个完整的应用，这两个完整的应用都有独立的流水线。使用 Lerna + **npm** 或者 **Yarn** 为每个项目安装依赖。

这意味着，在一些例子中，下载 1GB 的依赖可以降到 300MB。

有了 **pnpm**，我可以简单的运行下面的指令：

```bash
pnpm install --filter website
```

现在，只有根目录依赖和我的网站的依赖会被安装。

过滤的命令已经足够好了，但是它超越并且提供更多的灵活性。

我十分建议你阅读一下 [pnpm's "Filtering" 文档](https://pnpm.io/filtering) 然后看一下它是有多么令人惊讶。

> 另一个建议：["pnpm vs Lerna：多包仓库中筛选"](https://medium.com/pnpm/pnpm-vs-lerna-filtering-in-a-multi-package-repository-1f68bc644d6a)

这看起来是一件特别小的事情，但是这些小细节在不同环境中，在整个工作中造成了很大的不同。

## 迁移

如果你想看我已经合并包含整个迁移 PR，你可以看[这里](https://github.com/raulfdm/raulmelo-studio/pull/803) 。我将只高亮所有我需要展示的改变。

### 替代命令

我有一堆脚本，执行 `yarn` CLI。对于这些，我只需要用 `pnpm <command>` 或者 `pnpm run <command>` 替换；

### 移除 Yarn Workspace 配置

在我的 package.json，我已经为 Yarn 声明工作区区域并且定义一些没有提升到根 node_modules 的包；

```json
{
"workspaces": {
   "packages": [
     "packages/*",
     "apps/*"
   ],
   "nohoist": [
     "**/netlify-lambda",
     "**/netlify-lambda/**"
   ]
 }
}
```

这一切都过去了。

### 用 `pnpm-workspace.yml` 代替 `lerna.json` 

我已经移除了下面的配置：

```json
{
  "version": "independent",
  "packages": ["packages/*", "apps/*"],
  "npmClient": "yarn",
  "useWorkspaces": true,
  "command": {
    "version": {
      "allowBranch": "main"
    },
    "publish": {
      "conventionalCommits": true,
      "message": "chore(release): publish",
      "ignoreChanges": ["*.md", "!packages/**/*"]
    }
  }
}
```

换成:

```yml
prefer-workspace-packages: true
packages:
  - 'packages/*'
  - 'apps/*'
```

### 适配 pipelines，Dockerfile，和主平台

一件事我必须改变的是，在安装我 Github Actions 里的依赖，Docker 镜像，和 Vercel's 安装脚本之前，确保我安装了 `pnpm`：

```bash
npm install -g pnpm && pnpm install --filter <project-name>
```

这是必不可少的步骤因为大多数环境包含开箱即用的 yarn，而不是 pnpm（我希望这会很快改变）。

### 移除 `yarn.lock` 文件

这个文件已经不再需要了。Pnpm 创建了它自身的 `pnpm-lock.yaml` 锁文件去控制依赖版本。

### 适配构建命令

在我为我的网站运行 `lerna run build` 时，它自动理解，它必须构建我的网站使用的包。

对于 **pnpm**，我必须明确说明：

```bash
pnpm run build --filter website # Only build the website

pnpm run build --filter website... # Builds first all dependencies from the website and only then, the website
```

这些声明是十分重要的，因为我不是把所有包发布在 NPM。

### 添加 `.npmrc`

pnpm 通过 CLI 接收一堆标识和选项。如果我不想一直通过它们，我们可以在一个 `.npmrc` 文件中定义它们。

我添加在那里的唯一选项是：

```bash
shamefully-hoist=true
```

就像我前面解释的，Strapi 并不能使用 pnpm 的安装 node_modules 方式，这是惭愧的。

通过提交这些文件，我确保了，无论在哪，我运行 `pnpm install` 时，依赖会被正确的安装。

### 用 Changesets 替换 semantic-release

我必须坦白，我现在还没有完全测试这个。

总结，在我之前的步骤，我被迫以一种特定的方式编写提交，以便语义发布能够检查我的更改，通过读取消息自动知道更改了什么，更改版本并发布我的包。

我之前运行得很好，但一些陷阱，特别是考虑 Github Actions 环境是如何工作的。

虽然，Pnpm 建议我们用 [Atlassian 的 changesets](https://pnpm.io/using-changesets) 。

这个方式有点不同。现在如果我做一个改变，我必须创建一个带有一些 meta 信息和描述哪些改变的 .md 文件，基于这个文件，明白如何生成改变 long，以及应该更改哪个版本。

我仍然需要完成这个准备工作和可能写一篇关于那个的文章。 😅

## 结论

这就是我需要用 **pnpm** workspaces 代替 Lerna + Yarn Workspaces 的全部基本知识。

诚实讲，它比我最初设想的更简单。

我用 **pnpm** 更多，我更享受它。项目是可靠的，用户体验是令人愉快的。

## 参考

* [https://pnpm.io](https://pnpm.io)
* [https://github.com/lerna/lerna](https://github.com/lerna/lerna)
* [https://classic.yarnpkg.com/lang/en/docs/workspaces/](https://classic.yarnpkg.com/lang/en/docs/workspaces/)
* [https://medium.com/pnpm/pnpm-vs-lerna-filtering-in-a-multi-package-repository-1f68bc644d6a](https://medium.com/pnpm/pnpm-vs-lerna-filtering-in-a-multi-package-repository-1f68bc644d6a)
* [https://github.com/raulfdm/raulmelo-studio/pull/803/files](https://github.com/raulfdm/raulmelo-studio/pull/803/files)
* [https://medium.com/@307/hard-links-and-symbolic-links-a-comparison-7f2b56864cdd](https://medium.com/@307/hard-links-and-symbolic-links-a-comparison-7f2b56864cdd)

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。

---

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。
