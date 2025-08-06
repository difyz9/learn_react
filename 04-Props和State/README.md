# 第四章：Props 和 State

## 学习目标
- 深入理解Props的概念和使用方法
- 掌握State的定义、更新和管理
- 理解单向数据流的重要性
- 学会处理不可变数据
- 掌握状态提升的技巧

## 4.1 Props 深入理解

### 什么是Props？
Props（properties的缩写）是组件的外部配置，类似于函数的参数。Props是只读的，组件不能修改自己的props。

### Props的特点
1. **只读性**：组件不能修改props
2. **单向数据流**：数据从父组件流向子组件
3. **类型多样**：可以传递任何JavaScript数据类型
4. **默认值**：可以设置默认属性值

### 基本Props使用
```javascript
function Welcome({ name, age, email }) {
  return (
    <div>
      <h1>欢迎，{name}！</h1>
      <p>年龄：{age}</p>
      <p>邮箱：{email}</p>
    </div>
  );
}

// 使用组件
<Welcome name="张三" age={25} email="zhangsan@example.com" />
```

### Props的类型

#### 1. 基本数据类型
```javascript
function DataTypeProps({ 
  str, 
  num, 
  bool, 
  nullValue, 
  undefinedValue 
}) {
  return (
    <div>
      <p>字符串：{str}</p>
      <p>数字：{num}</p>
      <p>布尔值：{bool ? '真' : '假'}</p>
      <p>空值：{nullValue || '无'}</p>
      <p>未定义：{undefinedValue || '无'}</p>
    </div>
  );
}

// 使用
<DataTypeProps 
  str="Hello"
  num={42}
  bool={true}
  nullValue={null}
  undefinedValue={undefined}
/>
```

#### 2. 对象和数组
```javascript
function ComplexProps({ user, hobbies, config }) {
  return (
    <div>
      <h2>用户信息</h2>
      <p>姓名：{user.name}</p>
      <p>年龄：{user.age}</p>
      
      <h3>爱好</h3>
      <ul>
        {hobbies.map((hobby, index) => (
          <li key={index}>{hobby}</li>
        ))}
      </ul>
      
      <h3>配置</h3>
      <p>主题：{config.theme}</p>
      <p>语言：{config.language}</p>
    </div>
  );
}

// 使用
const userData = { name: '李四', age: 30 };
const userHobbies = ['阅读', '游泳', '编程'];
const appConfig = { theme: 'dark', language: 'zh-CN' };

<ComplexProps 
  user={userData}
  hobbies={userHobbies}
  config={appConfig}
/>
```

#### 3. 函数作为Props
```javascript
function Counter({ count, onIncrement, onDecrement, onReset }) {
  return (
    <div>
      <p>计数：{count}</p>
      <button onClick={onIncrement}>+1</button>
      <button onClick={onDecrement}>-1</button>
      <button onClick={onReset}>重置</button>
    </div>
  );
}

// 父组件
function App() {
  const [count, setCount] = useState(0);
  
  return (
    <Counter
      count={count}
      onIncrement={() => setCount(count + 1)}
      onDecrement={() => setCount(count - 1)}
      onReset={() => setCount(0)}
    />
  );
}
```

#### 4. JSX作为Props
```javascript
function Card({ title, children, footer }) {
  return (
    <div className="card">
      <div className="card-header">
        <h3>{title}</h3>
      </div>
      <div className="card-body">
        {children}
      </div>
      {footer && (
        <div className="card-footer">
          {footer}
        </div>
      )}
    </div>
  );
}

// 使用
<Card 
  title="用户信息"
  footer={<button>保存</button>}
>
  <p>这里是卡片内容</p>
  <input type="text" placeholder="输入内容" />
</Card>
```

### Props默认值

#### 函数组件默认值
```javascript
// 方法1：参数默认值
function Greeting({ name = '访客', age = 0, showAge = true }) {
  return (
    <div>
      <h1>你好，{name}！</h1>
      {showAge && <p>年龄：{age}</p>}
    </div>
  );
}

// 方法2：defaultProps（类组件风格）
function UserCard({ name, email, role, avatar }) {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
      <span className="role">{role}</span>
    </div>
  );
}

UserCard.defaultProps = {
  avatar: '/default-avatar.png',
  role: '用户'
};
```

#### 类组件默认值
```javascript
class Welcome extends React.Component {
  static defaultProps = {
    name: '访客',
    age: 0,
    showAge: true
  };
  
  render() {
    const { name, age, showAge } = this.props;
    return (
      <div>
        <h1>你好，{name}！</h1>
        {showAge && <p>年龄：{age}</p>}
      </div>
    );
  }
}
```

### Props验证（PropTypes）
```javascript
import PropTypes from 'prop-types';

function UserProfile({ user, onEdit, showActions }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      {showActions && (
        <button onClick={() => onEdit(user.id)}>
          编辑
        </button>
      )}
    </div>
  );
}

UserProfile.propTypes = {
  user: PropTypes.shape({
    id: PropTypes.number.isRequired,
    name: PropTypes.string.isRequired,
    email: PropTypes.string.isRequired
  }).isRequired,
  onEdit: PropTypes.func,
  showActions: PropTypes.bool
};

UserProfile.defaultProps = {
  showActions: false
};
```

## 4.2 State 深入理解

### 什么是State？
State是组件的内部状态，用于存储组件的数据。与Props不同，State是可变的，组件可以通过setState（类组件）或useState（函数组件）来更新状态。

### State的特点
1. **可变性**：组件可以更新自己的状态
2. **私有性**：状态属于组件内部
3. **异步更新**：状态更新可能是异步的
4. **触发重渲染**：状态变化会触发组件重新渲染

### 函数组件中的State（useState）

#### 基本用法
```javascript
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>计数：{count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setCount(count - 1)}>-1</button>
      <button onClick={() => setCount(0)}>重置</button>
    </div>
  );
}
```

#### 多个状态变量
```javascript
function UserForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    
    try {
      // 模拟API调用
      await new Promise(resolve => setTimeout(resolve, 2000));
      console.log('提交数据：', { name, email, age });
      
      // 重置表单
      setName('');
      setEmail('');
      setAge(0);
    } catch (error) {
      console.error('提交失败：', error);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>姓名：</label>
        <input 
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />
      </div>
      
      <div>
        <label>邮箱：</label>
        <input 
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
      </div>
      
      <div>
        <label>年龄：</label>
        <input 
          type="number"
          value={age}
          onChange={(e) => setAge(parseInt(e.target.value) || 0)}
        />
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '提交中...' : '提交'}
      </button>
    </form>
  );
}
```

#### 对象状态
```javascript
function UserProfile() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0,
    preferences: {
      theme: 'light',
      language: 'zh-CN'
    }
  });
  
  const updateField = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };
  
  const updatePreference = (key, value) => {
    setUser(prevUser => ({
      ...prevUser,
      preferences: {
        ...prevUser.preferences,
        [key]: value
      }
    }));
  };
  
  return (
    <div>
      <input 
        placeholder="姓名"
        value={user.name}
        onChange={(e) => updateField('name', e.target.value)}
      />
      
      <input 
        placeholder="邮箱"
        value={user.email}
        onChange={(e) => updateField('email', e.target.value)}
      />
      
      <select 
        value={user.preferences.theme}
        onChange={(e) => updatePreference('theme', e.target.value)}
      >
        <option value="light">浅色</option>
        <option value="dark">深色</option>
      </select>
      
      <div>
        <h3>用户信息</h3>
        <pre>{JSON.stringify(user, null, 2)}</pre>
      </div>
    </div>
  );
}
```

#### 数组状态
```javascript
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState('');
  
  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos(prevTodos => [
        ...prevTodos,
        {
          id: Date.now(),
          text: inputValue,
          completed: false
        }
      ]);
      setInputValue('');
    }
  };
  
  const toggleTodo = (id) => {
    setTodos(prevTodos => 
      prevTodos.map(todo =>
        todo.id === id 
          ? { ...todo, completed: !todo.completed }
          : todo
      )
    );
  };
  
  const deleteTodo = (id) => {
    setTodos(prevTodos => 
      prevTodos.filter(todo => todo.id !== id)
    );
  };
  
  return (
    <div>
      <div>
        <input 
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="添加待办事项"
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
        />
        <button onClick={addTodo}>添加</button>
      </div>
      
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input 
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span 
              style={{ 
                textDecoration: todo.completed ? 'line-through' : 'none' 
              }}
            >
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>删除</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### 类组件中的State

#### 基本用法
```javascript
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  
  increment = () => {
    this.setState({ count: this.state.count + 1 });
  }
  
  decrement = () => {
    this.setState({ count: this.state.count - 1 });
  }
  
  reset = () => {
    this.setState({ count: 0 });
  }
  
  render() {
    return (
      <div>
        <p>计数：{this.state.count}</p>
        <button onClick={this.increment}>+1</button>
        <button onClick={this.decrement}>-1</button>
        <button onClick={this.reset}>重置</button>
      </div>
    );
  }
}
```

#### 函数式更新
```javascript
class AsyncCounter extends React.Component {
  state = { count: 0 };
  
  // 使用函数式更新确保状态正确
  incrementAsync = () => {
    this.setState(prevState => ({
      count: prevState.count + 1
    }));
  }
  
  // 批量更新示例
  multipleUpdates = () => {
    // 这些更新会被批量处理
    this.setState(prevState => ({ count: prevState.count + 1 }));
    this.setState(prevState => ({ count: prevState.count + 1 }));
    this.setState(prevState => ({ count: prevState.count + 1 }));
  }
  
  render() {
    return (
      <div>
        <p>计数：{this.state.count}</p>
        <button onClick={this.incrementAsync}>异步+1</button>
        <button onClick={this.multipleUpdates}>+3</button>
      </div>
    );
  }
}
```

## 4.3 单向数据流

### 理解单向数据流
React采用单向数据流（也称为单向绑定），数据从父组件流向子组件，子组件不能直接修改父组件的状态。

```javascript
function App() {
  const [userList, setUserList] = useState([
    { id: 1, name: '张三', active: true },
    { id: 2, name: '李四', active: false },
    { id: 3, name: '王五', active: true }
  ]);
  
  // 父组件控制数据更新
  const toggleUserStatus = (userId) => {
    setUserList(prevUsers => 
      prevUsers.map(user =>
        user.id === userId 
          ? { ...user, active: !user.active }
          : user
      )
    );
  };
  
  return (
    <div>
      <h1>用户管理</h1>
      <UserList 
        users={userList} 
        onToggleStatus={toggleUserStatus}
      />
    </div>
  );
}

function UserList({ users, onToggleStatus }) {
  return (
    <div>
      {users.map(user => (
        <UserItem 
          key={user.id}
          user={user}
          onToggle={() => onToggleStatus(user.id)}
        />
      ))}
    </div>
  );
}

function UserItem({ user, onToggle }) {
  return (
    <div className="user-item">
      <span>{user.name}</span>
      <span>状态：{user.active ? '活跃' : '非活跃'}</span>
      <button onClick={onToggle}>
        {user.active ? '禁用' : '启用'}
      </button>
    </div>
  );
}
```

### 数据流示意图
```
App (userList, toggleUserStatus)
  ↓ props
UserList (users, onToggleStatus)
  ↓ props
UserItem (user, onToggle)
  ↑ 事件回调
UserList
  ↑ 事件回调
App (更新 userList)
```

## 4.4 不可变数据

### 为什么需要不可变数据？
1. **性能优化**：React可以快速比较引用来决定是否重渲染
2. **调试容易**：状态变化轨迹清晰
3. **避免副作用**：防止意外修改数据

### 不可变更新模式

#### 数组不可变更新
```javascript
function ArrayStateExample() {
  const [items, setItems] = useState(['苹果', '香蕉', '橙子']);
  
  // 添加项目
  const addItem = (newItem) => {
    setItems(prevItems => [...prevItems, newItem]);
  };
  
  // 删除项目
  const removeItem = (index) => {
    setItems(prevItems => 
      prevItems.filter((_, i) => i !== index)
    );
  };
  
  // 更新项目
  const updateItem = (index, newValue) => {
    setItems(prevItems => 
      prevItems.map((item, i) => 
        i === index ? newValue : item
      )
    );
  };
  
  // 插入项目
  const insertItem = (index, newItem) => {
    setItems(prevItems => [
      ...prevItems.slice(0, index),
      newItem,
      ...prevItems.slice(index)
    ]);
  };
  
  return (
    <div>
      <ul>
        {items.map((item, index) => (
          <li key={index}>
            {item}
            <button onClick={() => removeItem(index)}>删除</button>
            <button onClick={() => updateItem(index, `${item}(已更新)`)}>
              更新
            </button>
          </li>
        ))}
      </ul>
      
      <button onClick={() => addItem('新水果')}>添加</button>
      <button onClick={() => insertItem(1, '插入的水果')}>插入</button>
    </div>
  );
}
```

#### 对象不可变更新
```javascript
function ObjectStateExample() {
  const [user, setUser] = useState({
    name: '张三',
    age: 25,
    address: {
      city: '北京',
      street: '长安街'
    },
    hobbies: ['阅读', '游泳']
  });
  
  // 更新顶层属性
  const updateName = (newName) => {
    setUser(prevUser => ({
      ...prevUser,
      name: newName
    }));
  };
  
  // 更新嵌套对象
  const updateCity = (newCity) => {
    setUser(prevUser => ({
      ...prevUser,
      address: {
        ...prevUser.address,
        city: newCity
      }
    }));
  };
  
  // 更新数组
  const addHobby = (newHobby) => {
    setUser(prevUser => ({
      ...prevUser,
      hobbies: [...prevUser.hobbies, newHobby]
    }));
  };
  
  const removeHobby = (index) => {
    setUser(prevUser => ({
      ...prevUser,
      hobbies: prevUser.hobbies.filter((_, i) => i !== index)
    }));
  };
  
  return (
    <div>
      <h3>用户信息</h3>
      <p>姓名：{user.name}</p>
      <p>年龄：{user.age}</p>
      <p>城市：{user.address.city}</p>
      <p>街道：{user.address.street}</p>
      
      <h4>爱好：</h4>
      <ul>
        {user.hobbies.map((hobby, index) => (
          <li key={index}>
            {hobby}
            <button onClick={() => removeHobby(index)}>删除</button>
          </li>
        ))}
      </ul>
      
      <button onClick={() => updateName('李四')}>更新姓名</button>
      <button onClick={() => updateCity('上海')}>更新城市</button>
      <button onClick={() => addHobby('编程')}>添加爱好</button>
    </div>
  );
}
```

### 使用Immer库简化不可变更新
```javascript
import { useImmer } from 'use-immer';

function ImmerExample() {
  const [user, updateUser] = useImmer({
    name: '张三',
    age: 25,
    address: {
      city: '北京',
      street: '长安街'
    },
    hobbies: ['阅读', '游泳']
  });
  
  const updateName = (newName) => {
    updateUser(draft => {
      draft.name = newName;
    });
  };
  
  const updateCity = (newCity) => {
    updateUser(draft => {
      draft.address.city = newCity;
    });
  };
  
  const addHobby = (newHobby) => {
    updateUser(draft => {
      draft.hobbies.push(newHobby);
    });
  };
  
  // ... 渲染逻辑
}
```

## 4.5 状态提升

### 什么是状态提升？
当多个组件需要共享同一状态时，将状态提升到它们的共同父组件中。

### 状态提升示例
```javascript
function TemperatureApp() {
  const [temperature, setTemperature] = useState('');
  const [scale, setScale] = useState('c');
  
  const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
  const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;
  
  return (
    <div>
      <TemperatureInput
        scale="c"
        temperature={celsius}
        onTemperatureChange={(temp) => {
          setTemperature(temp);
          setScale('c');
        }}
      />
      
      <TemperatureInput
        scale="f"
        temperature={fahrenheit}
        onTemperatureChange={(temp) => {
          setTemperature(temp);
          setScale('f');
        }}
      />
      
      <BoilingVerdict celsius={parseFloat(celsius)} />
    </div>
  );
}

function TemperatureInput({ scale, temperature, onTemperatureChange }) {
  const scaleNames = {
    c: '摄氏度',
    f: '华氏度'
  };
  
  return (
    <fieldset>
      <legend>请输入{scaleNames[scale]}温度：</legend>
      <input
        value={temperature}
        onChange={(e) => onTemperatureChange(e.target.value)}
      />
    </fieldset>
  );
}

function BoilingVerdict({ celsius }) {
  if (celsius >= 100) {
    return <p>水会沸腾。</p>;
  }
  return <p>水不会沸腾。</p>;
}

// 辅助函数
function toCelsius(fahrenheit) {
  return (fahrenheit - 32) * 5 / 9;
}

function toFahrenheit(celsius) {
  return (celsius * 9 / 5) + 32;
}

function tryConvert(temperature, convert) {
  const input = parseFloat(temperature);
  if (Number.isNaN(input)) {
    return '';
  }
  const output = convert(input);
  const rounded = Math.round(output * 1000) / 1000;
  return rounded.toString();
}
```

## 4.6 实践案例：计数器应用

让我们创建一个功能完整的计数器应用，展示Props和State的实际应用。

### 主应用组件
```javascript
import React, { useState } from 'react';
import Counter from './Counter';
import CounterList from './CounterList';
import './CounterApp.css';

function CounterApp() {
  const [counters, setCounters] = useState([
    { id: 1, value: 0, step: 1 },
    { id: 2, value: 0, step: 2 },
    { id: 3, value: 0, step: 5 }
  ]);
  
  const [globalSettings, setGlobalSettings] = useState({
    maxValue: 100,
    minValue: -100,
    defaultStep: 1
  });
  
  // 添加新计数器
  const addCounter = () => {
    const newId = Math.max(...counters.map(c => c.id)) + 1;
    setCounters(prev => [...prev, {
      id: newId,
      value: 0,
      step: globalSettings.defaultStep
    }]);
  };
  
  // 删除计数器
  const removeCounter = (id) => {
    setCounters(prev => prev.filter(counter => counter.id !== id));
  };
  
  // 更新计数器值
  const updateCounterValue = (id, newValue) => {
    const { maxValue, minValue } = globalSettings;
    const clampedValue = Math.max(minValue, Math.min(maxValue, newValue));
    
    setCounters(prev => 
      prev.map(counter =>
        counter.id === id 
          ? { ...counter, value: clampedValue }
          : counter
      )
    );
  };
  
  // 更新计数器步长
  const updateCounterStep = (id, newStep) => {
    setCounters(prev =>
      prev.map(counter =>
        counter.id === id
          ? { ...counter, step: Math.max(1, newStep) }
          : counter
      )
    );
  };
  
  // 重置所有计数器
  const resetAllCounters = () => {
    setCounters(prev =>
      prev.map(counter => ({ ...counter, value: 0 }))
    );
  };
  
  // 计算统计信息
  const totalValue = counters.reduce((sum, counter) => sum + counter.value, 0);
  const averageValue = counters.length > 0 ? totalValue / counters.length : 0;
  
  return (
    <div className="counter-app">
      <header className="app-header">
        <h1>多计数器应用</h1>
        
        <div className="global-settings">
          <h3>全局设置</h3>
          <div className="setting-group">
            <label>
              最大值：
              <input
                type="number"
                value={globalSettings.maxValue}
                onChange={(e) => setGlobalSettings(prev => ({
                  ...prev,
                  maxValue: parseInt(e.target.value) || 100
                }))}
              />
            </label>
            
            <label>
              最小值：
              <input
                type="number"
                value={globalSettings.minValue}
                onChange={(e) => setGlobalSettings(prev => ({
                  ...prev,
                  minValue: parseInt(e.target.value) || -100
                }))}
              />
            </label>
            
            <label>
              默认步长：
              <input
                type="number"
                min="1"
                value={globalSettings.defaultStep}
                onChange={(e) => setGlobalSettings(prev => ({
                  ...prev,
                  defaultStep: parseInt(e.target.value) || 1
                }))}
              />
            </label>
          </div>
        </div>
        
        <div className="statistics">
          <h3>统计信息</h3>
          <p>计数器数量：{counters.length}</p>
          <p>总值：{totalValue}</p>
          <p>平均值：{averageValue.toFixed(2)}</p>
        </div>
        
        <div className="global-actions">
          <button onClick={addCounter}>添加计数器</button>
          <button onClick={resetAllCounters}>重置所有</button>
        </div>
      </header>
      
      <main className="app-main">
        <CounterList
          counters={counters}
          onCounterChange={updateCounterValue}
          onStepChange={updateCounterStep}
          onCounterRemove={removeCounter}
          globalSettings={globalSettings}
        />
      </main>
    </div>
  );
}

export default CounterApp;
```

### 计数器列表组件
```javascript
import React from 'react';
import Counter from './Counter';

function CounterList({ 
  counters, 
  onCounterChange, 
  onStepChange, 
  onCounterRemove,
  globalSettings 
}) {
  if (counters.length === 0) {
    return (
      <div className="empty-state">
        <p>暂无计数器，点击"添加计数器"创建一个</p>
      </div>
    );
  }
  
  return (
    <div className="counter-list">
      {counters.map(counter => (
        <Counter
          key={counter.id}
          id={counter.id}
          value={counter.value}
          step={counter.step}
          maxValue={globalSettings.maxValue}
          minValue={globalSettings.minValue}
          onValueChange={(newValue) => onCounterChange(counter.id, newValue)}
          onStepChange={(newStep) => onStepChange(counter.id, newStep)}
          onRemove={() => onCounterRemove(counter.id)}
        />
      ))}
    </div>
  );
}

export default CounterList;
```

### 单个计数器组件
```javascript
import React, { useState, useEffect } from 'react';

function Counter({ 
  id, 
  value, 
  step, 
  maxValue, 
  minValue, 
  onValueChange, 
  onStepChange, 
  onRemove 
}) {
  const [isAnimating, setIsAnimating] = useState(false);
  const [history, setHistory] = useState([value]);
  
  // 记录值的变化历史
  useEffect(() => {
    setHistory(prev => [...prev.slice(-4), value]);
  }, [value]);
  
  const increment = () => {
    const newValue = value + step;
    if (newValue <= maxValue) {
      triggerAnimation();
      onValueChange(newValue);
    }
  };
  
  const decrement = () => {
    const newValue = value - step;
    if (newValue >= minValue) {
      triggerAnimation();
      onValueChange(newValue);
    }
  };
  
  const reset = () => {
    triggerAnimation();
    onValueChange(0);
  };
  
  const triggerAnimation = () => {
    setIsAnimating(true);
    setTimeout(() => setIsAnimating(false), 300);
  };
  
  const getProgressPercentage = () => {
    const range = maxValue - minValue;
    const position = value - minValue;
    return (position / range) * 100;
  };
  
  const isAtMax = value >= maxValue;
  const isAtMin = value <= minValue;
  
  return (
    <div className={`counter ${isAnimating ? 'animating' : ''}`}>
      <div className="counter-header">
        <h3>计数器 #{id}</h3>
        <button 
          className="remove-button"
          onClick={onRemove}
          title="删除计数器"
        >
          ×
        </button>
      </div>
      
      <div className="counter-display">
        <div className={`value ${isAnimating ? 'pulse' : ''}`}>
          {value}
        </div>
        <div className="progress-bar">
          <div 
            className="progress-fill"
            style={{ width: `${getProgressPercentage()}%` }}
          ></div>
        </div>
        <div className="range-info">
          <span>{minValue}</span>
          <span>{maxValue}</span>
        </div>
      </div>
      
      <div className="counter-controls">
        <button 
          onClick={decrement}
          disabled={isAtMin}
          className="control-button decrement"
        >
          -{step}
        </button>
        
        <button 
          onClick={reset}
          className="control-button reset"
        >
          重置
        </button>
        
        <button 
          onClick={increment}
          disabled={isAtMax}
          className="control-button increment"
        >
          +{step}
        </button>
      </div>
      
      <div className="counter-settings">
        <label>
          步长：
          <input
            type="number"
            min="1"
            max="10"
            value={step}
            onChange={(e) => onStepChange(parseInt(e.target.value) || 1)}
          />
        </label>
      </div>
      
      <div className="counter-history">
        <h4>历史记录</h4>
        <div className="history-list">
          {history.slice(-5).map((historyValue, index) => (
            <span key={index} className="history-item">
              {historyValue}
            </span>
          ))}
        </div>
      </div>
    </div>
  );
}

export default Counter;
```

### 样式文件 (CounterApp.css)
```css
.counter-app {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
}

.app-header {
  background: #f8f9fa;
  padding: 30px;
  border-radius: 12px;
  margin-bottom: 30px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.app-header h1 {
  color: #333;
  margin-bottom: 30px;
  text-align: center;
}

.global-settings {
  margin-bottom: 20px;
}

.global-settings h3 {
  color: #555;
  margin-bottom: 15px;
}

.setting-group {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
}

.setting-group label {
  display: flex;
  flex-direction: column;
  gap: 5px;
  font-weight: 500;
}

.setting-group input {
  padding: 8px;
  border: 2px solid #ddd;
  border-radius: 6px;
  width: 100px;
}

.statistics {
  background: white;
  padding: 20px;
  border-radius: 8px;
  margin: 20px 0;
  border-left: 4px solid #007bff;
}

.statistics h3 {
  margin-bottom: 10px;
  color: #007bff;
}

.statistics p {
  margin: 5px 0;
  color: #666;
}

.global-actions {
  display: flex;
  gap: 10px;
  justify-content: center;
}

.global-actions button {
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.global-actions button:first-child {
  background: #28a745;
  color: white;
}

.global-actions button:first-child:hover {
  background: #218838;
}

.global-actions button:last-child {
  background: #6c757d;
  color: white;
}

.global-actions button:last-child:hover {
  background: #545b62;
}

.counter-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 20px;
}

.counter {
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  border: 2px solid transparent;
  transition: all 0.3s ease;
}

.counter:hover {
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.15);
  border-color: #007bff;
}

.counter.animating {
  transform: scale(1.02);
}

.counter-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.counter-header h3 {
  margin: 0;
  color: #333;
}

.remove-button {
  background: #dc3545;
  color: white;
  border: none;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}

.remove-button:hover {
  background: #c82333;
}

.counter-display {
  text-align: center;
  margin-bottom: 20px;
}

.value {
  font-size: 48px;
  font-weight: bold;
  color: #007bff;
  margin-bottom: 15px;
  transition: all 0.3s ease;
}

.value.pulse {
  animation: pulse 0.3s ease-in-out;
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: #e9ecef;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 10px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #28a745, #007bff, #dc3545);
  transition: width 0.3s ease;
}

.range-info {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #6c757d;
}

.counter-controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.control-button {
  flex: 1;
  padding: 12px;
  border: none;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.control-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.decrement {
  background: #dc3545;
  color: white;
}

.decrement:hover:not(:disabled) {
  background: #c82333;
}

.increment {
  background: #28a745;
  color: white;
}

.increment:hover:not(:disabled) {
  background: #218838;
}

.reset {
  background: #6c757d;
  color: white;
}

.reset:hover {
  background: #545b62;
}

.counter-settings {
  margin-bottom: 15px;
}

.counter-settings label {
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 500;
}

.counter-settings input {
  width: 60px;
  padding: 6px;
  border: 2px solid #ddd;
  border-radius: 4px;
}

.counter-history h4 {
  margin: 0 0 10px 0;
  color: #555;
  font-size: 14px;
}

.history-list {
  display: flex;
  gap: 8px;
}

.history-item {
  background: #f8f9fa;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  color: #666;
  border: 1px solid #dee2e6;
}

.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: #6c757d;
  font-size: 18px;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .counter-list {
    grid-template-columns: 1fr;
  }
  
  .setting-group {
    flex-direction: column;
  }
  
  .global-actions {
    flex-direction: column;
  }
}
```

## 4.7 总结

### 本章重点
1. **Props**：组件的外部配置，只读不可变
2. **State**：组件的内部状态，可变且触发重渲染
3. **单向数据流**：数据从父组件流向子组件
4. **不可变数据**：通过创建新对象来更新状态
5. **状态提升**：将共享状态提升到共同父组件

### 关键概念理解
- **Props vs State**：外部配置 vs 内部状态
- **不可变更新**：保持数据不变性的重要性
- **状态设计**：如何合理设计组件状态结构
- **数据流动**：理解React中数据的流动方向

### 下章预告
下一章我们将学习React中的事件处理，包括合成事件、事件绑定、事件传参等核心概念。

### 练习作业
1. 创建一个购物车应用，实现商品添加、删除、数量修改、价格计算等功能
2. 构建一个学生成绩管理系统，包含学生列表、成绩录入、统计分析等功能
3. 实现一个简单的博客文章管理系统，支持文章的增删改查操作

### 常见问题
1. **Q: 什么时候使用Props，什么时候使用State？**
   A: Props用于接收外部数据，State用于管理组件内部变化的数据。

2. **Q: 为什么要保持数据不可变性？**
   A: 便于React进行性能优化，避免意外的副作用，使状态变化可追踪。

3. **Q: 如何决定状态应该放在哪个组件中？**
   A: 将状态放在需要该状态的所有组件的最近共同父组件中。
