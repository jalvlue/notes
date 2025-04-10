css 定义了一组 html 渲染规则, 属于规则集

![[Pasted image 20231227213524.png]]

##### 选择器: selector
选择了一个或多个需要添加样式的元素, 上图中是 `<p>` 段落

##### 声明, 属性和值
改变 HTML 元素的规则
在 HTML 的 `<head>` 中引入
```html
<link href="styles/style.css" rel="stylesheet" />
```
使用 link 元素引入

同时修改多个属性:
```css
p {
  color: red;
  width: 500px;
  border: 1px solid black;
}
```

同时选择多个元素:
```css
p,
li,
h1 {
  color: red;
}
```

常用选择器:
![[Pasted image 20231227213904.png]]

完整 css:
```css
html {

  font-size: 10px;

  /* px 表示“像素（pixel）”: 基础字号为 10 像素 */

  font-family: "Open Sans", sans-serif;

  /* 这应该是你从 Google Fonts 得到的其余输出。 */

}

  

h1 {

  font-size: 60px;

  text-align: center;

}

  

p,

li {

  font-size: 16px;

  line-height: 2;

  letter-spacing: 1px;

}

  

html {

  background-color: #00539f;

}

  

body {

  width: 600px;

  margin: 0 auto;

  background-color: #ff9500;

  padding: 0 20px 20px 20px;

  border: 5px solid black;

}

  

h1 {

  margin: 0;

  padding: 20px 0;

  color: #00539f;

  text-shadow: 3px 3px 1px black;

}

  

img {

  width: 400px;

  display: block;

  margin: 0 auto;

}
```