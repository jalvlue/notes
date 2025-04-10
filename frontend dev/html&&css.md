# web concepts
- URL: 统一资源定位器, 互连网上的每一个资源都会有自己独一无二的定位器, 通常是一个 http 链接, 客户端可以通过请求这个 http 链接来获得该资源
- HTML: 一种标记语言, 用于向浏览器阐述应该如何组织网页, H 代表 hypertext, 超文本, 也就是 HTML 中不只有文本, 可能会有指向其他资源的链接
- 访问网站: 首先会请求该页面的主 HTML, 然后再请求 HTML 上的超链接, 所有的东西都拿到之后, 浏览器可以根据 HTML 将这些东西渲染成网页 ![[Pasted image 20241024214133.png]] ![[Pasted image 20241024214146.png]]


# HTML
- HTML 由一个个元素组成
- `DOCTYPE`
- `html`
- `head`
- `title`
- `body`
- `img`
- `p`

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My first web page</title>
  </head>

  <body>
    <img src="images/pretty.jpg" />
    <p>@pretty</p>
    <p>I love you</p>
  </body>
</html>

```
![[Pasted image 20241024215922.png]]

# CSS
- CSS 就是加在 HTML 元素上的样式, 可以通过元素标签和属性加
- `width`
- `border-radius`
- `float`
- `margin-right`
```html
<!DOCTYPE html>
<html>
  <head>
    <title>My first web page</title>
    <style>
      img {
        width: 100px;
        border-radius: 50px;
        float: left;
        margin-right: 10px;
      }

      p.username {
        font-weight: bold;
      }
    </style>
  </head>

  <body>
    <img src="images/pretty.jpg" />
    <p class="username">@pretty</p>
    <p>I love you</p>
  </body>
</html>

```
![[Pasted image 20241024221029.png]]


# essentials
- `meta`: 展示网页的元信息, 通常是网页的基本设置
- 在最佳实践中, `HTML` 只是用来组织网页结构的, 所有样式的东西都应该在 `CSS` 中做
- `entity`: `HTML` 中有些符号被用来做 `关键字` 了, 因此需要使用 `entity` 来表示这些符号
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta name="keywords" content="HTML, CSS" />
    <meta name="description" content="..." />
    <title>Document</title>
    <style>
      em {
        color: red;
        font-style: normal;
      }
    </style>
  </head>
  <body>
    <h1>heading 1</h1>
    <h2>heading 2</h2>
    <h3>heading 3</h3>
    <p>I love &lt;you&gt; &copy;</p>
    <h2>heading 2</h2>
    <h3>heading 3</h3>
    <h4>heading 4</h4>
    <h5>heading 5</h5>
    <h6>heading 6</h6>
    <p>I love <em>you</em></p>
  </body>
</html>

```


- 超链接是 HTML 中很重要的一部分, 可以是文档图片其他网页链接等等
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>index</title>
  </head>
  <body>
    <a href="./about.html">About me</a>
    <a href="./images/pretty.jpg">Image</a>
    <a href="#section-css">CSS</a>
    <a href="https://google.com" target="_blank">Google</a>
    <a href="mailto:example@example.com">Email</a>
    <h2>HTML</h2>
    <p>
      fjkdasljfkldsajgkldsjaklgdjsfiowejktljlkj kfldajsklfg jslfjkdslafj lsjf
      klasjdlkfawe jfkle wjfklewaeslfewjf klsdaj ljsdkf jsdla jfdksj aflsj l j
    </p>
    <h2>CSS</h2>
    <p id="section-css">
      fjklasd sjd klf jwspe fjskld fjls fdsjf lkse jafklds eoiwjfelsjt klsdfj
      klsdj fklsdj fklsdj lkfjds lfk j
    </p>
  </body>
</html>

```
