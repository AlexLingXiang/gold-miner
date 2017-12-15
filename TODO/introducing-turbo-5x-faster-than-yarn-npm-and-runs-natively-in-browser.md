> * 原文地址：[Introducing Turbo: 5x faster than Yarn & NPM, and runs natively in-browser 🔥](https://medium.com/@ericsimons/introducing-turbo-5x-faster-than-yarn-npm-and-runs-natively-in-browser-cc2c39715403)
> * 原文作者：[Eric Simons](https://medium.com/@ericsimons?source=post_header_lockup)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/TODO/introducing-turbo-5x-faster-than-yarn-npm-and-runs-natively-in-browser.md](https://github.com/xitu/gold-miner/blob/master/TODO/introducing-turbo-5x-faster-than-yarn-npm-and-runs-natively-in-browser.md)
> * 译者：[Cherry](https://github.com/sunshine940326)
> * 校对者：

# 介绍 Turbo：比 Tarn 和 NPM 快 5 倍，可以在本地浏览器中运行🔥
![](https://cdn-images-1.medium.com/max/800/1*ZM5-cr-PRyZxEV7gegcU_g.png)

**注意** ：这是我在 Google’s Mountain View campus 演讲的一部分，[**欢迎加入！**](https://www.meetup.com/modernweb/events/244544544/)

在经过四个月的努力，我很兴奋的宣布 **Turbo** 诞生了！🎉

Turbo 是一个速度极快的 NPM 客户端，最初是为了 [StackBlitz](https://stackblitz.com) 创建的：

- **安装包的速度最少是 Yarn 和 NPM 的五倍 🔥**

- **将 **`node_modules`** 的大小减少到两个数量级😮**
- **用于生产级可靠性的多层冗余** 💪
- **完全在 Web 浏览器中工作，能够拥有闪电般的开发环境 ⚡️**

![在 StackBlitz.com 中使用 Turbo 安装 NPM 包的实际速度](https://cdn-images-1.medium.com/max/800/1*flSBzkA6MwhaGdXnHE9B1g.gif)

Actual installation speed of NPM packages using Turbo on [StackBlitz.com](https://stackblitz.com/)

在 [StackBlitz.com](https://stackblitz.com/) 中使用 Turbo 安装 NPM 包的实际速度
### 为什么呢？

当我们第一个 [使用 StackBlitz 开始我们的工作](https://medium.com/@ericsimons/stackblitz-online-vs-code-ide-for-angular-react-7d09348497f4)，我们的目标就是创建一个在线的 IDE，这个 IDE 可以让你感觉和超级跑车的轮子一样快：你只需要接受瞬间响应命令的喜悦即可。


和 Turb 不同的是，NPM 和 Yarn 是本地的。因为设计 NPM 和 Yarn 就是用来处理大量依赖后台代码库，需要本地二进制或其他资源。他们的安装速度和超级跑车的速度比就是卡车的速度。 但前端代码很少依靠这种大规模的依赖，有什么问题吗？当然，这些依赖仍然会进入安装过程作为开发和子依赖和下载和提取一样，导致臭名昭著的黑洞被称为 node_modules。


![Dank, relevant meme](https://cdn-images-1.medium.com/max/600/1*liNzl2MQKqg4tLMCF4jY5g.png)


为什么 NPM 不在本地的浏览器中工作，这是问题的关键。在 `node_modules` 文件中解析、下载、提取百兆字节（或千兆字节）的典型前端项目是一个挑战，在浏览器中并不适合这样做。此外，这也证明了为什么这个问题的服务器端解决方法是 [慢、不可靠、并且成本较高的](https://github.com/unpkg/unpkg/issues/35#issuecomment-317128917)。

> 所以，如果 NPM 本身可以再浏览器中运行，我们创建一个新的 NPM 客户端怎么样呢？


### 问题：一个专门为 Web 构建的更聪明、更快的包管理器📦


Turbo’s speed & efficiency is largely achieved by utilizing the same techniques modern frontend applications use for snappy performance—tree-shaking, lazy-loading, and plain ol’ XHR/fetch with gzip.
Turbo 的速度和效率大部分都是通过现代的前端应用技术 snappy、tree-shaking、懒加载实现的，一般的请求使用 gzip。

#### **只检索需要的文件，按需检索** 🚀
Turbo 很巧妙的只检索 main、typings、和其他相关文件需要的文件而不是下载整个压缩包。无论是个人项目还是大型项目，这都减轻了惊人的负载。

![ RxJS 和 RealWorld Angular 总有效载荷大小的比较](https://cdn-images-1.medium.com/max/800/1*zl-KV3eL7lSnAI45Hb_Rcw.png)

 [RxJS](http://npmjs.com/package/rxjs) 和 [RealWorld Angular](https://github.com/gothinkster/angular-realworld-example-app) 总有效载荷大小的比较

那么如果你的重要文件并没有被主文件引用会怎么样呢？例如一个 [Sass 文件
](https://stackblitz.com/edit/angular-material?file=theme.scss)，不用担心，Turb 按需进行懒加载并且一直保存以便将来使用，这个和微信新推出的 GVFS Git protocol](https://blogs.msdn.microsoft.com/devops/2017/02/03/announcing-gvfs-git-virtual-file-system/) 工作原理有些类似。

#### 具有多种故障转移策略的健壮缓存 🏋️

我们最近推出了一个具有 Turbo 特征的 CDN，所有的 NPM 包都在一个使用 gzip 打包的 JSON 请求中，大大提高了包安装的速度。这个概念就像是 NPM 的压缩包，这个包含所有文件的并且使用 gzip 压缩。然而，Turbo 的缓存智能的只包含你项目需要的文件并压缩他们。


每一个 Turbo 的客户端都是在浏览器中独立运行的，并且如果你引用的包文件再我们的缓存中，那么会直接从 [jsDelivr’s 提供的大量的 CDN 资源](https://www.jsdelivr.com/) 中自动按需下载。如果 jsDelivr 访问不了了怎么办？不要担心，会自动替换成 [Unpkg’s CDN](https://unpkg.com)，提供三层超可靠的独立的包安装工具👌。

#### 快速的依赖解决方案 ⚡️

为了确保最小的有效负载大小，Turb 使用一个定制的解析算法，在可能的情况下积极解决通用包版本。这也是出奇的快和冗余：解析器的无服务器版本可以访问到 NPM 的全部在内存中的数据集并且 **解析任何 package.json 在 85ms 以内。** Turbo 在连接无服务器版本的解析器时有任何的问题
，在失败的时候可以优雅的在浏览器中完整运行并且检索所有必需的元数据解决方案。

在客户端上执行依赖性解决方案也会带来一些新的令人兴奋的可能性，比如只需单击一次就可以安装缺少的对等依赖关系 😮:


![](https://cdn-images-1.medium.com/max/800/1*BTe1Q-cZda_1dB3H0wROzQ.gif)

因为没有人读这些 NPM 在控制台输出的警告 😜

#### 证明可以大规模使用 Turbo 📈

Turbo 目前能够可靠地处理每个月百万级别的请求数，我们很兴奋的宣布：Google 的 Angular 团队最近选择 StackBlitz 来支持他们文档中的实例，而有数以百万计的开发人员在使用他们的文档。

### 技术预览 🙌

Turbo 是依赖于 [StackBlitz.com](https://stackblitz.com) 的，并且通过技术预览阶段我们可以运行大量的测试和测速，检验效能和可靠性的改进，你的每一个反馈都是至关重要的，所以在使用中遇到问题，不假思索的向我们 [提 issues](https://github.com/stackblitz/core/issues) 来和我们沟通！🍻


然而 Turbo 最初是为生产级的使用而设计的，但在现实的 IDE（[stackblitz](https://stackblitz.com)）中，Turbo 已经找到了少数的在线应用场所，在社区，人们已经开始设计一种方法，使用 Turbo 使脚本类型与模块相等（很酷有没有！！！），我们迫不及待地想看到人们提出的其他惊人的东西，所以，一旦我们的 API 更加完善，我们会将其在[**我们的 Github**](https://github.com/stackblitz/core) 中完全开源（和 StackBlitz 的其他部分一起）以供全世界人们使用 🤘。

最近，我们非常感谢 Google 的 Angular 团队给我们的技术下的赌注，同时感谢 Google Cloud 团队将他们令人惊叹的服务赞助给 Turbo 使用！❤️

#### 一如既往，请随时通过 Tweet 联系我 
有任何的疑问、反馈、想法等等都可以通过 [@ericsimons40](https://twitter.com/ericsimons40) 或者 @[stackblitz](https://twitter.com/stackblitz) 联系我 ：）

另外，如果你有兴趣支持我们的工作，请考虑订阅 [Thinkster Pro](https://thinkster.io/pro)！我们创建了一个新系列关于我们是如何创建 Turbo 和 StackBlitz 的，我们也会将我们的所有目录内容：）

我希望你们能看下我再 12 月 6 日[在 Mountain View 的视频](https://www.meetup.com/modernweb/events/244544544/)。

---

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。
