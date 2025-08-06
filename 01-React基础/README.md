# 第一章：React 基础

## 学习目标
- 了解React框架的基本概念和核心思想
- 理解虚拟DOM的工作原理
- 掌握React开发环境的搭建
- 能够创建第一个React应用

## 1.1 React 简介

### 什么是React？
React是由Facebook开发的一个用于构建用户界面的JavaScript库。它专注于构建UI组件，采用声明式编程范式，让开发者更容易推理应用的状态变化。

### React的核心特点

#### 1. 声明式编程
- **描述想要的结果**，而不是如何实现
- 代码更容易理解和维护
- 减少了DOM操作的复杂性

```javascript
// 声明式 - React方式
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}

// 命令式 - 传统DOM操作方式
function createTodoList(todos) {
  const ul = document.createElement('ul');
  todos.forEach(todo => {
    const li = document.createElement('li');
    li.textContent = todo.text;
    ul.appendChild(li);
  });
  return ul;
}
```

#### 2. 组件化
- 将UI拆分成独立、可复用的组件
- 每个组件管理自己的状态
- 组件可以组合构建复杂的界面

#### 3. 学一次，到处写
- Web应用：React
- 移动应用：React Native
- 桌面应用：Electron + React

## 1.2 虚拟DOM

### 什么是虚拟DOM？
虚拟DOM（Virtual DOM）是React的核心概念之一，它是真实DOM的JavaScript表示。

### 虚拟DOM的工作原理

```javascript
// 虚拟DOM表示
const virtualDOM = {
  type: 'div',
  props: {
    className: 'container',
    children: [
      {
        type: 'h1',
        props: {
          children: 'Hello, React!'
        }
      }
    ]
  }
};

// 对应的真实DOM
// <div class="container">
//   <h1>Hello, React!</h1>
// </div>
```

### 虚拟DOM的优势

1. **性能优化**
   - 批量更新
   - Diff算法优化
   - 最小化DOM操作

2. **开发体验**
   - 声明式编程
   - 状态驱动视图更新
   - 更容易调试

3. **跨浏览器兼容**
   - 抽象了DOM操作差异
   - 统一的API接口

### Diff算法核心思想

```javascript
// 旧虚拟DOM
const oldVDOM = {
  type: 'ul',
  children: [
    { type: 'li', key: '1', children: 'Item 1' },
    { type: 'li', key: '2', children: 'Item 2' }
  ]
};

// 新虚拟DOM
const newVDOM = {
  type: 'ul',
  children: [
    { type: 'li', key: '1', children: 'Item 1' },
    { type: 'li', key: '3', children: 'Item 3' },
    { type: 'li', key: '2', children: 'Item 2' }
  ]
};

// Diff算法会计算出最小的变更操作：
// 1. 在位置1插入新的li元素（key='3'）
// 2. 移动key='2'的元素到新位置
```

## 1.3 开发环境搭建

### 前置要求
- Node.js (版本 >= 14.0.0)
- npm 或 yarn 包管理器
- 代码编辑器（推荐VS Code）

### 安装Node.js
```bash
# 检查Node.js版本
node --version

# 检查npm版本
npm --version
```

### VS Code插件推荐
1. **ES7+ React/Redux/React-Native snippets**
2. **Bracket Pair Colorizer**
3. **Auto Rename Tag**
4. **Prettier - Code formatter**
5. **ESLint**

## 1.4 Create React App

### 什么是Create React App？
Create React App是Facebook官方提供的创建React应用的脚手架工具，它预配置了：
- Webpack构建工具
- Babel编译器
- ESLint代码检查
- Jest测试框架
- 开发服务器

### 创建第一个React应用

```bash
# 使用npx创建应用（推荐）
npx create-react-app my-first-react-app

# 或使用npm
npm install -g create-react-app
create-react-app my-first-react-app

# 或使用yarn
yarn create react-app my-first-react-app
```

### 项目结构解析

```
my-first-react-app/
├── public/
│   ├── index.html          # HTML模板
│   ├── favicon.ico         # 网站图标
│   └── manifest.json       # PWA配置
├── src/
│   ├── index.js           # 应用入口文件
│   ├── App.js             # 主应用组件
│   ├── App.css            # 样式文件
│   ├── App.test.js        # 测试文件
│   ├── index.css          # 全局样式
│   ├── logo.svg           # React logo
│   └── reportWebVitals.js # 性能监控
├── package.json           # 项目配置和依赖
└── README.md             # 项目说明
```

### 启动开发服务器

```bash
cd my-first-react-app
npm start
# 或
yarn start
```

访问 http://localhost:3000 查看应用

## 1.5 React元素和渲染

### React元素
React元素是React应用的最小构建块，它描述了你想在屏幕上看到的内容。

```javascript
// 创建React元素
const element = React.createElement(
  'h1',
  { className: 'greeting' },
  'Hello, world!'
);

// 使用JSX创建元素（更常用）
const element = <h1 className="greeting">Hello, world!</h1>;
```

### 渲染元素到DOM

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';

// 创建根节点
const root = ReactDOM.createRoot(document.getElementById('root'));

// 渲染元素
const element = <h1>Hello, world!</h1>;
root.render(element);
```

### 更新已渲染的元素

```javascript
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  root.render(element);
}

setInterval(tick, 1000);
```

## 1.6 实践案例：Hello World应用

让我们创建一个简单的Hello World应用来实践所学内容。

### 修改 src/App.js

```javascript
import React from 'react';
import './App.css';

function App() {
  const currentTime = new Date().toLocaleString();
  const userName = 'React学习者';
  
  return (
    <div className="App">
      <header className="App-header">
        <h1>欢迎来到React世界！</h1>
        <p>你好，{userName}！</p>
        <p>当前时间：{currentTime}</p>
        <div className="info-section">
          <h2>React的特点</h2>
          <ul>
            <li>声明式编程</li>
            <li>组件化开发</li>
            <li>虚拟DOM</li>
            <li>单向数据流</li>
          </ul>
        </div>
      </header>
    </div>
  );
}

export default App;
```

### 修改 src/App.css

```css
.App {
  text-align: center;
}

.App-header {
  background-color: #282c34;
  padding: 20px;
  color: white;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.App-header h1 {
  color: #61dafb;
  margin-bottom: 20px;
}

.App-header p {
  font-size: 18px;
  margin: 10px 0;
}

.info-section {
  background-color: #3a3f47;
  padding: 20px;
  border-radius: 10px;
  margin-top: 20px;
  max-width: 400px;
}

.info-section h2 {
  color: #61dafb;
  margin-bottom: 15px;
}

.info-section ul {
  list-style: none;
  padding: 0;
}

.info-section li {
  background-color: #4a4f57;
  margin: 10px 0;
  padding: 10px;
  border-radius: 5px;
  border-left: 3px solid #61dafb;
}
```

## 1.7 动态时钟组件

让我们创建一个动态显示时间的组件：

### 创建 src/Clock.js

```javascript
import React, { useState, useEffect } from 'react';

function Clock() {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const timer = setInterval(() => {
      setTime(new Date());
    }, 1000);

    // 清理函数
    return () => clearInterval(timer);
  }, []);

  const formatTime = (date) => {
    return {
      date: date.toLocaleDateString('zh-CN'),
      time: date.toLocaleTimeString('zh-CN'),
      day: date.toLocaleDateString('zh-CN', { weekday: 'long' })
    };
  };

  const { date, time: currentTime, day } = formatTime(time);

  return (
    <div className="clock-container">
      <h3>实时时钟</h3>
      <div className="time-display">
        <div className="date">{date}</div>
        <div className="time">{currentTime}</div>
        <div className="day">{day}</div>
      </div>
    </div>
  );
}

export default Clock;
```

### 添加时钟样式到 App.css

```css
.clock-container {
  background-color: #2d3748;
  padding: 20px;
  border-radius: 15px;
  margin: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.clock-container h3 {
  color: #61dafb;
  margin-bottom: 15px;
}

.time-display {
  font-family: 'Courier New', monospace;
}

.date {
  font-size: 16px;
  color: #a0aec0;
  margin-bottom: 5px;
}

.time {
  font-size: 24px;
  color: #61dafb;
  font-weight: bold;
  margin-bottom: 5px;
}

.day {
  font-size: 14px;
  color: #e2e8f0;
}
```

### 在App.js中使用时钟组件

```javascript
import React from 'react';
import Clock from './Clock';
import './App.css';

function App() {
  const userName = 'React学习者';
  
  return (
    <div className="App">
      <header className="App-header">
        <h1>欢迎来到React世界！</h1>
        <p>你好，{userName}！</p>
        
        <Clock />
        
        <div className="info-section">
          <h2>React的特点</h2>
          <ul>
            <li>声明式编程</li>
            <li>组件化开发</li>
            <li>虚拟DOM</li>
            <li>单向数据流</li>
          </ul>
        </div>
      </header>
    </div>
  );
}

export default App;
```

## 1.8 总结

### 本章重点
1. **React基本概念**：声明式、组件化、虚拟DOM
2. **开发环境**：Node.js、Create React App、VS Code配置
3. **项目结构**：理解脚手架生成的文件结构
4. **React元素**：创建和渲染元素的基本方法
5. **第一个应用**：Hello World + 动态时钟

### 关键概念理解
- **声明式 vs 命令式**：描述想要什么，而不是如何做
- **虚拟DOM**：性能优化的核心机制
- **组件化**：构建可复用UI的基础

### 下章预告
下一章我们将学习JSX语法，这是React开发中不可或缺的语法糖，让我们能够在JavaScript中直接编写类似HTML的代码。

### 练习作业
1. 尝试修改时钟组件，添加不同的时间格式显示
2. 创建一个个人信息展示组件
3. 实践Create React App的其他命令（build、test、eject）

### 常见问题
1. **Q: 为什么选择React而不是其他框架？**
   A: React学习曲线相对平缓，生态丰富，社区活跃，就业机会多。

2. **Q: 必须使用Create React App吗？**
   A: 不是必须的，但对初学者友好，包含了最佳实践配置。

3. **Q: 虚拟DOM真的比直接操作DOM快吗？**
   A: 不一定总是更快，但它提供了更好的开发体验和更可预测的性能。
