- `react` 是 `nodejs` 生态圈中的一环, 支持通过 `npm` 安装和启动 [[node]]
- `npx create-react-app my-app && cd my-app && npm start`


# DOM
DOM（文档对象模型）是一个用于表示和操作 HTML 和 XML 文档的编程接口。它将文档视为一个树形结构，使开发者能够以编程方式访问和修改文档的内容、结构和样式。

### 主要特点
1. **树形结构**：DOM 将文档表示为一个节点树，文档的每个部分（如元素、属性和文本）都是树中的一个节点。
2. **语言无关**：虽然 DOM 最常与 JavaScript 一起使用，但它是与语言无关的，可以被多种编程语言访问。
3. **动态交互**：通过 DOM，开发者可以动态地添加、删除或修改页面内容，实现交互效果。

### DOM 的主要组件
- **节点（Node）**：树的基本单位，可以是元素节点、文本节点、属性节点等。
- **元素（Element）**：表示 HTML 标签，例如 `<div>`、`<p>` 等。
- **属性（Attribute）**：元素的特征，如 `class`、`id` 等。

### 使用示例
```javascript
// 获取元素
const element = document.getElementById('myElement');

// 修改内容
element.textContent = 'Hello, World!';

// 添加新元素
const newElement = document.createElement('p');
newElement.textContent = 'This is a new paragraph.';
document.body.appendChild(newElement);

```

### 根节点
在 DOM 中，根节点是一个文档树的最顶层节点。每个 HTML 或 XML 文档都有一个根节点，通常是整个文档的起始点。


1. **唯一性**：每个文档只有一个根节点。
2. **文档类型**：
   - 在 HTML 文档中，根节点通常是 `<html>` 标签。
   - 在 XML 文档中，根节点是包含所有其他节点的顶层元素。
3. **树结构**：根节点是 DOM 树的起始点，所有其他节点（如元素、文本和属性）都是其子节点。

在以下 HTML 文档中：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <h1>Hello, World!</h1>
</body>
</html>
```

- **根节点**：`<html>` 是根节点。
- **子节点**：`<head>` 和 `<body>` 是 `<html>` 的子节点。



### react

- 一个 `react` 页面由多个组件 `component` 组成
	- `class-base components`: 基于类的组件, 现代 react 基本不使用了
	- `functional components`: 基于函数的组件, 一个函数返回一个组件


### component
- `react` 神奇的地方就在于, 它可以直接返回一个作为 html 元素的组件 `component`, `react` 的组件编写语法被称为 `jsx`
- 通过 `component` 进行复用和嵌入
- `App` 是全局唯一的入口组件, 通常直接在 `index.js` 中嵌入, 一般不修改 `index.js` 中的内容, 而是在 `App.js` 中修改 `App` 组件的内容从而修改整个 app 的布局
- `props` 是一个很重要的性质, 允许在调用组件时传递参数

```js
import './App.css';

function Person(props) {
  return (
    <div>
      <h1>Hello, {props.name}</h1>
      <p>Age: {props.age}</p>
    </div>
  );
}

function App() {
  return (
    <div className="App">
      <Person name={"jalvlue"} age={20} />
    </div>
  );
}

export default App;

```

### state
- `react` 可以维护应用中某一个变量的状态, 从而达到不需要重新加载页面就可以更新状态的效果
- 使用 `useState hook`, 设置某一个状态变量, 并设置好函数来维护某一些行为之后, 该变量的状态
- `const [XXX, setXXX] = useState(initXXX)` 是使用 `useState hook` 的标准语法, `XXX` 就是该状态, `setXXX()` 则是 `useState` 钩子提供的一个回调函数, 用于修改该状态
- 这里通过 `useState()` 设置的 `XXX` 不仅是一个变量, 还是 `react` 状态的一部分, 在 `react` 的设定中手动修改会引发错误, 限定为只能使用 `setXXX()` 对该状态进行修改
- 通过调用 `setXXX()` 函数, 可以修改对应变量状态
	- `setXXX(newState)`: 直接赋值修改, 这里是异步进行修改, 如果直接赋值依赖于原状态的话可能会出错
	- `setXXX((prevXXX) => { ... }`: 传递一个闭包函数, 这个闭包函数的签名是 `function (prevXXX) newState`, 钩子会自动将前一个状态的值作为参数 `prevXXX` 传递进闭包函数中, 此时就可以依赖这个原状态

```js
import { useState } from 'react';
import './App.css';

function App() {
  const [counter, setCounter] = useState(0);

  return (
    <div className="App">

      <div className='buttons'>
        <button onClick={() => {
          setCounter((prevCount) => { return prevCount - 1 });
          alert("clicked");
        }}>-</button>

        <h1>{counter}</h1>

        <button onClick={() => {
          setCounter(counter + 1);
          alert("clicked");
        }}>+</button>
      </div>

    </div>
  );
}

export default App;

```


### effect
`useEffect` 是 React 中的一个钩子，用于处理副作用。副作用是指那些在组件渲染时需要执行的操作，比如数据获取、订阅事件或手动操作 DOM。

##### 主要特点
1. **执行时机**：
   - `useEffect` 在组件渲染后执行，可以在组件首次挂载时和每次更新时触发。
2. **清理副作用**：
   - 可以返回一个清理函数，以便在组件卸载时或在下一次执行效果前清理之前的副作用。
3. **依赖数组**：
   - 通过传递一个依赖数组，你可以控制 `useEffect` 何时执行。如果数组中的值发生变化，`useEffect` 会重新执行。

##### 基本用法
###### 1. 无依赖的副作用
```javascript
import React, { useEffect } from 'react';

function Example() {
  useEffect(() => {
    console.log('Component mounted or updated');

    return () => {
      console.log('Cleanup before next effect or unmount');
    };
  });

  return <div>Check the console!</div>;
}
```

###### 2. 有依赖的副作用
```javascript
function Example({ count }) {
  useEffect(() => {
    console.log(`Count has changed to: ${count}`);

    return () => {
      console.log('Cleanup for count change');
    };
  }, [count]); // 仅在 count 变化时执行

  return <div>Count: {count}</div>;
}
```

###### 3. 仅在挂载时执行
如果传入一个空数组，`useEffect` 只会在组件首次挂载时执行：

```javascript
useEffect(() => {
  console.log('Component mounted');

  return () => {
    console.log('Cleanup on unmount');
  };
}, []); // 只在挂载和卸载时执行
```

##### 总结
`useEffect` 是一个强大的工具，允许你处理副作用并管理组件的生命周期。通过控制依赖数组，你可以精确地控制副作用的执行时机，使得组件更加灵活和高效。


# app example: theater
```js
// App.js
import { useState, useEffect } from 'react';
import './App.css';
import SearchIcon from './search.svg'
import MovieCard from './MovieCard.jsx';

// 3b2f9d90
// http://www.omdbapi.com/?i=tt3896198&apikey=3b2f9d90
const API_URL = "http://www.omdbapi.com/?i=tt3896198&apikey=3b2f9d90";


function App() {
  const [movies, setMovies] = useState([]);
  const [searchTerm, setSearchTerm] = useState("");

  const serchMovie = async (title) => {
    const response = await fetch(`${API_URL}&s=${title}`);
    const data = await response.json();

    setMovies(data.Search);
  }

  useEffect(() => {
    serchMovie("detachment");
  }, []);

  return (
    <div className="app">
      <h1>Theater</h1>

      <div className='search'>
        <input
          placeholder='Search for moview'
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
        <img
          src={SearchIcon}
          alt="search"
          onClick={() => serchMovie(searchTerm)}
        />
      </div>

      {
        movies?.length > 0
          ? (
            <div className="container">
              {movies.map((movie) => (
                <MovieCard movie={movie} />
              ))}
            </div>
          ) : (
            <div className='empty'>
              <h2>No movies found</h2>
            </div>
          )
      }

    </div>
  );
}

export default App;


// MovieCard.jsx
import React from "react";

const MovieCard = ({ movie }) => {
  return (
    <div className='movie'>
      <div>
        <p>{movie.Year}</p>
      </div>

      <div>
        <img
          src={movie.Poster !== "N/A" ? movie.Poster : "https://via.placeholder.com/400"}
          alt={movie.Title}
        />
      </div>

      <div>
        <span>
          {movie.Type}
        </span>
        <h3>
          {movie.Title}
        </h3>

      </div>
    </div>
  )
}

export default MovieCard;

```

当然可以！下面是对你提供的 React 代码的详细解释。

### App. Js

```javascript
import { useState, useEffect } from 'react';
import './App.css';
import SearchIcon from './search.svg';
import MovieCard from './MovieCard.jsx';

const API_URL = "http://www.omdbapi.com/?i=tt3896198&apikey=3b2f9d90";
```
- **Imports**: 导入 React 的 `useState` 和 `useEffect` 钩子，用于管理状态和副作用。还导入 CSS 文件和其他组件 (`MovieCard`) 以及搜索图标。

```javascript
function App() {
  const [movies, setMovies] = useState([]);
  const [searchTerm, setSearchTerm] = useState("");
```
- **State Management**:
  - `movies`: 用于存储从 API 获取的电影数据，初始值为空数组。
  - `searchTerm`: 用于存储用户输入的搜索关键词，初始值为空字符串。

```javascript
  const serchMovie = async (title) => {
    const response = await fetch(`${API_URL}&s=${title}`);
    const data = await response.json();

    setMovies(data.Search);
  }
```
- **serchMovie**: 异步函数，用于根据电影标题从 API 获取电影数据。
  - 使用 `fetch` 请求 API，并将返回的数据解析为 JSON。
  - `setMovies(data.Search)`: 将获取的电影列表存储到 `movies` 状态中。

```javascript
  useEffect(() => {
    serchMovie("detachment");
  }, []);
```
- **useEffect**: 组件挂载后执行 `serchMovie("detachment")`，初次加载时会调用这个函数获取默认电影列表。

```javascript
  return (
    <div className="app">
      <h1>Theater</h1>
```
- **Render**: 渲染组件的结构和样式，包括标题和搜索框。

```javascript
      <div className='search'>
        <input
          placeholder='Search for movies'
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
        <img
          src={SearchIcon}
          alt="search"
          onClick={() => serchMovie(searchTerm)}
        />
      </div>
```
- **Search Input**:
  - 输入框用于接受用户输入的搜索关键词，并通过 `onChange` 更新 `searchTerm` 状态。
  - 点击搜索图标时，调用 `serchMovie` 函数进行搜索。

```javascript
      {
        movies?.length > 0
          ? (
            <div className="container">
              {movies.map((movie) => (
                <MovieCard movie={movie} />
              ))}
            </div>
          ) : (
            <div className='empty'>
              <h2>No movies found</h2>
            </div>
          )
      }
```
- **Conditional Rendering**: 根据 `movies` 数组的长度决定渲染内容。
  - 如果有电影，使用 `MovieCard` 组件渲染每个电影。
  - 如果没有电影，显示“未找到电影”的提示。

```javascript
export default App;
```
- **Export**: 导出 `App` 组件，供其他文件使用。

### MovieCard. Jsx

```javascript
import React from "react";

const MovieCard = ({ movie }) => {
  return (
    <div className='movie'>
      <div>
        <p>{movie.Year}</p>
      </div>

      <div>
        <img
          src={movie.Poster !== "N/A" ? movie.Poster : "https://via.placeholder.com/400"}
          alt={movie.Title}
        />
      </div>

      <div>
        <span>
          {movie.Type}
        </span>
        <h3>
          {movie.Title}
        </h3>
      </div>
    </div>
  )
}

export default MovieCard;
```
- **MovieCard Component**: 用于展示单个电影的信息。
  - **Props**: 接收 `movie` 对象作为属性。
  - 渲染电影的年份、海报和标题，使用条件渲染确保海报存在。

### 总结
这个项目是一个简单的电影搜索应用，使用 React 管理状态和组件，调用外部 API 获取数据，并根据用户输入动态渲染结果。通过组件化的方式，代码结构清晰，易于维护和扩展。