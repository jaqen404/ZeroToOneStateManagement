---
theme: juejin
---

本章知识点：

1. 什么是 [Observable](https://rxjs.dev/guide/observable)
2. 什么是 [Subject](https://rxjs.dev/guide/subject)
3. 什么是 [BehaviorSubject](https://rxjs.dev/guide/subject#behaviorsubject)
4. 什么是 [from](https://rxjs.dev/api/index/function/from)

## Observable

[Observable](https://rxjs.dev/guide/observable)是[RxJS](https://rxjs.dev/guide/overview)的`核心概念`，可是说掌握了 Observable 就掌握了 RxJS 的核心技术，其它所有的东西都是围绕 Observable 服务的。

接下来我就用小学二年级的大白话，介绍一下 Observable。
Observable 是一个出版社，他只能生产书籍杂志，接收消费者的订阅，然后生产发货。
现在我们来建立这个出版社（我们把所有`可以被订阅`的单位后面都加上一个`$`符号，代表它可以被订阅）：

```js
const playboy$ = new Observable();
```

出版社建立好之后，我们就需要制定一个生产流，先假定一个订阅者`Subscriber`。
出版社可以有三个行为：

1. next 向订阅者发送一件商品。
2. error 报错，生产或发送商品时出错了
3. complete 生产流结束，完成这次交易

```js
const playboy$ = new Observable((subscriber) => {
    // 发送 武侠第一期
    subscriber.next("武侠第一期");
    // 发送 武侠第二期
    subscriber.next("武侠第二期");
});
```

此时生产流制定出来了，但实际上什么也不会发生，因为并没有人订阅我们的书。要生产流启动，需要有消费者来订阅。订阅的方式就是`subscribe`。

```js
// 小明订阅了武侠杂志
playboy$.subscribe({
    next(x) {
        console.log("小明收到" + x);
    },
});
```

控制台会输出：

```
小明收到武侠第一期
小明收到武侠第二期
```

`注意：A.subcribe(B)，是 B 订阅了 A。`这点一定要理解记住。

如果你提前完成这个生产流，那后面的东西就都不会发送了。

```js
const playboy$ = new Observable((subscriber) => {
    // 发送 武侠第一期
    subscriber.next("武侠第一期");
    // 可以喝了，确认收货
    subscriber.complete();
    // 武侠第二期，不会发送，因为已经结束
    subscriber.next("武侠第二期");
});
// 小明订阅了武侠杂志
playboy$.subscribe({
    next(x) {
        console.log("小明收到" + x);
    },
    complete() {
        console.log("停产");
    },
});

// logs:
// 小明收到武侠第一期
// 停产
```

如果你报出错误，后面的东西也都不会发送了。

```js
const playboy$ = new Observable((subscriber) => {
    // 发送 武侠第一期
    subscriber.next("武侠第一期");
    // 报错，后面杂志都不会发送
    subscriber.error("有内鬼终止交易");
    // 武侠第二期，不会发送，因为已经结束
    subscriber.next("武侠第二期");
});
// 小明订阅了武侠杂志
playboy$.subscribe({
    next(x) {
        console.log("小明收到" + x);
    },
    error(err) {
        console.error("警告：" + err);
    },
});

// logs:
// 小明收到花花公子第一期
// 警告：有内鬼终止交易
```

好了，以上就是 RxJS 最核心的内容了。围绕这个出版社，可以`玩出花来`，这出版社里可以同步生产，还可以异步生产，可以合并生产线，还可以控制生产节奏。丰富的操作符[Operators](https://rxjs.dev/guide/operators)后面会慢慢一起学习。

## Subject

[Subject](https://rxjs.dev/guide/subject)是一个中间商，他既可以向出版社订阅书籍进货，也可以接收消费者的订阅发货（而 observable，出版社只能发货，`不能进货`）。

现在我们来建立一个中间商（中间商也是可以被订阅的，所以加上`$`后缀）：

```js
const businessman$ = new Subject();
```

和出版社一样，也需要有人订阅：

```js
// 小明订阅了
businessman$.subscribe({
    next: (v) => console.log("小明收到" + v),
});
// 小刚订阅了
businessman$.subscribe({
    next: (v) => console.log("小红收到" + v),
});
```

此时控制台不会有任何反应，尽管已经有消费者订阅了，但中间商并没有发货。
我们发送两本杂志试试：

```js
businessman$.next("寻秦记");
businessman$.next("射雕英雄传");

// logs:
// 小明收到寻秦记
// 小刚收到寻秦记
// 小明收到射雕英雄传
// 小刚收到射雕英雄传
```

大家都收到了名著，皆大欢喜。
中间商最重要的当然是从出版社进货，然后再发货给消费者，看看是怎样表现的：

```js
// 一个出版社
const playboy$ = from(["寻秦记", "射雕英雄传"]);
// 来了一个中间商
const businessman$ = new Subject();
// 中间商订阅了武侠
playboy$.subscribe(businessman$);
// 小明订阅了中间商
businessman$.subscribe({
    next: (v) => console.log("小明收到" + v),
});

// logs:
// 小明收到寻秦记
// 小明收到射雕英雄传
```

好的，我们现在遇到了我们的第一个操作符[from](https://rxjs.dev/api/index/function/from)。from 可以将数组、promise 或迭代器转换成 observable。

此例中，我们将一个生产流数组转化成了一个出版社，他会有序生产数组中的产品。然后在有人订阅后发出。

此时会有一个疑惑，为什么没有发送（`next`），也能收（`subscribe`）到东西。

因为：

```js
const playboy$ = from(["寻秦记", "射雕英雄传"]);
```

就相当于：

```js
const playboy$ = new Observable((subscriber) => {
    subscriber.next("寻秦记");
    subscriber.next("射雕英雄传");
});
```

这就是操作符的作用，简化了你的工作。

当然订阅与发送的顺序也一定要注意，普通的订阅是收不到订阅之前的数据的，如下例：

```js
const businessman$ = new Subject();
// 这个在subscrib之前，所以小明收不到
businessman$.next("寻秦记");
// 此时小明订阅，他错过了寻秦记
businessman$.subscribe({
    next: (v) => console.log("小明收到" + v),
});
// 只会发 射雕英雄传
businessman.next("射雕英雄传");

// logs:
// 小明收到射雕英雄传
```

Subject 本质还是一个 Observable， 但它既可以订阅，也可以发送，更加的灵活，更加多的适用场景。

## BehaviorSubject

[BehaviorSubject](https://rxjs.dev/guide/subject#behaviorsubject)是我们状态管理器的`关键性`零件。

我们先来看看 BehaviorSubject 是什么。

看名字就知道，它其实是一个`有能力`的 Subject。它的能力是：它永远会存一个`最新`的值。这个中间商发且只发最新的一期杂志。

```js
// BehaviorSubject 必须有初始值
// 中间商手中第一期（初始值）：寻秦记
const businessman$ = new BehaviorSubject("寻秦记");
// 小明订阅了
businessman$.subscribe({
    next: (v) => console.log("小明收到" + v),
});
// 中间商第二期：武侠周年版
businessman$.next("武侠周年版");
// 中间商第三期：射雕英雄传
businessman$.next("射雕英雄传");
// 小刚订阅了，只会收到最新的一期
businessman$.subscribe({
    next: (v) => console.log("小刚收到" + v),
});

// logs:
// 小明收到寻秦记
// 小明收到武侠周年版
// 小明收到射雕英雄传
// 小刚收到射雕英雄传
```

只要我订阅了，就总能接收到最新的数据，实时更新。这是不是超级契合我们前端的状态管理器，还是响应式的。

## 结语

RxJS 的前期铺垫工作就完成了，下一章我们就开始围绕`BehaviorSubject`来搭建我们的状态管理器。

## 其它

#### brass 状态管理器

按本系列文章思路实现的响应式状态管理器：

-   [brass](https://github.com/jaqen404/brass)
-   [vue-brass](https://github.com/jaqen404/vue-brass)
