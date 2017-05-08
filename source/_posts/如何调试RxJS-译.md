---
title: 如何调试RxJS(译)
date: 2017-05-05 16:53:13
tags: rxjs
---
[原文How to debug RxJS code](https://staltz.com/how-to-debug-rxjs-code.html)

开始学习RxJS时通常都会遇到一个共同的问题, 如何调试RxJS.

简短一点的答案是: 你需要大量依赖在纸上画Marble Diagram图并且在代码的operators后添加`.do(x => console.log(x))`

下面是长答案.
一旦你的代码中到处都是Observables并且RxJS订阅了控制流, 传统的断点调试方式就没那么有用了, 你的代码完全使用Observables编码, 断点调试工具对于使用了RxJS库的代码束手无策.

在责怪RxJS的不可调试之前, 我们先从宏观的角度来看看这个问题. 传统的调试是建立在面向过程的编码方式上的, 而不是其他编程泛型. 你的代码越是面向过程, 传统调试越有用. 使用RxJS的代码比普通的面向过程的js代码更加抽象, 使用传统的调试方式通常并不能帮我们真正解决问题. 同样用于调试Promises或者基于回调函数的代码. 与其说Promise不易调试, 不如直接使用专为Promise设计的调试工具, 比如[Chrome的Promise开发者工具](https://www.youtube.com/watch?v=o9c3U5_8tGY)(已经被移除)

这就是说, 当下主要有3种方式调试RxJS:
- [添加`.do(x => console.log(x))`来追踪log](#first)
- [画依赖图来跟踪据流](#second)
- [画`marble diagram`](#third)

调试的目标是使代码在执行过程中我们有一个清晰的思维模型, 下面的这三种技术能够帮我们实现.

<h2 id="first">追踪log</h2>

这是最基本的方式: 把事件发生时的数据流传递给`console.log`

声明一个Observable变量
``` javascript
var shortLowerCaseName$ = name$
.map(name => name.toLowerCase())
.filter(name => name.length < 5)
```

数据有两次变换, 我们可以在两次operators之间插入`.do(x => console.log(x))`
``` javascript
var shortLowerCaseName$ = name$
.map(name => name.toLowerCase())
.do(x => console.log(x))
.filter(name => name.length < 5);
```
`do`就像一次什么都没有做的map操作一样, 上面的代码等价于:
``` javascript
var shortLowerCaseName$ = name$
.map(name => name.toLowerCase())
.map(x => {
console.log(x);
return x;
})
.filter(name => name.length < 5);
```

和直接订阅`shortLowerCaseName$`不同
``` javascript
var shortLowerCaseName$ = name$
.map(name => name.toLowerCase())
.filter(name => name.length < 5);

shortLowerCaseName$.subscribe(name => console.log(x));
```

因为Observables在订阅之前是不会执行的(类似于generator), 订阅会触发operator的链式执行, 如果在`do`中使用了`log`但是没有订阅, `console.log`永远都不会执行.

所以`.do(x => console.log(x))`是一种无侵入的追踪技术, 不会改变代码本身的行为, 只是简单的反应中实际执行过程中的情况, 而订阅是侵入式的, 因为它会向operator链请求数据, 改变代码本身的运行, 特别是在调试.

注意`do()`也是一个operator: 它会返回一个新的Observable. 如果你只是想简单的用`do()`替换`subscribe()`, 没什么卵用, 因为你只能获取一个新的Observable但是并没有拉取数据.

``` javascript
var shortLowerCaseName$ = name$
.map(name => name.toLowerCase())
.filter(name => name.length < 5);

// This console.log will never happen!
shortLowerCaseName$.do(name => console.log(name));
```

想要获取`do()`的输出必须把它放在operator链中并且最后订阅数据流.

<h2 id="second">画依赖图来跟踪数据</h2>

假如有如下的代码:

``` javascript
var shortLowerCaseName$ = name$
.map(name => name.toLowerCase())
.filter(name => name.length < 5);

var bmi$ = weight$.combineLatest(height$, (weight, height) =>
Math.round(weight / (height * height * 0.0001))
);

var fullInfo$ = shortLowerCaseName$.combineLatest(bmi$);
```

我们顺着依赖关系(例如: shortLowerCaseName$ 依赖于 name$)可以建立依赖关系图, 如下:

![](https://staltz.com/img/debugging-dep-graph.png)

简单分析代码之后我们可以很容易的画出来这个图. 每个圈是一个Observable, 并且每个变量声明`var b$ = a$.flatMap(...)` 在图上就是一个箭头 `a$ --> b$`

Observable的依赖图应该是我们想到的第一个调试工具来检查代码中是否有bug, 对代码结构来说是非常有用的概括.
如果在纸上画出依赖图, 你可以在代码的关键位置添加`.do(x => console.log(x))`, 来检查Observable每一次操作的情况. 接下来代码执行时你可以通过依赖图看到数据流是如何传递的.

(冷信号, 每次订阅都从头开始, 没有状态, 热信号是所有的订阅者共享信号的状态)
要时刻清楚的意识到依赖图中的Observable也是有冷热的, 例如, $a有两个输出箭头, a$ --> b$ 和 a$ --> c$, a$可能使用了.share()变成了b$和c$的热信号. 尽管有时a$需要是个冷信号, 但是我们应该时刻清楚的判断一个信号到底是冷还是热.

通常依赖图已经足够指出数据流中的哪部分和我们所期望的不一致, 进一步的, 我们可以针对某个Observable放大来看, 使用marble diagram来调试它.

<h2 id="third">画`marble diagram`</h2>

大部分基础的RxJS的操作都有[marble diagrams](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html#instance-method-map), marble diagram把一个operator操作的Observable的输入(带点的箭头, 在上面)和输出(带点的箭头, 在下面)用图表示出来, 来解释operator是如何工作的. 因为operator只是一个返回新的Observable的函数, 所以你也可以绘制任何输入一个Observables输出一个Observable的函数.

![](http://reactivex.io/rxjs/img/map.png)

如果你不确定某个operator是如何运行的, 在这个operator的前后添加`.do(x => console.log(x))`, 然后执行代码, 绘制marble diagram, 然后很快就可以知道这个operator是如何工作了. 使用[RxMarbles](http://rxmarbles.com/), 拖动marbles来查看基础的operator是如何创建输出的Observable.

## 未来

如介绍中所说, 传统的调试在调试RxJS时并没有多大卵用, 并且你可能已经见过了在RxJS库中巨大的函数调用栈.

在下个版本(现在已经是了)会有大幅改进, 调用栈会浅很多, 来使传统方式调试RxJS代码变的可以忍受. 

像Promise得到了专有的调试工具(又没了...), RxJS的Observables也可以有专有的调试工具. 
注入行为到operator链上的所有的观察者, 来自Netflix公司的Ben Lesh 提到未来计划开发一个基于`lift()`的RxJS调试工具.

在不远的将来, 我们可能会看到实时渲染的依赖关系图或者实时的marble diagrams. 前者可能是使用静态分析或者使用lift架构.

在RxJS5的单元测试中, 大面积的使用marble diagrams(文本格式), 例如:

``` javascript
it.asDiagram('debounce')('should debounce events', () => {
  var e1 =   hot('-a--bc--d---|');
  var e2 =  cold( '--|         ');
  var expected = '---a---c--d-|';
  var result = e1.debounce(() => e2);
  expectObservable(result).toBe(expected);
});
```

这些基于文本的marble diagrams已经有用户在RxJS5中使用. 而且, PNG图也可以自动生成. 上述代码的PNG图如下:

![](https://staltz.com/img/debounce.png)

自动静态分析这些marble diagrams是自动渲染的第一步.

RxJS的调试工具依然很薄弱, 但是我们正在付出巨大的努力来获得强大的适合Observables的工具