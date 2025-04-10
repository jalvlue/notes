# 元素
HTML 由元素组成,
每一个元素包括标签和内容, 标签用于设置内容

![[Pasted image 20231227210414.png]]
例如:
`<p>...<\p>` 表示了一个段落元素

元素也可以有属性
![[Pasted image 20231227210554.png]]

属性包含了元素的额外信息, 为元素提供一个标识名称, 方便进一步使用(一般都使用双引号括起来)

元素也可以嵌套:
```html
<p>My cat is <strong>very</strong> grumpy.</p>
```
![[Pasted image 20231227210824.png]]
这里吧 `<strong>...</strong>` 嵌套在段落中

空元素: 
不含任何内容的元素
```html
<img src="images/firefox-icon.png" alt="My test image" />
```
其中 src 为图片地址, alt 为图片描述文字(无障碍访问)

完整例子:
```html
<!doctype html>
<html lang="en-US">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>My test page</title>
  </head>
  <body>
    <img src="images/firefox-icon.png" alt="My test image" />
  </body>
</html>
```

`<html></html>` 根元素
`<head></head>` header, 不向用户展示, 但对浏览器有一些作用
`<body></body>` 页面主体, 让用户看到的页面

# 标记文本
### 标题
```html
<h1>主标题</h1>
<h2>顶层标题</h2>
<h3>子标题</h3>
<h4>次子标题</h4>
```

# 段落
```html
<p>这是一个段落</p>
```

# 列表
html 中通常需要使用很多并列的元素, 因此可以使用列表
`<ul></ul>` 表示无序列表
`<ol></ol>` 表示有序列表

首先使用 `<ul>` 或 `<ol>` 包围
然后列表内并列元素使用 `<li>...elem...</li>` 表示

```html
<p>At Mozilla, we're a global community of</p>

<ul>
  <li>technologists</li>
  <li>thinkers</li>
  <li>builders</li>
</ul>

<p>working together…</p>
```

### 链接
`<a>...</a>`, a-anchor, 植入链接
使用 `herf` 属性, 指定链接的地址
```html
<a href="https://www.mozilla.org/zh-CN/about/manifesto/"
  >Mozilla Manifesto</a
>
```
herf - Hypertext REFerence

完整文件:
```html
<!doctype html>

<html lang="en-US">

  <head>

    <meta charset="utf-8" />

    <meta name="viewport" content="width=device-width" />

    <title>My test page</title>

  </head>

  <body>

    <h1>testtest</h1>

    <h2>testtest</h2>

    <p>My cat is <strong>very</strong> grumpy.</p>

    <img src="images/test-img.jpg" alt="My test image" />

    <p>very nice picture</p>

    <p>another paragraph</p>

    <ul>

      <li>test</li>

      <li>test</li>

      <li>test</li>

    </ul>

    <a href="https://www.mozilla.com">mozilla</a>

  </body>

</html>
```

