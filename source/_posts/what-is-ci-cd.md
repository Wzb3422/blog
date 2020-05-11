---
title: 👷‍♂️什么是持续集成、持续交付、持续部署？
keywords: JavaScript
thumbnail: 'https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511140912.png'
tags: 前端工程化
categories:
  - JavaScript
date: 2020-02-28 19:18:56
---

## 持续集成 Continuous Integration (CI)
在传统的软件开发模式下，我们等到项目组中每个人负责的功能模块完成之后，用较长的一段时间将各自的部分集成起来。（这通常是一个痛苦的过程。）

<!-- MORE -->

> 集成：开发者个人研发的部分向软件整体进行部分交付。

持续集成是对如此形式的一种改进。在开发过程中，每当开发者做出一些变更与提交之后，便自动进行构建与测试。若测试通过，则进行集成并通知开发者结果。
<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511140415.png" alt="image.png" style="zoom:50%;" />
开发者一般会在一个称为 _CI Server_ 的服务器上进行自动化的构建与测试，以测试提交的代码是否符合预期。正因此，成功实践 CI 的一个前提就是——开发者需要编写好所负责部分的测试代码（单元测试）。当所有的单元测试都通过时，会看到一个 _green build_ 。它意味着代码的变更经过测试后成功地集成到了软件整体。不过，虽然我们得到了 green build ，但并不是说现在的软件整体就可以进入生产环境进行部署了。我们还需要看看持续集成之后发生的事情——持续交付。

## 持续交付 Continuous Delivery (CD)

我们回顾一下上面的场景：开发者们的提交都进行了持续集成，在 CI Server 上通过了自动化的构建与测试。并且部署到了测试环境(production-like)。我们将代码在一系列环境下进行部署与测试的流程，称为 deployment pipeline。在每个阶段，开发者需要找到此阶段发现的 bug 以及可以优化的地方，完成了上一个阶段的测试之后，才能进入下一个阶段。
一般 deployment pipeline 是：test environment -> staging environment -> production

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511140440.png" alt="image.png" style="zoom:50%;" />

在每个阶段，不断地解决和优化软件中发现的问题。这样使得开发者对自己的代码越来越有信心，也减少了生产环境下可能会让用户发现的问题。最后通过人工操作的方式，部署至生产环境。
## 持续部署 Continuos Deployment
在持续交付实践之后，若是再让进入生产环境的这一个环节自动化，就成为了持续部署。持续部署使得每一次持续集成的小的变更，通过持续交付检验之后，直接进入生产环境。
是否要对持续部署进行实践，取决于你的具体业务与实际情况。

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511140502.png" alt="image.png" style="zoom:50%;" />

> 持续集成是持续交付的前提，持续交付是持续部署的前提。

