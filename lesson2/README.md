---
theme: juejin
---

本章知识点：

1. 什么是 pipe
2. 什么是 [map](https://rxjs.dev/api/index/function/map)
3. 什么是 [mergeWith](https://rxjs.dev/api/index/function/mergeWith)
4. 什么是 [fromEvent](https://rxjs.dev/api/index/function/fromEvent)

## 回顾

```js
// 中间商手中第一期：寻秦记
const businessman$ = new BehaviorSubject("寻秦记");
// 小明订阅了
businessman$.subscribe({
    next: (v) => console.log("小明收到" + v),
});
// 中间商第二期：武侠周年版
businessman.next("武侠周年版");
// 中间商第三期：射雕英雄传
businessman.next("射雕英雄传");
```

回顾一下上一章的 BehaviorSubject，我们把它稍微改造一下，就是一个响应式状态管理器的雏形了。

```js
// 数据初始值
const state = {
    book: "寻秦记",
};
// 设置状态管理器的初始值
const state$ = new BehaviorSubject(state);
// 订阅state$，subscribe里可以改成只接收一个next的方法
state$.subscribe((newState) => {
    // 每当状态有改变，就会在这里输出
    // 响应式的拿到最新的state
    console.log(newState);
});
state$.next({ book: "射雕英雄传" });

// logs:
// {book: "寻秦记"}
// {book: "射雕英雄传"}
```

接下来我们一步步改进这个简陋的状态管理器。
状态管理是要遵循特定的约定去变更状态，那我们就要遵循统一约定定义方法，再通过这些方法去改变状态，从而让状态的变化可预测。这些方法就像一个个加工站点，初始出厂的产品会通过这些站点，加工成更精美丰富的产品。

先建立一个加工站点，把书本包装起来，让《寻秦记》变成《精装寻秦记》：

```js
const parcel = (state) => {
    const newState = state;
    newState.book = "精装" + newState.book;
    return newState;
};
```

那么如何把这个站点加入生产流呢？这就要用到一个新的辅助方法，`pipe`，和一个新的 Operator，`map`。

## map

先看一下 js 数组中的 map：

```js
const list = [{ book: "寻秦记" }, { book: "金鳞岂是池中物" }].map((item) => {
    return { book: "精装" + item.book };
});
console.log(list);

// logs:
// [{book: '精装寻秦记'},  {book: '精装金鳞岂是池中物'}]
```

数组中的 map 方法，就是按照原始数组元素顺序依次处理元素，并返回一个新的数组。
而 RxJS 中的 map 方法，也非常类似，就是把源值传递给转化函数以获得相应的输出值。

```js
// 精装生产流的普通书入口
const entrance$ = new Subject();
// 把从入口进来的普通书打包成精装
const parcel$ = entrance$.pipe(
    map((item) => {
        return { book: "精装" + item.book };
    })
);
// 小明订阅了精装版
parcel$.subscribe((item) => {
    console.log("小明收到了" + item.book);
});
// 小刚订阅了普通版，在普通书入口就把书拿走了
entrance$.subscribe((item) => {
    console.log("小刚收到了" + item.book);
});
// 往精装生产流的普通书入口发送一本寻秦记
entrance$.next({ book: "寻秦记" });
// 往精装生产流的普通书入口发送一本射雕英雄传
entrance$.next({ book: "射雕英雄传" });

// logs:
// 小明收到了精装寻秦记
// 小刚收到了寻秦记
// 小明收到了精装射雕英雄传
// 小刚收到了射雕英雄传
```

上例中的`pipe`就是一个管道，他把所有的 operators 包裹起来。在这个管道中，商品会按顺序经过这些`operator`，然后返回一个 Observabel。

```js
const parcel$ = playboy$.pipe(
	operator1(),
	operator2(),
	operator3(),
	...
)
```

## mergeWith

现在我们精装加工站点已经建好了，接下来就是把精装生产流汇入我们的主生产流。
这就要用到`mergeWith`这个操作符，`mergeWith`就是将两个流合并成一个。
在这里还要介绍一个操作符`fromEvent`，这个操作符可以将`事件（event）`转化为流，每当这个事件被触发时，都会往这个流里发射一个信息。

```js
// 将单击事件转化为流，并用map把每次发射出的信息转化为字符串’click‘
const clicks$ = fromEvent(document, "click").pipe(map(() => "click"));
// 将鼠标移动事件转化为流，并用map把每次发射出的信息转化为字符串’mousemove‘
const mousemoves$ = fromEvent(document, "mousemove").pipe(
    map(() => "mousemove")
);
// 将双击事件转化为流，并用map把每次发射出的信息转化为字符串’dblclick‘
const dblclicks$ = fromEvent(document, "dblclick").pipe(map(() => "dblclick"));
// 把单击流 和 双击流 用 mergeWith 并入 移动流
mousemoves$
    .pipe(mergeWith(clicks$, dblclicks$))
    .subscribe((x) => console.log(x));
```

只需要订阅一次，就可以订阅这三个流，移动会输出，单击会输出，双击也会输出：

```
mousemove
mousemove
mousemove
click
dblclick
```

学会了`mergeWith`，我们再新建两个加工站点，一起合并到主生产流中去。

```js
// 初始状态
const state = {
    before: "",
    book: "",
    after: "",
};
const parcelEntrance$ = new Subject();
const parcel$ = parcelEntrance$.pipe(
    map((item) => {
        return { before: "精装", book: item, after: "" };
    })
);
// 新建一个合集生产流
const collectionEntrance$ = new Subject();
const collection$ = collectionEntrance$.pipe(
    map((item) => {
        return { before: "", book: item, after: "大合集" };
    })
);
// 把 精装生产流 和 合集生产流 合并到主生产流中去
const state$ = new BehaviorSubject(state).pipe(mergeWith(parcel$, collection$));
state$.subscribe((newState) => {
    console.log(newState.before + newState.book + newState.after);
});
// 往精装生产流的普通书入口发送一本鬼吹灯
parcelEntrance$.next("鬼吹灯");
// 往合集生产流的普通书入口发送一本鬼吹灯
collectionEntrance$.next("鬼吹灯");

// logs:
// 精装鬼吹灯
// 鬼吹灯大合集
```

## 结语

好的，我们现在已经可以用不同的方法来处理状态，我们的状态管理器又更进了一步。
但现在要解决一个问题，我们的出版社只能出版：精装 A，精装 B，A 大合集，B 大合集。
但是生产不了`精装A大合集`，精装的大合集没有，肯定是达不到我们需求的。
下一章，我们将重点解决这个问题。出版社将迎来`精装鬼吹灯大合集`。

## 其它

#### 文章和源码

[本系列 github 地址](https://github.com/jaqen404/ZeroToOneStateManager)

#### brass 状态管理器

按本系列文章思路实现的响应式状态管理器：

-   [brass](https://github.com/jaqen404/brass)
-   [vue-brass](https://github.com/jaqen404/vue-brass)
