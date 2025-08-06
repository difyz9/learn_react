# 第九章：路由管理

## 学习目标
- 理解单页应用(SPA)的路由概念
- 掌握React Router的基本使用
- 学会实现嵌套路由和动态路由
- 了解路由守卫和权限控制
- 能够构建复杂的路由系统

## 9.1 路由基础概念

### 什么是路由？
路由是根据不同的URL路径展示不同内容的机制。在React中，路由允许我们创建单页应用，通过改变URL来切换不同的组件，而无需重新加载页面。

### SPA vs 传统多页应用
```javascript
// 传统多页应用
// /home.html -> 服务器返回home.html
// /about.html -> 服务器返回about.html
// /contact.html -> 服务器返回contact.html

// 单页应用 (SPA)
// / -> React Router渲染Home组件
// /about -> React Router渲染About组件
// /contact -> React Router渲染Contact组件
```

### 安装React Router
```bash
npm install react-router-dom
```

## 9.2 基础路由配置

### 简单路由设置
```javascript
// App.js
import React from 'react';
import {
  BrowserRouter as Router,
  Routes,
  Route,
  Link,
  Navigate
} from 'react-router-dom';

// 页面组件
function Home() {
  return (
    <div>
      <h1>首页</h1>
      <p>欢迎来到我们的网站！</p>
    </div>
  );
}

function About() {
  return (
    <div>
      <h1>关于我们</h1>
      <p>我们是一家专业的技术公司...</p>
    </div>
  );
}

function Contact() {
  return (
    <div>
      <h1>联系我们</h1>
      <p>电话：123-456-7890</p>
      <p>邮箱：contact@example.com</p>
    </div>
  );
}

function NotFound() {
  return (
    <div>
      <h1>404 - 页面未找到</h1>
      <p>抱歉，您访问的页面不存在。</p>
      <Link to="/">返回首页</Link>
    </div>
  );
}

// 导航组件
function Navigation() {
  return (
    <nav style={{ padding: '20px', borderBottom: '1px solid #ccc' }}>
      <Link to="/" style={{ marginRight: '20px' }}>首页</Link>
      <Link to="/about" style={{ marginRight: '20px' }}>关于我们</Link>
      <Link to="/contact" style={{ marginRight: '20px' }}>联系我们</Link>
    </nav>
  );
}

// 主应用
function App() {
  return (
    <Router>
      <div>
        <Navigation />
        <main style={{ padding: '20px' }}>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/contact" element={<Contact />} />
            <Route path="/404" element={<NotFound />} />
            <Route path="*" element={<Navigate to="/404" replace />} />
          </Routes>
        </main>
      </div>
    </Router>
  );
}

export default App;
```

### 使用NavLink进行导航
```javascript
import { NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav className="navigation">
      <NavLink 
        to="/" 
        className={({ isActive }) => 
          isActive ? 'nav-link active' : 'nav-link'
        }
      >
        首页
      </NavLink>
      
      <NavLink 
        to="/about"
        style={({ isActive }) => ({
          color: isActive ? '#007bff' : '#333',
          fontWeight: isActive ? 'bold' : 'normal'
        })}
      >
        关于我们
      </NavLink>
      
      <NavLink 
        to="/contact"
        className="nav-link"
        end // 精确匹配
      >
        联系我们
      </NavLink>
    </nav>
  );
}
```

## 9.3 动态路由和参数

### URL参数
```javascript
import { useParams, useNavigate } from 'react-router-dom';

// 用户详情页
function UserProfile() {
  const { userId } = useParams();
  const navigate = useNavigate();
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const fetchUser = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) {
          throw new Error('用户不存在');
        }
        const userData = await response.json();
        setUser(userData);
      } catch (error) {
        console.error('获取用户失败:', error);
        navigate('/users', { replace: true });
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId, navigate]);
  
  if (loading) return <div>加载中...</div>;
  if (!user) return <div>用户不存在</div>;
  
  return (
    <div>
      <h1>用户资料</h1>
      <div className="user-profile">
        <img src={user.avatar} alt={user.name} />
        <h2>{user.name}</h2>
        <p>邮箱: {user.email}</p>
        <p>加入时间: {new Date(user.joinDate).toLocaleDateString()}</p>
        
        <button onClick={() => navigate('/users')}>
          返回用户列表
        </button>
        <button onClick={() => navigate(`/users/${userId}/edit`)}>
          编辑资料
        </button>
      </div>
    </div>
  );
}

// 用户列表页
function UserList() {
  const [users, setUsers] = useState([]);
  const navigate = useNavigate();
  
  useEffect(() => {
    fetch('/api/users')
      .then(response => response.json())
      .then(setUsers);
  }, []);
  
  return (
    <div>
      <h1>用户列表</h1>
      <div className="user-grid">
        {users.map(user => (
          <div key={user.id} className="user-card">
            <img src={user.avatar} alt={user.name} />
            <h3>{user.name}</h3>
            <button 
              onClick={() => navigate(`/users/${user.id}`)}
            >
              查看详情
            </button>
          </div>
        ))}
      </div>
    </div>
  );
}

// 路由配置
function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/users" element={<UserList />} />
        <Route path="/users/:userId" element={<UserProfile />} />
        <Route path="/users/:userId/edit" element={<EditUser />} />
      </Routes>
    </Router>
  );
}
```

### 查询参数
```javascript
import { useSearchParams } from 'react-router-dom';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const [products, setProducts] = useState([]);
  
  // 获取查询参数
  const category = searchParams.get('category') || 'all';
  const page = parseInt(searchParams.get('page')) || 1;
  const sortBy = searchParams.get('sort') || 'name';
  const search = searchParams.get('search') || '';
  
  useEffect(() => {
    const fetchProducts = async () => {
      const params = new URLSearchParams({
        category,
        page: page.toString(),
        sort: sortBy,
        ...(search && { search })
      });
      
      const response = await fetch(`/api/products?${params}`);
      const data = await response.json();
      setProducts(data.products);
    };
    
    fetchProducts();
  }, [category, page, sortBy, search]);
  
  // 更新查询参数
  const updateSearchParams = (updates) => {
    const newParams = new URLSearchParams(searchParams);
    
    Object.entries(updates).forEach(([key, value]) => {
      if (value) {
        newParams.set(key, value);
      } else {
        newParams.delete(key);
      }
    });
    
    setSearchParams(newParams);
  };
  
  return (
    <div>
      <div className="filters">
        <select 
          value={category}
          onChange={(e) => updateSearchParams({ category: e.target.value, page: 1 })}
        >
          <option value="all">所有分类</option>
          <option value="electronics">电子产品</option>
          <option value="clothing">服装</option>
          <option value="books">图书</option>
        </select>
        
        <select
          value={sortBy}
          onChange={(e) => updateSearchParams({ sort: e.target.value })}
        >
          <option value="name">按名称排序</option>
          <option value="price">按价格排序</option>
          <option value="rating">按评分排序</option>
        </select>
        
        <input
          type="text"
          placeholder="搜索产品..."
          value={search}
          onChange={(e) => updateSearchParams({ search: e.target.value, page: 1 })}
        />
      </div>
      
      <div className="product-grid">
        {products.map(product => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
      
      <div className="pagination">
        <button 
          disabled={page <= 1}
          onClick={() => updateSearchParams({ page: page - 1 })}
        >
          上一页
        </button>
        <span>第 {page} 页</span>
        <button 
          onClick={() => updateSearchParams({ page: page + 1 })}
        >
          下一页
        </button>
      </div>
    </div>
  );
}
```

## 9.4 嵌套路由

### 基础嵌套路由
```javascript
import { Outlet, useLocation } from 'react-router-dom';

// 布局组件
function DashboardLayout() {
  const location = useLocation();
  
  return (
    <div className="dashboard-layout">
      <aside className="sidebar">
        <nav>
          <NavLink to="/dashboard" end>概览</NavLink>
          <NavLink to="/dashboard/users">用户管理</NavLink>
          <NavLink to="/dashboard/products">产品管理</NavLink>
          <NavLink to="/dashboard/orders">订单管理</NavLink>
          <NavLink to="/dashboard/analytics">数据分析</NavLink>
          <NavLink to="/dashboard/settings">系统设置</NavLink>
        </nav>
      </aside>
      
      <main className="main-content">
        <header className="content-header">
          <h1>管理后台</h1>
          <breadcrumb currentPath={location.pathname} />
        </header>
        
        <div className="content-body">
          {/* 嵌套路由内容会在这里渲染 */}
          <Outlet />
        </div>
      </main>
    </div>
  );
}

// 面包屑导航
function Breadcrumb({ currentPath }) {
  const pathSegments = currentPath.split('/').filter(Boolean);
  
  const breadcrumbItems = pathSegments.map((segment, index) => {
    const path = '/' + pathSegments.slice(0, index + 1).join('/');
    const isLast = index === pathSegments.length - 1;
    
    return {
      path,
      label: segment,
      isLast
    };
  });
  
  return (
    <nav className="breadcrumb">
      {breadcrumbItems.map((item, index) => (
        <span key={item.path}>
          {index > 0 && <span className="separator"> > </span>}
          {item.isLast ? (
            <span className="current">{item.label}</span>
          ) : (
            <Link to={item.path}>{item.label}</Link>
          )}
        </span>
      ))}
    </nav>
  );
}

// 概览页面
function DashboardOverview() {
  return (
    <div>
      <h2>系统概览</h2>
      <div className="stats-grid">
        <StatCard title="总用户数" value="1,234" />
        <StatCard title="总产品数" value="567" />
        <StatCard title="待处理订单" value="89" />
        <StatCard title="本月销售额" value="$12,345" />
      </div>
    </div>
  );
}

// 用户管理页面布局
function UserManagement() {
  return (
    <div>
      <div className="page-header">
        <h2>用户管理</h2>
        <Link to="/dashboard/users/new" className="btn btn-primary">
          添加用户
        </Link>
      </div>
      
      <div className="user-management">
        <div className="user-tabs">
          <NavLink to="/dashboard/users" end>用户列表</NavLink>
          <NavLink to="/dashboard/users/roles">角色管理</NavLink>
          <NavLink to="/dashboard/users/permissions">权限管理</NavLink>
        </div>
        
        <div className="tab-content">
          <Outlet />
        </div>
      </div>
    </div>
  );
}

// 用户列表
function UserList() {
  const [users, setUsers] = useState([]);
  
  return (
    <div>
      <div className="search-bar">
        <input type="text" placeholder="搜索用户..." />
        <button>搜索</button>
      </div>
      
      <table className="user-table">
        <thead>
          <tr>
            <th>用户名</th>
            <th>邮箱</th>
            <th>角色</th>
            <th>状态</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.id}>
              <td>{user.name}</td>
              <td>{user.email}</td>
              <td>{user.role}</td>
              <td>{user.status}</td>
              <td>
                <Link to={`/dashboard/users/${user.id}`}>编辑</Link>
                <button>删除</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

// 路由配置
function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />
        
        {/* 嵌套路由 */}
        <Route path="/dashboard" element={<DashboardLayout />}>
          <Route index element={<DashboardOverview />} />
          
          <Route path="users" element={<UserManagement />}>
            <Route index element={<UserList />} />
            <Route path="roles" element={<RoleManagement />} />
            <Route path="permissions" element={<PermissionManagement />} />
            <Route path="new" element={<CreateUser />} />
            <Route path=":userId" element={<EditUser />} />
          </Route>
          
          <Route path="products" element={<ProductManagement />}>
            <Route index element={<ProductList />} />
            <Route path="categories" element={<CategoryManagement />} />
            <Route path="new" element={<CreateProduct />} />
            <Route path=":productId" element={<EditProduct />} />
          </Route>
          
          <Route path="orders" element={<OrderManagement />} />
          <Route path="analytics" element={<Analytics />} />
          <Route path="settings" element={<Settings />} />
        </Route>
        
        <Route path="*" element={<NotFound />} />
      </Routes>
    </Router>
  );
}
```

## 9.5 路由守卫和权限控制

### 身份验证守卫
```javascript
import { useContext, createContext } from 'react';

// 认证Context
const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // 检查用户是否已登录
    const token = localStorage.getItem('token');
    if (token) {
      fetch('/api/me', {
        headers: { Authorization: `Bearer ${token}` }
      })
      .then(response => {
        if (response.ok) {
          return response.json();
        }
        throw new Error('Token无效');
      })
      .then(setUser)
      .catch(() => {
        localStorage.removeItem('token');
      })
      .finally(() => setLoading(false));
    } else {
      setLoading(false);
    }
  }, []);
  
  const login = async (credentials) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    });
    
    if (response.ok) {
      const { user, token } = await response.json();
      localStorage.setItem('token', token);
      setUser(user);
      return user;
    }
    
    throw new Error('登录失败');
  };
  
  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => useContext(AuthContext);

// 受保护的路由组件
function ProtectedRoute({ children, requiredRole = null }) {
  const { user, loading } = useAuth();
  const location = useLocation();
  
  if (loading) {
    return <div className="loading">验证用户身份中...</div>;
  }
  
  if (!user) {
    // 重定向到登录页，并保存当前位置
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  if (requiredRole && !user.roles.includes(requiredRole)) {
    return <Navigate to="/unauthorized" replace />;
  }
  
  return children;
}

// 角色守卫组件
function RoleGuard({ children, roles, fallback = null }) {
  const { user } = useAuth();
  
  if (!user) return null;
  
  const hasRequiredRole = roles.some(role => user.roles.includes(role));
  
  if (!hasRequiredRole) {
    return fallback || <div>您没有权限访问此内容</div>;
  }
  
  return children;
}

// 登录组件
function Login() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  
  const [credentials, setCredentials] = useState({ username: '', password: '' });
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  
  const from = location.state?.from?.pathname || '/dashboard';
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError('');
    
    try {
      await login(credentials);
      navigate(from, { replace: true });
    } catch (error) {
      setError(error.message);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="login-page">
      <form onSubmit={handleSubmit}>
        <h2>登录</h2>
        
        {error && <div className="error">{error}</div>}
        
        <input
          type="text"
          placeholder="用户名"
          value={credentials.username}
          onChange={(e) => setCredentials(prev => ({
            ...prev,
            username: e.target.value
          }))}
          required
        />
        
        <input
          type="password"
          placeholder="密码"
          value={credentials.password}
          onChange={(e) => setCredentials(prev => ({
            ...prev,
            password: e.target.value
          }))}
          required
        />
        
        <button type="submit" disabled={loading}>
          {loading ? '登录中...' : '登录'}
        </button>
      </form>
    </div>
  );
}

// 更新路由配置
function App() {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/login" element={<Login />} />
          <Route path="/unauthorized" element={<Unauthorized />} />
          
          {/* 受保护的路由 */}
          <Route 
            path="/dashboard/*" 
            element={
              <ProtectedRoute>
                <DashboardRoutes />
              </ProtectedRoute>
            } 
          />
          
          {/* 管理员路由 */}
          <Route 
            path="/admin/*" 
            element={
              <ProtectedRoute requiredRole="admin">
                <AdminRoutes />
              </ProtectedRoute>
            } 
          />
        </Routes>
      </Router>
    </AuthProvider>
  );
}
```

### 权限控制Hook
```javascript
// 权限控制Hook
function usePermissions() {
  const { user } = useAuth();
  
  const hasPermission = useCallback((permission) => {
    if (!user) return false;
    return user.permissions.includes(permission) || user.roles.includes('admin');
  }, [user]);
  
  const hasRole = useCallback((role) => {
    if (!user) return false;
    return user.roles.includes(role);
  }, [user]);
  
  const hasAnyRole = useCallback((roles) => {
    if (!user) return false;
    return roles.some(role => user.roles.includes(role));
  }, [user]);
  
  const can = useCallback((action, resource) => {
    if (!user) return false;
    
    // 基于角色的权限检查
    const permission = `${action}:${resource}`;
    return hasPermission(permission);
  }, [user, hasPermission]);
  
  return {
    hasPermission,
    hasRole,
    hasAnyRole,
    can,
    user
  };
}

// 条件渲染组件
function CanAccess({ permission, role, roles, children, fallback = null }) {
  const { hasPermission, hasRole, hasAnyRole } = usePermissions();
  
  let hasAccess = true;
  
  if (permission && !hasPermission(permission)) {
    hasAccess = false;
  }
  
  if (role && !hasRole(role)) {
    hasAccess = false;
  }
  
  if (roles && !hasAnyRole(roles)) {
    hasAccess = false;
  }
  
  return hasAccess ? children : fallback;
}

// 使用示例
function UserManagement() {
  const { can } = usePermissions();
  
  return (
    <div>
      <h2>用户管理</h2>
      
      <CanAccess permission="create:user">
        <button>添加用户</button>
      </CanAccess>
      
      <table>
        <thead>
          <tr>
            <th>用户名</th>
            <th>邮箱</th>
            <CanAccess role="admin">
              <th>操作</th>
            </CanAccess>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.id}>
              <td>{user.name}</td>
              <td>{user.email}</td>
              <CanAccess role="admin">
                <td>
                  {can('update', 'user') && (
                    <button>编辑</button>
                  )}
                  {can('delete', 'user') && (
                    <button>删除</button>
                  )}
                </td>
              </CanAccess>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

## 9.6 路由性能优化

### 代码分割和懒加载
```javascript
import { lazy, Suspense } from 'react';

// 懒加载组件
const Dashboard = lazy(() => import('./pages/Dashboard'));
const UserManagement = lazy(() => import('./pages/UserManagement'));
const ProductManagement = lazy(() => import('./pages/ProductManagement'));
const Analytics = lazy(() => 
  import('./pages/Analytics').then(module => ({
    default: module.Analytics
  }))
);

// 带错误边界的懒加载组件
const LazyComponentWithErrorBoundary = ({ component: Component, ...props }) => (
  <ErrorBoundary>
    <Suspense fallback={<PageLoader />}>
      <Component {...props} />
    </Suspense>
  </ErrorBoundary>
);

// 页面加载器
function PageLoader() {
  return (
    <div className="page-loader">
      <div className="spinner"></div>
      <p>页面加载中...</p>
    </div>
  );
}

// 错误边界
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('页面加载错误:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-page">
          <h2>页面加载失败</h2>
          <p>请刷新页面重试</p>
          <button onClick={() => window.location.reload()}>
            刷新页面
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// 路由配置
function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<Home />} />
        
        <Route 
          path="/dashboard" 
          element={
            <LazyComponentWithErrorBoundary component={Dashboard} />
          } 
        />
        
        <Route 
          path="/users" 
          element={
            <LazyComponentWithErrorBoundary component={UserManagement} />
          } 
        />
        
        <Route 
          path="/products" 
          element={
            <LazyComponentWithErrorBoundary component={ProductManagement} />
          } 
        />
        
        <Route 
          path="/analytics" 
          element={
            <LazyComponentWithErrorBoundary component={Analytics} />
          } 
        />
      </Routes>
    </Router>
  );
}
```

### 路由预加载
```javascript
// 路由预加载Hook
function useRoutePreloader() {
  const preloadedRoutes = useRef(new Set());
  
  const preloadRoute = useCallback((routePath) => {
    if (preloadedRoutes.current.has(routePath)) {
      return; // 已经预加载过
    }
    
    preloadedRoutes.current.add(routePath);
    
    // 根据路由路径预加载对应的组件
    switch (routePath) {
      case '/dashboard':
        import('./pages/Dashboard');
        break;
      case '/users':
        import('./pages/UserManagement');
        break;
      case '/products':
        import('./pages/ProductManagement');
        break;
      case '/analytics':
        import('./pages/Analytics');
        break;
      default:
        break;
    }
  }, []);
  
  return { preloadRoute };
}

// 智能导航组件
function SmartNavigation() {
  const { preloadRoute } = useRoutePreloader();
  
  const handleMouseEnter = (routePath) => {
    // 鼠标悬停时预加载路由
    preloadRoute(routePath);
  };
  
  return (
    <nav>
      <NavLink 
        to="/dashboard"
        onMouseEnter={() => handleMouseEnter('/dashboard')}
      >
        仪表板
      </NavLink>
      
      <NavLink 
        to="/users"
        onMouseEnter={() => handleMouseEnter('/users')}
      >
        用户管理
      </NavLink>
      
      <NavLink 
        to="/products"
        onMouseEnter={() => handleMouseEnter('/products')}
      >
        产品管理
      </NavLink>
    </nav>
  );
}
```

## 9.7 实践案例：博客系统路由

让我们构建一个完整的博客系统路由结构。

```javascript
// 路由配置
function BlogApp() {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          {/* 公共路由 */}
          <Route path="/" element={<BlogLayout />}>
            <Route index element={<HomePage />} />
            <Route path="posts" element={<PostList />} />
            <Route path="posts/:slug" element={<PostDetail />} />
            <Route path="categories" element={<CategoryList />} />
            <Route path="categories/:categorySlug" element={<CategoryPosts />} />
            <Route path="tags/:tagSlug" element={<TagPosts />} />
            <Route path="authors/:authorId" element={<AuthorProfile />} />
            <Route path="search" element={<SearchResults />} />
            <Route path="about" element={<AboutPage />} />
            <Route path="contact" element={<ContactPage />} />
          </Route>
          
          {/* 认证路由 */}
          <Route path="/auth" element={<AuthLayout />}>
            <Route path="login" element={<Login />} />
            <Route path="register" element={<Register />} />
            <Route path="forgot-password" element={<ForgotPassword />} />
            <Route path="reset-password/:token" element={<ResetPassword />} />
          </Route>
          
          {/* 用户仪表板 */}
          <Route 
            path="/dashboard" 
            element={
              <ProtectedRoute>
                <UserDashboardLayout />
              </ProtectedRoute>
            }
          >
            <Route index element={<DashboardHome />} />
            <Route path="profile" element={<UserProfile />} />
            <Route path="posts" element={<UserPosts />} />
            <Route path="posts/new" element={<CreatePost />} />
            <Route path="posts/:postId/edit" element={<EditPost />} />
            <Route path="drafts" element={<UserDrafts />} />
            <Route path="comments" element={<UserComments />} />
            <Route path="settings" element={<UserSettings />} />
          </Route>
          
          {/* 管理员面板 */}
          <Route 
            path="/admin" 
            element={
              <ProtectedRoute requiredRole="admin">
                <AdminLayout />
              </ProtectedRoute>
            }
          >
            <Route index element={<AdminDashboard />} />
            <Route path="posts" element={<AdminPostManagement />} />
            <Route path="users" element={<AdminUserManagement />} />
            <Route path="categories" element={<AdminCategoryManagement />} />
            <Route path="comments" element={<AdminCommentManagement />} />
            <Route path="analytics" element={<AdminAnalytics />} />
            <Route path="settings" element={<AdminSettings />} />
          </Route>
          
          {/* 错误页面 */}
          <Route path="/404" element={<NotFound />} />
          <Route path="/500" element={<ServerError />} />
          <Route path="*" element={<Navigate to="/404" replace />} />
        </Routes>
      </Router>
    </AuthProvider>
  );
}

// 博客布局组件
function BlogLayout() {
  return (
    <div className="blog-layout">
      <Header />
      <main className="main-content">
        <Outlet />
      </main>
      <Footer />
    </div>
  );
}

// 博客头部
function Header() {
  const location = useLocation();
  const { user, logout } = useAuth();
  
  return (
    <header className="blog-header">
      <div className="container">
        <Link to="/" className="logo">
          <h1>我的博客</h1>
        </Link>
        
        <nav className="main-nav">
          <NavLink to="/">首页</NavLink>
          <NavLink to="/posts">文章</NavLink>
          <NavLink to="/categories">分类</NavLink>
          <NavLink to="/about">关于</NavLink>
          <NavLink to="/contact">联系</NavLink>
        </nav>
        
        <div className="header-actions">
          <SearchBox />
          
          {user ? (
            <div className="user-menu">
              <button className="user-avatar">
                <img src={user.avatar} alt={user.name} />
              </button>
              <div className="dropdown-menu">
                <Link to="/dashboard">我的仪表板</Link>
                <Link to="/dashboard/posts/new">写文章</Link>
                {user.roles.includes('admin') && (
                  <Link to="/admin">管理面板</Link>
                )}
                <button onClick={logout}>登出</button>
              </div>
            </div>
          ) : (
            <div className="auth-buttons">
              <Link to="/auth/login" className="btn btn-outline">
                登录
              </Link>
              <Link to="/auth/register" className="btn btn-primary">
                注册
              </Link>
            </div>
          )}
        </div>
      </div>
    </header>
  );
}

// 文章详情页
function PostDetail() {
  const { slug } = useParams();
  const navigate = useNavigate();
  const [post, setPost] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchPost = async () => {
      try {
        const response = await fetch(`/api/posts/${slug}`);
        
        if (!response.ok) {
          if (response.status === 404) {
            navigate('/404', { replace: true });
            return;
          }
          throw new Error('获取文章失败');
        }
        
        const postData = await response.json();
        setPost(postData);
        
        // 更新页面标题
        document.title = `${postData.title} - 我的博客`;
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchPost();
  }, [slug, navigate]);
  
  if (loading) return <PostSkeleton />;
  if (error) return <ErrorMessage message={error} />;
  if (!post) return <NotFound />;
  
  return (
    <article className="post-detail">
      <header className="post-header">
        <nav className="breadcrumb">
          <Link to="/">首页</Link>
          <span>/</span>
          <Link to="/posts">文章</Link>
          <span>/</span>
          <span>{post.title}</span>
        </nav>
        
        <h1>{post.title}</h1>
        
        <div className="post-meta">
          <Link to={`/authors/${post.author.id}`} className="author">
            <img src={post.author.avatar} alt={post.author.name} />
            {post.author.name}
          </Link>
          
          <time>{new Date(post.publishedAt).toLocaleDateString()}</time>
          
          <div className="post-categories">
            {post.categories.map(category => (
              <Link 
                key={category.id}
                to={`/categories/${category.slug}`}
                className="category-tag"
              >
                {category.name}
              </Link>
            ))}
          </div>
          
          <div className="post-tags">
            {post.tags.map(tag => (
              <Link 
                key={tag.id}
                to={`/tags/${tag.slug}`}
                className="tag"
              >
                #{tag.name}
              </Link>
            ))}
          </div>
        </div>
      </header>
      
      <div className="post-content">
        <div dangerouslySetInnerHTML={{ __html: post.content }} />
      </div>
      
      <footer className="post-footer">
        <div className="post-actions">
          <LikeButton postId={post.id} />
          <ShareButtons url={window.location.href} title={post.title} />
        </div>
        
        <RelatedPosts postId={post.id} />
        <CommentSection postId={post.id} />
      </footer>
    </article>
  );
}

// 搜索组件
function SearchBox() {
  const [query, setQuery] = useState('');
  const navigate = useNavigate();
  const [searchParams] = useSearchParams();
  
  useEffect(() => {
    const q = searchParams.get('q');
    if (q) setQuery(q);
  }, [searchParams]);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (query.trim()) {
      navigate(`/search?q=${encodeURIComponent(query.trim())}`);
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="search-box">
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="搜索文章..."
      />
      <button type="submit">
        <SearchIcon />
      </button>
    </form>
  );
}
```

## 9.8 总结

### 本章重点
1. **路由基础**：理解SPA路由的工作原理
2. **React Router**：掌握现代路由库的使用
3. **动态路由**：URL参数和查询参数的处理
4. **嵌套路由**：构建复杂的路由层次结构
5. **路由守卫**：实现身份验证和权限控制
6. **性能优化**：代码分割和懒加载

### 最佳实践
- 合理组织路由结构，保持层次清晰
- 使用路由守卫保护敏感页面
- 实现代码分割减少初始加载时间
- 提供友好的加载和错误状态
- 正确处理浏览器历史记录

### 下章预告
下一章我们将学习表单处理，包括受控组件、表单验证、第三方表单库等内容。

### 练习作业
1. 构建一个电商网站的完整路由系统
2. 实现基于角色的权限控制系统
3. 添加路由过渡动画和页面加载优化

### 常见问题
1. **Q: 什么时候使用嵌套路由？**
   A: 当页面有公共布局或需要组织复杂的路由层次时使用嵌套路由。

2. **Q: 如何处理路由参数验证？**
   A: 可以在组件中使用useEffect验证参数，无效时重定向到错误页面。

3. **Q: 路由懒加载的最佳实践？**
   A: 按页面级别进行代码分割，配合Suspense和错误边界使用。
