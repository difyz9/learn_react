# 第三章：组件开发

## 学习目标
- 理解React组件的概念和作用
- 掌握函数组件和类组件的创建方法
- 学会组件的组合和拆分技巧
- 理解组件间的通信方式
- 能够构建可复用的组件库

## 3.1 组件概念

### 什么是组件？
组件是React应用的基本构建块，它是一个可重用的代码片段，负责渲染页面的一部分。组件就像JavaScript函数一样，接受任意的输入（称为"props"），并返回描述页面显示内容的React元素。

### 组件的特点
1. **可复用**：一次编写，多处使用
2. **独立性**：每个组件管理自己的状态
3. **可组合**：小组件组成大组件
4. **声明式**：描述UI应该是什么样子

### 组件的分类

#### 按功能分类
- **展示组件（Presentational Components）**：专注于UI渲染
- **容器组件（Container Components）**：专注于数据逻辑

#### 按实现方式分类
- **函数组件（Function Components）**：使用函数定义
- **类组件（Class Components）**：使用ES6类定义

## 3.2 函数组件

### 基本语法
```javascript
// 最简单的函数组件
function Welcome() {
  return <h1>Hello, World!</h1>;
}

// 箭头函数写法
const Welcome = () => {
  return <h1>Hello, World!</h1>;
};

// 更简洁的写法（省略return）
const Welcome = () => <h1>Hello, World!</h1>;
```

### 带参数的函数组件
```javascript
function Greeting(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// 使用解构赋值
function Greeting({ name, age }) {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>You are {age} years old.</p>
    </div>
  );
}

// 设置默认参数
function Greeting({ name = 'Guest', age = 0 }) {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>You are {age} years old.</p>
    </div>
  );
}
```

### 复杂函数组件示例
```javascript
function UserProfile({ user, onEdit, onDelete }) {
  // 辅助函数
  const formatDate = (date) => {
    return new Date(date).toLocaleDateString('zh-CN');
  };
  
  const getStatusColor = (status) => {
    const colors = {
      active: '#28a745',
      inactive: '#6c757d',
      pending: '#ffc107'
    };
    return colors[status] || '#6c757d';
  };
  
  // 条件渲染函数
  const renderActions = () => {
    if (!user.canEdit) return null;
    
    return (
      <div className="actions">
        <button onClick={() => onEdit(user.id)}>编辑</button>
        <button onClick={() => onDelete(user.id)}>删除</button>
      </div>
    );
  };
  
  return (
    <div className="user-profile">
      <div className="avatar">
        <img src={user.avatar} alt={user.name} />
        <div 
          className="status-indicator"
          style={{ backgroundColor: getStatusColor(user.status) }}
        ></div>
      </div>
      
      <div className="info">
        <h3>{user.name}</h3>
        <p>邮箱：{user.email}</p>
        <p>注册时间：{formatDate(user.createdAt)}</p>
        <p>状态：{user.status}</p>
      </div>
      
      {renderActions()}
    </div>
  );
}
```

## 3.3 类组件

### 基本语法
```javascript
import React, { Component } from 'react';

class Welcome extends Component {
  render() {
    return <h1>Hello, World!</h1>;
  }
}

// 带参数的类组件
class Greeting extends Component {
  render() {
    const { name, age } = this.props;
    return (
      <div>
        <h1>Hello, {name}!</h1>
        <p>You are {age} years old.</p>
      </div>
    );
  }
}
```

### 带状态的类组件
```javascript
class Counter extends Component {
  constructor(props) {
    super(props);
    
    // 初始化状态
    this.state = {
      count: 0,
      isEven: true
    };
    
    // 绑定事件处理方法
    this.increment = this.increment.bind(this);
    this.decrement = this.decrement.bind(this);
  }
  
  increment() {
    this.setState(prevState => ({
      count: prevState.count + 1,
      isEven: (prevState.count + 1) % 2 === 0
    }));
  }
  
  decrement() {
    this.setState(prevState => ({
      count: prevState.count - 1,
      isEven: (prevState.count - 1) % 2 === 0
    }));
  }
  
  render() {
    const { count, isEven } = this.state;
    
    return (
      <div className="counter">
        <h2>计数器: {count}</h2>
        <p>当前数字是{isEven ? '偶数' : '奇数'}</p>
        <button onClick={this.increment}>+1</button>
        <button onClick={this.decrement}>-1</button>
      </div>
    );
  }
}
```

### 类组件生命周期示例
```javascript
class TimerComponent extends Component {
  constructor(props) {
    super(props);
    this.state = {
      seconds: 0,
      isRunning: false
    };
  }
  
  componentDidMount() {
    console.log('组件已挂载');
    // 组件挂载后的逻辑
  }
  
  componentDidUpdate(prevProps, prevState) {
    console.log('组件已更新');
    // 如果开始计时，启动定时器
    if (!prevState.isRunning && this.state.isRunning) {
      this.startTimer();
    }
    // 如果停止计时，清除定时器
    if (prevState.isRunning && !this.state.isRunning) {
      this.stopTimer();
    }
  }
  
  componentWillUnmount() {
    console.log('组件即将卸载');
    this.stopTimer();
  }
  
  startTimer = () => {
    this.timer = setInterval(() => {
      this.setState(prevState => ({
        seconds: prevState.seconds + 1
      }));
    }, 1000);
  }
  
  stopTimer = () => {
    if (this.timer) {
      clearInterval(this.timer);
      this.timer = null;
    }
  }
  
  toggleTimer = () => {
    this.setState(prevState => ({
      isRunning: !prevState.isRunning
    }));
  }
  
  resetTimer = () => {
    this.setState({
      seconds: 0,
      isRunning: false
    });
  }
  
  render() {
    const { seconds, isRunning } = this.state;
    
    return (
      <div className="timer">
        <h2>计时器: {seconds}秒</h2>
        <button onClick={this.toggleTimer}>
          {isRunning ? '暂停' : '开始'}
        </button>
        <button onClick={this.resetTimer}>重置</button>
      </div>
    );
  }
}
```

## 3.4 组件组合

### 基本组合
```javascript
// 子组件
function Header({ title }) {
  return (
    <header className="header">
      <h1>{title}</h1>
    </header>
  );
}

function Sidebar({ items }) {
  return (
    <aside className="sidebar">
      <ul>
        {items.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </aside>
  );
}

function MainContent({ children }) {
  return (
    <main className="main-content">
      {children}
    </main>
  );
}

function Footer({ copyright }) {
  return (
    <footer className="footer">
      <p>&copy; {copyright}</p>
    </footer>
  );
}

// 父组件组合子组件
function App() {
  const sidebarItems = [
    { id: 1, name: '首页' },
    { id: 2, name: '关于' },
    { id: 3, name: '联系' }
  ];
  
  return (
    <div className="app">
      <Header title="我的应用" />
      <div className="app-body">
        <Sidebar items={sidebarItems} />
        <MainContent>
          <h2>欢迎来到主页</h2>
          <p>这里是主要内容区域。</p>
        </MainContent>
      </div>
      <Footer copyright="2023 我的公司" />
    </div>
  );
}
```

### 使用children属性
```javascript
function Card({ title, children }) {
  return (
    <div className="card">
      {title && <div className="card-header">{title}</div>}
      <div className="card-body">
        {children}
      </div>
    </div>
  );
}

function Button({ variant = 'primary', children, ...props }) {
  return (
    <button className={`btn btn-${variant}`} {...props}>
      {children}
    </button>
  );
}

// 使用组合组件
function UserCard({ user }) {
  return (
    <Card title="用户信息">
      <p>姓名：{user.name}</p>
      <p>邮箱：{user.email}</p>
      <div className="actions">
        <Button variant="primary">编辑</Button>
        <Button variant="secondary">删除</Button>
      </div>
    </Card>
  );
}
```

### 高阶组合模式
```javascript
// 布局组件
function Layout({ children }) {
  return (
    <div className="layout">
      <nav className="navigation">
        <ul>
          <li><a href="#home">首页</a></li>
          <li><a href="#about">关于</a></li>
          <li><a href="#contact">联系</a></li>
        </ul>
      </nav>
      <main className="content">
        {children}
      </main>
    </div>
  );
}

// 容器组件
function Container({ maxWidth = '1200px', children }) {
  return (
    <div 
      className="container" 
      style={{ maxWidth, margin: '0 auto', padding: '0 20px' }}
    >
      {children}
    </div>
  );
}

// 网格组件
function Grid({ columns = 12, children }) {
  return (
    <div className={`grid grid-${columns}`}>
      {children}
    </div>
  );
}

function GridItem({ span = 1, children }) {
  return (
    <div className={`grid-item span-${span}`}>
      {children}
    </div>
  );
}

// 使用布局组件
function HomePage() {
  return (
    <Layout>
      <Container>
        <h1>欢迎来到首页</h1>
        <Grid columns={12}>
          <GridItem span={8}>
            <Card title="主要内容">
              <p>这里是主要内容区域...</p>
            </Card>
          </GridItem>
          <GridItem span={4}>
            <Card title="侧边栏">
              <p>这里是侧边栏内容...</p>
            </Card>
          </GridItem>
        </Grid>
      </Container>
    </Layout>
  );
}
```

## 3.5 组件通信

### 父子组件通信

#### 父传子：通过Props
```javascript
function Parent() {
  const [message, setMessage] = useState('Hello from Parent');
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <h2>父组件</h2>
      <input 
        value={message} 
        onChange={(e) => setMessage(e.target.value)} 
      />
      <button onClick={() => setCount(count + 1)}>
        增加计数
      </button>
      
      <Child 
        message={message} 
        count={count} 
        parentName="ParentComponent"
      />
    </div>
  );
}

function Child({ message, count, parentName }) {
  return (
    <div>
      <h3>子组件</h3>
      <p>来自{parentName}的消息：{message}</p>
      <p>计数：{count}</p>
    </div>
  );
}
```

#### 子传父：通过回调函数
```javascript
function Parent() {
  const [receivedData, setReceivedData] = useState('');
  
  const handleDataFromChild = (data) => {
    setReceivedData(data);
  };
  
  return (
    <div>
      <h2>父组件</h2>
      <p>从子组件接收到：{receivedData}</p>
      
      <Child onSendData={handleDataFromChild} />
    </div>
  );
}

function Child({ onSendData }) {
  const [inputValue, setInputValue] = useState('');
  
  const sendDataToParent = () => {
    onSendData(inputValue);
  };
  
  return (
    <div>
      <h3>子组件</h3>
      <input 
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="输入要发送的数据"
      />
      <button onClick={sendDataToParent}>发送给父组件</button>
    </div>
  );
}
```

### 兄弟组件通信
```javascript
function App() {
  const [sharedData, setSharedData] = useState('初始数据');
  
  return (
    <div>
      <h1>兄弟组件通信</h1>
      <Sibling1 data={sharedData} onUpdate={setSharedData} />
      <Sibling2 data={sharedData} onUpdate={setSharedData} />
    </div>
  );
}

function Sibling1({ data, onUpdate }) {
  const updateData = () => {
    onUpdate(`来自Sibling1的数据：${Date.now()}`);
  };
  
  return (
    <div>
      <h2>兄弟组件1</h2>
      <p>共享数据：{data}</p>
      <button onClick={updateData}>更新数据</button>
    </div>
  );
}

function Sibling2({ data, onUpdate }) {
  const updateData = () => {
    onUpdate(`来自Sibling2的数据：${Date.now()}`);
  };
  
  return (
    <div>
      <h2>兄弟组件2</h2>
      <p>共享数据：{data}</p>
      <button onClick={updateData}>更新数据</button>
    </div>
  );
}
```

## 3.6 实践案例：可复用的UI组件库

让我们创建一个小型的UI组件库，包含常用的组件。

### 按钮组件 (Button.js)
```javascript
import React from 'react';
import './Button.css';

function Button({ 
  children, 
  variant = 'primary', 
  size = 'medium', 
  disabled = false,
  loading = false,
  onClick,
  ...props 
}) {
  const getButtonClass = () => {
    let classes = ['btn'];
    classes.push(`btn-${variant}`);
    classes.push(`btn-${size}`);
    if (disabled) classes.push('btn-disabled');
    if (loading) classes.push('btn-loading');
    return classes.join(' ');
  };
  
  const handleClick = (e) => {
    if (disabled || loading) return;
    onClick && onClick(e);
  };
  
  return (
    <button 
      className={getButtonClass()}
      disabled={disabled || loading}
      onClick={handleClick}
      {...props}
    >
      {loading && <span className="btn-spinner">⟳</span>}
      {children}
    </button>
  );
}

export default Button;
```

### 输入框组件 (Input.js)
```javascript
import React, { useState } from 'react';
import './Input.css';

function Input({ 
  label,
  error,
  helpText,
  type = 'text',
  placeholder,
  value,
  onChange,
  required = false,
  disabled = false,
  ...props 
}) {
  const [focused, setFocused] = useState(false);
  
  const getInputClass = () => {
    let classes = ['input'];
    if (error) classes.push('input-error');
    if (disabled) classes.push('input-disabled');
    if (focused) classes.push('input-focused');
    return classes.join(' ');
  };
  
  return (
    <div className="input-group">
      {label && (
        <label className="input-label">
          {label}
          {required && <span className="required">*</span>}
        </label>
      )}
      
      <input
        type={type}
        className={getInputClass()}
        placeholder={placeholder}
        value={value}
        onChange={onChange}
        disabled={disabled}
        onFocus={() => setFocused(true)}
        onBlur={() => setFocused(false)}
        {...props}
      />
      
      {error && <div className="input-error-message">{error}</div>}
      {helpText && !error && (
        <div className="input-help-text">{helpText}</div>
      )}
    </div>
  );
}

export default Input;
```

### 模态框组件 (Modal.js)
```javascript
import React, { useEffect } from 'react';
import './Modal.css';

function Modal({ 
  isOpen, 
  onClose, 
  title, 
  children,
  size = 'medium',
  closeOnOverlayClick = true 
}) {
  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === 'Escape') {
        onClose();
      }
    };
    
    if (isOpen) {
      document.addEventListener('keydown', handleEscape);
      document.body.style.overflow = 'hidden';
    }
    
    return () => {
      document.removeEventListener('keydown', handleEscape);
      document.body.style.overflow = 'unset';
    };
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  const handleOverlayClick = (e) => {
    if (e.target === e.currentTarget && closeOnOverlayClick) {
      onClose();
    }
  };
  
  return (
    <div className="modal-overlay" onClick={handleOverlayClick}>
      <div className={`modal modal-${size}`}>
        <div className="modal-header">
          <h2 className="modal-title">{title}</h2>
          <button className="modal-close" onClick={onClose}>
            ×
          </button>
        </div>
        
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>
  );
}

export default Modal;
```

### 卡片组件 (Card.js)
```javascript
import React from 'react';
import './Card.css';

function Card({ 
  title, 
  subtitle,
  children, 
  actions,
  image,
  hoverable = false,
  ...props 
}) {
  const getCardClass = () => {
    let classes = ['card'];
    if (hoverable) classes.push('card-hoverable');
    return classes.join(' ');
  };
  
  return (
    <div className={getCardClass()} {...props}>
      {image && (
        <div className="card-image">
          <img src={image.src} alt={image.alt} />
        </div>
      )}
      
      <div className="card-content">
        {(title || subtitle) && (
          <div className="card-header">
            {title && <h3 className="card-title">{title}</h3>}
            {subtitle && <p className="card-subtitle">{subtitle}</p>}
          </div>
        )}
        
        <div className="card-body">
          {children}
        </div>
        
        {actions && (
          <div className="card-actions">
            {actions}
          </div>
        )}
      </div>
    </div>
  );
}

export default Card;
```

### 使用组件库的示例应用
```javascript
import React, { useState } from 'react';
import Button from './components/Button';
import Input from './components/Input';
import Modal from './components/Modal';
import Card from './components/Card';

function ComponentLibraryDemo() {
  const [modalOpen, setModalOpen] = useState(false);
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  const [errors, setErrors] = useState({});
  const [loading, setLoading] = useState(false);
  
  const handleInputChange = (field) => (e) => {
    setFormData(prev => ({
      ...prev,
      [field]: e.target.value
    }));
    
    // 清除错误
    if (errors[field]) {
      setErrors(prev => ({
        ...prev,
        [field]: ''
      }));
    }
  };
  
  const validateForm = () => {
    const newErrors = {};
    
    if (!formData.name.trim()) {
      newErrors.name = '姓名不能为空';
    }
    
    if (!formData.email.trim()) {
      newErrors.email = '邮箱不能为空';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = '邮箱格式不正确';
    }
    
    if (!formData.message.trim()) {
      newErrors.message = '消息不能为空';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = async () => {
    if (!validateForm()) return;
    
    setLoading(true);
    
    // 模拟API调用
    await new Promise(resolve => setTimeout(resolve, 2000));
    
    setLoading(false);
    setModalOpen(false);
    alert('提交成功！');
    
    // 重置表单
    setFormData({ name: '', email: '', message: '' });
  };
  
  return (
    <div className="demo-app">
      <h1>组件库演示</h1>
      
      <div className="demo-section">
        <h2>按钮组件</h2>
        <div className="button-group">
          <Button variant="primary">主要按钮</Button>
          <Button variant="secondary">次要按钮</Button>
          <Button variant="success">成功按钮</Button>
          <Button variant="danger">危险按钮</Button>
          <Button disabled>禁用按钮</Button>
          <Button loading>加载中...</Button>
        </div>
        
        <div className="button-group">
          <Button size="small">小按钮</Button>
          <Button size="medium">中按钮</Button>
          <Button size="large">大按钮</Button>
        </div>
      </div>
      
      <div className="demo-section">
        <h2>卡片组件</h2>
        <div className="card-grid">
          <Card 
            title="基础卡片"
            subtitle="这是一个基础卡片示例"
            hoverable
          >
            <p>这里是卡片的内容区域，可以放置任何内容。</p>
          </Card>
          
          <Card 
            title="带操作的卡片"
            actions={
              <div>
                <Button size="small" variant="secondary">取消</Button>
                <Button size="small">确定</Button>
              </div>
            }
          >
            <p>这个卡片包含了操作按钮。</p>
          </Card>
          
          <Card 
            title="带图片的卡片"
            image={{
              src: 'https://via.placeholder.com/300x200',
              alt: '示例图片'
            }}
            actions={
              <Button size="small" onClick={() => setModalOpen(true)}>
                打开表单
              </Button>
            }
          >
            <p>这个卡片包含了图片和操作按钮。</p>
          </Card>
        </div>
      </div>
      
      <Modal
        isOpen={modalOpen}
        onClose={() => setModalOpen(false)}
        title="联系表单"
        size="medium"
      >
        <div className="form">
          <Input
            label="姓名"
            placeholder="请输入您的姓名"
            value={formData.name}
            onChange={handleInputChange('name')}
            error={errors.name}
            required
          />
          
          <Input
            label="邮箱"
            type="email"
            placeholder="请输入您的邮箱"
            value={formData.email}
            onChange={handleInputChange('email')}
            error={errors.email}
            helpText="我们不会泄露您的邮箱地址"
            required
          />
          
          <Input
            label="消息"
            placeholder="请输入您的消息"
            value={formData.message}
            onChange={handleInputChange('message')}
            error={errors.message}
            required
          />
          
          <div className="form-actions">
            <Button 
              variant="secondary" 
              onClick={() => setModalOpen(false)}
            >
              取消
            </Button>
            <Button 
              onClick={handleSubmit}
              loading={loading}
            >
              提交
            </Button>
          </div>
        </div>
      </Modal>
    </div>
  );
}

export default ComponentLibraryDemo;
```

## 3.7 组件设计原则

### 1. 单一职责原则
每个组件应该只负责一个功能。

```javascript
// ✅ 好的做法 - 单一职责
function UserAvatar({ src, alt, size = 'medium' }) {
  return (
    <img 
      className={`avatar avatar-${size}`}
      src={src} 
      alt={alt} 
    />
  );
}

function UserName({ name, role }) {
  return (
    <div className="user-name">
      <span className="name">{name}</span>
      {role && <span className="role">{role}</span>}
    </div>
  );
}

// ❌ 避免的做法 - 职责过多
function UserInfo({ user, onEdit, onDelete, showActions }) {
  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <span>{user.name}</span>
      <span>{user.role}</span>
      {showActions && (
        <div>
          <button onClick={onEdit}>编辑</button>
          <button onClick={onDelete}>删除</button>
        </div>
      )}
    </div>
  );
}
```

### 2. 可复用性
组件应该设计得足够通用，能在不同场景下使用。

```javascript
// ✅ 好的做法 - 可复用
function List({ items, renderItem, emptyMessage = '暂无数据' }) {
  if (!items.length) {
    return <div className="empty-state">{emptyMessage}</div>;
  }
  
  return (
    <ul className="list">
      {items.map((item, index) => (
        <li key={item.id || index}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}

// 使用
<List 
  items={users}
  renderItem={(user) => (
    <UserCard user={user} />
  )}
  emptyMessage="暂无用户"
/>
```

### 3. 组合优于继承
使用组合来构建复杂组件，而不是继承。

```javascript
// ✅ 好的做法 - 使用组合
function FormField({ label, error, children }) {
  return (
    <div className="form-field">
      <label className="label">{label}</label>
      {children}
      {error && <div className="error">{error}</div>}
    </div>
  );
}

// 使用
<FormField label="用户名" error={nameError}>
  <input 
    type="text" 
    value={username}
    onChange={handleUsernameChange}
  />
</FormField>
```

## 3.8 总结

### 本章重点
1. **组件概念**：React应用的基本构建块
2. **函数组件**：使用函数定义，简洁直观
3. **类组件**：使用ES6类定义，功能完整
4. **组件组合**：通过组合构建复杂界面
5. **组件通信**：Props传递和回调函数

### 关键概念理解
- **声明式组件**：描述UI应该是什么样子
- **Props流向**：数据自上而下流动
- **组件职责**：每个组件专注于特定功能
- **可复用性**：编写一次，多处使用

### 下章预告
下一章我们将深入学习Props和State，理解React中数据流动的核心机制。

### 练习作业
1. 创建一个完整的用户管理界面，包含用户列表、添加、编辑、删除功能
2. 设计一个简单的博客组件系统，包含文章列表、文章详情、评论等组件
3. 构建一个小型的电商产品展示页面，包含产品卡片、分类筛选、购物车等组件

### 常见问题
1. **Q: 什么时候使用函数组件，什么时候使用类组件？**
   A: 现在推荐优先使用函数组件配合Hooks，类组件主要用于特殊的生命周期需求。

2. **Q: 如何决定组件的粒度？**
   A: 遵循单一职责原则，一个组件只做一件事。如果组件变得复杂，考虑拆分。

3. **Q: 组件间通信有哪些方式？**
   A: Props传递、回调函数、Context API、状态管理库等。
