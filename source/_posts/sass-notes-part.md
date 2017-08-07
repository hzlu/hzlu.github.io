---
title: Sass 语法小结
tags: [css]
categories:
- Sports
- foo
---

## `Sass` 与 `Scss` 的区别

- `scss` 使用花括号与分号分隔，`sass`要求严格的换行和缩进
- 使用 `@import` 的区别

~~~sass
// for sass
@import foo/bar
~~~

~~~scss
// for scss
@import "foo/bar";
~~~

## 变量

声明变量时，变量值也可以引用其他变量。

~~~scss
$highlight-color: #f90;
$highlight-border: 1px solid $highlight-color;
~~~

使用中划线 `-` 声明的变量可以通过下划线 `_` 的方式引用，反之亦然。

~~~scss
$link-color: blue;
a {
  color: $link_color;
}
~~~

## 选择器

### 父选择器标识符 `&`

~~~scss
#content aside {
  color: red;
  body.ie & {color: green}
}
/* 编译后 */
#content aside {color: red;}
body.ie #content aside {color: green;}
~~~

### 子元素选择器 `>`
### 兄弟相邻选择器 `+`
### 同级元素选择器 `~`

~~~scss
article {
  ~ article {...}
  > section {...}
  dl > {
    dt {...}
    dd {...}
  }
  nav + & {...}
}
/* 编译后 */
article ~ article {...}
article > section {...}
article dl > dt {...}
article dl > dd {...}
nav + article {...}
~~~

嵌套css属性

~~~scss
#footer {
  border: {
    width: 1px;
    color: #ccc;
    style: solid;
  }
}
// 指名例外规则
nav {
  border: 1px solid #ccc {
    left: 0px;
    right: 0px;
  }
}
~~~

## 使用混合器 `@mixin`

`@mixin` 标志符给一大段样式赋予一个名字，可以通过引用 `@include` 这个名字重用样式

~~~scss
@mixin rounded-corners {
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
}

notice {
  background-color: green;
  @include rounded-corners;
}
~~~

### 混合器传参

~~~scss
@mixin link-colors($normal, $hover, $visited) {
  color: $normal;
  &:hover {color: $hover;}
  &:visited {color: $visited;}
}

a {
  @include link-colors(blue, red, green);
}
~~~

### 自定义传参顺序

~~~scss
a {
  @include link-colors(
    $normal: blue,
    $visited: green,
    $hover: red
  );
}
~~~

### 默认参数

~~~scss
@mixin link-colors(
  $normal,
  $hover: $normal,
  $visited: $normal)
{
  color: $normal;
  &:hover {color: $hover;}
  &:visited {color: $visited;}
}

a {
  @include link-colors(blue);
}
~~~

## 使用继承 `@extend`

通过选择器继承样式

~~~scss
.error {
  border: 1px red;
  background-color: #fdd;
}
.seriousError {
  @extend .error;
  border-width: 3px;
}
~~~

**！注意，不仅会继承`.error`自身的所有样式，任何跟`.error`有关的组合选择器样式也会被继承**

~~~scss
// 以下样式也会应用到.seriousError a中
.error a {
  color: red;
  font-weight: 100;
}
// 以下样式也会应用到h1.seriousError中
h1.error {
  font-size: 1.2rem;
}
~~~

**！注意，如果一个选择器 `#foo` 继承了另一个选择器 `.bar.baz` ，那么只有 `.bar.baz` 或 `h1.bar.baz` 的样式会被继承，但是 `.bar` 或 `.baz` 下的样式则不会被继承，也就是说继承是看成一个`整体`进行的**


### 继承的要点

- 继承仅仅是重复选择器，而不会重复属性，所以使用继承比使用混合器生成的css体积更小。
- 继承遵从CSS层叠规则，根据权重决定应用哪个样式。

### 继承最佳实践

- 不要使用后代选择器去继承规则`.foo .bar {@extend .baz;}`
- 可以放心去继承有后代选择器修饰规则的选择器。

## 导入sass文件

### 导入局部文件 `@import`

约定文件名以下划线 `_` 开头，`@import` 一个局部文件可以不写下划线和扩展名。如： 命令 `@import "foo/bar";` 可以导入文件`foo/_bar.scss`。

### 默认变量值 `!default`

如果这个变量被声明赋值了，就用它声明的值，否则才使用默认值，与 `!important` 刚好相反。`$fancybox-width: 400px !default;`

### 嵌套导入

可以嵌套导入局部文件

~~~scss
// _blue-theme.scss文件
aside {
  background: blue;
  color: white;
}

// 在嵌套中导入
.blue-theme {@import "blue-theme";}

// 生成结果
.blue-theme {
  aside {
    background: blue;
    color: #fff;
  }
}
~~~

- css的 `@import` 规则：只有执行到`@import`时浏览器才会去下载css文件
- sass的 `@import` 规则：在生成css文件时就把相关文件导入到一个css文件中

在sass中使用css的import：

- 被导入文件以 `.css` 结尾
- 被导入文件的名字是一个URL地址
- 被导入文件的名字是css的 `url()` 的值
