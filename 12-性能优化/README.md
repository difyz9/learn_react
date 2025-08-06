# 第十二章：性能优化

## 学习目标
- 理解React性能优化的原理和方法
- 掌握渲染优化技术和最佳实践
- 学会使用React DevTools分析性能问题
- 了解代码分割和懒加载技术
- 能够构建高性能的React应用

## 12.1 性能基础概念

### React渲染过程
```javascript
// React渲染的三个阶段
/*
1. Reconciliation (协调阶段)
   - 比较新旧Virtual DOM
   - 计算需要更新的节点
   - 可以被中断

2. Commit (提交阶段)
   - 更新DOM
   - 执行副作用
   - 不能被中断

3. Cleanup (清理阶段)
   - 清理副作用
   - 清理引用
*/

// 性能问题的常见原因
/*
1. 不必要的重新渲染
2. 昂贵的组件计算
3. 过大的组件树
4. 内存泄漏
5. 阻塞主线程的操作
*/
```

### 性能监测工具
```javascript
// 1. React DevTools Profiler
// 在开发环境中使用，可以可视化组件渲染时间

// 2. Performance API
function measureRenderTime(name, fn) {
  performance.mark(`${name}-start`);
  fn();
  performance.mark(`${name}-end`);
  performance.measure(name, `${name}-start`, `${name}-end`);
  
  const measure = performance.getEntriesByName(name)[0];
  console.log(`${name} took ${measure.duration.toFixed(2)}ms`);
}

// 使用示例
measureRenderTime('component-render', () => {
  ReactDOM.render(<App />, document.getElementById('root'));
});

// 3. 自定义性能Hook
function usePerformanceMonitor(componentName) {
  const renderCountRef = useRef(0);
  const startTimeRef = useRef(null);
  
  useEffect(() => {
    renderCountRef.current += 1;
    const endTime = performance.now();
    
    if (startTimeRef.current) {
      const renderTime = endTime - startTimeRef.current;
      console.log(
        `${componentName} render #${renderCountRef.current}: ${renderTime.toFixed(2)}ms`
      );
    }
    
    startTimeRef.current = performance.now();
  });
  
  return {
    renderCount: renderCountRef.current,
    logRenderReason: (reason) => {
      console.log(`${componentName} re-rendered because: ${reason}`);
    }
  };
}

// 使用性能监控Hook
function MyComponent({ userId, data }) {
  const { renderCount, logRenderReason } = usePerformanceMonitor('MyComponent');
  
  useEffect(() => {
    logRenderReason('userId changed');
  }, [userId, logRenderReason]);
  
  useEffect(() => {
    logRenderReason('data changed');
  }, [data, logRenderReason]);
  
  return (
    <div>
      <p>Render count: {renderCount}</p>
      {/* 组件内容 */}
    </div>
  );
}
```

## 12.2 渲染优化

### React.memo和组件缓存
```javascript
// 基础memo使用
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data, onUpdate }) {
  console.log('ExpensiveComponent rendered');
  
  // 模拟昂贵的计算
  const processedData = useMemo(() => {
    console.log('Processing data...');
    return data.map(item => ({
      ...item,
      processed: true,
      timestamp: Date.now()
    }));
  }, [data]);
  
  return (
    <div>
      {processedData.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
      <button onClick={() => onUpdate('new data')}>Update</button>
    </div>
  );
});

// 自定义比较函数
const SmartComponent = React.memo(
  function SmartComponent({ user, settings, onAction }) {
    return (
      <div>
        <h2>{user.name}</h2>
        <p>Theme: {settings.theme}</p>
        <button onClick={onAction}>Action</button>
      </div>
    );
  },
  (prevProps, nextProps) => {
    // 自定义比较逻辑
    return (
      prevProps.user.id === nextProps.user.id &&
      prevProps.user.name === nextProps.user.name &&
      prevProps.settings.theme === nextProps.settings.theme
      // 注意：不比较onAction，因为它可能每次都不同
    );
  }
);

// 深度比较Hook
function useDeepMemo(value) {
  const ref = useRef();
  
  if (!ref.current || !deepEqual(ref.current, value)) {
    ref.current = value;
  }
  
  return ref.current;
}

function deepEqual(a, b) {
  if (a === b) return true;
  if (a == null || b == null) return false;
  if (typeof a !== 'object' || typeof b !== 'object') return false;
  
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  
  if (keysA.length !== keysB.length) return false;
  
  for (let key of keysA) {
    if (!keysB.includes(key) || !deepEqual(a[key], b[key])) {
      return false;
    }
  }
  
  return true;
}

// 使用深度比较的组件
function DeepComparisonComponent({ complexData }) {
  const memoizedData = useDeepMemo(complexData);
  
  const processedData = useMemo(() => {
    return processComplexData(memoizedData);
  }, [memoizedData]);
  
  return <div>{/* 渲染逻辑 */}</div>;
}
```

### useMemo和useCallback优化
```javascript
// useMemo优化昂贵计算
function DataAnalysis({ data, filters, sortBy }) {
  // 过滤数据
  const filteredData = useMemo(() => {
    console.log('Filtering data...');
    return data.filter(item => {
      return Object.entries(filters).every(([key, value]) => {
        if (!value) return true;
        return item[key]?.toString().toLowerCase().includes(value.toLowerCase());
      });
    });
  }, [data, filters]);
  
  // 排序数据
  const sortedData = useMemo(() => {
    console.log('Sorting data...');
    if (!sortBy) return filteredData;
    
    return [...filteredData].sort((a, b) => {
      const aValue = a[sortBy.field];
      const bValue = b[sortBy.field];
      
      if (sortBy.direction === 'desc') {
        return bValue > aValue ? 1 : -1;
      }
      return aValue > bValue ? 1 : -1;
    });
  }, [filteredData, sortBy]);
  
  // 统计信息
  const statistics = useMemo(() => {
    console.log('Calculating statistics...');
    return {
      total: sortedData.length,
      average: sortedData.reduce((sum, item) => sum + item.value, 0) / sortedData.length,
      max: Math.max(...sortedData.map(item => item.value)),
      min: Math.min(...sortedData.map(item => item.value))
    };
  }, [sortedData]);
  
  return (
    <div>
      <div>Total: {statistics.total}</div>
      <div>Average: {statistics.average.toFixed(2)}</div>
      <div>Max: {statistics.max}</div>
      <div>Min: {statistics.min}</div>
      
      <table>
        {sortedData.map(item => (
          <tr key={item.id}>
            <td>{item.name}</td>
            <td>{item.value}</td>
          </tr>
        ))}
      </table>
    </div>
  );
}

// useCallback优化事件处理器
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');
  
  // 优化的添加函数
  const addTodo = useCallback((text) => {
    setTodos(prev => [
      ...prev,
      {
        id: Date.now(),
        text,
        completed: false
      }
    ]);
  }, []);
  
  // 优化的切换函数
  const toggleTodo = useCallback((id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  }, []);
  
  // 优化的删除函数
  const deleteTodo = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []);
  
  // 过滤后的todos
  const filteredTodos = useMemo(() => {
    switch (filter) {
      case 'active':
        return todos.filter(todo => !todo.completed);
      case 'completed':
        return todos.filter(todo => todo.completed);
      default:
        return todos;
    }
  }, [todos, filter]);
  
  return (
    <div>
      <TodoInput onAdd={addTodo} />
      <TodoFilter currentFilter={filter} onFilterChange={setFilter} />
      <TodoList 
        todos={filteredTodos}
        onToggle={toggleTodo}
        onDelete={deleteTodo}
      />
    </div>
  );
}

// 优化的Todo组件
const TodoItem = React.memo(function TodoItem({ todo, onToggle, onDelete }) {
  const handleToggle = useCallback(() => {
    onToggle(todo.id);
  }, [todo.id, onToggle]);
  
  const handleDelete = useCallback(() => {
    onDelete(todo.id);
  }, [todo.id, onDelete]);
  
  return (
    <div className={`todo-item ${todo.completed ? 'completed' : ''}`}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={handleToggle}
      />
      <span>{todo.text}</span>
      <button onClick={handleDelete}>Delete</button>
    </div>
  );
});
```

### 虚拟列表优化
```javascript
// 固定高度虚拟列表
function FixedVirtualList({ items, itemHeight = 50, containerHeight = 400 }) {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef();
  
  // 计算可见范围
  const visibleRange = useMemo(() => {
    const start = Math.floor(scrollTop / itemHeight);
    const visibleCount = Math.ceil(containerHeight / itemHeight);
    const end = Math.min(start + visibleCount + 1, items.length);
    
    return { start, end };
  }, [scrollTop, itemHeight, containerHeight, items.length]);
  
  // 可见项目
  const visibleItems = useMemo(() => {
    return items.slice(visibleRange.start, visibleRange.end).map((item, index) => ({
      ...item,
      index: visibleRange.start + index
    }));
  }, [items, visibleRange]);
  
  const handleScroll = useCallback((e) => {
    setScrollTop(e.target.scrollTop);
  }, []);
  
  const totalHeight = items.length * itemHeight;
  const offsetY = visibleRange.start * itemHeight;
  
  return (
    <div
      ref={containerRef}
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={handleScroll}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div
          style={{
            transform: `translateY(${offsetY}px)`,
            position: 'absolute',
            top: 0,
            left: 0,
            right: 0
          }}
        >
          {visibleItems.map(item => (
            <VirtualListItem
              key={item.id}
              item={item}
              height={itemHeight}
              index={item.index}
            />
          ))}
        </div>
      </div>
    </div>
  );
}

const VirtualListItem = React.memo(function VirtualListItem({ item, height, index }) {
  return (
    <div
      style={{
        height,
        display: 'flex',
        alignItems: 'center',
        padding: '0 16px',
        borderBottom: '1px solid #eee'
      }}
    >
      <span>#{index + 1}</span>
      <span style={{ marginLeft: 16 }}>{item.name}</span>
      <span style={{ marginLeft: 'auto' }}>{item.value}</span>
    </div>
  );
});

// 动态高度虚拟列表
function DynamicVirtualList({ items, containerHeight = 400, estimatedItemHeight = 50 }) {
  const [scrollTop, setScrollTop] = useState(0);
  const [itemHeights, setItemHeights] = useState(new Map());
  const containerRef = useRef();
  const itemRefs = useRef(new Map());
  
  // 测量项目高度
  const measureItem = useCallback((index, element) => {
    if (element) {
      const height = element.getBoundingClientRect().height;
      setItemHeights(prev => new Map(prev).set(index, height));
      itemRefs.current.set(index, element);
    }
  }, []);
  
  // 计算累积高度
  const cumulativeHeights = useMemo(() => {
    const heights = [0];
    for (let i = 0; i < items.length; i++) {
      const itemHeight = itemHeights.get(i) || estimatedItemHeight;
      heights.push(heights[i] + itemHeight);
    }
    return heights;
  }, [items.length, itemHeights, estimatedItemHeight]);
  
  // 查找可见范围
  const visibleRange = useMemo(() => {
    const start = cumulativeHeights.findIndex(height => height > scrollTop) - 1;
    const startIndex = Math.max(0, start);
    
    let endIndex = startIndex;
    let currentHeight = cumulativeHeights[startIndex];
    
    while (currentHeight < scrollTop + containerHeight && endIndex < items.length - 1) {
      endIndex++;
      currentHeight = cumulativeHeights[endIndex];
    }
    
    return { start: startIndex, end: Math.min(endIndex + 1, items.length) };
  }, [cumulativeHeights, scrollTop, containerHeight, items.length]);
  
  const visibleItems = items.slice(visibleRange.start, visibleRange.end);
  const totalHeight = cumulativeHeights[items.length] || 0;
  const offsetY = cumulativeHeights[visibleRange.start] || 0;
  
  return (
    <div
      ref={containerRef}
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={(e) => setScrollTop(e.target.scrollTop)}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div
          style={{
            transform: `translateY(${offsetY}px)`,
            position: 'absolute',
            top: 0,
            left: 0,
            right: 0
          }}
        >
          {visibleItems.map((item, i) => {
            const actualIndex = visibleRange.start + i;
            return (
              <DynamicListItem
                key={item.id}
                item={item}
                index={actualIndex}
                onMeasure={measureItem}
              />
            );
          })}
        </div>
      </div>
    </div>
  );
}

const DynamicListItem = React.memo(function DynamicListItem({ item, index, onMeasure }) {
  const ref = useRef();
  
  useEffect(() => {
    if (ref.current) {
      onMeasure(index, ref.current);
    }
  }, [index, onMeasure]);
  
  return (
    <div ref={ref} style={{ padding: '16px', borderBottom: '1px solid #eee' }}>
      <h3>{item.title}</h3>
      <p>{item.description}</p>
      {item.tags && (
        <div>
          {item.tags.map(tag => (
            <span key={tag} style={{ marginRight: 8, padding: '2px 8px', background: '#f0f0f0' }}>
              {tag}
            </span>
          ))}
        </div>
      )}
    </div>
  );
});
```

## 12.3 代码分割和懒加载

### 组件级代码分割
```javascript
// 1. React.lazy基础用法
import React, { Suspense, lazy } from 'react';

// 懒加载组件
const LazyDashboard = lazy(() => import('./components/Dashboard'));
const LazyProfile = lazy(() => import('./components/Profile'));
const LazySettings = lazy(() => import('./components/Settings'));

// 带错误处理的懒加载
const LazyAdminPanel = lazy(() =>
  import('./components/AdminPanel').catch(error => {
    console.error('Failed to load AdminPanel:', error);
    // 返回fallback组件
    return { default: () => <div>Failed to load admin panel</div> };
  })
);

// 预加载策略
const preloadDashboard = () => {
  const componentImport = import('./components/Dashboard');
  return componentImport;
};

// 路由级代码分割
import { Routes, Route } from 'react-router-dom';

function App() {
  // 预加载常用组件
  useEffect(() => {
    // 用户登录后预加载Dashboard
    if (user) {
      preloadDashboard();
    }
  }, [user]);
  
  return (
    <div className="app">
      <Header />
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<LazyDashboard />} />
          <Route path="/profile" element={<LazyProfile />} />
          <Route path="/settings" element={<LazySettings />} />
          <Route path="/admin" element={<LazyAdminPanel />} />
        </Routes>
      </Suspense>
      <Footer />
    </div>
  );
}

// 智能加载组件
function SmartLoader({ loader, fallback, retryCount = 3 }) {
  const [LoadedComponent, setLoadedComponent] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);
  const retryCountRef = useRef(0);
  
  const loadComponent = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const component = await loader();
      setLoadedComponent(() => component.default);
    } catch (err) {
      console.error('Component loading failed:', err);
      
      if (retryCountRef.current < retryCount) {
        retryCountRef.current++;
        setTimeout(loadComponent, 1000 * retryCountRef.current);
      } else {
        setError(err);
      }
    } finally {
      setLoading(false);
    }
  }, [loader, retryCount]);
  
  useEffect(() => {
    loadComponent();
  }, [loadComponent]);
  
  if (loading) {
    return fallback || <div>Loading...</div>;
  }
  
  if (error) {
    return (
      <div>
        <p>Failed to load component</p>
        <button onClick={() => {
          retryCountRef.current = 0;
          loadComponent();
        }}>
          Retry
        </button>
      </div>
    );
  }
  
  return LoadedComponent ? <LoadedComponent /> : null;
}
```

### 动态导入和模块分割
```javascript
// 1. 动态导入工具函数
async function importModule(modulePath) {
  try {
    const module = await import(modulePath);
    return module.default || module;
  } catch (error) {
    console.error(`Failed to import module: ${modulePath}`, error);
    throw error;
  }
}

// 2. 条件性加载
async function loadFeature(featureName) {
  const featureModules = {
    charts: () => import('./features/Charts'),
    analytics: () => import('./features/Analytics'),
    reports: () => import('./features/Reports'),
    admin: () => import('./features/Admin')
  };
  
  const loader = featureModules[featureName];
  if (!loader) {
    throw new Error(`Unknown feature: ${featureName}`);
  }
  
  return loader();
}

// 3. 功能开关和动态加载
function FeatureLoader({ feature, children, fallback }) {
  const [FeatureComponent, setFeatureComponent] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    async function loadFeatureComponent() {
      try {
        setLoading(true);
        const module = await loadFeature(feature);
        setFeatureComponent(() => module.default);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    }
    
    loadFeatureComponent();
  }, [feature]);
  
  if (loading) return fallback || <div>Loading feature...</div>;
  if (error) return <div>Feature unavailable</div>;
  if (!FeatureComponent) return null;
  
  return <FeatureComponent>{children}</FeatureComponent>;
}

// 使用示例
function App() {
  const [enabledFeatures, setEnabledFeatures] = useState(['charts', 'analytics']);
  
  return (
    <div>
      {enabledFeatures.includes('charts') && (
        <FeatureLoader feature="charts" fallback={<ChartsSkeleton />}>
          <ChartsContent />
        </FeatureLoader>
      )}
      
      {enabledFeatures.includes('analytics') && (
        <FeatureLoader feature="analytics" fallback={<AnalyticsSkeleton />}>
          <AnalyticsContent />
        </FeatureLoader>
      )}
    </div>
  );
}

// 4. 第三方库的动态加载
function ChartComponent({ data, type }) {
  const [ChartLibrary, setChartLibrary] = useState(null);
  
  useEffect(() => {
    async function loadChartLibrary() {
      // 根据图表类型加载不同的库
      if (type === 'line' || type === 'bar') {
        const { default: Chart } = await import('chart.js/auto');
        setChartLibrary(() => Chart);
      } else if (type === 'd3') {
        const d3 = await import('d3');
        setChartLibrary(() => d3);
      }
    }
    
    loadChartLibrary();
  }, [type]);
  
  if (!ChartLibrary) {
    return <div>Loading chart library...</div>;
  }
  
  return <Chart library={ChartLibrary} data={data} type={type} />;
}
```

## 12.4 内存管理和泄漏预防

### 内存泄漏检测和预防
```javascript
// 1. 副作用清理
function TimerComponent() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(prev => prev + 1);
    }, 1000);
    
    // 清理定时器
    return () => {
      clearInterval(interval);
    };
  }, []);
  
  useEffect(() => {
    const handleResize = () => {
      console.log('Window resized');
    };
    
    window.addEventListener('resize', handleResize);
    
    // 清理事件监听器
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);
  
  return <div>Count: {count}</div>;
}

// 2. AbortController for API calls
function DataFetcher({ url }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const abortControllerRef = useRef(null);
  
  useEffect(() => {
    async function fetchData() {
      // 取消之前的请求
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
      
      abortControllerRef.current = new AbortController();
      
      try {
        setLoading(true);
        const response = await fetch(url, {
          signal: abortControllerRef.current.signal
        });
        const result = await response.json();
        setData(result);
      } catch (error) {
        if (error.name !== 'AbortError') {
          console.error('Fetch error:', error);
        }
      } finally {
        setLoading(false);
      }
    }
    
    if (url) {
      fetchData();
    }
    
    // 清理函数
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [url]);
  
  return loading ? <div>Loading...</div> : <div>{JSON.stringify(data)}</div>;
}

// 3. 内存泄漏检测Hook
function useMemoryLeakDetector(componentName) {
  const mountedRef = useRef(true);
  const timersRef = useRef(new Set());
  const listenersRef = useRef(new Set());
  
  // 注册定时器
  const registerTimer = useCallback((timerId) => {
    timersRef.current.add(timerId);
  }, []);
  
  // 注册事件监听器
  const registerListener = useCallback((element, event, handler) => {
    const listener = { element, event, handler };
    listenersRef.current.add(listener);
    element.addEventListener(event, handler);
  }, []);
  
  // 清理所有资源
  const cleanup = useCallback(() => {
    // 清理定时器
    timersRef.current.forEach(timerId => {
      clearTimeout(timerId);
      clearInterval(timerId);
    });
    timersRef.current.clear();
    
    // 清理事件监听器
    listenersRef.current.forEach(({ element, event, handler }) => {
      element.removeEventListener(event, handler);
    });
    listenersRef.current.clear();
  }, []);
  
  useEffect(() => {
    return () => {
      mountedRef.current = false;
      cleanup();
      
      // 检测潜在内存泄漏
      setTimeout(() => {
        if (timersRef.current.size > 0) {
          console.warn(`${componentName}: ${timersRef.current.size} timers not cleaned up`);
        }
        if (listenersRef.current.size > 0) {
          console.warn(`${componentName}: ${listenersRef.current.size} listeners not cleaned up`);
        }
      }, 1000);
    };
  }, [componentName, cleanup]);
  
  return {
    isMounted: () => mountedRef.current,
    registerTimer,
    registerListener,
    cleanup
  };
}

// 使用内存泄漏检测
function SafeComponent() {
  const { isMounted, registerTimer, registerListener } = useMemoryLeakDetector('SafeComponent');
  const [data, setData] = useState(null);
  
  useEffect(() => {
    // 安全的异步操作
    const fetchData = async () => {
      const response = await fetch('/api/data');
      const result = await response.json();
      
      if (isMounted()) {
        setData(result);
      }
    };
    
    fetchData();
    
    // 注册定时器
    const timerId = setInterval(() => {
      if (isMounted()) {
        console.log('Timer tick');
      }
    }, 1000);
    registerTimer(timerId);
    
    // 注册事件监听器
    const handleClick = () => {
      if (isMounted()) {
        console.log('Document clicked');
      }
    };
    registerListener(document, 'click', handleClick);
  }, [isMounted, registerTimer, registerListener]);
  
  return <div>{data ? JSON.stringify(data) : 'Loading...'}</div>;
}
```

### WeakMap和WeakSet使用
```javascript
// 1. 使用WeakMap存储私有数据
const privateData = new WeakMap();

class ComponentManager {
  constructor(component) {
    privateData.set(this, {
      component,
      listeners: new Set(),
      timers: new Set()
    });
  }
  
  addListener(element, event, handler) {
    const data = privateData.get(this);
    data.listeners.add({ element, event, handler });
    element.addEventListener(event, handler);
  }
  
  addTimer(callback, delay) {
    const data = privateData.get(this);
    const timerId = setTimeout(callback, delay);
    data.timers.add(timerId);
    return timerId;
  }
  
  cleanup() {
    const data = privateData.get(this);
    
    // 清理监听器
    data.listeners.forEach(({ element, event, handler }) => {
      element.removeEventListener(event, handler);
    });
    
    // 清理定时器
    data.timers.forEach(timerId => {
      clearTimeout(timerId);
    });
    
    // 清理私有数据
    privateData.delete(this);
  }
}

// 2. 组件实例缓存
const componentCache = new WeakMap();

function getCachedComponent(props) {
  if (componentCache.has(props)) {
    return componentCache.get(props);
  }
  
  const component = createExpensiveComponent(props);
  componentCache.set(props, component);
  return component;
}

// 3. DOM节点关联数据
const nodeData = new WeakMap();

function attachDataToNode(node, data) {
  nodeData.set(node, data);
}

function getNodeData(node) {
  return nodeData.get(node);
}

// 当DOM节点被垃圾回收时，关联数据也会自动清理
```

## 12.5 实际案例：性能优化实践

### 大列表优化案例
```javascript
// 优化前：性能问题的列表组件
function SlowProductList({ products, onProductClick }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [sortBy, setSortBy] = useState('name');
  const [category, setCategory] = useState('all');
  
  // 问题：每次渲染都重新计算
  const filteredProducts = products
    .filter(product => {
      if (category !== 'all' && product.category !== category) return false;
      if (searchTerm && !product.name.toLowerCase().includes(searchTerm.toLowerCase())) return false;
      return true;
    })
    .sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      if (sortBy === 'price') return a.price - b.price;
      return 0;
    });
  
  return (
    <div>
      <div>
        <input
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          placeholder="Search products..."
        />
        <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
          <option value="name">Sort by Name</option>
          <option value="price">Sort by Price</option>
        </select>
        <select value={category} onChange={(e) => setCategory(e.target.value)}>
          <option value="all">All Categories</option>
          <option value="electronics">Electronics</option>
          <option value="clothing">Clothing</option>
        </select>
      </div>
      
      {/* 问题：大列表直接渲染 */}
      <div>
        {filteredProducts.map(product => (
          <div key={product.id} onClick={() => onProductClick(product)}>
            <img src={product.image} alt={product.name} />
            <h3>{product.name}</h3>
            <p>${product.price}</p>
            <p>{product.description}</p>
          </div>
        ))}
      </div>
    </div>
  );
}

// 优化后：高性能列表组件
function OptimizedProductList({ products, onProductClick }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [sortBy, setSortBy] = useState('name');
  const [category, setCategory] = useState('all');
  
  // 优化：使用防抖的搜索
  const debouncedSearchTerm = useDebounce(searchTerm, 300);
  
  // 优化：缓存过滤和排序结果
  const filteredProducts = useMemo(() => {
    console.log('Filtering and sorting products...');
    
    let result = products;
    
    // 过滤
    if (category !== 'all') {
      result = result.filter(product => product.category === category);
    }
    
    if (debouncedSearchTerm) {
      const searchLower = debouncedSearchTerm.toLowerCase();
      result = result.filter(product => 
        product.name.toLowerCase().includes(searchLower) ||
        product.description.toLowerCase().includes(searchLower)
      );
    }
    
    // 排序
    result = [...result].sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      if (sortBy === 'price') return a.price - b.price;
      if (sortBy === 'rating') return b.rating - a.rating;
      return 0;
    });
    
    return result;
  }, [products, debouncedSearchTerm, sortBy, category]);
  
  // 优化：稳定的事件处理器
  const handleProductClick = useCallback((product) => {
    onProductClick(product);
  }, [onProductClick]);
  
  return (
    <div>
      <ProductFilters
        searchTerm={searchTerm}
        onSearchChange={setSearchTerm}
        sortBy={sortBy}
        onSortChange={setSortBy}
        category={category}
        onCategoryChange={setCategory}
      />
      
      {/* 优化：使用虚拟列表 */}
      <VirtualizedProductList
        products={filteredProducts}
        onProductClick={handleProductClick}
        itemHeight={200}
        containerHeight={600}
      />
    </div>
  );
}

// 优化的产品项组件
const ProductItem = React.memo(function ProductItem({ product, onClick, style }) {
  const handleClick = useCallback(() => {
    onClick(product);
  }, [product, onClick]);
  
  // 优化：图片懒加载
  const [imgSrc, setImgSrc] = useState(null);
  const [imgLoaded, setImgLoaded] = useState(false);
  const imgRef = useRef();
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setImgSrc(product.image);
          observer.disconnect();
        }
      },
      { threshold: 0.1 }
    );
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, [product.image]);
  
  return (
    <div style={style} className="product-item" onClick={handleClick}>
      <div ref={imgRef} className="product-image">
        {imgSrc ? (
          <img
            src={imgSrc}
            alt={product.name}
            onLoad={() => setImgLoaded(true)}
            style={{ opacity: imgLoaded ? 1 : 0 }}
          />
        ) : (
          <div className="image-placeholder">Loading...</div>
        )}
      </div>
      
      <div className="product-info">
        <h3>{product.name}</h3>
        <p className="price">${product.price}</p>
        <p className="description">{product.description}</p>
        <div className="rating">
          {Array.from({ length: 5 }, (_, i) => (
            <span key={i} className={i < product.rating ? 'star filled' : 'star'}>
              ★
            </span>
          ))}
        </div>
      </div>
    </div>
  );
});

// 虚拟化产品列表
function VirtualizedProductList({ products, onProductClick, itemHeight, containerHeight }) {
  return (
    <FixedVirtualList
      items={products}
      itemHeight={itemHeight}
      containerHeight={containerHeight}
      renderItem={({ item, style }) => (
        <ProductItem
          key={item.id}
          product={item}
          onClick={onProductClick}
          style={style}
        />
      )}
    />
  );
}

// 防抖Hook
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
```

## 12.6 Bundle分析和优化

### Webpack Bundle Analyzer
```javascript
// 安装和配置
// npm install --save-dev webpack-bundle-analyzer

// 在package.json中添加脚本
{
  "scripts": {
    "analyze": "npm run build && npx webpack-bundle-analyzer build/static/js/*.js"
  }
}

// 自定义webpack配置 (如果使用eject)
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'server',
      openAnalyzer: true,
      generateStatsFile: true
    })
  ]
};

// 分析脚本
function analyzeBundleSize() {
  const fs = require('fs');
  const path = require('path');
  
  const buildDir = path.join(__dirname, 'build/static/js');
  const files = fs.readdirSync(buildDir);
  
  const bundles = files
    .filter(file => file.endsWith('.js'))
    .map(file => {
      const filePath = path.join(buildDir, file);
      const stats = fs.statSync(filePath);
      return {
        name: file,
        size: stats.size,
        sizeKB: (stats.size / 1024).toFixed(2)
      };
    })
    .sort((a, b) => b.size - a.size);
  
  console.log('Bundle Analysis:');
  console.log('================');
  bundles.forEach(bundle => {
    console.log(`${bundle.name}: ${bundle.sizeKB} KB`);
  });
  
  const totalSize = bundles.reduce((sum, bundle) => sum + bundle.size, 0);
  console.log(`Total: ${(totalSize / 1024).toFixed(2)} KB`);
}
```

### 代码分割策略
```javascript
// 1. 路由级分割
const routes = [
  {
    path: '/',
    component: lazy(() => import('./pages/Home'))
  },
  {
    path: '/dashboard',
    component: lazy(() => import('./pages/Dashboard'))
  },
  {
    path: '/profile',
    component: lazy(() => import('./pages/Profile'))
  }
];

// 2. 功能级分割
const FeatureA = lazy(() => import('./features/FeatureA'));
const FeatureB = lazy(() => import('./features/FeatureB'));

// 3. 第三方库分割
// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          enforce: true
        }
      }
    }
  }
};

// 4. 动态polyfill加载
async function loadPolyfills() {
  const polyfills = [];
  
  if (!window.Promise) {
    polyfills.push(import('es6-promise/auto'));
  }
  
  if (!window.fetch) {
    polyfills.push(import('whatwg-fetch'));
  }
  
  if (!Array.prototype.includes) {
    polyfills.push(import('core-js/features/array/includes'));
  }
  
  await Promise.all(polyfills);
}

// 在应用启动前加载polyfills
async function startApp() {
  await loadPolyfills();
  
  const React = await import('react');
  const ReactDOM = await import('react-dom');
  const App = await import('./App');
  
  ReactDOM.render(<App.default />, document.getElementById('root'));
}

startApp();
```

## 12.7 总结

### 性能优化清单
```javascript
// 组件层面
/*
✅ 使用React.memo包装纯组件
✅ 使用useMemo缓存昂贵计算
✅ 使用useCallback稳定事件处理器
✅ 避免在render中创建对象和函数
✅ 合理使用key属性
✅ 避免不必要的状态提升
*/

// 渲染层面
/*
✅ 实现虚拟滚动处理大列表
✅ 使用Intersection Observer实现懒加载
✅ 分片渲染大量数据
✅ 防抖和节流用户输入
✅ 避免深层嵌套和过度渲染
*/

// 资源层面
/*
✅ 实现代码分割和懒加载
✅ 优化图片加载和尺寸
✅ 压缩和minify代码
✅ 使用CDN加速资源加载
✅ 实现有效的缓存策略
*/

// 内存层面
/*
✅ 及时清理副作用
✅ 避免内存泄漏
✅ 正确使用WeakMap和WeakSet
✅ 监控内存使用情况
*/
```

### 本章重点
1. **性能监测**：学会使用工具分析性能问题
2. **渲染优化**：掌握React渲染优化的核心技术
3. **代码分割**：实现智能的代码分割策略
4. **内存管理**：预防和解决内存泄漏问题
5. **实践案例**：将优化技术应用到实际项目中

### 下章预告
下一章我们将学习部署，包括构建优化、服务器配置、CI/CD等内容。

### 练习作业
1. 分析一个现有React应用的性能瓶颈并优化
2. 实现一个高性能的无限滚动列表
3. 设计一个大型应用的代码分割策略

### 常见问题
1. **Q: 什么时候应该使用React.memo？**
   A: 当组件的props变化不频繁，或者组件渲染成本较高时考虑使用。

2. **Q: useMemo和useCallback的使用原则？**
   A: 只在确实需要优化时使用，过度使用可能适得其反。

3. **Q: 如何平衡代码分割的粒度？**
   A: 根据用户访问模式和功能相关性来决定分割边界。
