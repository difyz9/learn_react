# 第五章：事件处理

## 学习目标
- 理解React中的合成事件系统
- 掌握事件绑定的各种方法
- 学会事件传参和阻止默认行为
- 了解事件委托和性能优化
- 能够处理各种常见的用户交互

## 5.1 React事件系统概述

### 合成事件（SyntheticEvent）
React使用合成事件来包装原生DOM事件，提供了跨浏览器的一致性API。

#### 合成事件的特点
1. **跨浏览器兼容**：统一的事件接口
2. **事件委托**：React在根节点统一处理事件
3. **事件对象**：包含原生事件的所有属性和方法
4. **性能优化**：事件对象可以被重用

```javascript
function EventDemo() {
  const handleClick = (event) => {
    console.log('事件类型:', event.type);
    console.log('目标元素:', event.target);
    console.log('当前元素:', event.currentTarget);
    console.log('原生事件:', event.nativeEvent);
    
    // 阻止默认行为
    event.preventDefault();
    
    // 阻止事件冒泡
    event.stopPropagation();
  };
  
  return (
    <button onClick={handleClick}>
      点击我查看事件对象
    </button>
  );
}
```

### 常见事件类型

#### 鼠标事件
```javascript
function MouseEvents() {
  const handleMouseEvent = (eventName) => (event) => {
    console.log(`${eventName}:`, {
      clientX: event.clientX,
      clientY: event.clientY,
      button: event.button,
      buttons: event.buttons
    });
  };
  
  return (
    <div 
      style={{ 
        width: '200px', 
        height: '200px', 
        background: '#f0f0f0',
        border: '1px solid #ccc',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center'
      }}
      onClick={handleMouseEvent('click')}
      onDoubleClick={handleMouseEvent('doubleClick')}
      onMouseDown={handleMouseEvent('mouseDown')}
      onMouseUp={handleMouseEvent('mouseUp')}
      onMouseEnter={handleMouseEvent('mouseEnter')}
      onMouseLeave={handleMouseEvent('mouseLeave')}
      onMouseMove={handleMouseEvent('mouseMove')}
      onContextMenu={handleMouseEvent('contextMenu')}
    >
      鼠标事件区域
    </div>
  );
}
```

#### 键盘事件
```javascript
function KeyboardEvents() {
  const [inputValue, setInputValue] = useState('');
  const [keyLog, setKeyLog] = useState([]);
  
  const handleKeyEvent = (eventName) => (event) => {
    const logEntry = {
      event: eventName,
      key: event.key,
      code: event.code,
      keyCode: event.keyCode,
      ctrlKey: event.ctrlKey,
      shiftKey: event.shiftKey,
      altKey: event.altKey,
      metaKey: event.metaKey
    };
    
    setKeyLog(prev => [logEntry, ...prev.slice(0, 9)]);
    
    // 处理特殊按键
    if (event.key === 'Enter') {
      console.log('回车键被按下');
    } else if (event.key === 'Escape') {
      setInputValue('');
      console.log('ESC键被按下，清空输入');
    } else if (event.ctrlKey && event.key === 's') {
      event.preventDefault();
      console.log('Ctrl+S 保存快捷键');
    }
  };
  
  return (
    <div>
      <input
        type="text"
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        onKeyDown={handleKeyEvent('keyDown')}
        onKeyUp={handleKeyEvent('keyUp')}
        onKeyPress={handleKeyEvent('keyPress')}
        placeholder="输入文本，尝试各种按键..."
        style={{ width: '300px', padding: '10px' }}
      />
      
      <div style={{ marginTop: '20px' }}>
        <h4>按键日志：</h4>
        <div style={{ maxHeight: '200px', overflow: 'auto' }}>
          {keyLog.map((log, index) => (
            <div key={index} style={{ 
              fontSize: '12px', 
              padding: '5px',
              backgroundColor: index % 2 === 0 ? '#f9f9f9' : 'white'
            }}>
              {log.event}: {log.key} (code: {log.code})
              {log.ctrlKey && ' [Ctrl]'}
              {log.shiftKey && ' [Shift]'}
              {log.altKey && ' [Alt]'}
              {log.metaKey && ' [Meta]'}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
```

#### 表单事件
```javascript
function FormEvents() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    gender: '',
    interests: [],
    agreement: false
  });
  
  const [focusedField, setFocusedField] = useState('');
  
  const handleInputChange = (event) => {
    const { name, value, type, checked } = event.target;
    
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };
  
  const handleMultiSelect = (event) => {
    const { value, checked } = event.target;
    
    setFormData(prev => ({
      ...prev,
      interests: checked 
        ? [...prev.interests, value]
        : prev.interests.filter(interest => interest !== value)
    }));
  };
  
  const handleSubmit = (event) => {
    event.preventDefault();
    console.log('表单提交:', formData);
    alert('表单提交成功！查看控制台');
  };
  
  const handleReset = (event) => {
    console.log('表单重置');
    setFormData({
      username: '',
      email: '',
      password: '',
      gender: '',
      interests: [],
      agreement: false
    });
  };
  
  return (
    <form onSubmit={handleSubmit} onReset={handleReset}>
      <div>
        <label>
          用户名：
          <input
            type="text"
            name="username"
            value={formData.username}
            onChange={handleInputChange}
            onFocus={(e) => setFocusedField(e.target.name)}
            onBlur={() => setFocusedField('')}
            required
          />
          {focusedField === 'username' && (
            <span style={{ color: 'blue' }}>（正在输入用户名）</span>
          )}
        </label>
      </div>
      
      <div>
        <label>
          邮箱：
          <input
            type="email"
            name="email"
            value={formData.email}
            onChange={handleInputChange}
            onFocus={(e) => setFocusedField(e.target.name)}
            onBlur={() => setFocusedField('')}
            required
          />
        </label>
      </div>
      
      <div>
        <label>
          密码：
          <input
            type="password"
            name="password"
            value={formData.password}
            onChange={handleInputChange}
            onFocus={(e) => setFocusedField(e.target.name)}
            onBlur={() => setFocusedField('')}
            required
          />
        </label>
      </div>
      
      <div>
        <label>性别：</label>
        <label>
          <input
            type="radio"
            name="gender"
            value="male"
            checked={formData.gender === 'male'}
            onChange={handleInputChange}
          />
          男
        </label>
        <label>
          <input
            type="radio"
            name="gender"
            value="female"
            checked={formData.gender === 'female'}
            onChange={handleInputChange}
          />
          女
        </label>
      </div>
      
      <div>
        <label>兴趣爱好：</label>
        {['阅读', '游泳', '编程', '音乐'].map(interest => (
          <label key={interest}>
            <input
              type="checkbox"
              value={interest}
              checked={formData.interests.includes(interest)}
              onChange={handleMultiSelect}
            />
            {interest}
          </label>
        ))}
      </div>
      
      <div>
        <label>
          <input
            type="checkbox"
            name="agreement"
            checked={formData.agreement}
            onChange={handleInputChange}
            required
          />
          我同意用户协议
        </label>
      </div>
      
      <div>
        <button type="submit">提交</button>
        <button type="reset">重置</button>
      </div>
      
      <div style={{ marginTop: '20px' }}>
        <h4>表单数据预览：</h4>
        <pre>{JSON.stringify(formData, null, 2)}</pre>
      </div>
    </form>
  );
}
```

## 5.2 事件绑定方法

### 函数组件中的事件绑定

#### 内联箭头函数
```javascript
function InlineEventBinding() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>计数: {count}</p>
      {/* 内联箭头函数 */}
      <button onClick={() => setCount(count + 1)}>
        增加
      </button>
      
      {/* 内联箭头函数传参 */}
      <button onClick={(e) => {
        console.log('事件对象:', e);
        setCount(count - 1);
      }}>
        减少
      </button>
    </div>
  );
}
```

#### 函数引用
```javascript
function FunctionReference() {
  const [message, setMessage] = useState('');
  
  // 事件处理函数
  const handleClick = () => {
    setMessage('按钮被点击了！');
  };
  
  const handleInputChange = (event) => {
    setMessage(event.target.value);
  };
  
  const handleClear = () => {
    setMessage('');
  };
  
  return (
    <div>
      <input 
        type="text" 
        value={message}
        onChange={handleInputChange}
        placeholder="输入消息"
      />
      <button onClick={handleClick}>设置消息</button>
      <button onClick={handleClear}>清空</button>
      <p>消息: {message}</p>
    </div>
  );
}
```

#### useCallback优化
```javascript
import React, { useState, useCallback } from 'react';

function OptimizedEventBinding() {
  const [count, setCount] = useState(0);
  const [multiplier, setMultiplier] = useState(1);
  
  // 使用useCallback优化事件处理函数
  const handleIncrement = useCallback(() => {
    setCount(prev => prev + multiplier);
  }, [multiplier]);
  
  const handleDecrement = useCallback(() => {
    setCount(prev => prev - multiplier);
  }, [multiplier]);
  
  const handleReset = useCallback(() => {
    setCount(0);
  }, []);
  
  const handleMultiplierChange = useCallback((event) => {
    setMultiplier(parseInt(event.target.value) || 1);
  }, []);
  
  return (
    <div>
      <p>计数: {count}</p>
      <p>倍数: {multiplier}</p>
      
      <input
        type="number"
        value={multiplier}
        onChange={handleMultiplierChange}
        min="1"
      />
      
      <button onClick={handleIncrement}>增加</button>
      <button onClick={handleDecrement}>减少</button>
      <button onClick={handleReset}>重置</button>
    </div>
  );
}
```

### 类组件中的事件绑定

#### 构造函数中绑定
```javascript
class ConstructorBinding extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    
    // 在构造函数中绑定this
    this.handleIncrement = this.handleIncrement.bind(this);
    this.handleDecrement = this.handleDecrement.bind(this);
  }
  
  handleIncrement() {
    this.setState(prevState => ({
      count: prevState.count + 1
    }));
  }
  
  handleDecrement() {
    this.setState(prevState => ({
      count: prevState.count - 1
    }));
  }
  
  render() {
    return (
      <div>
        <p>计数: {this.state.count}</p>
        <button onClick={this.handleIncrement}>+</button>
        <button onClick={this.handleDecrement}>-</button>
      </div>
    );
  }
}
```

#### 箭头函数方法
```javascript
class ArrowFunctionBinding extends React.Component {
  state = { count: 0 };
  
  // 箭头函数自动绑定this
  handleIncrement = () => {
    this.setState(prevState => ({
      count: prevState.count + 1
    }));
  }
  
  handleDecrement = () => {
    this.setState(prevState => ({
      count: prevState.count - 1
    }));
  }
  
  render() {
    return (
      <div>
        <p>计数: {this.state.count}</p>
        <button onClick={this.handleIncrement}>+</button>
        <button onClick={this.handleDecrement}>-</button>
      </div>
    );
  }
}
```

## 5.3 事件传参

### 基本传参方法
```javascript
function EventParameterExample() {
  const [selectedItem, setSelectedItem] = useState(null);
  const [actionLog, setActionLog] = useState([]);
  
  const items = [
    { id: 1, name: '苹果', price: 5 },
    { id: 2, name: '香蕉', price: 3 },
    { id: 3, name: '橙子', price: 4 }
  ];
  
  const handleItemClick = (item, action) => {
    setSelectedItem(item);
    
    const logEntry = {
      timestamp: new Date().toLocaleTimeString(),
      action,
      item: item.name
    };
    
    setActionLog(prev => [logEntry, ...prev.slice(0, 4)]);
  };
  
  const handleQuantityChange = (itemId, quantity) => {
    console.log(`商品 ${itemId} 数量变更为: ${quantity}`);
  };
  
  return (
    <div>
      <h3>商品列表</h3>
      {items.map(item => (
        <div key={item.id} style={{ 
          padding: '10px', 
          margin: '5px', 
          border: '1px solid #ccc',
          backgroundColor: selectedItem?.id === item.id ? '#e3f2fd' : 'white'
        }}>
          <span>{item.name} - ¥{item.price}</span>
          
          {/* 方法1: 箭头函数传参 */}
          <button onClick={() => handleItemClick(item, '选择')}>
            选择
          </button>
          
          {/* 方法2: 箭头函数传多个参数 */}
          <button onClick={() => handleItemClick(item, '收藏')}>
            收藏
          </button>
          
          {/* 方法3: 传递事件对象和参数 */}
          <button onClick={(e) => {
            e.stopPropagation();
            handleItemClick(item, '删除');
          }}>
            删除
          </button>
          
          {/* 方法4: 数字输入框传参 */}
          <input
            type="number"
            min="1"
            defaultValue="1"
            onChange={(e) => handleQuantityChange(item.id, e.target.value)}
            style={{ width: '60px', marginLeft: '10px' }}
          />
        </div>
      ))}
      
      {selectedItem && (
        <div style={{ marginTop: '20px', padding: '10px', backgroundColor: '#f5f5f5' }}>
          <h4>已选择: {selectedItem.name}</h4>
          <p>价格: ¥{selectedItem.price}</p>
        </div>
      )}
      
      <div style={{ marginTop: '20px' }}>
        <h4>操作日志:</h4>
        {actionLog.map((log, index) => (
          <div key={index} style={{ fontSize: '12px', color: '#666' }}>
            {log.timestamp} - {log.action} {log.item}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 高阶函数传参
```javascript
function HigherOrderFunctionParams() {
  const [buttons, setButtons] = useState([
    { id: 1, label: '按钮1', clickCount: 0 },
    { id: 2, label: '按钮2', clickCount: 0 },
    { id: 3, label: '按钮3', clickCount: 0 }
  ]);
  
  // 高阶函数：返回一个事件处理函数
  const createClickHandler = (buttonId) => {
    return (event) => {
      console.log('点击的按钮ID:', buttonId);
      console.log('事件对象:', event);
      
      setButtons(prev => 
        prev.map(button =>
          button.id === buttonId
            ? { ...button, clickCount: button.clickCount + 1 }
            : button
        )
      );
    };
  };
  
  // 更复杂的高阶函数
  const createActionHandler = (action, data) => {
    return (event) => {
      event.preventDefault();
      
      switch (action) {
        case 'reset':
          setButtons(prev => 
            prev.map(button => ({ ...button, clickCount: 0 }))
          );
          break;
        case 'increment':
          setButtons(prev => 
            prev.map(button =>
              button.id === data.id
                ? { ...button, clickCount: button.clickCount + data.amount }
                : button
            )
          );
          break;
        default:
          console.log('未知操作:', action);
      }
    };
  };
  
  return (
    <div>
      <h3>高阶函数事件处理</h3>
      
      {buttons.map(button => (
        <div key={button.id} style={{ margin: '10px 0' }}>
          <button onClick={createClickHandler(button.id)}>
            {button.label} (点击次数: {button.clickCount})
          </button>
          
          <button 
            onClick={createActionHandler('increment', { id: button.id, amount: 5 })}
            style={{ marginLeft: '10px' }}
          >
            +5
          </button>
        </div>
      ))}
      
      <button 
        onClick={createActionHandler('reset')}
        style={{ marginTop: '20px', backgroundColor: '#dc3545', color: 'white' }}
      >
        重置所有计数
      </button>
    </div>
  );
}
```

## 5.4 阻止默认行为和事件冒泡

### 阻止默认行为
```javascript
function PreventDefaultExample() {
  const [submittedData, setSubmittedData] = useState(null);
  
  const handleFormSubmit = (event) => {
    // 阻止表单默认提交行为
    event.preventDefault();
    
    const formData = new FormData(event.target);
    const data = Object.fromEntries(formData.entries());
    
    setSubmittedData(data);
    console.log('表单数据:', data);
  };
  
  const handleLinkClick = (event) => {
    // 阻止链接默认跳转行为
    event.preventDefault();
    
    alert('链接点击被拦截，执行自定义逻辑');
  };
  
  const handleContextMenu = (event) => {
    // 阻止右键菜单
    event.preventDefault();
    
    alert('右键菜单被禁用');
  };
  
  return (
    <div>
      <h3>阻止默认行为示例</h3>
      
      <form onSubmit={handleFormSubmit}>
        <div>
          <input name="username" placeholder="用户名" required />
        </div>
        <div>
          <input name="email" type="email" placeholder="邮箱" required />
        </div>
        <button type="submit">提交</button>
      </form>
      
      <p>
        <a href="https://www.example.com" onClick={handleLinkClick}>
          这个链接的默认行为被阻止了
        </a>
      </p>
      
      <div 
        onContextMenu={handleContextMenu}
        style={{ 
          padding: '20px', 
          border: '1px solid #ccc',
          backgroundColor: '#f9f9f9'
        }}
      >
        右键点击这个区域试试
      </div>
      
      {submittedData && (
        <div style={{ marginTop: '20px' }}>
          <h4>提交的数据：</h4>
          <pre>{JSON.stringify(submittedData, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

### 阻止事件冒泡
```javascript
function EventBubblingExample() {
  const [clickLog, setClickLog] = useState([]);
  
  const addLog = (element) => {
    const timestamp = new Date().toLocaleTimeString();
    setClickLog(prev => [`${timestamp} - 点击了 ${element}`, ...prev.slice(0, 9)]);
  };
  
  const handleOuterClick = () => {
    addLog('外层容器');
  };
  
  const handleMiddleClick = (event) => {
    addLog('中层容器');
    // 如果需要阻止冒泡，取消注释下面的行
    // event.stopPropagation();
  };
  
  const handleInnerClick = (event) => {
    addLog('内层按钮');
    // 阻止事件冒泡
    event.stopPropagation();
  };
  
  const handleStopPropagationClick = (event) => {
    addLog('停止冒泡按钮');
    event.stopPropagation();
  };
  
  const clearLog = () => {
    setClickLog([]);
  };
  
  return (
    <div>
      <h3>事件冒泡示例</h3>
      
      <div 
        onClick={handleOuterClick}
        style={{ 
          padding: '30px', 
          backgroundColor: '#ffebee',
          border: '2px solid #f44336'
        }}
      >
        外层容器（红色）
        
        <div 
          onClick={handleMiddleClick}
          style={{ 
            padding: '20px', 
            backgroundColor: '#e8f5e8',
            border: '2px solid #4caf50',
            margin: '10px'
          }}
        >
          中层容器（绿色）
          
          <button 
            onClick={handleInnerClick}
            style={{ 
              padding: '10px',
              backgroundColor: '#e3f2fd',
              border: '2px solid #2196f3'
            }}
          >
            内层按钮（蓝色）- 阻止冒泡
          </button>
          
          <button 
            onClick={handleStopPropagationClick}
            style={{ 
              padding: '10px',
              backgroundColor: '#fff3e0',
              border: '2px solid #ff9800',
              marginLeft: '10px'
            }}
          >
            停止冒泡按钮（橙色）
          </button>
        </div>
      </div>
      
      <div style={{ marginTop: '20px' }}>
        <button onClick={clearLog}>清空日志</button>
        
        <div style={{ marginTop: '10px' }}>
          <h4>点击日志：</h4>
          <div style={{ 
            maxHeight: '200px', 
            overflow: 'auto',
            backgroundColor: '#f5f5f5',
            padding: '10px'
          }}>
            {clickLog.map((log, index) => (
              <div key={index} style={{ fontSize: '14px', marginBottom: '5px' }}>
                {log}
              </div>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
}
```

## 5.5 实践案例：交互式待办事项

让我们创建一个功能完整的待办事项应用，展示各种事件处理技巧。

```javascript
import React, { useState, useCallback, useRef, useEffect } from 'react';

function InteractiveTodoApp() {
  const [todos, setTodos] = useState([
    { id: 1, text: '学习React事件处理', completed: false, priority: 'high', category: '学习' },
    { id: 2, text: '完成项目文档', completed: true, priority: 'medium', category: '工作' },
    { id: 3, text: '锻炼身体', completed: false, priority: 'low', category: '健康' }
  ]);
  
  const [newTodo, setNewTodo] = useState('');
  const [filter, setFilter] = useState('all');
  const [editingId, setEditingId] = useState(null);
  const [editingText, setEditingText] = useState('');
  const [draggedItem, setDraggedItem] = useState(null);
  const [selectedTodos, setSelectedTodos] = useState(new Set());
  
  const inputRef = useRef(null);
  const editInputRef = useRef(null);
  
  // 自动聚焦到编辑输入框
  useEffect(() => {
    if (editingId && editInputRef.current) {
      editInputRef.current.focus();
      editInputRef.current.select();
    }
  }, [editingId]);
  
  // 添加新待办事项
  const handleAddTodo = useCallback((event) => {
    event.preventDefault();
    
    if (newTodo.trim()) {
      const newTodoItem = {
        id: Date.now(),
        text: newTodo.trim(),
        completed: false,
        priority: 'medium',
        category: '其他'
      };
      
      setTodos(prev => [newTodoItem, ...prev]);
      setNewTodo('');
      inputRef.current?.focus();
    }
  }, [newTodo]);
  
  // 切换完成状态
  const handleToggleComplete = useCallback((id) => {
    setTodos(prev => 
      prev.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }, []);
  
  // 删除待办事项
  const handleDeleteTodo = useCallback((id, event) => {
    event.stopPropagation();
    
    if (window.confirm('确定要删除这个待办事项吗？')) {
      setTodos(prev => prev.filter(todo => todo.id !== id));
      setSelectedTodos(prev => {
        const newSet = new Set(prev);
        newSet.delete(id);
        return newSet;
      });
    }
  }, []);
  
  // 开始编辑
  const handleStartEdit = useCallback((id, text, event) => {
    event.stopPropagation();
    setEditingId(id);
    setEditingText(text);
  }, []);
  
  // 保存编辑
  const handleSaveEdit = useCallback((event) => {
    event.preventDefault();
    
    if (editingText.trim()) {
      setTodos(prev =>
        prev.map(todo =>
          todo.id === editingId
            ? { ...todo, text: editingText.trim() }
            : todo
        )
      );
    }
    
    setEditingId(null);
    setEditingText('');
  }, [editingId, editingText]);
  
  // 取消编辑
  const handleCancelEdit = useCallback((event) => {
    if (event.key === 'Escape') {
      setEditingId(null);
      setEditingText('');
    }
  }, []);
  
  // 更新优先级
  const handlePriorityChange = useCallback((id, priority, event) => {
    event.stopPropagation();
    
    setTodos(prev =>
      prev.map(todo =>
        todo.id === id ? { ...todo, priority } : todo
      )
    );
  }, []);
  
  // 多选处理
  const handleTodoSelect = useCallback((id, event) => {
    if (event.ctrlKey || event.metaKey) {
      setSelectedTodos(prev => {
        const newSet = new Set(prev);
        if (newSet.has(id)) {
          newSet.delete(id);
        } else {
          newSet.add(id);
        }
        return newSet;
      });
    } else {
      setSelectedTodos(new Set([id]));
    }
  }, []);
  
  // 批量操作
  const handleBatchOperation = useCallback((operation) => {
    if (selectedTodos.size === 0) return;
    
    switch (operation) {
      case 'complete':
        setTodos(prev =>
          prev.map(todo =>
            selectedTodos.has(todo.id)
              ? { ...todo, completed: true }
              : todo
          )
        );
        break;
      case 'incomplete':
        setTodos(prev =>
          prev.map(todo =>
            selectedTodos.has(todo.id)
              ? { ...todo, completed: false }
              : todo
          )
        );
        break;
      case 'delete':
        if (window.confirm(`确定要删除选中的 ${selectedTodos.size} 个待办事项吗？`)) {
          setTodos(prev =>
            prev.filter(todo => !selectedTodos.has(todo.id))
          );
          setSelectedTodos(new Set());
        }
        break;
    }
  }, [selectedTodos]);
  
  // 拖拽处理
  const handleDragStart = useCallback((id, event) => {
    setDraggedItem(id);
    event.dataTransfer.effectAllowed = 'move';
  }, []);
  
  const handleDragOver = useCallback((event) => {
    event.preventDefault();
    event.dataTransfer.dropEffect = 'move';
  }, []);
  
  const handleDrop = useCallback((targetId, event) => {
    event.preventDefault();
    
    if (draggedItem && draggedItem !== targetId) {
      setTodos(prev => {
        const newTodos = [...prev];
        const draggedIndex = newTodos.findIndex(todo => todo.id === draggedItem);
        const targetIndex = newTodos.findIndex(todo => todo.id === targetId);
        
        if (draggedIndex !== -1 && targetIndex !== -1) {
          const [removed] = newTodos.splice(draggedIndex, 1);
          newTodos.splice(targetIndex, 0, removed);
        }
        
        return newTodos;
      });
    }
    
    setDraggedItem(null);
  }, [draggedItem]);
  
  // 过滤待办事项
  const filteredTodos = todos.filter(todo => {
    switch (filter) {
      case 'active':
        return !todo.completed;
      case 'completed':
        return todo.completed;
      default:
        return true;
    }
  });
  
  // 获取优先级颜色
  const getPriorityColor = (priority) => {
    const colors = {
      high: '#f44336',
      medium: '#ff9800',
      low: '#4caf50'
    };
    return colors[priority] || '#9e9e9e';
  };
  
  return (
    <div className="todo-app" style={{ maxWidth: '600px', margin: '0 auto', padding: '20px' }}>
      <h1>交互式待办事项</h1>
      
      {/* 添加新待办事项 */}
      <form onSubmit={handleAddTodo} style={{ marginBottom: '20px' }}>
        <input
          ref={inputRef}
          type="text"
          value={newTodo}
          onChange={(e) => setNewTodo(e.target.value)}
          placeholder="添加新的待办事项..."
          style={{ 
            width: '70%', 
            padding: '10px',
            border: '2px solid #ddd',
            borderRadius: '4px'
          }}
          onKeyDown={(e) => {
            if (e.key === 'Escape') {
              setNewTodo('');
            }
          }}
        />
        <button 
          type="submit"
          style={{ 
            width: '25%', 
            padding: '10px',
            backgroundColor: '#007bff',
            color: 'white',
            border: 'none',
            borderRadius: '4px',
            marginLeft: '5%'
          }}
        >
          添加
        </button>
      </form>
      
      {/* 过滤器 */}
      <div style={{ marginBottom: '20px' }}>
        {['all', 'active', 'completed'].map(filterType => (
          <button
            key={filterType}
            onClick={() => setFilter(filterType)}
            style={{
              padding: '8px 16px',
              margin: '0 5px',
              backgroundColor: filter === filterType ? '#007bff' : '#f8f9fa',
              color: filter === filterType ? 'white' : '#333',
              border: '1px solid #ddd',
              borderRadius: '4px',
              cursor: 'pointer'
            }}
          >
            {filterType === 'all' ? '全部' : 
             filterType === 'active' ? '未完成' : '已完成'}
          </button>
        ))}
      </div>
      
      {/* 批量操作 */}
      {selectedTodos.size > 0 && (
        <div style={{ 
          marginBottom: '20px', 
          padding: '10px',
          backgroundColor: '#e3f2fd',
          borderRadius: '4px'
        }}>
          <span>已选择 {selectedTodos.size} 项：</span>
          <button onClick={() => handleBatchOperation('complete')}>标记完成</button>
          <button onClick={() => handleBatchOperation('incomplete')}>标记未完成</button>
          <button onClick={() => handleBatchOperation('delete')}>删除</button>
        </div>
      )}
      
      {/* 待办事项列表 */}
      <div>
        {filteredTodos.map(todo => (
          <div
            key={todo.id}
            draggable
            onDragStart={(e) => handleDragStart(todo.id, e)}
            onDragOver={handleDragOver}
            onDrop={(e) => handleDrop(todo.id, e)}
            onClick={(e) => handleTodoSelect(todo.id, e)}
            style={{
              padding: '15px',
              margin: '10px 0',
              border: `2px solid ${selectedTodos.has(todo.id) ? '#007bff' : '#ddd'}`,
              borderRadius: '8px',
              backgroundColor: todo.completed ? '#f8f9fa' : 'white',
              cursor: 'pointer',
              opacity: draggedItem === todo.id ? 0.5 : 1,
              transition: 'all 0.2s ease'
            }}
          >
            <div style={{ display: 'flex', alignItems: 'center', gap: '10px' }}>
              {/* 完成状态复选框 */}
              <input
                type="checkbox"
                checked={todo.completed}
                onChange={() => handleToggleComplete(todo.id)}
                onClick={(e) => e.stopPropagation()}
                style={{ transform: 'scale(1.2)' }}
              />
              
              {/* 待办事项文本 */}
              {editingId === todo.id ? (
                <form onSubmit={handleSaveEdit} style={{ flex: 1 }}>
                  <input
                    ref={editInputRef}
                    type="text"
                    value={editingText}
                    onChange={(e) => setEditingText(e.target.value)}
                    onKeyDown={handleCancelEdit}
                    onBlur={handleSaveEdit}
                    style={{ 
                      width: '100%',
                      padding: '5px',
                      border: '1px solid #007bff',
                      borderRadius: '4px'
                    }}
                  />
                </form>
              ) : (
                <span
                  style={{
                    flex: 1,
                    textDecoration: todo.completed ? 'line-through' : 'none',
                    color: todo.completed ? '#666' : '#333'
                  }}
                  onDoubleClick={(e) => handleStartEdit(todo.id, todo.text, e)}
                >
                  {todo.text}
                </span>
              )}
              
              {/* 优先级选择器 */}
              <select
                value={todo.priority}
                onChange={(e) => handlePriorityChange(todo.id, e.target.value, e)}
                onClick={(e) => e.stopPropagation()}
                style={{
                  padding: '4px',
                  borderColor: getPriorityColor(todo.priority),
                  color: getPriorityColor(todo.priority),
                  fontWeight: 'bold'
                }}
              >
                <option value="low">低</option>
                <option value="medium">中</option>
                <option value="high">高</option>
              </select>
              
              {/* 操作按钮 */}
              <div style={{ display: 'flex', gap: '5px' }}>
                <button
                  onClick={(e) => handleStartEdit(todo.id, todo.text, e)}
                  style={{
                    padding: '5px 10px',
                    backgroundColor: '#28a745',
                    color: 'white',
                    border: 'none',
                    borderRadius: '4px',
                    cursor: 'pointer'
                  }}
                >
                  编辑
                </button>
                
                <button
                  onClick={(e) => handleDeleteTodo(todo.id, e)}
                  style={{
                    padding: '5px 10px',
                    backgroundColor: '#dc3545',
                    color: 'white',
                    border: 'none',
                    borderRadius: '4px',
                    cursor: 'pointer'
                  }}
                >
                  删除
                </button>
              </div>
            </div>
          </div>
        ))}
      </div>
      
      {/* 统计信息 */}
      <div style={{ 
        marginTop: '20px',
        padding: '15px',
        backgroundColor: '#f8f9fa',
        borderRadius: '8px'
      }}>
        <h3>统计信息</h3>
        <p>总计：{todos.length} 项</p>
        <p>已完成：{todos.filter(t => t.completed).length} 项</p>
        <p>未完成：{todos.filter(t => !t.completed).length} 项</p>
        {selectedTodos.size > 0 && (
          <p>已选择：{selectedTodos.size} 项</p>
        )}
      </div>
      
      {/* 使用说明 */}
      <div style={{ 
        marginTop: '20px',
        padding: '15px',
        backgroundColor: '#fff3cd',
        border: '1px solid #ffeaa7',
        borderRadius: '8px'
      }}>
        <h4>使用说明：</h4>
        <ul style={{ paddingLeft: '20px' }}>
          <li>单击选择单个项目，Ctrl/Cmd+点击多选</li>
          <li>双击文本开始编辑，ESC键取消编辑</li>
          <li>拖拽项目可以重新排序</li>
          <li>使用过滤器查看不同状态的项目</li>
          <li>选择多个项目后可以进行批量操作</li>
        </ul>
      </div>
    </div>
  );
}

export default InteractiveTodoApp;
```

## 5.6 性能优化

### 事件处理性能优化
```javascript
import React, { useState, useCallback, useMemo } from 'react';

function PerformanceOptimizedList() {
  const [items, setItems] = useState(
    Array.from({ length: 1000 }, (_, i) => ({
      id: i + 1,
      name: `项目 ${i + 1}`,
      value: Math.floor(Math.random() * 100)
    }))
  );
  
  const [filter, setFilter] = useState('');
  
  // 使用 useCallback 优化事件处理函数
  const handleItemClick = useCallback((id) => {
    console.log('点击项目:', id);
  }, []);
  
  const handleItemDelete = useCallback((id) => {
    setItems(prev => prev.filter(item => item.id !== id));
  }, []);
  
  const handleFilterChange = useCallback((event) => {
    setFilter(event.target.value);
  }, []);
  
  // 使用 useMemo 优化过滤计算
  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);
  
  return (
    <div>
      <input
        type="text"
        value={filter}
        onChange={handleFilterChange}
        placeholder="搜索项目..."
        style={{ marginBottom: '20px', padding: '10px', width: '200px' }}
      />
      
      <p>显示 {filteredItems.length} / {items.length} 项</p>
      
      <div style={{ maxHeight: '400px', overflow: 'auto' }}>
        {filteredItems.map(item => (
          <OptimizedListItem
            key={item.id}
            item={item}
            onItemClick={handleItemClick}
            onItemDelete={handleItemDelete}
          />
        ))}
      </div>
    </div>
  );
}

// 使用 React.memo 优化子组件
const OptimizedListItem = React.memo(({ item, onItemClick, onItemDelete }) => {
  const handleClick = useCallback(() => {
    onItemClick(item.id);
  }, [item.id, onItemClick]);
  
  const handleDelete = useCallback((event) => {
    event.stopPropagation();
    onItemDelete(item.id);
  }, [item.id, onItemDelete]);
  
  return (
    <div
      onClick={handleClick}
      style={{
        padding: '10px',
        border: '1px solid #ddd',
        margin: '5px 0',
        cursor: 'pointer',
        display: 'flex',
        justifyContent: 'space-between',
        alignItems: 'center'
      }}
    >
      <span>{item.name} - 值: {item.value}</span>
      <button onClick={handleDelete}>删除</button>
    </div>
  );
});
```

## 5.7 总结

### 本章重点
1. **合成事件**：React统一的事件处理系统
2. **事件绑定**：多种绑定方法和性能优化
3. **事件传参**：传递参数给事件处理函数
4. **阻止默认行为**：使用preventDefault()
5. **阻止事件冒泡**：使用stopPropagation()

### 关键概念理解
- **事件委托**：React在根节点统一处理事件
- **合成事件对象**：跨浏览器兼容的事件接口
- **事件处理优化**：使用useCallback和React.memo
- **用户交互**：创建响应式的用户界面

### 下章预告
下一章我们将学习React组件的生命周期，了解组件从创建到销毁的完整过程。

### 练习作业
1. 创建一个图片查看器，支持鼠标悬停放大、键盘导航等交互
2. 实现一个简单的绘图应用，支持鼠标绘制、撤销重做等功能
3. 构建一个表格组件，支持排序、筛选、行选择等交互功能

### 常见问题
1. **Q: 为什么要使用合成事件而不是原生事件？**
   A: 合成事件提供跨浏览器兼容性，统一的API，以及性能优化。

2. **Q: 什么时候需要阻止事件冒泡？**
   A: 当不希望父元素的事件处理函数被触发时，比如在嵌套的可点击元素中。

3. **Q: 如何优化大量事件处理器的性能？**
   A: 使用事件委托、useCallback优化函数引用、React.memo优化组件渲染。
