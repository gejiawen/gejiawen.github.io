title: "CSS3多列布局"
date: 2015-04-14 00:15:38
categories: [CSS]
tags: [CSS拾遗系列, css3]

---

**注意**：由于CSS3中的部分内容尚未完全定稿，所以本文的部分可能会随时更新。

CSS3中新出现的多列布局([multi-column](http://www.w3.org/TR/css3-multicol/))是传统HTML网页中块状布局模式的有力扩充。这种新语法能够让WEB开发人员轻松的让文本呈现多列显示。本文将会详细阐述`column`属性的相关用法。

# 语法及兼容性

| 属性名 | 含义 | 可取值 | 说明 |
| :---  | :--- | :---  | :--- |
| `column-count` | 列数目 | - | - |
| `column-width` | 每列宽度 | - | 不一定会使用，浏览器将会根据`column-count`及`column-width`作自适应计算 |
| `column-rule` | 列分割线的样式 | - | 不占用真实空间，设置的方式跟`border`一致，可拆分为`column-rule-width`、`column-rule-style`、`column-rule-color` |
| `column-gap` | 各列之间的间隙宽度 | - | `column-gap`将占用真实的空间大小 |
| `column-span` | 允许一个元素的宽度跨越多列 | `none` 、`all` | - |


除了上述的几个常用的属性之外，还有

- `column-fill`属性，此属性用于标识分列的高度是否统一。不过博主在Chrome41上实验时貌似还不支持。
- `break-before`、`break-after`、`break-inside`属性，这三个属性是用于标识分列之间的行为的。这三个属性目前来说并不常用。

其兼容性如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3多列布局-001.png)

不出所料，IE9及其以下的浏览器是不支持的。而且`column`目前并不完全支持W3C的标准语法，需要针对不同的浏览器内核添加不同的前缀。（下面的示例中将默认使用`-webkit-`前缀）

# 示例

让我们先来看个简单的示例，

```html
<style>
    div {
        -webkit-column-count: 3;
        -webkit-column-width: 100px;
        -webkit-column-rule: 10px solid red;
        -webkit-column-gap: 20px;
    }
</style>

<div>
    blablabla.....
</div>
```

打开这个[demo](http://runjs.cn/detail/8ghuaw7o)，看一下效果。其效果如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3多列布局-002.png)

此时，我们如果注释掉css代码中的`column-count`或者`column-width`属性，得到的结果是不一样的。各位看客可在给出的demo链接中自行实验。

**结论**：其实`column-width`并不是必须的，浏览器会自动根据`column-count`和`column-width`计算出自适应的布局。上面的代码中我们设置`column-width`为100px，`column-count`为3列，得到的结果虽然是分成了3列，但是每一列的宽度并不是100px。

我们再来看看`column-span`属性。此属性目前只能有两个可取值：`none`或者`all`，标识某一元素要么不进行跨列要么就跨了所有的列。具体的效果可以参见这个[demo](http://runjs.cn/detail/ggiy1z6r)，其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3多列布局-003.png)

各位看官可自行将`column-span`属性变更成`none`或者`all`观察其效果变化。


# 简化语法

`column-count`及`column-width`属性可以合并起来使用，如下

```html
<style>
div {
    -webkit-columns: 3 15em
}
</style>

<div>
    blablabla......
</div>
```

其中`columns: 3 15em`与下面的写法是等效的，

```css
div {
    -webkit-column-count: 3;
    -webkit-column-width: 15em;
}
```

# 参考列表

- [http://www.w3.org/TR/css3-multicol/](http://www.w3.org/TR/css3-multicol/)


End! All rights reserved `@gejiawen`.


