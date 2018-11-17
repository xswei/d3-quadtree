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

Adds the specified array of *data* to the quadtree, deriving each element’s coordinates ⟨*x*,*y*⟩ using the current [*x*-](#quadtree_x) and [*y*-](#quadtree_y)accessors, and return this quadtree. This is approximately equivalent to calling [*quadtree*.add](#quadtree_add) repeatedly:

```js
for (var i = 0, n = data.length; i < n; ++i) {
  quadtree.add(data[i]);
}
```

However, this method results in a more compact quadtree because the extent of the *data* is computed first before adding the data.

<a name="quadtree_remove" href="#quadtree_remove">#</a> <i>quadtree</i>.<b>remove</b>(<i>datum</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/remove.js "Source")

Removes the specified *datum* to the quadtree, deriving its coordinates ⟨*x*,*y*⟩ using the current [*x*-](#quadtree_x) and [*y*-](#quadtree_y)accessors, and returns the quadtree. If the specified *datum* does not exist in this quadtree, this method does nothing.

<a name="quadtree_removeAll" href="#quadtree_removeAll">#</a> <i>quadtree</i>.<b>removeAll</b>(<i>data</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/remove.js#L59 "Source")

…

<a name="quadtree_copy" href="#quadtree_copy">#</a> <i>quadtree</i>.<b>copy</b>()

Returns a copy of the quadtree. All [nodes](#nodes) in the returned quadtree are identical copies of the corresponding node in the quadtree; however, any data in the quadtree is shared by reference and not copied.

<a name="quadtree_root" href="#quadtree_root">#</a> <i>quadtree</i>.<b>root</b>()
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/root.js "Source")

Returns the root [node](#nodes) of the quadtree.

<a name="quadtree_data" href="#quadtree_data">#</a> <i>quadtree</i>.<b>data</b>()
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/data.js "Source")

Returns an array of all data in the quadtree.

<a name="quadtree_size" href="#quadtree_size">#</a> <i>quadtree</i>.<b>size</b>()
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/size.js "Source")

Returns the total number of data in the quadtree.

<a name="quadtree_find" href="#quadtree_find">#</a> <i>quadtree</i>.<b>find</b>(<i>x</i>, <i>y</i>[, <i>radius</i>])
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/find.js "Source")

Returns the datum closest to the position ⟨*x*,*y*⟩ with the given search *radius*. If *radius* is not specified, it defaults to infinity. If there is no datum within the search area, returns undefined.

<a name="quadtree_visit" href="#quadtree_visit">#</a> <i>quadtree</i>.<b>visit</b>(<i>callback</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/visit.js "Source")

Visits each [node](#nodes) in the quadtree in pre-order traversal, invoking the specified *callback* with arguments *node*, *x0*, *y0*, *x1*, *y1* for each node, where *node* is the node being visited, ⟨*x0*, *y0*⟩ are the lower bounds of the node, and ⟨*x1*, *y1*⟩ are the upper bounds, and returns the quadtree. (Assuming that positive *x* is right and positive *y* is down, as is typically the case in Canvas and SVG, ⟨*x0*, *y0*⟩ is the top-left corner and ⟨*x1*, *y1*⟩ is the lower-right corner; however, the coordinate system is arbitrary, so more formally *x0* <= *x1* and *y0* <= *y1*.)

If the *callback* returns true for a given node, then the children of that node are not visited; otherwise, all child nodes are visited. This can be used to quickly visit only parts of the tree, for example when using the [Barnes–Hut approximation](https://en.wikipedia.org/wiki/Barnes–Hut_simulation). Note, however, that child quadrants are always visited in sibling order: top-left, top-right, bottom-left, bottom-right. In cases such as [search](#quadtree_find), visiting siblings in a specific order may be faster.

<a name="quadtree_visitAfter" href="#quadtree_visitAfter">#</a> <i>quadtree</i>.<b>visitAfter</b>(<i>callback</i>)
 [<源码>](https://github.com/d3/d3-quadtree/blob/master/src/visitAfter.js "Source")

Visits each [node](#nodes) in the quadtree in post-order traversal, invoking the specified *callback* with arguments *node*, *x0*, *y0*, *x1*, *y1* for each node, where *node* is the node being visited, ⟨*x0*, *y0*⟩ are the lower bounds of the node, and ⟨*x1*, *y1*⟩ are the upper bounds, and returns the quadtree. (Assuming that positive *x* is right and positive *y* is down, as is typically the case in Canvas and SVG, ⟨*x0*, *y0*⟩ is the top-left corner and ⟨*x1*, *y1*⟩ is the lower-right corner; however, the coordinate system is arbitrary, so more formally *x0* <= *x1* and *y0* <= *y1*.) Returns *root*.

### Nodes

Internal nodes of the quadtree are represented as four-element arrays in left-to-right, top-to-bottom order:

* `0` - the top-left quadrant, if any.
* `1` - the top-right quadrant, if any.
* `2` - the bottom-left quadrant, if any.
* `3` - the bottom-right quadrant, if any.

A child quadrant may be undefined if it is empty.

Leaf nodes are represented as objects with the following properties:

* `data` - the data associated with this point, as passed to [*quadtree*.add](#quadtree_add).
* `next` - the next datum in this leaf, if any.

The `length` property may be used to distinguish leaf nodes from internal nodes: it is undefined for leaf nodes, and 4 for internal nodes. For example, to iterate over all data in a leaf node:

```js
if (!node.length) do console.log(node.data); while (node = node.next);
```

The point’s *x*- and *y*-coordinates **must not be modified** while the point is in the quadtree. To update a point’s position, [remove](#quadtree_remove) the point and then re-[add](#quadtree_add) it to the quadtree at the new position. Alternatively, you may discard the existing quadtree entirely and create a new one from scratch; this may be more efficient if many of the points have moved.
