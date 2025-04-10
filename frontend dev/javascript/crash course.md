- `js` 最初被发明用来在浏览器中执行, 需要依赖浏览器中的引擎, 例如 `chrome V8`
- `nodejs` 将 `chrome V8` 拿出来并做了一些处理形成了一个 `js runtime`, 让代码可以在浏览器之外, 使用 `node` 运行

### html 嵌入
- `js` 原始的方式是在 `html::<script> ...jscode... </script>` 中嵌入代码, 以让浏览器执行
- 嵌入 `<script></script>` 的最佳实践是写在 `<body></body>` 的最末尾, 浏览器从上至下解析 html, 让浏览器先渲染出页面, 然后再处理脚本


### js 代码分离
- 工程中, `js` 逻辑可能很多很复杂, 直接嵌入到 html 是不好的
- 因此可以将 `js` 代码分离成单独的文件, 例如 `index.js`, 然后在 ` html::<script></script> ` 中设置 ` js ` 代码源 ` <script src="index.js"></script> `
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <h1>Hello world</h1>
  <script src="index.js"></script>
</body>
</html>
```


### 使用 node 直接运行 js 代码
- `node index.js`


### 变量&常量
- `var`: 声明全局作用域的变量, 几乎废弃, 非常少用
- `let`: 声明变量
- `const`: 声明常量, 必须声明时初始化

```js
console.log('Hello world from index.js');

let a = 10;
console.log(a);
a = a + 1;
console.log(a);

const b = 20;
console.log(b);
// b = b + 1;   >>> error
console.log(b);
```


### 原生数据类型
- `string`
- `number`: 包括整型和浮点数
- `boolean`
- `object`: null 等
- `undefined`: 未赋值的变量等未定义行为

```js
const username = "john_doe";
console.log(typeof username);

const age = 30;
console.log(typeof age);

const rate = 4.5;
console.log(typeof rate);

const t = true;
console.log(typeof t);

const n = null;
console.log(typeof n);

const u = undefined;
console.log(typeof u);

let z;
console.log(typeof z);

```


### 模板字符串
- 反引号括起来的, 内嵌变量的字符串

```js
let username = "John";
let age = 25;
let hello = `hello, i am ${username} and i am ${age} years old`;

console.log(hello);

```


### 字符串内置属性和方法
- `js` 是面向对象的, 所有的元素都是一个对象
```js
const s = "Hello World!";

console.log(s);
console.log("Length of string is: " + s.length);
console.log(s.toUpperCase());
console.log(s.toLowerCase());
console.log(s.substring(0, 5));

```


### 数组
- `js` 数组类似其他语言中的 `tuple`, 可以存储不同类型的元素
- 数组支持索引
```js
let fruit = new Array("apple", "banana", "mango", "orange", 1, false, null, undefined);
console.log(fruit);

let tree = ["oak", "pine", "maple", "birch", 1, false, null, undefined];
console.log(tree);

const random = ["apple", "mango"];
random.push("orange");
random[2] = "banana";
random.push("grape");
random.unshift("kiwi");
random.pop();
console.log(random);
```


### 对象
- 与其说是对象不如说是一组数据的集合
- 突出一个随意, 可以随意增加属性
```js
let person = {
  name: 'John',
  age: 30,
  city: {
    name: 'New York',
    zip: 10001
  }
};

person.city.zip = 10002;
person.email = "hello@email.com"

console.log(person);

let { name, age, city } = person;
console.log(name, age, city);

```


### 对象数组和 json
- `js` 数组也可以是对象数组, 当然其中的对象
- `js` 中的对象和 `json, JavaScript Object Notation` 非常容易地转换
```js
const todos = [
  {
    id: 1,
    title: "Buy milk",
    completed: false,
  },
  {
    id: 2,
    title: "Buy bread",
    completed: true,
  },
  {
    id: 3,
    title: "Buy butter",
    completed: false,
  },
];
console.log(todos);

const todoJSON = JSON.stringify(todos);

console.log(todoJSON);

```

### if
- `js` 中, `==` 在进行比较时会进行类型转换
- `===`, 严格相等运算符, 不会进行类型转换

```js
console.log(5 == '5'); // true，因为字符串 '5' 被转换为数字 5
console.log(null == undefined); // true

console.log(5 === '5'); // false，因为类型不同
console.log(null === undefined); // false，因为类型不同
```


### 三目运算符
- `const color = x > 10 ? "red" : "blue"`


### switch
- `js` 的 `switch` 跟 C 语言中的类似, 需要在每一个 case 中显式加上 `break`

```js
const color = x > 10 ? "red" : "blue";

switch (color) {
  case "red":
    console.log("Color is red");
    break;
  case "blue":
    console.log("Color is blue");
    break;
  default:
    console.log("Color is neither red nor blue");
    break;
}

```

### while & for
```js
for (let i = 0; i < 10; i++) {
  console.log(`Hello, World! ${i}`);
}

let i = 0;
while (i < 10) {
  console.log(`Hello, World! ${i}`);
  i++;
}

let todos = [
  {
    id: 1,
    text: 'Buy milk',
    done: false,
  },
  {
    id: 2,
    text: 'Learn JavaScript',
    done: false,
  },
  {
    id: 3,
    text: 'Drink water',
    done: true,
  },
];

for (let todo of todos) {
  console.log(todo.text);
}

```


### functions
```js
console.log('Hello, World!');
console.log('test');

function square(number) {
  return number * number;
}

console.log(square(5));

const square = function (params) {
  return number * number;
}

const square = (params) => {
  return number * number;
}

```


# async await
当然可以！下面是对这段代码中 `async`、`await` 和 `fetch` 的详细解释。

### 1. `async`

- **定义**：`async` 是一个关键字，用于定义一个异步函数。异步函数可以在内部使用 `await`，并且总是返回一个 Promise。
- **作用**：使得函数能够处理异步操作，使代码看起来更像同步代码，便于理解和维护。

### 2. `await`

- **定义**：`await` 关键字用于暂停执行异步函数，直到 Promise 被解决（fulfilled）或拒绝（rejected）。它只能在 `async` 函数内部使用。
- **作用**：使得代码在等待 Promise 解决的同时，不会阻塞主线程。可以用来处理异步操作的结果。

### 3. `fetch`

- **定义**：`fetch` 是一个用于发起网络请求的 Web API，可以用于获取资源。它返回一个 Promise，表示请求的结果。
- **用法**：
  - `fetch(url)` 会发起一个 GET 请求，返回一个 Promise。
  - 你可以使用 `.then()` 链接处理结果，或者在 `async` 函数中使用 `await`。

### 代码分析

```javascript
const serchMovie = async (title) => {
    const response = await fetch(`${API_URL}&s=${title}`);
    const data = await response.json();

    setMovies(data.Search);
}
```

1. **函数定义**：
   - `const serchMovie = async (title) => {...}`：定义一个异步函数 `serchMovie`，接受一个参数 `title`。

2. **发起请求**：


3. **处理响应**：
   - `const data = await response.json();`：
     - `response.json()` 解析响应体为 JSON 格式，同样返回一个 Promise。
     - `await` 再次暂停，直到解析完成。

4. **更新状态**：
   - `setMovies(data.Search);`：
     - 将获取到的电影数据存储到组件的状态中，以便在 UI 上显示。

### 总结

- `async` 和 `await` 使得处理异步请求更简单清晰，而 `fetch` 则是发起网络请求的工具。这种组合使得在 React 中处理 API 请求非常方便。


是的，`fetch` 是 JavaScript 的内置函数，属于 Web API 的一部分。它用于发起网络请求，获取资源。以下是一些关于 `fetch` 的关键点：

### 主要特点

1. **基于 Promise**：
   - `fetch` 返回一个 Promise，使得可以使用 `.then()` 和 `.catch()` 方法来处理请求结果和错误，或在 `async` 函数中使用 `await`。

2. **简化请求**：
   - 提供了一个简单的接口来发起 HTTP 请求，可以发送 GET、POST、PUT、DELETE 等请求。

3. **响应处理**：
   - 返回的响应对象包含了状态信息和数据，可以通过方法（如 `.json()`、`.text()`、`.blob()`）来解析响应体。

### 示例用法

```javascript
fetch('https://api.example.com/data')
  .then(response => {
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    return response.json(); // 解析 JSON 数据
  })
  .then(data => console.log(data))
  .catch(error => console.error('There was a problem with the fetch operation:', error));
```

### 总结

`fetch` 是现代 JavaScript 中用于处理网络请求的标准方法，支持异步操作并且使用起来非常灵活。

