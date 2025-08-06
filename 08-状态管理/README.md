# 第八章：状态管理

## 学习目标
- 理解React应用中的状态管理概念
- 掌握Context API的高级用法
- 学习状态管理的最佳实践
- 了解流行的状态管理库（Redux基础）
- 能够选择合适的状态管理方案

## 8.1 状态管理概述

### 为什么需要状态管理？
随着应用复杂度增加，组件间的状态共享和传递变得困难：

1. **Props钻取问题**：深层嵌套组件间传递数据
2. **状态分散**：状态散落在各个组件中，难以维护
3. **状态同步**：多个组件需要共享相同状态
4. **状态预测性**：状态变化难以跟踪和调试

### 状态管理方案对比
```javascript
// 1. 本地状态 - 适合简单组件
function SimpleCounter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// 2. Props传递 - 适合浅层级
function Parent() {
  const [user, setUser] = useState(null);
  return <Child user={user} onUserChange={setUser} />;
}

// 3. Context - 适合中等复杂度
const UserContext = createContext();
function App() {
  return (
    <UserContext.Provider value={user}>
      <DeepNestedComponent />
    </UserContext.Provider>
  );
}

// 4. 状态管理库 - 适合复杂应用
// Redux, Zustand, Recoil 等
```

## 8.2 高级Context模式

### 多Context组合
```javascript
import React, { createContext, useContext, useReducer, useCallback } from 'react';

// 1. 认证Context
const AuthContext = createContext();

const authInitialState = {
  user: null,
  isAuthenticated: false,
  loading: false,
  error: null
};

function authReducer(state, action) {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, loading: true, error: null };
    case 'LOGIN_SUCCESS':
      return {
        ...state,
        user: action.payload,
        isAuthenticated: true,
        loading: false,
        error: null
      };
    case 'LOGIN_FAILURE':
      return {
        ...state,
        user: null,
        isAuthenticated: false,
        loading: false,
        error: action.payload
      };
    case 'LOGOUT':
      return {
        ...state,
        user: null,
        isAuthenticated: false,
        error: null
      };
    default:
      return state;
  }
}

function AuthProvider({ children }) {
  const [state, dispatch] = useReducer(authReducer, authInitialState);
  
  const login = useCallback(async (credentials) => {
    dispatch({ type: 'LOGIN_START' });
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      });
      
      if (!response.ok) {
        throw new Error('登录失败');
      }
      
      const user = await response.json();
      dispatch({ type: 'LOGIN_SUCCESS', payload: user });
      
      // 保存到localStorage
      localStorage.setItem('user', JSON.stringify(user));
    } catch (error) {
      dispatch({ type: 'LOGIN_FAILURE', payload: error.message });
    }
  }, []);
  
  const logout = useCallback(() => {
    dispatch({ type: 'LOGOUT' });
    localStorage.removeItem('user');
  }, []);
  
  const value = {
    ...state,
    login,
    logout
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// 2. 购物车Context
const CartContext = createContext();

const cartInitialState = {
  items: [],
  total: 0,
  itemCount: 0
};

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItem = state.items.find(item => item.id === action.payload.id);
      
      if (existingItem) {
        const updatedItems = state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
        return calculateCartTotals({ ...state, items: updatedItems });
      } else {
        const newItems = [...state.items, { ...action.payload, quantity: 1 }];
        return calculateCartTotals({ ...state, items: newItems });
      }
    }
    
    case 'REMOVE_ITEM': {
      const updatedItems = state.items.filter(item => item.id !== action.payload);
      return calculateCartTotals({ ...state, items: updatedItems });
    }
    
    case 'UPDATE_QUANTITY': {
      const { id, quantity } = action.payload;
      if (quantity <= 0) {
        const updatedItems = state.items.filter(item => item.id !== id);
        return calculateCartTotals({ ...state, items: updatedItems });
      } else {
        const updatedItems = state.items.map(item =>
          item.id === id ? { ...item, quantity } : item
        );
        return calculateCartTotals({ ...state, items: updatedItems });
      }
    }
    
    case 'CLEAR_CART':
      return cartInitialState;
      
    default:
      return state;
  }
}

function calculateCartTotals(state) {
  const total = state.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  const itemCount = state.items.reduce((sum, item) => sum + item.quantity, 0);
  return { ...state, total, itemCount };
}

function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, cartInitialState);
  
  const addItem = useCallback((product) => {
    dispatch({ type: 'ADD_ITEM', payload: product });
  }, []);
  
  const removeItem = useCallback((productId) => {
    dispatch({ type: 'REMOVE_ITEM', payload: productId });
  }, []);
  
  const updateQuantity = useCallback((productId, quantity) => {
    dispatch({ type: 'UPDATE_QUANTITY', payload: { id: productId, quantity } });
  }, []);
  
  const clearCart = useCallback(() => {
    dispatch({ type: 'CLEAR_CART' });
  }, []);
  
  const value = {
    ...state,
    addItem,
    removeItem,
    updateQuantity,
    clearCart
  };
  
  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
}

// 3. 主题Context
const ThemeContext = createContext();

const themes = {
  light: {
    background: '#ffffff',
    foreground: '#000000',
    primary: '#007bff',
    secondary: '#6c757d'
  },
  dark: {
    background: '#121212',
    foreground: '#ffffff',
    primary: '#bb86fc',
    secondary: '#03dac6'
  }
};

function ThemeProvider({ children }) {
  const [currentTheme, setCurrentTheme] = useState('light');
  
  const toggleTheme = useCallback(() => {
    setCurrentTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);
  
  const theme = themes[currentTheme];
  
  const value = {
    currentTheme,
    theme,
    toggleTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 4. 组合Provider
function AppProviders({ children }) {
  return (
    <ThemeProvider>
      <AuthProvider>
        <CartProvider>
          {children}
        </CartProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}

// 5. 自定义Hook
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth必须在AuthProvider内使用');
  }
  return context;
}

function useCart() {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart必须在CartProvider内使用');
  }
  return context;
}

function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme必须在ThemeProvider内使用');
  }
  return context;
}
```

### Context性能优化
```javascript
// 1. 分离读写Context
const StateContext = createContext();
const DispatchContext = createContext();

function AppProvider({ children }) {
  const [state, dispatch] = useReducer(appReducer, initialState);
  
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

function useAppState() {
  const context = useContext(StateContext);
  if (!context) {
    throw new Error('useAppState必须在AppProvider内使用');
  }
  return context;
}

function useAppDispatch() {
  const context = useContext(DispatchContext);
  if (!context) {
    throw new Error('useAppDispatch必须在AppProvider内使用');
  }
  return context;
}

// 2. 选择性订阅
function createSelector(selector) {
  return function useSelector() {
    const state = useAppState();
    return useMemo(() => selector(state), [state, selector]);
  };
}

// 使用选择器
const useUserName = createSelector(state => state.user?.name);
const useCartTotal = createSelector(state => state.cart.total);

function UserProfile() {
  const userName = useUserName(); // 只在用户名变化时重新渲染
  return <div>Hello, {userName}</div>;
}

// 3. Context分割
function createStore(reducer, initialState) {
  const StateContext = createContext();
  const DispatchContext = createContext();
  
  function Provider({ children }) {
    const [state, dispatch] = useReducer(reducer, initialState);
    
    return (
      <StateContext.Provider value={state}>
        <DispatchContext.Provider value={dispatch}>
          {children}
        </DispatchContext.Provider>
      </StateContext.Provider>
    );
  }
  
  function useState() {
    const context = useContext(StateContext);
    if (context === undefined) {
      throw new Error('useState必须在Provider内使用');
    }
    return context;
  }
  
  function useDispatch() {
    const context = useContext(DispatchContext);
    if (context === undefined) {
      throw new Error('useDispatch必须在Provider内使用');
    }
    return context;
  }
  
  return { Provider, useState, useDispatch };
}
```

## 8.3 状态管理模式

### Flux模式
```javascript
// 1. Action Creators
const ActionTypes = {
  ADD_TODO: 'ADD_TODO',
  TOGGLE_TODO: 'TOGGLE_TODO',
  DELETE_TODO: 'DELETE_TODO',
  SET_FILTER: 'SET_FILTER'
};

const actionCreators = {
  addTodo: (text) => ({
    type: ActionTypes.ADD_TODO,
    payload: { text, id: Date.now() }
  }),
  
  toggleTodo: (id) => ({
    type: ActionTypes.TOGGLE_TODO,
    payload: id
  }),
  
  deleteTodo: (id) => ({
    type: ActionTypes.DELETE_TODO,
    payload: id
  }),
  
  setFilter: (filter) => ({
    type: ActionTypes.SET_FILTER,
    payload: filter
  })
};

// 2. Reducer
function todoReducer(state = initialState, action) {
  switch (action.type) {
    case ActionTypes.ADD_TODO:
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: action.payload.id,
            text: action.payload.text,
            completed: false,
            createdAt: new Date().toISOString()
          }
        ]
      };
      
    case ActionTypes.TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
      
    case ActionTypes.DELETE_TODO:
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
      
    case ActionTypes.SET_FILTER:
      return {
        ...state,
        filter: action.payload
      };
      
    default:
      return state;
  }
}

// 3. Store
function createAppStore() {
  const initialState = {
    todos: [],
    filter: 'all'
  };
  
  const [state, dispatch] = useReducer(todoReducer, initialState);
  
  const actions = useMemo(() => ({
    addTodo: (text) => dispatch(actionCreators.addTodo(text)),
    toggleTodo: (id) => dispatch(actionCreators.toggleTodo(id)),
    deleteTodo: (id) => dispatch(actionCreators.deleteTodo(id)),
    setFilter: (filter) => dispatch(actionCreators.setFilter(filter))
  }), [dispatch]);
  
  return { state, actions };
}
```

### MVVM模式
```javascript
// 1. Model层
class TodoModel {
  constructor() {
    this.todos = [];
    this.filter = 'all';
  }
  
  addTodo(text) {
    const todo = {
      id: Date.now(),
      text,
      completed: false,
      createdAt: new Date()
    };
    this.todos.push(todo);
    return todo;
  }
  
  toggleTodo(id) {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.completed = !todo.completed;
    }
    return todo;
  }
  
  deleteTodo(id) {
    const index = this.todos.findIndex(t => t.id === id);
    if (index > -1) {
      return this.todos.splice(index, 1)[0];
    }
  }
  
  getFilteredTodos() {
    switch (this.filter) {
      case 'active':
        return this.todos.filter(t => !t.completed);
      case 'completed':
        return this.todos.filter(t => t.completed);
      default:
        return this.todos;
    }
  }
}

// 2. ViewModel层
function useTodoViewModel() {
  const [model] = useState(() => new TodoModel());
  const [, forceUpdate] = useReducer(x => x + 1, 0);
  
  const addTodo = useCallback((text) => {
    model.addTodo(text);
    forceUpdate();
  }, [model]);
  
  const toggleTodo = useCallback((id) => {
    model.toggleTodo(id);
    forceUpdate();
  }, [model]);
  
  const deleteTodo = useCallback((id) => {
    model.deleteTodo(id);
    forceUpdate();
  }, [model]);
  
  const setFilter = useCallback((filter) => {
    model.filter = filter;
    forceUpdate();
  }, [model]);
  
  return {
    todos: model.getFilteredTodos(),
    filter: model.filter,
    totalCount: model.todos.length,
    completedCount: model.todos.filter(t => t.completed).length,
    actions: {
      addTodo,
      toggleTodo,
      deleteTodo,
      setFilter
    }
  };
}

// 3. View层
function TodoView() {
  const viewModel = useTodoViewModel();
  const [inputValue, setInputValue] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (inputValue.trim()) {
      viewModel.actions.addTodo(inputValue.trim());
      setInputValue('');
    }
  };
  
  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="添加新待办事项"
        />
        <button type="submit">添加</button>
      </form>
      
      <div>
        {['all', 'active', 'completed'].map(filter => (
          <button
            key={filter}
            onClick={() => viewModel.actions.setFilter(filter)}
            style={{
              fontWeight: viewModel.filter === filter ? 'bold' : 'normal'
            }}
          >
            {filter}
          </button>
        ))}
      </div>
      
      <ul>
        {viewModel.todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => viewModel.actions.toggleTodo(todo.id)}
            />
            <span>{todo.text}</span>
            <button onClick={() => viewModel.actions.deleteTodo(todo.id)}>
              删除
            </button>
          </li>
        ))}
      </ul>
      
      <div>
        总计: {viewModel.totalCount}, 
        已完成: {viewModel.completedCount}
      </div>
    </div>
  );
}
```

## 8.4 Redux基础

### Redux核心概念
```javascript
// 1. 安装Redux Toolkit (推荐方式)
// npm install @reduxjs/toolkit react-redux

// 2. 创建Slice
import { createSlice } from '@reduxjs/toolkit';

const userSlice = createSlice({
  name: 'user',
  initialState: {
    currentUser: null,
    isLoading: false,
    error: null
  },
  reducers: {
    loginStart: (state) => {
      state.isLoading = true;
      state.error = null;
    },
    loginSuccess: (state, action) => {
      state.isLoading = false;
      state.currentUser = action.payload;
      state.error = null;
    },
    loginFailure: (state, action) => {
      state.isLoading = false;
      state.error = action.payload;
    },
    logout: (state) => {
      state.currentUser = null;
      state.error = null;
    }
  }
});

export const { loginStart, loginSuccess, loginFailure, logout } = userSlice.actions;
export default userSlice.reducer;

// 3. 异步Action (Thunk)
export const loginAsync = (credentials) => async (dispatch) => {
  dispatch(loginStart());
  try {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    });
    
    if (!response.ok) {
      throw new Error('登录失败');
    }
    
    const user = await response.json();
    dispatch(loginSuccess(user));
  } catch (error) {
    dispatch(loginFailure(error.message));
  }
};

// 4. 创建Store
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';
import cartReducer from './cartSlice';

const store = configureStore({
  reducer: {
    user: userReducer,
    cart: cartReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST']
      }
    })
});

// 5. Provider设置
import { Provider } from 'react-redux';

function App() {
  return (
    <Provider store={store}>
      <AppContent />
    </Provider>
  );
}

// 6. 在组件中使用
import { useSelector, useDispatch } from 'react-redux';

function LoginForm() {
  const { isLoading, error } = useSelector(state => state.user);
  const dispatch = useDispatch();
  const [credentials, setCredentials] = useState({ username: '', password: '' });
  
  const handleSubmit = (e) => {
    e.preventDefault();
    dispatch(loginAsync(credentials));
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={credentials.username}
        onChange={(e) => setCredentials(prev => ({
          ...prev,
          username: e.target.value
        }))}
        placeholder="用户名"
      />
      <input
        type="password"
        value={credentials.password}
        onChange={(e) => setCredentials(prev => ({
          ...prev,
          password: e.target.value
        }))}
        placeholder="密码"
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? '登录中...' : '登录'}
      </button>
      {error && <div style={{ color: 'red' }}>{error}</div>}
    </form>
  );
}
```

### Redux中间件
```javascript
// 1. 自定义中间件
const loggerMiddleware = (store) => (next) => (action) => {
  console.group(`Action: ${action.type}`);
  console.log('Previous State:', store.getState());
  console.log('Action:', action);
  
  const result = next(action);
  
  console.log('Next State:', store.getState());
  console.groupEnd();
  
  return result;
};

// 2. 错误处理中间件
const errorMiddleware = (store) => (next) => (action) => {
  try {
    return next(action);
  } catch (error) {
    console.error('Redux错误:', error);
    // 可以dispatch错误action
    store.dispatch({
      type: 'ERROR_OCCURRED',
      payload: error.message
    });
    throw error;
  }
};

// 3. 本地存储中间件
const localStorageMiddleware = (store) => (next) => (action) => {
  const result = next(action);
  
  // 保存特定状态到localStorage
  const state = store.getState();
  if (action.type.includes('user/')) {
    localStorage.setItem('user', JSON.stringify(state.user.currentUser));
  }
  
  return result;
};

// 4. 配置中间件
const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware()
      .concat(loggerMiddleware)
      .concat(errorMiddleware)
      .concat(localStorageMiddleware)
});
```

## 8.5 实践案例：电商购物应用

让我们构建一个完整的电商购物应用状态管理系统。

```javascript
// store/slices/productsSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// 异步action: 获取产品列表
export const fetchProducts = createAsyncThunk(
  'products/fetchProducts',
  async ({ category, page = 1, limit = 12 }, { rejectWithValue }) => {
    try {
      const response = await fetch(
        `/api/products?category=${category}&page=${page}&limit=${limit}`
      );
      
      if (!response.ok) {
        throw new Error('获取产品失败');
      }
      
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const productsSlice = createSlice({
  name: 'products',
  initialState: {
    items: [],
    categories: [],
    currentCategory: 'all',
    loading: false,
    error: null,
    pagination: {
      page: 1,
      totalPages: 0,
      total: 0
    },
    filters: {
      priceRange: [0, 1000],
      rating: 0,
      inStock: false
    }
  },
  reducers: {
    setCategory: (state, action) => {
      state.currentCategory = action.payload;
      state.pagination.page = 1; // 重置页码
    },
    setFilters: (state, action) => {
      state.filters = { ...state.filters, ...action.payload };
    },
    clearFilters: (state) => {
      state.filters = {
        priceRange: [0, 1000],
        rating: 0,
        inStock: false
      };
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchProducts.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchProducts.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload.products;
        state.pagination = action.payload.pagination;
      })
      .addCase(fetchProducts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

export const { setCategory, setFilters, clearFilters } = productsSlice.actions;
export default productsSlice.reducer;

// store/slices/cartSlice.js
const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    total: 0,
    itemCount: 0,
    shippingCost: 0,
    discountCode: null,
    discountAmount: 0
  },
  reducers: {
    addToCart: (state, action) => {
      const { product, quantity = 1 } = action.payload;
      const existingItem = state.items.find(item => item.id === product.id);
      
      if (existingItem) {
        existingItem.quantity += quantity;
      } else {
        state.items.push({
          ...product,
          quantity
        });
      }
      
      cartSlice.caseReducers.calculateTotals(state);
    },
    
    removeFromCart: (state, action) => {
      const productId = action.payload;
      state.items = state.items.filter(item => item.id !== productId);
      cartSlice.caseReducers.calculateTotals(state);
    },
    
    updateQuantity: (state, action) => {
      const { productId, quantity } = action.payload;
      const item = state.items.find(item => item.id === productId);
      
      if (item) {
        if (quantity <= 0) {
          state.items = state.items.filter(item => item.id !== productId);
        } else {
          item.quantity = quantity;
        }
      }
      
      cartSlice.caseReducers.calculateTotals(state);
    },
    
    applyDiscountCode: (state, action) => {
      const { code, discountAmount } = action.payload;
      state.discountCode = code;
      state.discountAmount = discountAmount;
      cartSlice.caseReducers.calculateTotals(state);
    },
    
    clearCart: (state) => {
      state.items = [];
      state.total = 0;
      state.itemCount = 0;
      state.discountCode = null;
      state.discountAmount = 0;
    },
    
    calculateTotals: (state) => {
      const subtotal = state.items.reduce(
        (sum, item) => sum + (item.price * item.quantity), 
        0
      );
      state.itemCount = state.items.reduce(
        (sum, item) => sum + item.quantity, 
        0
      );
      
      // 计算运费（满99免邮）
      state.shippingCost = subtotal >= 99 ? 0 : 10;
      
      // 应用折扣
      const total = subtotal + state.shippingCost - state.discountAmount;
      state.total = Math.max(0, total);
    }
  }
});

export const { 
  addToCart, 
  removeFromCart, 
  updateQuantity, 
  applyDiscountCode, 
  clearCart 
} = cartSlice.actions;
export default cartSlice.reducer;

// store/slices/orderSlice.js
export const createOrder = createAsyncThunk(
  'orders/createOrder',
  async (orderData, { getState, rejectWithValue }) => {
    try {
      const { cart } = getState();
      const response = await fetch('/api/orders', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          ...orderData,
          items: cart.items,
          total: cart.total
        })
      });
      
      if (!response.ok) {
        throw new Error('创建订单失败');
      }
      
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const orderSlice = createSlice({
  name: 'orders',
  initialState: {
    currentOrder: null,
    orderHistory: [],
    loading: false,
    error: null
  },
  reducers: {
    clearCurrentOrder: (state) => {
      state.currentOrder = null;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(createOrder.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(createOrder.fulfilled, (state, action) => {
        state.loading = false;
        state.currentOrder = action.payload;
        state.orderHistory.unshift(action.payload);
      })
      .addCase(createOrder.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

export const { clearCurrentOrder } = orderSlice.actions;
export default orderSlice.reducer;

// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import productsReducer from './slices/productsSlice';
import cartReducer from './slices/cartSlice';
import orderReducer from './slices/orderSlice';
import userReducer from './slices/userSlice';

const store = configureStore({
  reducer: {
    products: productsReducer,
    cart: cartReducer,
    orders: orderReducer,
    user: userReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE']
      }
    }),
  devTools: process.env.NODE_ENV !== 'production'
});

export default store;

// 选择器 (selectors)
export const selectFilteredProducts = (state) => {
  const { items, filters } = state.products;
  
  return items.filter(product => {
    const { priceRange, rating, inStock } = filters;
    
    // 价格过滤
    if (product.price < priceRange[0] || product.price > priceRange[1]) {
      return false;
    }
    
    // 评分过滤
    if (product.rating < rating) {
      return false;
    }
    
    // 库存过滤
    if (inStock && product.stock <= 0) {
      return false;
    }
    
    return true;
  });
};

export const selectCartSummary = (state) => ({
  items: state.cart.items,
  itemCount: state.cart.itemCount,
  subtotal: state.cart.items.reduce((sum, item) => sum + (item.price * item.quantity), 0),
  shipping: state.cart.shippingCost,
  discount: state.cart.discountAmount,
  total: state.cart.total
});

// 组件使用示例
function ProductList() {
  const dispatch = useDispatch();
  const { loading, error, currentCategory } = useSelector(state => state.products);
  const filteredProducts = useSelector(selectFilteredProducts);
  
  useEffect(() => {
    dispatch(fetchProducts({ category: currentCategory }));
  }, [dispatch, currentCategory]);
  
  const handleAddToCart = (product) => {
    dispatch(addToCart({ product, quantity: 1 }));
  };
  
  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误: {error}</div>;
  
  return (
    <div className="product-grid">
      {filteredProducts.map(product => (
        <div key={product.id} className="product-card">
          <img src={product.image} alt={product.name} />
          <h3>{product.name}</h3>
          <p>${product.price}</p>
          <button onClick={() => handleAddToCart(product)}>
            加入购物车
          </button>
        </div>
      ))}
    </div>
  );
}

function CartSummary() {
  const cartSummary = useSelector(selectCartSummary);
  const dispatch = useDispatch();
  
  return (
    <div className="cart-summary">
      <h3>购物车 ({cartSummary.itemCount})</h3>
      <p>小计: ${cartSummary.subtotal.toFixed(2)}</p>
      <p>运费: ${cartSummary.shipping.toFixed(2)}</p>
      {cartSummary.discount > 0 && (
        <p>折扣: -${cartSummary.discount.toFixed(2)}</p>
      )}
      <h4>总计: ${cartSummary.total.toFixed(2)}</h4>
    </div>
  );
}
```

## 8.6 状态管理最佳实践

### 1. 状态结构设计
```javascript
// ✅ 好的状态结构
const goodState = {
  entities: {
    users: {
      byId: {
        1: { id: 1, name: 'Alice' },
        2: { id: 2, name: 'Bob' }
      },
      allIds: [1, 2]
    }
  },
  ui: {
    currentUserId: 1,
    isLoading: false,
    error: null
  }
};

// ❌ 不好的状态结构
const badState = {
  users: [
    { id: 1, name: 'Alice', isSelected: true },
    { id: 2, name: 'Bob', isSelected: false }
  ],
  isLoading: false,
  error: null
};
```

### 2. 不可变更新
```javascript
// ✅ 正确的不可变更新
const updateUser = (state, action) => {
  return {
    ...state,
    users: {
      ...state.users,
      byId: {
        ...state.users.byId,
        [action.payload.id]: {
          ...state.users.byId[action.payload.id],
          ...action.payload.updates
        }
      }
    }
  };
};

// ❌ 错误的可变更新
const badUpdateUser = (state, action) => {
  state.users.byId[action.payload.id].name = action.payload.name; // 直接修改
  return state;
};
```

### 3. 选择器模式
```javascript
// 创建记忆化选择器
import { createSelector } from '@reduxjs/toolkit';

const selectUsers = state => state.entities.users;
const selectCurrentUserId = state => state.ui.currentUserId;

const selectCurrentUser = createSelector(
  [selectUsers, selectCurrentUserId],
  (users, currentUserId) => users.byId[currentUserId]
);

const selectUsersByStatus = createSelector(
  [selectUsers, (state, status) => status],
  (users, status) => users.allIds
    .map(id => users.byId[id])
    .filter(user => user.status === status)
);
```

### 4. 错误边界处理
```javascript
// 全局错误处理
const errorSlice = createSlice({
  name: 'error',
  initialState: {
    global: null,
    fields: {}
  },
  reducers: {
    setGlobalError: (state, action) => {
      state.global = action.payload;
    },
    clearGlobalError: (state) => {
      state.global = null;
    },
    setFieldError: (state, action) => {
      state.fields[action.payload.field] = action.payload.error;
    },
    clearFieldError: (state, action) => {
      delete state.fields[action.payload];
    },
    clearAllErrors: (state) => {
      state.global = null;
      state.fields = {};
    }
  }
});

// 错误处理中间件
const errorHandlingMiddleware = (store) => (next) => (action) => {
  if (action.type.endsWith('/rejected')) {
    store.dispatch(setGlobalError(action.payload));
  }
  return next(action);
};
```

## 8.7 总结

### 本章重点
1. **状态管理概念**：理解为什么需要状态管理
2. **Context模式**：适合中小型应用的状态共享
3. **Redux基础**：大型应用的状态管理解决方案
4. **最佳实践**：状态设计、不可变更新、性能优化

### 选择建议
- **本地状态**: 简单组件内部状态
- **Props传递**: 父子组件通信，层级较浅
- **Context API**: 中等复杂度，避免props drilling
- **Redux**: 大型应用，复杂状态逻辑，需要时间旅行调试

### 下章预告
下一章我们将学习React Router，构建单页应用的路由系统。

### 练习作业
1. 使用Context API实现一个多语言切换系统
2. 用Redux Toolkit构建一个博客应用的状态管理
3. 比较不同状态管理方案的性能差异

### 常见问题
1. **Q: 什么时候使用Redux而不是Context？**
   A: 当状态逻辑复杂、需要时间旅行调试、有复杂的异步逻辑时考虑Redux。

2. **Q: 如何避免Context的性能问题？**
   A: 分离Context、使用选择器、合理组织Context层次结构。

3. **Q: Redux的学习曲线如何？**
   A: Redux Toolkit大大简化了Redux的使用，推荐从RTK开始学习。
