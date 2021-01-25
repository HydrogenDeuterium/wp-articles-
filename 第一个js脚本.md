# 第一个 js 脚本

我对 js 历来是深恶痛绝，主要反感的地方在于其糟糕的语言设计——主要是弱类型导致的大量意料外行为，这甚至逼迫大家都不得不用三等于。

比较常见的所谓“三位一体”这里不进一步重复，给大家分享一个更加令人意外的。

```JavaScript
// "[object Object][object Object]"
>>({}+{}).length
30
```

当然，这点儿问题还不至于影响 js 的地位，但说实话没有能够很好的解决这个问题（而且在我看起来并不难以解决）仍然让我丢光了对它的好感。

闲话已经说的够多了，让我们回到正题上来。

这个脚本的替代功能是这样：对于某个网页希望对其进行一定的操作之后打印。操作过程不是很麻烦，但是对于人手来说还是不合理的，因此写了这个脚本来减轻负担。

事实上我对网页（html，css，js）这部分基本上是外行，干啥基本靠云。最后（在恶劣的生产环境下） xjb 翻找各种傻逼教程，勉强凑出来了这玩意。

```javascript
let table=document.getElementsByTagName("table")[0];
//去除流水号
table.getElementsByTagName("th"）[0].parentNode.remove();
//去除成功时间
if (table.getElementsByClassName("rbox-label"）[0]!==undefined){
    table.getElementsByClassName("rbox-label")[0].parentNode.remove();}
```

先看脚本的第一部分，找到指定的表，然后清除第一行和指定行。
总的来说这里需要的是`getElementBy***`系列方法来找到需要的 html 标签，然后 `parentNode` 来获取父节点，`remove`来删除。

流水号这部分现在的实现是盲目删除第一个，理论上不太好，应该判断一下这一行的内容再说的，但是我对 html 的表格，js 的文本判断等内容都不熟悉，最重要的原因是现在这部分也可以用，所以暂且就先这样了。

后面去除成功时间的部分，蠢蠢的先判断存在再删除，理论上用 try 可能好，但是最后也懒得改了。

```js
//缩小
document.getElementsByTagName("body")[0].style.zoom = 0.80;
```

这部分是缩小网页，主要目的是使得打印的比例正确，不会多一页几乎没有内容的东西出来。code 已经给了我提示说，“zoom”已被弃用，并且会有改变显示比例的副作用，但是还是那句话，既然现在已经能用了，那就不要改了。

好吧，其实主要原因是翻了一下修改打印比例的部分，发现没法在几分钟之内看明白，所以就放弃了，反正现在又不是不能用。去看文档的话，应该也能找到替代 zoom 的方法，然而还是我懒。

```js
setTimeout((function(){document.getElementById("onlineService").remove();}),300);
```

这里是关闭网页上一个每次进去的在线帮助悬浮窗，不关闭会导致打印出一堆乱七八糟的东西，还非常浪费纸。一开始在 console 里测试的结果非常好，但是部署到 tampermonkey 上执行结果就一直报错说是找不到这个标签，改来改去也没想明白。

后来我突然领悟了，应该是这个部分实际上是有另外的脚本来执行生成，导致我自己写的脚本执行的时候这个部分还没有生成，因此导致这时候试图删除找不到内容。

在脚本管理器里几次调整了脚本执行时间都没有效果，后来想了想能不能在脚本里等一会等这玩意加载出来再执行，感觉不太行，因为这样可能会涉及异步操作或者多线程之类的问题。

俗话说得好，如果你有一个问题，当你考虑多线程（还是进程？这方面我没什么研究）的时候，很好，你有了两个问题了。但转念一想，既然网页的交互具有很高的实时性，应当不会容许一个地方卡住导致网页无法继续运行。

这也就意味着，js 的语言设计者应该不会蠢到在这个地方卡住（如果会的话，那么毫无疑问这玩意又会多一个黑点了）。找了一下有这个 `setTimeout()`函数，试了一下果然可以。

```js
//打印
//var newwin =window.open("");
//newwin.document.write(table.outerHTML);
//newwin.document.getElementsByTagName("body")[0].style.zoom=0.75;
//newwin.print();
//newwin.close();
```

最后的这部分打印，可以看到我全部注释掉了。原因是这会导致进入页面的时候自动打印，这经常是不必要的。

理论上来说更好的方案是在页面上生成一个按钮之类的东西来进行控制（就像我大一用来刷网课的超星脚本那样），但是首先我对这方面并没有研究，同时时间也不太容许我现场研究，所以关闭了这个自动打印的功能。

实际上来说如果使用了这个自动打印（实际原理是打开一个新窗口写入指定内容打印），因为只需要指定内容（即前文获取和处理 table）的部分，那么处理关闭在线帮助标签就是不必要的了，但是没有做成，所以最后只好注释掉。

最后放出完整的全部代码：

```js
(function() {

    'use strict';

    //这里删除的时候这个悬浮窗还没加载出来

    //console.log(document)

    //console.log("1,2,3,4");

    //console.log(document.getElementById("onlineService"));

    //console.log(document.getElementById("onlineService")==undefined);

    //删除在线服务悬浮窗。

    //document.getElementById("onlineService").remove();

    let table=document.getElementsByTagName("table")[0];

    //去除流水号

    table.getElementsByTagName("th")[0].parentNode.remove();

    //去除成功时间

    if (table.getElementsByClassName("rbox-label")[0]!==undefined){

        table.getElementsByClassName("rbox-label")[0].parentNode.remove();}

    //缩小

    document.getElementsByTagName("body")[0].style.zoom = 0.80;

    //打印

    //var newwin =window.open("");

    //newwin.document.write(table.outerHTML);

    //newwin.document.getElementsByTagName("body")[0].style.zoom=0.75;

    //newwin.print();

    //newwin.close();

})();

// 要在这里删除

setTimeout((function(){document.getElementById("onlineService").remove();}),300);
```
