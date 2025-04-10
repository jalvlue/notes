js 为 html 页面提供动态的信息

使用 script 元素引入,
```html
<script src="scripts/main.js" defer></script>
```
通常要放在 `<body>` 的最底部, 
因为浏览器顺序加载 html 文件, 
脚本放前面先运行的话可能会导致脚本读到了未加载的元素而失效,
因此放在最底部是比较合适的

main.js
```js
let myHeading = document.querySelector("h1");

myHeading.textContent = "Hello world!";
```
这里首先获得了 html 文档的一级标题,
然后把标题修改成了 `Hello world!`
思想是想修改什么就要先获得什么

# js 基础

### 变量
使用 `let` 或 `var` 关键字声明

变量类型:
![[Pasted image 20231227220058.png]]

需要注意的是, js 使用 `===` 和 `!==` 用于逻辑表达式

### 条件语句
与 c 类似
```js
let iceCream = "chocolate";
if (iceCream === "chocolate") {
  alert("我最喜欢巧克力冰淇淋了。");
} else {
  alert("但是巧克力才是我的最爱呀……");
}
```

### 函数
使用 `function` 关键字开始定义
```js
function multiply(num1, num2) {
  let result = num1 * num2;
  return result;
}

multiply(4, 7);
multiply(20, 20);
multiply(0.5, 3);
```

### 事件
给网页添加交互能力, 能捕捉浏览器的操作, 并运行一些代码作为响应
```js
document.querySelector("html").addEventListener("click", function () {
  alert("别戳我，我怕疼。");
});

document.querySelector("html").addEventListener("click", () => {
  alert("别戳我，我怕疼。");
});

```
以上两种都是选择了 html 元素, 然后给这个元素添加了监听 `click` 的事件, 并定义好了对应的响应函数(匿名函数, 也可以直接使用其他的应该)


js 会调用很多浏览器提供的库函数
```js
// let myHeading = document.querySelector("h1");

// myHeading.textContent = "Hello world!";

  

let myVar = "hello";

alert(myVar);

console.log(myVar);

  

// document.querySelector("html").addEventListener("click", function () {

//   alert("Ouch! Stop poking me!");

// })

  

let myImage = document.querySelector("img");

  

myImage.onclick = function () {

  let mySrc = myImage.getAttribute("src");

  if (mySrc === "images/test-img.jpg") {

    myImage.setAttribute("src", "images/test-img2.jpg");

  } else {

    myImage.setAttribute("src", "images/test-img.jpg");

  }

};

  

let myButton = document.querySelector("button");

let myHeading = document.querySelector("h1");

  

function setUserName() {

  let myName = prompt("Please enter your name.");

  if (!myName) {

    alert("re-enter name")

    setUserName();

  } else {

    localStorage.setItem("name", myName);

    myHeading.textContent = "Mozilla is cool, " + myName;

  }

}

  

if (!localStorage.getItem("name")) {

  setUserName();

} else {

  let storedName = localStorage.getItem("name");

  myHeading.textContent = "Mozilla is coll, " + storedName

}

  

myButton.onclick = function () {

  setUserName();

};
```

