# HTML 5

## HTML常用标签

HTML 标签的大小写无关的，推荐使用小写。

**双标签**

```html
<html></html>
<head></head>
<title></title>
<body></body>
<h1></h1>
<p></p>
<div></div>
<span></span>
<a></a>
<ul></ul>
```

双标签中间的内容称为元素。

**单标签**

```html
<br />
<!--换行-->
<hr />
<!--水平分隔线-->
<meta />
<img />
```

**标签介绍**

+ h标签，表示标题，`<h1>一级标题</h1>`。

+ p标签，表示段落

+ 图片标签，`<img src="http://..."/>`

+ a标签，表示超链接，`<a href="http://...">超链接</a>`

+ div标签，表示块元素，可以设置宽高，浏览器自动在`<div>`和`</div>`所标记的区域前后自动放置一个换行符。

  ```html
  <div style="width:200px;height:200px;background:pink">
    块级元素
  </div>
  ```

+ `<span>`标签，通常作为文本容器，是内联容器，在一行内显示，不会自动添加换行符。

+ 在浏览器显示页面时，浏览器会移除源代码中多余的空格和空行，所有连续的空格或空行都会被认为是一个空格。换行标签`<br/>`，空格标签`&nbsp;`

+ `<hr/>`标签，水平分割线

+ 列表标签，`<ul>`无序标签，`<ol>`有序标签，`<li>`列表项

  ```html
  <p>无序列表</p>
  <ul>
    <li>列表项1</li>
    <li>列表项2</li>
  </ul>
  
  <p>有序列表</p>
  <ol>
    <li>列表项1</li>
    <li>列表项2</li>
  </ol>
  ```

  列表标签可以设置`type`属性，定义标签符号的类型。

  自定义标签`<dl>`.

**HTML样式**

`style`属性可以为所有html标签定义样式。

+ 背景颜色

  ```html
  <html>
    <body>
      <p style="background-color:red">实验楼</p>
    </body>
  </html>
  ```

+ 字体、颜色、尺寸

  ```html
  <html>
    <body>
      <p style="font-family:arial;color:green;font-size:50px;">实验楼</p>
    </body>
  </html>
  ```

+ 文本对齐

  ```html
  <html>
    <body>
      <h1 style="text-align:center">实验楼</h1>
    </body>
  </html>
  ```

## HTML表格

表格由`<table>`标签定义，表格行由`<tr>`标签定义，表格单元格由标签`<td>`标签定义。

`border`属性定义边框，`width`属性定义表格宽度，`height`属性定义表格高度

`<caption>`标签定义表格标题

`<th>`标签定义表格表头，常用于列名字

```html
<table border="1">
  <caption> table title </caption>
  <tr>
    <th>first column</th>
    <th>second column</th>
  </tr>
  <tr>
    <td>第一行第一列</td>
    <td>第一行第二列</td>
  </tr>
  <tr>
    <td>第二行第一列</td>
    <td>第二行第二列</td>
  </tr>
</table>
```

**单元格跨行和跨列**

```html
<table border="1" width="300px" height="150px">
  <h3>单元跨两行</h3>
  <tr>
    <th>姓名</th>
    <th colspan="2">电话</th>
  </tr>
  <tr>
    <td>张三</td>
    <td>344 22 112</td>
    <td>211 32 123</td>
  </tr>
  <tr>
    <td>李四</td>
    <td>433 31 212</td>
    <td>234 21 654</td>
  </tr>
</table>

<table border="1" cellspacing="0">
  <h3>单元跨两列</h3>
  <tr>
    <th>姓名</th>
    <td>张三</td>
  </tr>
  <tr>
    <th rowspan="2">电话</th>
    <td>344 22 112</td>
  </tr>
  <tr>
    <td>234 21 654</td>
  </tr>
</table>
```

## HTML表单

HTML 表单是一个包含表单元素的区域，是网页中提供的一种交互式手段操作，用于搜集不同类型的用户输入。

`<form>`标签用于创建HTML表单，常见的表单格式为：`<form name="" method="" action=""></form>`

+ name：定义表单的名字；
+ method：定义表单结果从浏览器传送到服务器的方式，默认参数为`GET`；
+ action：发送数据要去的地址，如果没有提供此属性或`action="#"`则将数据发送到包含表单页面的URL。

表单包含多种表单元素，如文字字段、单选按钮、复选框、按钮等等。

**文字字段**

用户可以在文字字段中输入字符或者是单行文本。语法是：

```html
<input
  type="text"
  name="控件名称"
  value="文字字段的默认取值"
  size="控件的长度"
  maxlength="最长字符数"
/>
```

上述参数中只有`type`是必需的，其他参数是可选的。

```html
<form name="formBox" method="post" action="">
  姓名: <input type="text" name="name" size="20" /><br />
  年龄: <input type="text" name="age" size="40" value="10" maxlength="3" />
</form>
```

密码输入框是一种特殊的文字字段，输入的内容以`*`代替。

`<input type="password" name="pwd" />`

**单选按钮**

单选按钮可以使用户从选择列表中选择一个选项，几个单选按钮可以连接在一起，只需要把它们的 `name` 值设置为相同的。同一组中只有一个按钮可以同时被选。如果没有选中任何一个，那么整个单选按钮池就被认为处于未知状态，且不会随表单提交。

```html
<form name="formBox" method="post" action="">
  <input type="radio" name="sex" value="male" checked />男<br />
  <input type="radio" name="sex" value="female" />女
</form>
```

**复选框**

复选框可以让用户从一个选项列表中选择超出一个的选项。

```html
<form name="formBox" method="post" action="">
  <input type="checkbox" name="music" checked />音乐<br />
  <input type="checkbox" name="art" />美术<br />
  <input type="checkbox" name="math" />数学<br />
</form>
```

**按钮**

HTML 表单中，有三种按钮：提交按钮，重置按钮，匿名按钮。我们可以使用 `<input>` 元素或者 `<button>` 元素来创建一个按钮。`type` 属性的值指定显示什么类型的按钮。

提交按钮`type="submit"`，将表单数据发送给服务器;重置按钮`type="reset"`，用来清除用户在表单中的输入内容；匿名按钮`type="button"`

```html
<form name="formBox" method="post" action="">
  <input type="text" value="输入的内容" />
  <button type="submit">
    This a submit button
  </button>

  <!--or-->

  <input type="reset" value="This is a reset button" />
</form>
```

**下拉菜单**

下拉菜单能够节省页面空间，正常状态下显示一个选项，单击展开所以选项。

```html
<form name="formBox" method="post" action="">
  <select name="select">
    <option value="成都">成都</option>
    <option value="广州">广州 </option>
    <option value="四川">四川</option>
    <option value="上海">上海</option>
  </select>
</form>
```

注意：下拉菜单的宽度是由 `` 标记中包含的最长文本的宽度决定的。

**文本域**

当用户想要填入多行文本时，就应该使用文本域而不是文本字段。文本域使用 `textarea` 标记。

```html
<form name="formBox" method="post" action="">
  留下您的联系方式：
  <textarea name="textarea" cols="35" rows="5"></textarea>
</form>
```

## HTML图像与框架

### 图像

在 HTML 中，图像由 `` 标签定义。语法为：

```html
<img
  src="http://labfile.oss.aliyuncs.com/courses/1236/coder.jpg"
  alt="程序员"
  title="你好啊"
  width="700px"
  height="600px"
/>
```

`<img>`是单标签，只包含属性，没有闭合标签。`src`的值是图像文件的URL；`alt`的值是图片的替代文本，当图片未正常显示时显示的文本内容；`title`设置鼠标悬停时显示的内容;`width\height`设置图片的宽和高。

相对路径是以引用文件的网页的所在位置为参考基础。

### 框架

使用框架，你可以在同一个浏览器窗口中显示不止一个页面。在框架中显示其他页面。

`<iframe src="https://www.shiyanlou.com/" width="400" height="400"></iframe>`

`frameborder` 属性用于定义 iframe 表示是否显示边框。设置属性值为 "0" 移除 iframe 的边框。

iframe 可以显示一个超链接`<a href=“”>`的页面，超链接的`target`属性的值必须与 iframe 的`name`属性的值相同。

```html
<p><a href="https://www.shiyanlou.com/" target="shiyanlou">实验楼</a></p>
<iframe width="400" height="400" name="shiyanlou"></iframe>
```

## HTML5简介

**`<datalist>`**

`<datalist>`是html5的新标签，`<datalist> `标签规定了 `<input>` 元素可能的选项列表。请使用 `<input> `元素的 list 属性来绑定` <datalist> `元素。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title></title>
  </head>
  <body>
    <label for="myColor">What's your favorite color?</label>
    <input type="text" name="myColor" id="myColor" list="mySuggestion" />
    <datalist id="mySuggestion">
      <option value="black"> </option>
      <option value="blue"> </option>
      <option value="green"> </option>
      <option value="red"> </option>
      <option value="white"> </option>
      <option value="yellow"> </option>
    </datalist>
  </body>
</html>
```

**autocomplete属性**

`autocomplete` 属性规定表单是否应该启用自动完成功能：自动完成允许浏览器预测对字段的输入，当用户在字段开始键入时，浏览器基于之前键入过的值，应该显示出在字段中填写的选项。当 `autocomplete` 属性值为 `on` 时表示启用自动完成功能，为 `off` 时表示关闭。`autocomplete` 属性适用于 ``，以及下面的 `` 类型：`text`, `search`, `url`, `telephone`, `email`, `password`, `datepickers`, `range` 以及 `color`。

```html
<body>
  <form action="#" method="get" autocomplete="on">
    Name:<input type="text" name="name" /><br />
    E-mail: <input type="email" name="email" autocomplete="off" /><br />
    <input type="submit" value="提交" />
  </form>
</body>
```

**autofocus属性**

`autofocus` 属性规定在页面加载时，域自动地获得焦点。适用于所有 `<input>` 标签的类型。

```html
<body>
  <form action="#" method="get">
    Name：<input type="text" name="user_name" autofocus="autofocus" /><br />
    <input type="submit" value="提交" />
  </form>
</body>
```

**form属性**

`form` 属性规定输入域所属的一个或多个表单。`form` 属性适用于所有 `` 标签的类型。`form` 属性必须引用所属表单的 `id`。

```html
<body>
  <form action="#" method="get" id="user_form">
    First name:<input type="text" name="fname" />
    <input type="submit" value="提交" />
  </form>

  <p>
    下面的输入域在 form 元素之外，但仍然是表单的一部分， 也就是说提交按钮会把
    first name 和 last name的值都提交。
  </p>

  Last name: <input type="text" name="lname" form="user_form" />
</body>
```

**multiple属性**

`multiple` 属性规定输入域中可选择多个值，适用于以下类型的 `<input>` 标签：`email` 和 `file`。（file属性是上传文件）

```html
<body>
  <form action="#" method="get" id="user_form">
    选择文件: <input type="file" name="file" multiple="multiple" />
    <input type="submit" value="提交" />
  </form>
</body>
```

**pattern属性**

`pattern` 属性规定用于验证 `input` 域的模式（pattern）。模式（pattern） 是正则表达式。

```html
<body>
  <form action="#" method="get">
    Name:
    <input type="text" name="name" pattern="[A-z]{4}" title="请输入四个字母" />
    <input type="submit" value="提交" />
  </form>
</body>
```

**placeholder 属性**

`placeholder` 属性提供一种提示（hint），描述输入域所期待的值。适用于以下类型的 `` 标签：`text`, `search`, `url`, `telephone`, `email` 以及 `password`。提示（hint）会在输入域为空时显示出现，会在输入域获得焦点时消失。

```html
<body>
  <form action="#" method="get">
    <input type="search" name="user_search" placeholder="实验楼" />
    <input type="submit" value="提交" />
  </form>
</body>
```

**required属性**

`required` 属性规定必须在提交之前填写输入域（不能为空）。适用于以下类型的 `` 标签：`text`, `search`, `url`, `telephone`, `email`, `password`, `date pickers`, `number`, `checkbox`, `radio` 以及 `file`。

```html
<body>
  <form action="#" method="get">
    <input type="search" name="user_search" required="required" />
    <input type="submit" value="提交" />
  </form>
</body>
```

**input输入类型**

+ email，用于应该包含 e-mail 地址的输入域。在提交表单时，会自动验证 email 域的值。

+ url，

+ number，于应该包含数值的输入域。属性 `max` 设定允许输入的最大值，属性 `min` 设定允许输入的最小值，属性 `value` 设定默认值，属性 `step` 设定合法的数字间隔（比如 `step` 的值为 `2`，则合法的数字为 `-2`,`0`,`2`,`4` 等）。

+ range，用于应该包含一定范围内数字值的输入域。`range` 类型显示为滑动条。同样的 `range` 也有 `max`，`min`，`value`，`step` 属性与上面所讲的 `number` 中的一致。

+ date pickers，html5拥有多个日期和时间的输入类型：date month week time datetime datetime-local

  ```html
  <body>
    <form action="#" method="get">
      Date: <input type="date" name="user_date" /><br />
      <input type="submit" value="提交" />
      <br /><br />
      Month: <input type="month" name="user_month" /><br />
      <input type="submit" value="提交" />
    </form>
  </body>
  ```

+ 

