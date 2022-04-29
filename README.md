## 状态管理

### 什么是状态管理？

> 状态管理就是，**把组件之间需要共享的状态抽取出来，遵循特定的约定，统一来管理，让状态的变化可以预测**。

### 为什么需要状态管理？

#### 状态共享

组件之间通常会有一些共享的状态，在 Vue 或者 React 中我们一般会将这部分状态提升至公共父组件的 `props` 中，由父组件来统一管理共享的状态，状态的改变也是由父组件执行并向下传递。这样会导致两个问题:

-   需要将共享的状态提升至公共的父组件，若无公共的父组件，往往需要自行构造
-   状态由父组件自上而下逐层传递，若组件层级过多，数据传递会变得很冗杂

#### 变化跟踪

在应用调试过程中，可能会有跟踪状态变化过程的需求，方便对某些应用场景的复现和回溯。这时候就需要统一对状态进行管理，并遵循特定的约定去变更状态，从而让状态的变化可预测。

## 我们要做什么

我们要从 0 到 1，一步一步写一个状态管理器。这个状态管理器基于 RxJS。

目标特性：

-   响应式，能响应式的获取最新的 state
-   不可变数据，应用状态对象不可变
-   单向数据流
-   \_**Store 模式**
-   适用各种 UI 框架

### 为什么要基于 RxJS

前端的`jquery`、`react`、`vue`属于军营里的戈、戟、矛，一寸长，一寸强。人人都要会，最常用也最实用的兵器。上了战场，就听号令按节奏往前戳就行了。

而`RxJS`，属于自己的傍身短刀，或其它独门武器，在特定场合，或关键时刻都能有奇效。

比如在状态管理上，`RxJS` 天生响应式，高效简洁，能发挥出巨大威力。

### 什么是 Store 模式

Store 模式是一种相对简单的状态管理模式，一般有以下约定：

-   状态存储在外部变量 `store` 里（也可以是全局变量）
-   `store` 中的 `state` 用于存储数据，由 `store` 实例维护
-   `store` 中的 `actions` 封装了改变 `state` 的逻辑

如果对 state 的变更均通过 actions，那么实现记录变更、保存快照、历史回滚就会很简单，但是 Store 模式并没有对此进行强制约束。

## 为什么要造轮子

这是一个学习的过程，轮子再多，也不影响我们边造边学。

### 能学到什么

-   从 0 到 1 深入理解状态管理
-   从 0 到 1 深度使用 RxJS
-   响应式编程入门
-   函数式编程入门
-   各种前端小知识
-   Immer 等....

## 免责申明

本人能力有限，文章都只是本人在学习和实践过程中的一些思考记录，`错误和疏漏`在所难免。
对于一些人来说，可能过于粗浅了，对于另一些人来说，又可能过于复杂。大家结合自身情况使用。欢迎大家`理性讨论`，挑错也是学习的过程，但`抬杠`就大可不必。阅读过程中`走火入魔`的，本人不负法律责任。

## 其它

#### brass 状态管理器

按本系列文章思路实现的响应式状态管理器：

[brass](https://github.com/jaqen404/brass)

[vue-brass](https://github.com/jaqen404/vue-brass)

参考：

-   [状态管理之 Flux、Redux、Vuex、MobX（概念篇）](https://juejin.cn/post/6844904013532495885)
