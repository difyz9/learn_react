# 第六章：生命周期

## 学习目标
- 理解React组件的生命周期概念
- 掌握类组件的生命周期方法
- 学会使用useEffect模拟生命周期
- 了解生命周期的应用场景
- 能够优化组件性能和避免内存泄漏

## 6.1 生命周期概述

### 什么是组件生命周期？
组件生命周期是指组件从创建到销毁的完整过程。React组件在这个过程中会经历不同的阶段，每个阶段都有相应的生命周期方法可以被调用。

### 生命周期的三个主要阶段
1. **挂载（Mounting）**：组件被创建并插入到DOM中
2. **更新（Updating）**：组件的props或state发生变化
3. **卸载（Unmounting）**：组件从DOM中被移除

## 6.2 类组件生命周期

### 挂载阶段（Mounting）

#### constructor()
```javascript
class LifecycleDemo extends React.Component {
  constructor(props) {
    super(props);
    
    console.log('1. constructor - 组件构造函数');
    
    // 初始化状态
    this.state = {
      count: 0,
      data: null,
      loading: true
    };
    
    // 绑定事件处理方法
    this.handleIncrement = this.handleIncrement.bind(this);
  }
  
  handleIncrement() {
    this.setState(prevState => ({
      count: prevState.count + 1
    }));
  }
  
  render() {
    console.log('4. render - 渲染组件');
    return (
      <div>
        <p>计数: {this.state.count}</p>
        <button onClick={this.handleIncrement}>增加</button>
      </div>
    );
  }
}
```

#### static getDerivedStateFromProps()
```javascript
class DerivedStateDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      derivedValue: props.initialValue * 2
    };
  }
  
  // 从props派生state
  static getDerivedStateFromProps(nextProps, prevState) {
    console.log('2. getDerivedStateFromProps - 从props派生state');
    
    // 如果props.initialValue改变，更新derivedValue
    if (nextProps.initialValue !== prevState.prevPropsValue) {
      return {
        derivedValue: nextProps.initialValue * 2,
        prevPropsValue: nextProps.initialValue
      };
    }
    
    // 返回null表示不需要更新state
    return null;
  }
  
  render() {
    return (
      <div>
        <p>原始值: {this.props.initialValue}</p>
        <p>派生值: {this.state.derivedValue}</p>
      </div>
    );
  }
}
```

#### componentDidMount()
```javascript
class DataFetchingComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [],
      loading: true,
      error: null
    };
  }
  
  componentDidMount() {
    console.log('3. componentDidMount - 组件已挂载');
    
    // 执行副作用操作
    this.fetchUsers();
    
    // 设置定时器
    this.timer = setInterval(() => {
      console.log('定时器执行');
    }, 5000);
    
    // 添加事件监听器
    window.addEventListener('resize', this.handleResize);
    
    // 获取DOM元素
    this.inputRef.focus();
  }
  
  componentWillUnmount() {
    // 清理工作
    clearInterval(this.timer);
    window.removeEventListener('resize', this.handleResize);
  }
  
  handleResize = () => {
    console.log('窗口大小改变');
  }
  
  fetchUsers = async () => {
    try {
      this.setState({ loading: true, error: null });
      
      // 模拟API调用
      const response = await fetch('/api/users');
      const users = await response.json();
      
      this.setState({ 
        users, 
        loading: false 
      });
    } catch (error) {
      this.setState({ 
        error: error.message, 
        loading: false 
      });
    }
  }
  
  render() {
    const { users, loading, error } = this.state;
    
    if (loading) return <div>加载中...</div>;
    if (error) return <div>错误: {error}</div>;
    
    return (
      <div>
        <input ref={ref => this.inputRef = ref} placeholder="搜索用户" />
        <ul>
          {users.map(user => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      </div>
    );
  }
}
```

### 更新阶段（Updating）

#### shouldComponentUpdate()
```javascript
class OptimizedComponent extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    console.log('5. shouldComponentUpdate - 决定是否更新');
    
    // 性能优化：只有特定props改变时才更新
    if (this.props.importantProp !== nextProps.importantProp) {
      return true;
    }
    
    if (this.state.count !== nextState.count) {
      return true;
    }
    
    // 避免不必要的重渲染
    return false;
  }
  
  render() {
    console.log('6. render - 重新渲染');
    return (
      <div>
        <p>重要属性: {this.props.importantProp}</p>
        <p>计数: {this.state.count}</p>
      </div>
    );
  }
}
```

#### getSnapshotBeforeUpdate()
```javascript
class ScrollPositionComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      messages: ['消息1', '消息2', '消息3']
    };
    this.listRef = React.createRef();
  }
  
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('7. getSnapshotBeforeUpdate - 获取更新前的快照');
    
    // 如果新增了消息，记录当前滚动位置
    if (prevState.messages.length < this.state.messages.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    
    return null;
  }
  
  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('8. componentDidUpdate - 组件已更新');
    
    // 如果有快照数据，恢复滚动位置
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }
  
  addMessage = () => {
    this.setState(prevState => ({
      messages: [...prevState.messages, `消息${prevState.messages.length + 1}`]
    }));
  }
  
  render() {
    return (
      <div>
        <button onClick={this.addMessage}>添加消息</button>
        <div 
          ref={this.listRef}
          style={{ height: '200px', overflow: 'auto', border: '1px solid #ccc' }}
        >
          {this.state.messages.map((message, index) => (
            <div key={index} style={{ padding: '10px', borderBottom: '1px solid #eee' }}>
              {message}
            </div>
          ))}
        </div>
      </div>
    );
  }
}
```

#### componentDidUpdate()
```javascript
class UserProfileComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      userInfo: null,
      loading: false
    };
  }
  
  componentDidMount() {
    this.fetchUserInfo(this.props.userId);
  }
  
  componentDidUpdate(prevProps, prevState) {
    console.log('componentDidUpdate - 组件已更新');
    
    // 如果userId改变，重新获取用户信息
    if (prevProps.userId !== this.props.userId) {
      this.fetchUserInfo(this.props.userId);
    }
    
    // 如果用户信息更新，执行其他操作
    if (prevState.userInfo !== this.state.userInfo && this.state.userInfo) {
      this.logUserActivity();
    }
  }
  
  fetchUserInfo = async (userId) => {
    this.setState({ loading: true });
    
    try {
      const response = await fetch(`/api/users/${userId}`);
      const userInfo = await response.json();
      this.setState({ userInfo, loading: false });
    } catch (error) {
      console.error('获取用户信息失败:', error);
      this.setState({ loading: false });
    }
  }
  
  logUserActivity = () => {
    console.log('用户信息已更新:', this.state.userInfo);
    // 记录用户活动
  }
  
  render() {
    const { userInfo, loading } = this.state;
    
    if (loading) return <div>加载中...</div>;
    if (!userInfo) return <div>用户不存在</div>;
    
    return (
      <div>
        <h2>{userInfo.name}</h2>
        <p>邮箱: {userInfo.email}</p>
        <p>注册时间: {userInfo.createdAt}</p>
      </div>
    );
  }
}
```

### 卸载阶段（Unmounting）

#### componentWillUnmount()
```javascript
class CleanupComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      seconds: 0
    };
  }
  
  componentDidMount() {
    console.log('组件挂载，开始设置资源');
    
    // 设置定时器
    this.timer = setInterval(() => {
      this.setState(prevState => ({
        seconds: prevState.seconds + 1
      }));
    }, 1000);
    
    // 添加事件监听器
    this.handleKeyPress = (event) => {
      if (event.key === 'Escape') {
        console.log('ESC键被按下');
      }
    };
    
    document.addEventListener('keydown', this.handleKeyPress);
    
    // WebSocket连接
    this.websocket = new WebSocket('ws://localhost:8080');
    this.websocket.onmessage = (event) => {
      console.log('收到消息:', event.data);
    };
    
    // 订阅外部事件
    this.subscription = this.props.eventEmitter?.subscribe('data-update', this.handleDataUpdate);
  }
  
  componentWillUnmount() {
    console.log('9. componentWillUnmount - 组件即将卸载');
    
    // 清理定时器
    if (this.timer) {
      clearInterval(this.timer);
      console.log('定时器已清理');
    }
    
    // 移除事件监听器
    document.removeEventListener('keydown', this.handleKeyPress);
    console.log('事件监听器已移除');
    
    // 关闭WebSocket连接
    if (this.websocket) {
      this.websocket.close();
      console.log('WebSocket连接已关闭');
    }
    
    // 取消订阅
    if (this.subscription) {
      this.subscription.unsubscribe();
      console.log('事件订阅已取消');
    }
    
    // 取消正在进行的网络请求
    if (this.abortController) {
      this.abortController.abort();
      console.log('网络请求已取消');
    }
  }
  
  handleDataUpdate = (data) => {
    console.log('数据更新:', data);
  }
  
  render() {
    return (
      <div>
        <p>运行时间: {this.state.seconds} 秒</p>
        <p>按ESC键测试事件监听器</p>
      </div>
    );
  }
}
```

### 错误边界（Error Boundaries）

#### componentDidCatch()
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null
    };
  }
  
  static getDerivedStateFromError(error) {
    // 更新state以显示错误界面
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    console.log('componentDidCatch - 捕获到错误');
    console.error('错误详情:', error);
    console.error('错误信息:', errorInfo);
    
    // 保存错误信息到state
    this.setState({
      error,
      errorInfo
    });
    
    // 可以将错误信息发送到错误报告服务
    this.logErrorToService(error, errorInfo);
  }
  
  logErrorToService = (error, errorInfo) => {
    // 发送错误报告到服务器
    console.log('发送错误报告到服务器');
    // fetch('/api/error-report', {
    //   method: 'POST',
    //   body: JSON.stringify({ error: error.toString(), errorInfo })
    // });
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div style={{ padding: '20px', border: '1px solid red', backgroundColor: '#ffebee' }}>
          <h2>出现了错误！</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            <summary>查看错误详情</summary>
            <p><strong>错误:</strong> {this.state.error && this.state.error.toString()}</p>
            <p><strong>堆栈信息:</strong></p>
            <pre>{this.state.errorInfo.componentStack}</pre>
          </details>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// 使用错误边界
function App() {
  return (
    <ErrorBoundary>
      <ProblematicComponent />
    </ErrorBoundary>
  );
}

// 可能出错的组件
class ProblematicComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { counter: 0 };
  }
  
  handleClick = () => {
    this.setState(({ counter }) => ({
      counter: counter + 1
    }));
  }
  
  render() {
    // 故意在counter为5时抛出错误
    if (this.state.counter === 5) {
      throw new Error('计数器达到5时的测试错误！');
    }
    
    return (
      <div>
        <p>计数器: {this.state.counter}</p>
        <button onClick={this.handleClick}>
          增加计数（达到5时会出错）
        </button>
      </div>
    );
  }
}
```

## 6.3 函数组件中的生命周期（useEffect）

### useEffect基本用法
```javascript
import React, { useState, useEffect } from 'react';

function FunctionLifecycleDemo() {
  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);
  
  // 模拟componentDidMount
  useEffect(() => {
    console.log('组件挂载 - 相当于componentDidMount');
    
    // 执行初始化操作
    fetchInitialData();
    
    // 返回清理函数 - 相当于componentWillUnmount
    return () => {
      console.log('组件卸载 - 相当于componentWillUnmount');
    };
  }, []); // 空依赖数组表示只在挂载和卸载时执行
  
  // 模拟componentDidUpdate
  useEffect(() => {
    console.log('count发生变化 - 相当于componentDidUpdate');
    
    // 当count变化时执行的逻辑
    if (count > 0) {
      document.title = `计数: ${count}`;
    }
    
    return () => {
      // 清理上一次effect的影响
      document.title = 'React应用';
    };
  }, [count]); // 依赖数组包含count，count变化时执行
  
  const fetchInitialData = async () => {
    try {
      // 模拟API调用
      setTimeout(() => {
        setData({ message: '数据加载完成' });
      }, 1000);
    } catch (error) {
      console.error('数据加载失败:', error);
    }
  };
  
  return (
    <div>
      <p>计数: {count}</p>
      <p>数据: {data ? data.message : '加载中...'}</p>
      <button onClick={() => setCount(count + 1)}>
        增加计数
      </button>
    </div>
  );
}
```

### 多个useEffect
```javascript
function MultipleEffectsDemo() {
  const [userId, setUserId] = useState(1);
  const [userInfo, setUserInfo] = useState(null);
  const [posts, setPosts] = useState([]);
  const [online, setOnline] = useState(navigator.onLine);
  
  // Effect 1: 获取用户信息
  useEffect(() => {
    let isCancelled = false;
    
    const fetchUserInfo = async () => {
      try {
        console.log('获取用户信息:', userId);
        // 模拟API调用
        setTimeout(() => {
          if (!isCancelled) {
            setUserInfo({
              id: userId,
              name: `用户${userId}`,
              email: `user${userId}@example.com`
            });
          }
        }, 500);
      } catch (error) {
        if (!isCancelled) {
          console.error('获取用户信息失败:', error);
        }
      }
    };
    
    fetchUserInfo();
    
    return () => {
      isCancelled = true;
    };
  }, [userId]);
  
  // Effect 2: 获取用户文章
  useEffect(() => {
    let isCancelled = false;
    
    const fetchUserPosts = async () => {
      try {
        console.log('获取用户文章:', userId);
        // 模拟API调用
        setTimeout(() => {
          if (!isCancelled) {
            setPosts([
              { id: 1, title: `用户${userId}的文章1` },
              { id: 2, title: `用户${userId}的文章2` }
            ]);
          }
        }, 800);
      } catch (error) {
        if (!isCancelled) {
          console.error('获取用户文章失败:', error);
        }
      }
    };
    
    fetchUserPosts();
    
    return () => {
      isCancelled = true;
    };
  }, [userId]);
  
  // Effect 3: 监听网络状态
  useEffect(() => {
    const handleOnline = () => setOnline(true);
    const handleOffline = () => setOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  
  // Effect 4: 日志记录
  useEffect(() => {
    console.log('组件状态更新:', {
      userId,
      userInfo: userInfo?.name,
      postsCount: posts.length,
      online
    });
  }); // 没有依赖数组，每次渲染都执行
  
  return (
    <div>
      <div>
        <label>用户ID: </label>
        <select value={userId} onChange={(e) => setUserId(Number(e.target.value))}>
          <option value={1}>用户1</option>
          <option value={2}>用户2</option>
          <option value={3}>用户3</option>
        </select>
      </div>
      
      <div>网络状态: {online ? '在线' : '离线'}</div>
      
      <div>
        <h3>用户信息</h3>
        {userInfo ? (
          <div>
            <p>姓名: {userInfo.name}</p>
            <p>邮箱: {userInfo.email}</p>
          </div>
        ) : (
          <p>加载中...</p>
        )}
      </div>
      
      <div>
        <h3>用户文章</h3>
        {posts.length > 0 ? (
          <ul>
            {posts.map(post => (
              <li key={post.id}>{post.title}</li>
            ))}
          </ul>
        ) : (
          <p>加载中...</p>
        )}
      </div>
    </div>
  );
}
```

### 自定义Hook封装生命周期逻辑
```javascript
// 自定义Hook: 数据获取
function useApi(url, dependencies = []) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let isCancelled = false;
    
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        // 模拟API调用
        setTimeout(() => {
          if (!isCancelled) {
            setData({ url, timestamp: new Date().toISOString() });
            setLoading(false);
          }
        }, 1000);
      } catch (err) {
        if (!isCancelled) {
          setError(err);
          setLoading(false);
        }
      }
    };
    
    fetchData();
    
    return () => {
      isCancelled = true;
    };
  }, [url, ...dependencies]);
  
  return { data, loading, error };
}

// 自定义Hook: 定时器
function useInterval(callback, delay) {
  const savedCallback = useRef();
  
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);
  
  useEffect(() => {
    if (delay !== null) {
      const id = setInterval(() => savedCallback.current(), delay);
      return () => clearInterval(id);
    }
  }, [delay]);
}

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

// 使用自定义Hook的组件
function CustomHookDemo() {
  const [userId, setUserId] = useState(1);
  const [count, setCount] = useLocalStorage('count', 0);
  
  // 使用数据获取Hook
  const { data: userData, loading: userLoading, error: userError } = useApi(
    `/api/users/${userId}`, 
    [userId]
  );
  
  // 使用定时器Hook
  useInterval(() => {
    setCount(count + 1);
  }, 2000);
  
  return (
    <div>
      <h3>自定义Hook演示</h3>
      
      <div>
        <label>用户ID: </label>
        <input 
          type="number" 
          value={userId} 
          onChange={(e) => setUserId(Number(e.target.value))} 
        />
      </div>
      
      <div>
        <h4>用户数据</h4>
        {userLoading && <p>加载中...</p>}
        {userError && <p>错误: {userError.message}</p>}
        {userData && (
          <pre>{JSON.stringify(userData, null, 2)}</pre>
        )}
      </div>
      
      <div>
        <h4>自动计数器</h4>
        <p>计数: {count} (每2秒自动增加)</p>
        <button onClick={() => setCount(0)}>重置</button>
      </div>
    </div>
  );
}
```

## 6.4 实践案例：数据加载组件

让我们创建一个完整的数据加载组件，展示生命周期的实际应用。

```javascript
import React, { useState, useEffect, useRef, useCallback } from 'react';

// 模拟API服务
const mockApiService = {
  async fetchUsers(page = 1, pageSize = 5) {
    // 模拟网络延迟
    await new Promise(resolve => setTimeout(resolve, 1000 + Math.random() * 1000));
    
    // 模拟偶尔的错误
    if (Math.random() < 0.1) {
      throw new Error('网络请求失败');
    }
    
    const startIndex = (page - 1) * pageSize;
    const users = Array.from({ length: pageSize }, (_, i) => ({
      id: startIndex + i + 1,
      name: `用户 ${startIndex + i + 1}`,
      email: `user${startIndex + i + 1}@example.com`,
      avatar: `https://via.placeholder.com/50?text=U${startIndex + i + 1}`,
      status: Math.random() > 0.5 ? 'active' : 'inactive'
    }));
    
    return {
      users,
      total: 50, // 总用户数
      page,
      pageSize,
      hasMore: page * pageSize < 50
    };
  },
  
  async searchUsers(query) {
    await new Promise(resolve => setTimeout(resolve, 500));
    
    const allUsers = Array.from({ length: 20 }, (_, i) => ({
      id: i + 1,
      name: `用户 ${i + 1}`,
      email: `user${i + 1}@example.com`,
      avatar: `https://via.placeholder.com/50?text=U${i + 1}`,
      status: Math.random() > 0.5 ? 'active' : 'inactive'
    }));
    
    return allUsers.filter(user => 
      user.name.toLowerCase().includes(query.toLowerCase()) ||
      user.email.toLowerCase().includes(query.toLowerCase())
    );
  }
};

function DataLoadingComponent() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  const [searchQuery, setSearchQuery] = useState('');
  const [searchResults, setSearchResults] = useState([]);
  const [searchLoading, setSearchLoading] = useState(false);
  const [retryCount, setRetryCount] = useState(0);
  
  // 使用ref避免useEffect依赖问题
  const abortControllerRef = useRef(null);
  const searchTimeoutRef = useRef(null);
  const isInitialMount = useRef(true);
  
  // 加载用户数据
  const loadUsers = useCallback(async (pageNum = 1, append = false) => {
    // 取消之前的请求
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
    
    abortControllerRef.current = new AbortController();
    
    try {
      setLoading(true);
      setError(null);
      
      const result = await mockApiService.fetchUsers(pageNum, 5);
      
      if (!abortControllerRef.current.signal.aborted) {
        if (append) {
          setUsers(prevUsers => [...prevUsers, ...result.users]);
        } else {
          setUsers(result.users);
        }
        
        setHasMore(result.hasMore);
        setPage(result.page);
        setRetryCount(0); // 重置重试次数
      }
    } catch (err) {
      if (!abortControllerRef.current.signal.aborted) {
        setError(err.message);
        console.error('加载用户失败:', err);
      }
    } finally {
      if (!abortControllerRef.current.signal.aborted) {
        setLoading(false);
      }
    }
  }, []);
  
  // 搜索用户
  const searchUsers = useCallback(async (query) => {
    if (!query.trim()) {
      setSearchResults([]);
      return;
    }
    
    try {
      setSearchLoading(true);
      const results = await mockApiService.searchUsers(query);
      setSearchResults(results);
    } catch (err) {
      console.error('搜索失败:', err);
      setSearchResults([]);
    } finally {
      setSearchLoading(false);
    }
  }, []);
  
  // 初始数据加载
  useEffect(() => {
    if (isInitialMount.current) {
      isInitialMount.current = false;
      loadUsers(1, false);
    }
    
    // 清理函数
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [loadUsers]);
  
  // 搜索防抖
  useEffect(() => {
    if (searchTimeoutRef.current) {
      clearTimeout(searchTimeoutRef.current);
    }
    
    searchTimeoutRef.current = setTimeout(() => {
      searchUsers(searchQuery);
    }, 300);
    
    return () => {
      if (searchTimeoutRef.current) {
        clearTimeout(searchTimeoutRef.current);
      }
    };
  }, [searchQuery, searchUsers]);
  
  // 组件卸载清理
  useEffect(() => {
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
      if (searchTimeoutRef.current) {
        clearTimeout(searchTimeoutRef.current);
      }
    };
  }, []);
  
  // 重试加载
  const handleRetry = () => {
    setRetryCount(prev => prev + 1);
    loadUsers(page, false);
  };
  
  // 加载更多
  const handleLoadMore = () => {
    if (!loading && hasMore) {
      loadUsers(page + 1, true);
    }
  };
  
  // 刷新数据
  const handleRefresh = () => {
    setPage(1);
    setHasMore(true);
    loadUsers(1, false);
  };
  
  // 渲染用户列表
  const renderUserList = (userList) => (
    <div className="user-list">
      {userList.map(user => (
        <div key={user.id} className="user-item">
          <img src={user.avatar} alt={user.name} />
          <div className="user-info">
            <h4>{user.name}</h4>
            <p>{user.email}</p>
            <span className={`status ${user.status}`}>
              {user.status === 'active' ? '活跃' : '非活跃'}
            </span>
          </div>
        </div>
      ))}
    </div>
  );
  
  return (
    <div className="data-loading-component">
      <div className="header">
        <h2>用户管理</h2>
        <button onClick={handleRefresh} disabled={loading}>
          {loading ? '刷新中...' : '刷新'}
        </button>
      </div>
      
      {/* 搜索框 */}
      <div className="search-section">
        <input
          type="text"
          value={searchQuery}
          onChange={(e) => setSearchQuery(e.target.value)}
          placeholder="搜索用户..."
          className="search-input"
        />
        {searchLoading && <span className="search-loading">搜索中...</span>}
      </div>
      
      {/* 搜索结果 */}
      {searchQuery && (
        <div className="search-results">
          <h3>搜索结果 ({searchResults.length})</h3>
          {searchResults.length > 0 ? (
            renderUserList(searchResults)
          ) : (
            !searchLoading && <p>未找到匹配的用户</p>
          )}
        </div>
      )}
      
      {/* 主要用户列表 */}
      {!searchQuery && (
        <div className="main-users">
          <h3>所有用户</h3>
          
          {error ? (
            <div className="error-state">
              <p>加载失败: {error}</p>
              <button onClick={handleRetry}>
                重试 {retryCount > 0 && `(${retryCount})`}
              </button>
            </div>
          ) : (
            <>
              {renderUserList(users)}
              
              {loading && (
                <div className="loading-state">
                  <p>加载中...</p>
                </div>
              )}
              
              {!loading && hasMore && (
                <button onClick={handleLoadMore} className="load-more-btn">
                  加载更多
                </button>
              )}
              
              {!loading && !hasMore && users.length > 0 && (
                <p className="end-message">没有更多数据了</p>
              )}
            </>
          )}
        </div>
      )}
      
      <style jsx>{`
        .data-loading-component {
          max-width: 600px;
          margin: 0 auto;
          padding: 20px;
        }
        
        .header {
          display: flex;
          justify-content: space-between;
          align-items: center;
          margin-bottom: 20px;
        }
        
        .search-section {
          position: relative;
          margin-bottom: 20px;
        }
        
        .search-input {
          width: 100%;
          padding: 10px;
          border: 2px solid #ddd;
          border-radius: 6px;
          font-size: 16px;
        }
        
        .search-loading {
          position: absolute;
          right: 10px;
          top: 50%;
          transform: translateY(-50%);
          color: #666;
        }
        
        .user-list {
          space-y: 10px;
        }
        
        .user-item {
          display: flex;
          align-items: center;
          padding: 15px;
          border: 1px solid #ddd;
          border-radius: 8px;
          background: white;
          margin-bottom: 10px;
        }
        
        .user-item img {
          width: 50px;
          height: 50px;
          border-radius: 50%;
          margin-right: 15px;
        }
        
        .user-info h4 {
          margin: 0 0 5px 0;
        }
        
        .user-info p {
          margin: 0 0 5px 0;
          color: #666;
        }
        
        .status {
          padding: 2px 8px;
          border-radius: 12px;
          font-size: 12px;
          font-weight: bold;
        }
        
        .status.active {
          background: #d4edda;
          color: #155724;
        }
        
        .status.inactive {
          background: #f8d7da;
          color: #721c24;
        }
        
        .error-state {
          text-align: center;
          padding: 40px;
          color: #dc3545;
        }
        
        .loading-state {
          text-align: center;
          padding: 20px;
          color: #666;
        }
        
        .load-more-btn {
          width: 100%;
          padding: 12px;
          background: #007bff;
          color: white;
          border: none;
          border-radius: 6px;
          font-size: 16px;
          cursor: pointer;
          margin: 20px 0;
        }
        
        .load-more-btn:hover {
          background: #0056b3;
        }
        
        .end-message {
          text-align: center;
          color: #666;
          padding: 20px;
        }
        
        .search-results {
          margin-bottom: 30px;
          padding-bottom: 20px;
          border-bottom: 2px solid #eee;
        }
      `}</style>
    </div>
  );
}

export default DataLoadingComponent;
```

## 6.5 总结

### 本章重点
1. **生命周期概念**：组件从创建到销毁的完整过程
2. **类组件生命周期**：挂载、更新、卸载阶段的方法
3. **函数组件生命周期**：使用useEffect模拟生命周期
4. **错误边界**：处理组件错误的机制
5. **资源清理**：避免内存泄漏的重要性

### 关键概念理解
- **副作用处理**：在正确的时机执行副作用操作
- **依赖数组**：控制useEffect的执行时机
- **清理函数**：清理定时器、事件监听器等资源
- **错误处理**：使用错误边界保护应用稳定性

### 下章预告
下一章我们将学习React Hooks，这是现代React开发的核心功能，包括useState、useEffect、useContext等常用Hooks。

### 练习作业
1. 创建一个图片懒加载组件，利用生命周期优化性能
2. 实现一个实时聊天组件，处理WebSocket连接的建立和清理
3. 构建一个数据仪表板，包含多个数据源的加载和更新

### 常见问题
1. **Q: 什么时候使用componentDidUpdate？**
   A: 当需要响应props或state变化，执行额外的操作时，比如网络请求或DOM操作。

2. **Q: useEffect的依赖数组应该包含什么？**
   A: 包含effect中使用的所有来自组件作用域的变量（props、state、函数等）。

3. **Q: 如何避免内存泄漏？**
   A: 在组件卸载时清理定时器、事件监听器、网络请求等资源。
