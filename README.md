# d3-quadtree

[quadtree](https://en.wikipedia.org/wiki/Quadtree) 将二维平面递归的划分为正方形，并且可以进一步将正方形划分为四个等大小的正方形。每一个单独的点都放置在一个单独的 [node](#nodes) 中；重合的点由链表表示。四叉树可以加速各种空间操作，比如用来计算电荷力模型的 [Barnes–Hut approximation](https://en.wikipedia.org/wiki/Barnes–Hut_simulation) 算法, 碰撞检测以及搜索邻近点。

<a href="http://bl.ocks.org/mbostock/9078690"><img src="http://bl.ocks.org/mbostock/raw/9078690/thumbnail.png" width="202"></a>
<a href="http://bl.ocks.org/mbostock/4343214"><img src="http://bl.ocks.org/mbostock/raw/4343214/thumbnail.png" width="202"></a>

## Installing

`NPM` 安装： `npm install d3-quadtree`. 此外还可以下载 [latest release](https://github.com/d3/d3-quadtree/releases/latest). 可以直接从 [d3js.org](https://d3js.org) 以 [standalone library(单独标准库)](https://d3js.org/d3-quadtree.v1.min.js) 或作为 [D3 4.0](https://github.com/d3/d3) 的一部分载入。支持 `AMD`, `CommonJS` 以及基本的标签引入形式. 如果使用标签引入会暴露 `d3` 全局变量:

```html
<script src="https://d3js.org/d3-quadtree.v1.min.js"></script>
<script>

var quadtree = d3.quadtree();

</script>
```

[在浏览器中测试 d3-quadtree](https://tonicdev.com/npm/d3-quadtree)

## API Reference

<a name="quadtree" href="#quadtree">#</a> d3.<b>quadtree</b>([<i>data</i>[, <i>x</i>, <i>y</i>]])
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/quadtree.js#L14 "Source")

使用空的 [extent](#quadtree_extent) 和默认的 [*x*-](#quadtree_x) 和 [*y*-](#quadtree_y) 访问器创建一个新的、空的四叉树。如果指定了 *data* 则将指定的数组 [adds](#quadtree_addAll) 到四叉树中。等价于:

```js
var tree = d3.quadtree()
    .addAll(data);
```

如果同时指定了 *x* 和 *y* 则会在将数据添加到四叉树之前设置 [*x*-](#quadtree_x) 和 [*y*-](#quadtree_y) 访问器为指定的函数, 等价于:

```js
var tree = d3.quadtree()
    .x(x)
    .y(y)
    .addAll(data);
```

<a name="quadtree_x" href="#quadtree_x">#</a> <i>quadtree</i>.<b>x</b>([<i>x</i>]) [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/x.js "Source")

如果指定了 *x* 则将当前 *x*-坐标的访问器设置为指定的值并返回当前四叉树。如果没有指定则返回当前 *x* 访问器，默认为:

```js
function x(d) {
  return d[0];
}
```

*x* 访问器用来在将数据 [adding](#quadtree_add) 到四叉树或者从四叉树中 [removing](#quadtree_remove) 时读取 *x* 坐标的值。在 [finding](#quadtree_find) 插入到树中的坐标时也会使用到。因此 *x* 和 *y* 访问器必须在相同的条件下返回确定的值。

<a name="quadtree_y" href="#quadtree_y">#</a> <i>quadtree</i>.<b>y</b>([<i>y</i>])
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/y.js "Source")

如果指定了 *y* 则将当前 *y*-坐标的访问器设置为指定的值并返回当前四叉树。如果没有指定则返回当前 *y* 访问器，默认为:

```js
function y(d) {
  return d[1];
}
```

*y* 访问器用来在将数据 [adding](#quadtree_add) 到四叉树或者从四叉树中 [removing](#quadtree_remove) 时读取 *y* 坐标的值。在 [finding](#quadtree_find) 插入到树中的坐标时也会使用到。因此 *x* 和 *y* 访问器必须在相同的条件下返回确定的值。

<a name="quadtree_extent" href="#quadtree_extent">#</a> <i>quadtree</i>.<b>extent</b>([*extent*])
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/extent.js "Source")

如果指定了 *extent* 则扩展四叉树的范围以 [cover(覆盖)](#quadtree_cover) 指定的点 [[*x0*, *y0*], [*x1*, *y1*]] 并返回四叉树。如果没有指定 *extent* 则返回当前的区间 [[*x0*, *y0*], [*x1*, *y1*]]，其中 *x0* 和 *y0* 表示区间的下界而 *x1* 和 *y1* 表示区间的上界，如果之前没有指定区间则会返回 `undefined`。扩展四叉树的区间必须使用 [*quadtree*.cover](#quadtree_cover) 或 [*quadtree*.add](#quadtree_add) 方法。

<a name="quadtree_cover" href="#quadtree_cover">#</a> <i>quadtree</i>.<b>cover</b>(<i>x</i>, <i>y</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/cover.js "Source")

扩展四叉树的区间使其覆盖指定的点 ⟨*x*,*y*⟩ 并返回四叉树。如果四叉树的区间已经覆盖了指定的点则什么都不做。如果当前四叉树的区间没有覆盖指定的点则会重复加倍以覆盖指定的点。如果四叉树为空，则区间会被初始化为 [[⌊*x*⌋, ⌊*y*⌋], [⌈*x*⌉, ⌈*y*⌉]]。（四舍五入是必要的，这样的话如果区间在之后扩展翻倍时，不会因为浮点数而出现误差）。

<a name="quadtree_add" href="#quadtree_add">#</a> <i>quadtree</i>.<b>add</b>(<i>datum</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/add.js "Source")

将指定的数据 *datum* 添加到四叉树中，并使用当前 [*x*-](#quadtree_x) 和 [*y*-](#quadtree_y) 访问器计算出坐标 ⟨*x*,*y*⟩，返回四叉树。如果新的点在当前 [extent](#quadtree_extent) 之外，则会自动调用 [cover](#quadtree_cover) 方法以覆盖新添加的点。

<a name="quadtree_addAll" href="#quadtree_addAll">#</a> <i>quadtree</i>.<b>addAll</b>(<i>data</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/add.js#L50 "Source")

将指定点数组 *data* 添加到四叉树中，并使用当前 [*x*-](#quadtree_x) 和 [*y*-](#quadtree_y) 访问器为每个元素计算  ⟨*x*,*y*⟩  坐标，返回四叉树。这是一种重复使用 [*quadtree*.add](#quadtree_add) 添加多个点的快捷方式：

```js
for (var i = 0, n = data.length; i < n; ++i) {
  quadtree.add(data[i]);
}
```

但是，这种方法会产生更紧凑的四叉树，因为数据的区间是在添加数据之前计算的。

<a name="quadtree_remove" href="#quadtree_remove">#</a> <i>quadtree</i>.<b>remove</b>(<i>datum</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/remove.js "Source")

移除四叉树中指定的数据 *datum*，使用 [*x*-](#quadtree_x) 和 [*y*-](#quadtree_y) 访问器计算数据的坐标 ⟨*x*,*y*⟩。如果指定的 *datum* 不在当前四叉树中则什么都不做。

<a name="quadtree_removeAll" href="#quadtree_removeAll">#</a> <i>quadtree</i>.<b>removeAll</b>(<i>data</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/remove.js#L59 "Source")

… 清空四叉树

<a name="quadtree_copy" href="#quadtree_copy">#</a> <i>quadtree</i>.<b>copy</b>()

返回四叉树的拷贝。返回的四叉树中所有的 [nodes](#nodes) 都会拷贝当前四叉树中所有的节点。但是四叉树中的数据是引用共享的，而不是拷贝的。

<a name="quadtree_root" href="#quadtree_root">#</a> <i>quadtree</i>.<b>root</b>()
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/root.js "Source")

返回当前四叉树的根 [node](#nodes)

<a name="quadtree_data" href="#quadtree_data">#</a> <i>quadtree</i>.<b>data</b>()
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/data.js "Source")

返回当前四叉树中所有的数据数组。

<a name="quadtree_size" href="#quadtree_size">#</a> <i>quadtree</i>.<b>size</b>()
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/size.js "Source")

返回当前四叉树中所有的数据个数。

<a name="quadtree_find" href="#quadtree_find">#</a> <i>quadtree</i>.<b>find</b>(<i>x</i>, <i>y</i>[, <i>radius</i>])
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/find.js "Source")

返回距离指定点 ⟨*x*,*y*⟩ 最近的数据，可以指定搜索半径  *radius*. 如果没有指定 *radius* 则默认为无穷大。如果在指定的搜索区域内没有找到数据则返回 `undefined`.

<a name="quadtree_visit" href="#quadtree_visit">#</a> <i>quadtree</i>.<b>visit</b>(<i>callback</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/visit.js "Source")

以前序遍历的方式遍历所有的 [node](#nodes)，并传入参数 *node*, *x0*, *y0*, *x1*, *y1* 调用指定的 *callback*，其中 *node* 为当前被遍历到的节点，⟨*x0*, *y0*⟩ 是当前节点的下界而 ⟨*x1*, *y1*⟩ 是当前节点的上界，返回四叉树。（假设 *x* 正方向为右，*y* 正方向为下，在 `Canvas` 和 `SVG` 中 ⟨*x0*, *y0*⟩ 为左上角坐标，⟨*x1*, *y1*⟩ 为右下角坐标，但是坐标系统也可以是其他形式，所以更一般的情况应该为 *x0* <= *x1* 并且 *y0* <= *y1*）

如果 *callback* 返回真值，则当前节点的子节点将不会被遍历，否则当前节点的子节点都会被遍历到。这在遍历四叉树中的部分分区时很有用，例如 [Barnes–Hut approximation](https://en.wikipedia.org/wiki/Barnes–Hut_simulation)。请注意，子象限总是按照兄弟节点的顺序遍历: 左上角、右上角、左下角、右下角。在 [search](#quadtree_find) 等情形下，按照指定的顺序遍历兄弟节点可能会更快。

<a name="quadtree_visitAfter" href="#quadtree_visitAfter">#</a> <i>quadtree</i>.<b>visitAfter</b>(<i>callback</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/visitAfter.js "Source")

以后序遍历的方式遍历所有的 [node](#nodes)，并传入参数 *node*, *x0*, *y0*, *x1*, *y1* 调用指定的 *callback*，其中 *node* 为当前被遍历到的节点，⟨*x0*, *y0*⟩ 是当前节点的下界而 ⟨*x1*, *y1*⟩ 是当前节点的上界，返回四叉树。（假设 *x* 正方向为右，*y* 正方向为下，在 `Canvas` 和 `SVG` 中 ⟨*x0*, *y0*⟩ 为左上角坐标，⟨*x1*, *y1*⟩ 为右下角坐标，但是坐标系统也可以是其他形式，所以更一般的情况应该为 *x0* <= *x1* 并且 *y0* <= *y1*）

### Nodes

四叉树的内部节点以从左到右、从上到下的顺序表示为四个元素数组:

* `0` - 左上象限，如果有.
* `1` - 右上象限，如果有.
* `2` - 左下象限，如果有.
* `3` - 右下象限，如果有.

如果子象限是空的，则它可能是未定义的。

叶节点表示为具有以下属性的对象:

* `data` - 与当前点关联的数据，也就是传递给 [*quadtree*.add](#quadtree_add) 的数据
* `next` - 当前叶节点的下一个数据，如果有的话.

`length` 属性可以用来区分叶节点和内部节点：如果是叶节点则为 `undefined`，如果是 `4` 则为内部节点。例如获取所有的叶节点的数据：

```js
if (!node.length) do console.log(node.data); while (node = node.next);
```

四叉树中的点的 *x*- 和 *y*-坐标 **必须不能修改**。更新一个点的位置需要首先 [remove](#quadtree_remove) 点然后使用新的位置重新 [add](#quadtree_add) 到四叉树中。或者，可以完全丢弃现有的四叉树，重新创建一个新的四叉树，如果大多数点的位置改变，则重新创建可能会更高效。

