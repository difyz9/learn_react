# 第七章：Hooks

## 学习目标
- 深入理解React Hooks的概念和意义
- 熟练掌握常用Hooks的使用方法
- 学会创建自定义Hook
- 理解Hook的规则和限制
- 能够使用Hooks重构类组件

## 7.1 Hooks 概述

### 什么是Hooks？
Hooks是React 16.8中引入的新特性，它允许在函数组件中使用state和其他React特性，而无需编写类组件。

### Hooks的优势
1. **更简洁的代码**：减少样板代码
2. **更好的逻辑复用**：通过自定义Hook
3. **更容易测试**：函数更容易测试
4. **更好的性能**：避免类组件的开销

### Hook规则
1. **只在顶层调用Hook**：不要在循环、条件或嵌套函数中调用
2. **只在React函数中调用Hook**：函数组件或自定义Hook中

```javascript
// ✅ 正确用法
function MyComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  useEffect(() => {
    // 副作用逻辑
  }, []);
  
  return <div>{count}</div>;
}

// ❌ 错误用法
function BadComponent() {
  if (someCondition) {
    const [count, setCount] = useState(0); // 违反规则1
  }
  
  return <div>Bad</div>;
}
```

## 7.2 useState Hook

### 基本用法
```javascript
import React, { useState } from 'react';

function Counter() {
  // 声明状态变量
  const [count, setCount] = useState(0);
  
  const increment = () => {
    setCount(count + 1);
  };
  
  const decrement = () => {
    setCount(prevCount => prevCount - 1); // 函数式更新
  };
  
  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

### 复杂状态管理
```javascript
function UserForm() {
  // 对象状态
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });
  
  // 数组状态
  const [hobbies, setHobbies] = useState([]);
  
  // 布尔状态
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const updateUser = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };
  
  const addHobby = (hobby) => {
    setHobbies(prevHobbies => [...prevHobbies, hobby]);
  };
  
  const removeHobby = (index) => {
    setHobbies(prevHobbies => 
      prevHobbies.filter((_, i) => i !== index)
    );
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    
    try {
      // 提交逻辑
      await submitUser(user, hobbies);
      // 重置表单
      setUser({ name: '', email: '', age: 0 });
      setHobbies([]);
    } catch (error) {
      console.error('提交失败:', error);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={user.name}
        onChange={(e) => updateUser('name', e.target.value)}
        placeholder="姓名"
      />
      
      <input
        type="email"
        value={user.email}
        onChange={(e) => updateUser('email', e.target.value)}
        placeholder="邮箱"
      />
      
      <input
        type="number"
        value={user.age}
        onChange={(e) => updateUser('age', parseInt(e.target.value) || 0)}
        placeholder="年龄"
      />
      
      <div>
        <h4>兴趣爱好:</h4>
        {hobbies.map((hobby, index) => (
          <span key={index}>
            {hobby}
            <button type="button" onClick={() => removeHobby(index)}>
              删除
            </button>
          </span>
        ))}
        <button 
          type="button" 
          onClick={() => addHobby('新爱好')}
        >
          添加爱好
        </button>
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '提交中...' : '提交'}
      </button>
    </form>
  );
}
```

### 懒初始化
```javascript
function ExpensiveComponent() {
  // 懒初始化：只在初始渲染时执行
  const [data, setData] = useState(() => {
    console.log('执行昂贵的初始化操作');
    return computeExpensiveValue();
  });
  
  return <div>{data}</div>;
}

function computeExpensiveValue() {
  // 模拟昂贵的计算
  let result = 0;
  for (let i = 0; i < 1000000; i++) {
    result += i;
  }
  return result;
}
```

## 7.3 useEffect Hook

### 基本副作用
```javascript
import React, { useState, useEffect } from 'react';

function DocumentTitle() {
  const [count, setCount] = useState(0);
  
  // 每次渲染后执行
  useEffect(() => {
    document.title = `计数: ${count}`;
  });
  
  return (
    <div>
      <p>当前计数: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        增加
      </button>
    </div>
  );
}
```

### 有依赖的副作用
```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    let isCancelled = false;
    
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        
        if (!isCancelled) {
          setUser(userData);
        }
      } catch (error) {
        if (!isCancelled) {
          console.error('获取用户失败:', error);
        }
      } finally {
        if (!isCancelled) {
          setLoading(false);
        }
      }
    };
    
    fetchUser();
    
    return () => {
      isCancelled = true;
    };
  }, [userId]); // 依赖userId
  
  if (loading) return <div>加载中...</div>;
  if (!user) return <div>用户不存在</div>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### 清理副作用
```javascript
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  
  useEffect(() => {
    let intervalId;
    
    if (isRunning) {
      intervalId = setInterval(() => {
        setSeconds(prevSeconds => prevSeconds + 1);
      }, 1000);
    }
    
    return () => {
      if (intervalId) {
        clearInterval(intervalId);
      }
    };
  }, [isRunning]);
  
  const toggleTimer = () => {
    setIsRunning(!isRunning);
  };
  
  const resetTimer = () => {
    setSeconds(0);
    setIsRunning(false);
  };
  
  return (
    <div>
      <h2>计时器: {seconds}秒</h2>
      <button onClick={toggleTimer}>
        {isRunning ? '暂停' : '开始'}
      </button>
      <button onClick={resetTimer}>重置</button>
    </div>
  );
}
```

## 7.4 useContext Hook

### 创建和使用Context
```javascript
import React, { createContext, useContext, useState } from 'react';

// 创建Context
const ThemeContext = createContext();
const UserContext = createContext();

// Provider组件
function AppProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState(null);
  
  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };
  
  const login = (userData) => {
    setUser(userData);
  };
  
  const logout = () => {
    setUser(null);
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <UserContext.Provider value={{ user, login, logout }}>
        {children}
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

// 自定义Hook简化Context使用
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme必须在ThemeProvider内使用');
  }
  return context;
}

function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser必须在UserProvider内使用');
  }
  return context;
}

// 使用Context的组件
function Header() {
  const { theme, toggleTheme } = useTheme();
  const { user, logout } = useUser();
  
  return (
    <header style={{
      backgroundColor: theme === 'light' ? '#fff' : '#333',
      color: theme === 'light' ? '#333' : '#fff',
      padding: '1rem'
    }}>
      <h1>我的应用</h1>
      <div>
        <button onClick={toggleTheme}>
          切换到{theme === 'light' ? '深色' : '浅色'}主题
        </button>
        
        {user ? (
          <div>
            <span>欢迎, {user.name}</span>
            <button onClick={logout}>登出</button>
          </div>
        ) : (
          <LoginForm />
        )}
      </div>
    </header>
  );
}

function LoginForm() {
  const { login } = useUser();
  const [username, setUsername] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (username.trim()) {
      login({ name: username });
      setUsername('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder="输入用户名"
      />
      <button type="submit">登录</button>
    </form>
  );
}

function MainContent() {
  const { theme } = useTheme();
  const { user } = useUser();
  
  return (
    <main style={{
      backgroundColor: theme === 'light' ? '#f5f5f5' : '#444',
      color: theme === 'light' ? '#333' : '#fff',
      padding: '2rem',
      minHeight: '500px'
    }}>
      <h2>主要内容</h2>
      {user ? (
        <p>欢迎回来, {user.name}!</p>
      ) : (
        <p>请先登录</p>
      )}
    </main>
  );
}

// 应用根组件
function App() {
  return (
    <AppProvider>
      <div>
        <Header />
        <MainContent />
      </div>
    </AppProvider>
  );
}
```

## 7.5 useReducer Hook

### 基本用法
```javascript
import React, { useReducer } from 'react';

// 定义初始状态
const initialState = {
  count: 0,
  step: 1
};

// 定义reducer函数
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + state.step };
    case 'DECREMENT':
      return { ...state, count: state.count - state.step };
    case 'RESET':
      return { ...state, count: 0 };
    case 'SET_STEP':
      return { ...state, step: action.payload };
    default:
      throw new Error(`未知的action类型: ${action.type}`);
  }
}

function AdvancedCounter() {
  const [state, dispatch] = useReducer(counterReducer, initialState);
  
  return (
    <div>
      <h2>高级计数器</h2>
      <p>当前值: {state.count}</p>
      <p>步长: {state.step}</p>
      
      <div>
        <button onClick={() => dispatch({ type: 'INCREMENT' })}>
          +{state.step}
        </button>
        <button onClick={() => dispatch({ type: 'DECREMENT' })}>
          -{state.step}
        </button>
        <button onClick={() => dispatch({ type: 'RESET' })}>
          重置
        </button>
      </div>
      
      <div>
        <label>
          步长:
          <input
            type="number"
            value={state.step}
            onChange={(e) => dispatch({
              type: 'SET_STEP',
              payload: parseInt(e.target.value) || 1
            })}
          />
        </label>
      </div>
    </div>
  );
}
```

### 复杂状态管理
```javascript
// 待办事项的reducer
const todoInitialState = {
  todos: [],
  filter: 'all',
  nextId: 1
};

function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: state.nextId,
            text: action.payload,
            completed: false,
            createdAt: new Date().toISOString()
          }
        ],
        nextId: state.nextId + 1
      };
      
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
      
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
      
    case 'EDIT_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, text: action.payload.text }
            : todo
        )
      };
      
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload
      };
      
    case 'CLEAR_COMPLETED':
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };
      
    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, todoInitialState);
  const [inputValue, setInputValue] = useState('');
  const [editingId, setEditingId] = useState(null);
  
  const addTodo = () => {
    if (inputValue.trim()) {
      dispatch({ type: 'ADD_TODO', payload: inputValue.trim() });
      setInputValue('');
    }
  };
  
  const filteredTodos = state.todos.filter(todo => {
    switch (state.filter) {
      case 'active':
        return !todo.completed;
      case 'completed':
        return todo.completed;
      default:
        return true;
    }
  });
  
  return (
    <div>
      <h2>待办事项应用</h2>
      
      {/* 添加新待办 */}
      <div>
        <input
          type="text"
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          placeholder="添加新的待办事项"
        />
        <button onClick={addTodo}>添加</button>
      </div>
      
      {/* 过滤器 */}
      <div>
        {['all', 'active', 'completed'].map(filter => (
          <button
            key={filter}
            onClick={() => dispatch({ type: 'SET_FILTER', payload: filter })}
            style={{
              fontWeight: state.filter === filter ? 'bold' : 'normal'
            }}
          >
            {filter === 'all' ? '全部' : 
             filter === 'active' ? '未完成' : '已完成'}
          </button>
        ))}
      </div>
      
      {/* 待办列表 */}
      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}
            />
            
            {editingId === todo.id ? (
              <input
                type="text"
                defaultValue={todo.text}
                onBlur={(e) => {
                  dispatch({
                    type: 'EDIT_TODO',
                    payload: { id: todo.id, text: e.target.value }
                  });
                  setEditingId(null);
                }}
                onKeyPress={(e) => {
                  if (e.key === 'Enter') {
                    dispatch({
                      type: 'EDIT_TODO',
                      payload: { id: todo.id, text: e.target.value }
                    });
                    setEditingId(null);
                  }
                }}
                autoFocus
              />
            ) : (
              <span
                onDoubleClick={() => setEditingId(todo.id)}
                style={{
                  textDecoration: todo.completed ? 'line-through' : 'none'
                }}
              >
                {todo.text}
              </span>
            )}
            
            <button onClick={() => dispatch({ type: 'DELETE_TODO', payload: todo.id })}>
              删除
            </button>
          </li>
        ))}
      </ul>
      
      {/* 统计信息 */}
      <div>
        <p>总计: {state.todos.length}</p>
        <p>已完成: {state.todos.filter(t => t.completed).length}</p>
        <p>未完成: {state.todos.filter(t => !t.completed).length}</p>
        
        {state.todos.some(t => t.completed) && (
          <button onClick={() => dispatch({ type: 'CLEAR_COMPLETED' })}>
            清除已完成
          </button>
        )}
      </div>
    </div>
  );
}
```

## 7.6 性能优化 Hooks

### useMemo Hook
```javascript
import React, { useState, useMemo } from 'react';

function ExpensiveCalculation({ items, multiplier }) {
  // 昂贵的计算，只有当依赖项改变时才重新计算
  const expensiveValue = useMemo(() => {
    console.log('执行昂贵的计算...');
    return items.reduce((sum, item) => sum + item.value, 0) * multiplier;
  }, [items, multiplier]);
  
  const [localCount, setLocalCount] = useState(0);
  
  return (
    <div>
      <p>计算结果: {expensiveValue}</p>
      <p>本地计数: {localCount}</p>
      <button onClick={() => setLocalCount(localCount + 1)}>
        增加本地计数（不会触发重新计算）
      </button>
    </div>
  );
}
```

### useCallback Hook
```javascript
import React, { useState, useCallback, memo } from 'react';

// 子组件使用memo优化
const ChildComponent = memo(({ onButtonClick, label }) => {
  console.log(`${label} 组件重新渲染`);
  
  return (
    <button onClick={onButtonClick}>
      {label}
    </button>
  );
});

function ParentComponent() {
  const [count1, setCount1] = useState(0);
  const [count2, setCount2] = useState(0);
  
  // 使用useCallback优化回调函数
  const handleClick1 = useCallback(() => {
    setCount1(prev => prev + 1);
  }, []); // 没有依赖，函数永远不变
  
  const handleClick2 = useCallback(() => {
    setCount2(prev => prev + 1);
  }, []); // 没有依赖，函数永远不变
  
  // 不使用useCallback的对比
  const handleClick3 = () => {
    console.log('这个函数每次都会重新创建');
  };
  
  return (
    <div>
      <p>计数1: {count1}</p>
      <p>计数2: {count2}</p>
      
      <ChildComponent 
        onButtonClick={handleClick1} 
        label="计数器1（优化）" 
      />
      
      <ChildComponent 
        onButtonClick={handleClick2} 
        label="计数器2（优化）" 
      />
      
      <ChildComponent 
        onButtonClick={handleClick3} 
        label="未优化按钮" 
      />
    </div>
  );
}
```

## 7.7 自定义Hook

### 基础自定义Hook
```javascript
// 自定义Hook: 本地存储
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error('读取localStorage失败:', error);
      return initialValue;
    }
  });
  
  const setValue = (value) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('写入localStorage失败:', error);
    }
  };
  
  return [storedValue, setValue];
}

// 自定义Hook: 网络请求
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let isCancelled = false;
    
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(url, options);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const result = await response.json();
        
        if (!isCancelled) {
          setData(result);
        }
      } catch (err) {
        if (!isCancelled) {
          setError(err);
        }
      } finally {
        if (!isCancelled) {
          setLoading(false);
        }
      }
    };
    
    fetchData();
    
    return () => {
      isCancelled = true;
    };
  }, [url, JSON.stringify(options)]);
  
  return { data, loading, error };
}

// 自定义Hook: 防抖
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// 使用自定义Hook的组件
function CustomHookExample() {
  const [name, setName] = useLocalStorage('name', '');
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 300);
  
  const { data, loading, error } = useFetch(
    debouncedSearchTerm 
      ? `/api/search?q=${debouncedSearchTerm}`
      : '/api/items'
  );
  
  return (
    <div>
      <div>
        <label>
          姓名 (保存到localStorage):
          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
          />
        </label>
      </div>
      
      <div>
        <label>
          搜索 (防抖):
          <input
            type="text"
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
            placeholder="输入搜索关键词"
          />
        </label>
      </div>
      
      <div>
        {loading && <p>加载中...</p>}
        {error && <p>错误: {error.message}</p>}
        {data && (
          <ul>
            {data.map(item => (
              <li key={item.id}>{item.name}</li>
            ))}
          </ul>
        )}
      </div>
    </div>
  );
}
```

### 高级自定义Hook
```javascript
// 自定义Hook: 表单处理
function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const setValue = (name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
    
    // 如果字段已经被触摸过，立即验证
    if (touched[name] && validate) {
      const fieldErrors = validate({ ...values, [name]: value });
      setErrors(prev => ({ ...prev, [name]: fieldErrors[name] }));
    }
  };
  
  const setTouched = (name) => {
    setTouched(prev => ({ ...prev, [name]: true }));
    
    // 验证字段
    if (validate) {
      const fieldErrors = validate(values);
      setErrors(prev => ({ ...prev, [name]: fieldErrors[name] }));
    }
  };
  
  const handleSubmit = async (onSubmit) => {
    setIsSubmitting(true);
    
    // 标记所有字段为已触摸
    const allTouched = Object.keys(values).reduce((acc, key) => {
      acc[key] = true;
      return acc;
    }, {});
    setTouched(allTouched);
    
    // 验证所有字段
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      
      // 如果有错误，不提交
      if (Object.keys(validationErrors).some(key => validationErrors[key])) {
        setIsSubmitting(false);
        return;
      }
    }
    
    try {
      await onSubmit(values);
      // 重置表单
      setValues(initialValues);
      setErrors({});
      setTouched({});
    } catch (error) {
      console.error('提交失败:', error);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  };
  
  return {
    values,
    errors,
    touched,
    isSubmitting,
    setValue,
    setTouched,
    handleSubmit,
    reset
  };
}

// 使用表单Hook
function RegistrationForm() {
  const initialValues = {
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  };
  
  const validate = (values) => {
    const errors = {};
    
    if (!values.username) {
      errors.username = '用户名不能为空';
    } else if (values.username.length < 3) {
      errors.username = '用户名至少3个字符';
    }
    
    if (!values.email) {
      errors.email = '邮箱不能为空';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      errors.email = '邮箱格式不正确';
    }
    
    if (!values.password) {
      errors.password = '密码不能为空';
    } else if (values.password.length < 6) {
      errors.password = '密码至少6个字符';
    }
    
    if (values.password !== values.confirmPassword) {
      errors.confirmPassword = '两次输入的密码不一致';
    }
    
    return errors;
  };
  
  const {
    values,
    errors,
    touched,
    isSubmitting,
    setValue,
    setTouched,
    handleSubmit,
    reset
  } = useForm(initialValues, validate);
  
  const onSubmit = async (formData) => {
    // 模拟API调用
    await new Promise(resolve => setTimeout(resolve, 2000));
    console.log('注册成功:', formData);
    alert('注册成功！');
  };
  
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      handleSubmit(onSubmit);
    }}>
      <div>
        <label>用户名:</label>
        <input
          type="text"
          value={values.username}
          onChange={(e) => setValue('username', e.target.value)}
          onBlur={() => setTouched('username')}
        />
        {touched.username && errors.username && (
          <span style={{ color: 'red' }}>{errors.username}</span>
        )}
      </div>
      
      <div>
        <label>邮箱:</label>
        <input
          type="email"
          value={values.email}
          onChange={(e) => setValue('email', e.target.value)}
          onBlur={() => setTouched('email')}
        />
        {touched.email && errors.email && (
          <span style={{ color: 'red' }}>{errors.email}</span>
        )}
      </div>
      
      <div>
        <label>密码:</label>
        <input
          type="password"
          value={values.password}
          onChange={(e) => setValue('password', e.target.value)}
          onBlur={() => setTouched('password')}
        />
        {touched.password && errors.password && (
          <span style={{ color: 'red' }}>{errors.password}</span>
        )}
      </div>
      
      <div>
        <label>确认密码:</label>
        <input
          type="password"
          value={values.confirmPassword}
          onChange={(e) => setValue('confirmPassword', e.target.value)}
          onBlur={() => setTouched('confirmPassword')}
        />
        {touched.confirmPassword && errors.confirmPassword && (
          <span style={{ color: 'red' }}>{errors.confirmPassword}</span>
        )}
      </div>
      
      <div>
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? '注册中...' : '注册'}
        </button>
        <button type="button" onClick={reset}>
          重置
        </button>
      </div>
    </form>
  );
}
```

## 7.8 实践案例：Hooks重构类组件

让我们将之前的类组件重构为使用Hooks的函数组件。

### 重构前的类组件
```javascript
class OldDataComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      data: null,
      loading: true,
      error: null,
      page: 1,
      hasMore: true
    };
  }
  
  componentDidMount() {
    this.fetchData();
    window.addEventListener('scroll', this.handleScroll);
  }
  
  componentDidUpdate(prevProps) {
    if (prevProps.category !== this.props.category) {
      this.fetchData(true);
    }
  }
  
  componentWillUnmount() {
    window.removeEventListener('scroll', this.handleScroll);
    if (this.abortController) {
      this.abortController.abort();
    }
  }
  
  handleScroll = () => {
    if (window.innerHeight + window.scrollY >= document.body.offsetHeight - 1000) {
      this.loadMore();
    }
  }
  
  fetchData = async (reset = false) => {
    if (this.abortController) {
      this.abortController.abort();
    }
    
    this.abortController = new AbortController();
    
    try {
      this.setState({ loading: true, error: null });
      
      const response = await fetch(`/api/data?category=${this.props.category}&page=${reset ? 1 : this.state.page}`, {
        signal: this.abortController.signal
      });
      
      const result = await response.json();
      
      this.setState(prevState => ({
        data: reset ? result.data : [...(prevState.data || []), ...result.data],
        loading: false,
        page: reset ? 2 : prevState.page + 1,
        hasMore: result.hasMore
      }));
    } catch (error) {
      if (error.name !== 'AbortError') {
        this.setState({ error: error.message, loading: false });
      }
    }
  }
  
  loadMore = () => {
    if (!this.state.loading && this.state.hasMore) {
      this.fetchData();
    }
  }
  
  render() {
    const { data, loading, error } = this.state;
    
    return (
      <div>
        {error && <div>错误: {error}</div>}
        {data && data.map(item => (
          <div key={item.id}>{item.name}</div>
        ))}
        {loading && <div>加载中...</div>}
      </div>
    );
  }
}
```

### 重构后的函数组件
```javascript
function NewDataComponent({ category }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  
  const abortControllerRef = useRef(null);
  
  // 自定义Hook: 无限滚动
  const useInfiniteScroll = (callback, hasMore, loading) => {
    useEffect(() => {
      const handleScroll = () => {
        if (
          window.innerHeight + window.scrollY >= 
          document.body.offsetHeight - 1000 &&
          hasMore &&
          !loading
        ) {
          callback();
        }
      };
      
      window.addEventListener('scroll', handleScroll);
      return () => window.removeEventListener('scroll', handleScroll);
    }, [callback, hasMore, loading]);
  };
  
  // 获取数据的函数
  const fetchData = useCallback(async (reset = false) => {
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
    
    abortControllerRef.current = new AbortController();
    
    try {
      setLoading(true);
      setError(null);
      
      const currentPage = reset ? 1 : page;
      const response = await fetch(
        `/api/data?category=${category}&page=${currentPage}`,
        { signal: abortControllerRef.current.signal }
      );
      
      const result = await response.json();
      
      setData(prevData => 
        reset ? result.data : [...(prevData || []), ...result.data]
      );
      setPage(reset ? 2 : page + 1);
      setHasMore(result.hasMore);
    } catch (error) {
      if (error.name !== 'AbortError') {
        setError(error.message);
      }
    } finally {
      setLoading(false);
    }
  }, [category, page]);
  
  // 加载更多数据
  const loadMore = useCallback(() => {
    fetchData();
  }, [fetchData]);
  
  // 组件挂载时获取数据
  useEffect(() => {
    fetchData(true);
    
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, []); // eslint-disable-line react-hooks/exhaustive-deps
  
  // 当category改变时重新获取数据
  useEffect(() => {
    setPage(1);
    setData(null);
    fetchData(true);
  }, [category]); // eslint-disable-line react-hooks/exhaustive-deps
  
  // 使用无限滚动Hook
  useInfiniteScroll(loadMore, hasMore, loading);
  
  return (
    <div>
      {error && <div>错误: {error}</div>}
      {data && data.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
      {loading && <div>加载中...</div>}
    </div>
  );
}
```

## 7.9 总结

### 本章重点
1. **Hook概念**：函数组件中使用状态和副作用的方式
2. **常用Hook**：useState、useEffect、useContext、useReducer
3. **性能优化**：useMemo、useCallback优化渲染性能
4. **自定义Hook**：封装可复用的状态逻辑
5. **重构技巧**：将类组件重构为函数组件

### 关键概念理解
- **Hook规则**：只在顶层调用，只在React函数中调用
- **依赖数组**：控制useEffect和其他Hook的执行时机
- **闭包陷阱**：理解Hook中的闭包问题及解决方案
- **性能优化**：合理使用memo、useMemo、useCallback

### 下章预告
下一章我们将学习状态管理，包括Context API、useReducer模式、Redux等复杂应用的状态管理方案。

### 练习作业
1. 创建一个购物车Hook，管理商品的添加、删除、数量修改等操作
2. 实现一个主题切换Hook，支持多种主题和本地存储
3. 构建一个数据缓存Hook，支持请求缓存和失效策略

### 常见问题
1. **Q: 什么时候使用useReducer而不是useState？**
   A: 当状态逻辑复杂，涉及多个子值或下一个状态依赖于之前的状态时。

2. **Q: 如何避免useEffect的无限循环？**
   A: 正确设置依赖数组，避免在依赖中包含每次都变化的值。

3. **Q: 自定义Hook和普通函数的区别？**
   A: 自定义Hook可以调用其他Hook，遵循Hook规则，用于共享状态逻辑。
