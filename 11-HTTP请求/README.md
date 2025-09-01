# 第十一章：HTTP请求

## 学习目标
- 掌握在React中发送HTTP请求的各种方法
- 学会使用Fetch API和Axios进行网络请求
- 理解异步数据获取和状态管理
- 掌握错误处理和加载状态管理
- 学会实现请求缓存和优化策略

## 11.1 Fetch API基础

### 基本Fetch使用
```javascript
// 基础GET请求
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchUser() {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(`/api/users/${userId}`);
        
        // 检查响应状态
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    if (userId) {
      fetchUser();
    }
  }, [userId]);

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误: {error}</div>;
  if (!user) return <div>用户不存在</div>;

  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>邮箱: {user.email}</p>
      <p>注册时间: {new Date(user.createdAt).toLocaleDateString()}</p>
    </div>
  );
}

// POST请求示例
function CreateUser() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    password: ''
  });
  const [submitting, setSubmitting] = useState(false);
  const [message, setMessage] = useState('');

  const handleSubmit = async (e) => {
    e.preventDefault();
    setSubmitting(true);
    setMessage('');

    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          // 如果需要认证
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        },
        body: JSON.stringify(formData)
      });

      const result = await response.json();

      if (!response.ok) {
        throw new Error(result.message || '创建失败');
      }

      setMessage('用户创建成功！');
      setFormData({ name: '', email: '', password: '' });
    } catch (err) {
      setMessage(`错误: ${err.message}`);
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>姓名:</label>
        <input
          type="text"
          value={formData.name}
          onChange={(e) => setFormData({...formData, name: e.target.value})}
          required
        />
      </div>
      
      <div>
        <label>邮箱:</label>
        <input
          type="email"
          value={formData.email}
          onChange={(e) => setFormData({...formData, email: e.target.value})}
          required
        />
      </div>
      
      <div>
        <label>密码:</label>
        <input
          type="password"
          value={formData.password}
          onChange={(e) => setFormData({...formData, password: e.target.value})}
          required
        />
      </div>

      <button type="submit" disabled={submitting}>
        {submitting ? '提交中...' : '创建用户'}
      </button>

      {message && <p>{message}</p>}
    </form>
  );
}

// 文件上传示例
function FileUpload() {
  const [file, setFile] = useState(null);
  const [uploading, setUploading] = useState(false);
  const [progress, setProgress] = useState(0);
  const [uploadedUrl, setUploadedUrl] = useState('');

  const handleFileChange = (e) => {
    const selectedFile = e.target.files[0];
    setFile(selectedFile);
    setUploadedUrl('');
  };

  const handleUpload = async () => {
    if (!file) return;

    setUploading(true);
    setProgress(0);

    const formData = new FormData();
    formData.append('file', file);

    try {
      // 使用XMLHttpRequest来监控上传进度
      const xhr = new XMLHttpRequest();

      xhr.upload.onprogress = (event) => {
        if (event.lengthComputable) {
          const percentComplete = (event.loaded / event.total) * 100;
          setProgress(Math.round(percentComplete));
        }
      };

      xhr.onload = () => {
        if (xhr.status === 200) {
          const response = JSON.parse(xhr.responseText);
          setUploadedUrl(response.url);
        } else {
          throw new Error('上传失败');
        }
        setUploading(false);
      };

      xhr.onerror = () => {
        setUploading(false);
        alert('上传失败');
      };

      xhr.open('POST', '/api/upload');
      xhr.setRequestHeader('Authorization', `Bearer ${localStorage.getItem('token')}`);
      xhr.send(formData);

    } catch (err) {
      setUploading(false);
      alert(`上传失败: ${err.message}`);
    }
  };

  return (
    <div>
      <input type="file" onChange={handleFileChange} />
      
      <button 
        onClick={handleUpload} 
        disabled={!file || uploading}
      >
        {uploading ? '上传中...' : '上传文件'}
      </button>

      {uploading && (
        <div>
          <div>上传进度: {progress}%</div>
          <div 
            style={{
              width: '100%',
              height: '10px',
              backgroundColor: '#f0f0f0',
              borderRadius: '5px'
            }}
          >
            <div
              style={{
                width: `${progress}%`,
                height: '100%',
                backgroundColor: '#4caf50',
                borderRadius: '5px',
                transition: 'width 0.3s ease'
              }}
            />
          </div>
        </div>
      )}

      {uploadedUrl && (
        <div>
          <p>上传成功！</p>
          <img src={uploadedUrl} alt="上传的文件" style={{maxWidth: '200px'}} />
        </div>
      )}
    </div>
  );
}
```

### 自定义Fetch Hook
```javascript
// 通用的fetch hook
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // 使用useCallback避免不必要的重新请求
  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);

      const response = await fetch(url, {
        ...options,
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        }
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [url, JSON.stringify(options)]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return {
    data,
    loading,
    error,
    refetch: fetchData
  };
}

// 带缓存的fetch hook
function useFetchWithCache(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  // 简单的内存缓存
  const cacheRef = useRef(new Map());
  const cacheKey = `${url}-${JSON.stringify(options)}`;

  useEffect(() => {
    async function fetchData() {
      // 检查缓存
      if (cacheRef.current.has(cacheKey)) {
        const cachedData = cacheRef.current.get(cacheKey);
        setData(cachedData);
        setLoading(false);
        return;
      }

      try {
        setLoading(true);
        setError(null);

        const response = await fetch(url, options);
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }

        const result = await response.json();
        
        // 存入缓存
        cacheRef.current.set(cacheKey, result);
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchData();
  }, [url, cacheKey]);

  const clearCache = useCallback(() => {
    cacheRef.current.clear();
  }, []);

  return { data, loading, error, clearCache };
}

// 使用示例
function UserList() {
  const { data: users, loading, error, refetch } = useFetch('/api/users');

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误: {error}</div>;

  return (
    <div>
      <button onClick={refetch}>刷新</button>
      <ul>
        {users?.map(user => (
          <li key={user.id}>{user.name} - {user.email}</li>
        ))}
      </ul>
    </div>
  );
}

// 分页数据获取
function usePaginatedFetch(baseUrl, initialPage = 1, pageSize = 10) {
  const [data, setData] = useState([]);
  const [currentPage, setCurrentPage] = useState(initialPage);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [hasMore, setHasMore] = useState(true);
  const [totalCount, setTotalCount] = useState(0);

  const fetchPage = useCallback(async (page) => {
    try {
      setLoading(true);
      setError(null);

      const url = `${baseUrl}?page=${page}&limit=${pageSize}`;
      const response = await fetch(url);

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      const result = await response.json();

      if (page === 1) {
        setData(result.data);
      } else {
        setData(prev => [...prev, ...result.data]);
      }

      setHasMore(result.data.length === pageSize);
      setTotalCount(result.total);
      setCurrentPage(page);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [baseUrl, pageSize]);

  useEffect(() => {
    fetchPage(1);
  }, [fetchPage]);

  const loadMore = useCallback(() => {
    if (!loading && hasMore) {
      fetchPage(currentPage + 1);
    }
  }, [loading, hasMore, currentPage, fetchPage]);

  const refresh = useCallback(() => {
    setData([]);
    setCurrentPage(1);
    fetchPage(1);
  }, [fetchPage]);

  return {
    data,
    loading,
    error,
    hasMore,
    totalCount,
    loadMore,
    refresh
  };
}

// 无限滚动列表组件
function InfiniteScrollList() {
  const { 
    data: posts, 
    loading, 
    error, 
    hasMore, 
    loadMore 
  } = usePaginatedFetch('/api/posts');

  const observerRef = useRef();
  const lastPostElementRef = useCallback(node => {
    if (loading) return;
    if (observerRef.current) observerRef.current.disconnect();
    
    observerRef.current = new IntersectionObserver(entries => {
      if (entries[0].isIntersecting && hasMore) {
        loadMore();
      }
    });
    
    if (node) observerRef.current.observe(node);
  }, [loading, hasMore, loadMore]);

  return (
    <div>
      {posts.map((post, index) => {
        const isLast = posts.length === index + 1;
        return (
          <div 
            key={post.id}
            ref={isLast ? lastPostElementRef : null}
            className="post-item"
          >
            <h3>{post.title}</h3>
            <p>{post.content}</p>
          </div>
        );
      })}
      
      {loading && <div>加载中...</div>}
      {error && <div>错误: {error}</div>}
      {!hasMore && <div>没有更多内容了</div>}
    </div>
  );
}
```

## 11.2 Axios库使用

### Axios基础配置
```javascript
// 安装: npm install axios
import axios from 'axios';

// 创建axios实例
const api = axios.create({
  baseURL: process.env.REACT_APP_API_BASE_URL || 'http://localhost:3001/api',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
});

// 请求拦截器
api.interceptors.request.use(
  (config) => {
    // 在发送请求之前做些什么
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    // 添加请求时间戳
    config.metadata = { startTime: new Date() };
    
    console.log('发送请求:', config.method?.toUpperCase(), config.url);
    return config;
  },
  (error) => {
    // 对请求错误做些什么
    console.error('请求错误:', error);
    return Promise.reject(error);
  }
);

// 响应拦截器
api.interceptors.response.use(
  (response) => {
    // 对响应数据做点什么
    const duration = new Date() - response.config.metadata.startTime;
    console.log(`请求完成: ${response.config.url} (${duration}ms)`);
    
    return response;
  },
  (error) => {
    // 对响应错误做点什么
    console.error('响应错误:', error);
    
    if (error.response) {
      // 服务器响应了错误状态码
      const { status, data } = error.response;
      
      switch (status) {
        case 401:
          // 未授权，清除token并跳转到登录页
          localStorage.removeItem('authToken');
          window.location.href = '/login';
          break;
        case 403:
          // 禁止访问
          alert('没有权限访问该资源');
          break;
        case 404:
          // 资源不存在
          console.error('请求的资源不存在');
          break;
        case 500:
          // 服务器错误
          alert('服务器内部错误，请稍后重试');
          break;
        default:
          console.error(`HTTP ${status}: ${data?.message || '未知错误'}`);
      }
    } else if (error.request) {
      // 请求已发出但没有收到响应
      console.error('网络错误，请检查网络连接');
      alert('网络连接失败，请检查网络设置');
    } else {
      // 其他错误
      console.error('请求配置错误:', error.message);
    }
    
    return Promise.reject(error);
  }
);

export default api;
```

### API服务层封装
```javascript
// services/userService.js
import api from './apiConfig';

class UserService {
  // 获取用户列表
  async getUsers(params = {}) {
    try {
      const response = await api.get('/users', { params });
      return response.data;
    } catch (error) {
      throw this.handleError(error);
    }
  }

  // 获取单个用户
  async getUserById(id) {
    try {
      const response = await api.get(`/users/${id}`);
      return response.data;
    } catch (error) {
      throw this.handleError(error);
    }
  }

  // 创建用户
  async createUser(userData) {
    try {
      const response = await api.post('/users', userData);
      return response.data;
    } catch (error) {
      throw this.handleError(error);
    }
  }

  // 更新用户
  async updateUser(id, userData) {
    try {
      const response = await api.put(`/users/${id}`, userData);
      return response.data;
    } catch (error) {
      throw this.handleError(error);
    }
  }

  // 部分更新用户
  async patchUser(id, userData) {
    try {
      const response = await api.patch(`/users/${id}`, userData);
      return response.data;
    } catch (error) {
      throw this.handleError(error);
    }
  }

  // 删除用户
  async deleteUser(id) {
    try {
      await api.delete(`/users/${id}`);
      return true;
    } catch (error) {
      throw this.handleError(error);
    }
  }

  // 批量操作
  async batchUpdateUsers(updates) {
    try {
      const response = await api.patch('/users/batch', { updates });
      return response.data;
    } catch (error) {
      throw this.handleError(error);
    }
  }

  // 搜索用户
  async searchUsers(query, filters = {}) {
    try {
      const response = await api.get('/users/search', {
        params: { q: query, ...filters }
      });
      return response.data;
    } catch (error) {
      throw this.handleError(error);
    }
  }

  // 上传用户头像
  async uploadAvatar(userId, file) {
    try {
      const formData = new FormData();
      formData.append('avatar', file);

      const response = await api.post(`/users/${userId}/avatar`, formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        },
        // 上传进度回调
        onUploadProgress: (progressEvent) => {
          const percentCompleted = Math.round(
            (progressEvent.loaded * 100) / progressEvent.total
          );
          console.log(`上传进度: ${percentCompleted}%`);
        }
      });

      return response.data;
    } catch (error) {
      throw this.handleError(error);
    }
  }

  // 统一错误处理
  handleError(error) {
    if (error.response) {
      // 服务器返回错误状态
      const { status, data } = error.response;
      return new Error(data.message || `HTTP ${status} Error`);
    } else if (error.request) {
      // 网络错误
      return new Error('网络连接失败');
    } else {
      // 其他错误
      return new Error(error.message || '未知错误');
    }
  }
}

export default new UserService();

// services/postService.js
import api from './apiConfig';

class PostService {
  async getPosts(page = 1, limit = 10, category = '') {
    const params = { page, limit };
    if (category) params.category = category;

    const response = await api.get('/posts', { params });
    return response.data;
  }

  async getPostById(id) {
    const response = await api.get(`/posts/${id}`);
    return response.data;
  }

  async createPost(postData) {
    const response = await api.post('/posts', postData);
    return response.data;
  }

  async updatePost(id, postData) {
    const response = await api.put(`/posts/${id}`, postData);
    return response.data;
  }

  async deletePost(id) {
    await api.delete(`/posts/${id}`);
    return true;
  }

  async likePost(id) {
    const response = await api.post(`/posts/${id}/like`);
    return response.data;
  }

  async addComment(postId, comment) {
    const response = await api.post(`/posts/${postId}/comments`, comment);
    return response.data;
  }
}

export default new PostService();
```

### 自定义Axios Hooks
```javascript
// hooks/useApi.js
import { useState, useEffect, useCallback } from 'react';
import axios from 'axios';

// 通用API hook
function useApi(apiFunction, dependencies = []) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const execute = useCallback(async (...args) => {
    try {
      setLoading(true);
      setError(null);
      const result = await apiFunction(...args);
      setData(result);
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, dependencies);

  useEffect(() => {
    execute();
  }, [execute]);

  return {
    data,
    loading,
    error,
    execute
  };
}

// 用于异步操作的hook
function useAsyncOperation() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const execute = useCallback(async (asyncFunction) => {
    try {
      setLoading(true);
      setError(null);
      const result = await asyncFunction();
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);

  return { loading, error, execute };
}

// 用户相关操作hook
function useUsers() {
  const [users, setUsers] = useState([]);
  const { loading, error, execute } = useAsyncOperation();

  const fetchUsers = useCallback(async (params) => {
    const result = await execute(() => userService.getUsers(params));
    setUsers(result.data);
    return result;
  }, [execute]);

  const createUser = useCallback(async (userData) => {
    const newUser = await execute(() => userService.createUser(userData));
    setUsers(prev => [...prev, newUser]);
    return newUser;
  }, [execute]);

  const updateUser = useCallback(async (id, userData) => {
    const updatedUser = await execute(() => userService.updateUser(id, userData));
    setUsers(prev => prev.map(user => 
      user.id === id ? updatedUser : user
    ));
    return updatedUser;
  }, [execute]);

  const deleteUser = useCallback(async (id) => {
    await execute(() => userService.deleteUser(id));
    setUsers(prev => prev.filter(user => user.id !== id));
  }, [execute]);

  return {
    users,
    loading,
    error,
    fetchUsers,
    createUser,
    updateUser,
    deleteUser
  };
}

// 使用示例
function UserManagement() {
  const {
    users,
    loading,
    error,
    fetchUsers,
    createUser,
    updateUser,
    deleteUser
  } = useUsers();

  useEffect(() => {
    fetchUsers();
  }, [fetchUsers]);

  const handleCreateUser = async (userData) => {
    try {
      await createUser(userData);
      alert('用户创建成功');
    } catch (error) {
      alert(`创建失败: ${error.message}`);
    }
  };

  const handleUpdateUser = async (id, userData) => {
    try {
      await updateUser(id, userData);
      alert('用户更新成功');
    } catch (error) {
      alert(`更新失败: ${error.message}`);
    }
  };

  const handleDeleteUser = async (id) => {
    if (window.confirm('确定要删除这个用户吗？')) {
      try {
        await deleteUser(id);
        alert('用户删除成功');
      } catch (error) {
        alert(`删除失败: ${error.message}`);
      }
    }
  };

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误: {error}</div>;

  return (
    <div>
      <h2>用户管理</h2>
      <button onClick={() => fetchUsers()}>刷新</button>
      
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name} - {user.email}
            <button onClick={() => handleUpdateUser(user.id, { name: '新名字' })}>
              更新
            </button>
            <button onClick={() => handleDeleteUser(user.id)}>
              删除
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## 11.3 请求状态管理

### 复杂状态管理
```javascript
// 使用useReducer管理复杂的请求状态
const initialState = {
  data: null,
  loading: false,
  error: null,
  lastFetch: null,
  retryCount: 0
};

function requestReducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':
      return {
        ...state,
        loading: true,
        error: null
      };
    
    case 'FETCH_SUCCESS':
      return {
        ...state,
        loading: false,
        data: action.payload,
        error: null,
        lastFetch: new Date(),
        retryCount: 0
      };
    
    case 'FETCH_ERROR':
      return {
        ...state,
        loading: false,
        error: action.payload,
        retryCount: state.retryCount + 1
      };
    
    case 'RETRY':
      return {
        ...state,
        error: null,
        retryCount: state.retryCount + 1
      };
    
    case 'RESET':
      return initialState;
    
    default:
      return state;
  }
}

function useAdvancedFetch(url, options = {}) {
  const [state, dispatch] = useReducer(requestReducer, initialState);
  const abortControllerRef = useRef(null);

  const fetchData = useCallback(async () => {
    // 取消之前的请求
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    abortControllerRef.current = new AbortController();

    dispatch({ type: 'FETCH_START' });

    try {
      const response = await fetch(url, {
        ...options,
        signal: abortControllerRef.current.signal
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      const data = await response.json();
      dispatch({ type: 'FETCH_SUCCESS', payload: data });
    } catch (error) {
      if (error.name !== 'AbortError') {
        dispatch({ type: 'FETCH_ERROR', payload: error.message });
      }
    }
  }, [url, JSON.stringify(options)]);

  const retry = useCallback(() => {
    dispatch({ type: 'RETRY' });
    fetchData();
  }, [fetchData]);

  const reset = useCallback(() => {
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
    dispatch({ type: 'RESET' });
  }, []);

  useEffect(() => {
    fetchData();
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [fetchData]);

  return {
    ...state,
    retry,
    reset,
    refetch: fetchData
  };
}

// 请求缓存管理
class RequestCache {
  constructor(maxSize = 100, ttl = 5 * 60 * 1000) { // 5分钟TTL
    this.cache = new Map();
    this.maxSize = maxSize;
    this.ttl = ttl;
  }

  generateKey(url, options) {
    return `${url}:${JSON.stringify(options)}`;
  }

  get(url, options) {
    const key = this.generateKey(url, options);
    const cached = this.cache.get(key);

    if (!cached) return null;

    // 检查是否过期
    if (Date.now() - cached.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }

    return cached.data;
  }

  set(url, options, data) {
    const key = this.generateKey(url, options);

    // 如果缓存已满，删除最老的条目
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }

    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
  }

  clear() {
    this.cache.clear();
  }

  delete(url, options) {
    const key = this.generateKey(url, options);
    this.cache.delete(key);
  }
}

const requestCache = new RequestCache();

// 带缓存的fetch hook
function useCachedFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchData() {
      // 检查缓存
      const cachedData = requestCache.get(url, options);
      if (cachedData) {
        setData(cachedData);
        setLoading(false);
        return;
      }

      try {
        setLoading(true);
        setError(null);

        const response = await fetch(url, options);
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }

        const result = await response.json();
        
        // 存入缓存
        requestCache.set(url, options, result);
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchData();
  }, [url, JSON.stringify(options)]);

  const clearCache = useCallback(() => {
    requestCache.clear();
  }, []);

  const invalidateCache = useCallback(() => {
    requestCache.delete(url, options);
  }, [url, options]);

  return { data, loading, error, clearCache, invalidateCache };
}
```

### 全局状态管理
```javascript
// 使用Context管理全局API状态
const ApiContext = createContext();

function ApiProvider({ children }) {
  const [requests, setRequests] = useState(new Map());

  const addRequest = useCallback((key, request) => {
    setRequests(prev => new Map(prev).set(key, request));
  }, []);

  const removeRequest = useCallback((key) => {
    setRequests(prev => {
      const newMap = new Map(prev);
      newMap.delete(key);
      return newMap;
    });
  }, []);

  const getRequest = useCallback((key) => {
    return requests.get(key);
  }, [requests]);

  const value = {
    requests,
    addRequest,
    removeRequest,
    getRequest
  };

  return (
    <ApiContext.Provider value={value}>
      {children}
    </ApiContext.Provider>
  );
}

function useApiContext() {
  const context = useContext(ApiContext);
  if (!context) {
    throw new Error('useApiContext must be used within ApiProvider');
  }
  return context;
}

// 全局请求状态hook
function useGlobalRequest(key, apiCall, dependencies = []) {
  const { addRequest, removeRequest, getRequest } = useApiContext();
  const [localState, setLocalState] = useState({
    data: null,
    loading: false,
    error: null
  });

  useEffect(() => {
    const existingRequest = getRequest(key);
    if (existingRequest) {
      setLocalState(existingRequest);
      return;
    }

    async function execute() {
      const requestState = {
        data: null,
        loading: true,
        error: null
      };

      setLocalState(requestState);
      addRequest(key, requestState);

      try {
        const result = await apiCall();
        const successState = {
          data: result,
          loading: false,
          error: null
        };

        setLocalState(successState);
        addRequest(key, successState);
      } catch (error) {
        const errorState = {
          data: null,
          loading: false,
          error: error.message
        };

        setLocalState(errorState);
        addRequest(key, errorState);
      }
    }

    execute();

    return () => {
      removeRequest(key);
    };
  }, [key, ...dependencies]);

  return localState;
}

// 使用示例
function UserProfile({ userId }) {
  const { data: user, loading, error } = useGlobalRequest(
    `user-${userId}`,
    () => userService.getUserById(userId),
    [userId]
  );

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误: {error}</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// 在应用根部使用
function App() {
  return (
    <ApiProvider>
      <Router>
        <Routes>
          <Route path="/user/:id" element={<UserProfile />} />
        </Routes>
      </Router>
    </ApiProvider>
  );
}
```

## 11.4 错误处理和重试机制

### 智能重试策略
```javascript
// 指数退避重试
function useRetryableFetch(url, options = {}, maxRetries = 3) {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null,
    retryCount: 0
  });

  const fetchWithRetry = useCallback(async (retryCount = 0) => {
    try {
      setState(prev => ({ ...prev, loading: true, error: null }));

      const response = await fetch(url, options);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      const data = await response.json();
      setState({
        data,
        loading: false,
        error: null,
        retryCount: 0
      });
    } catch (error) {
      if (retryCount < maxRetries) {
        // 指数退避：1s, 2s, 4s, 8s...
        const delay = Math.pow(2, retryCount) * 1000;
        
        setState(prev => ({
          ...prev,
          loading: false,
          error: `重试中... (${retryCount + 1}/${maxRetries})`,
          retryCount: retryCount + 1
        }));

        setTimeout(() => {
          fetchWithRetry(retryCount + 1);
        }, delay);
      } else {
        setState(prev => ({
          ...prev,
          loading: false,
          error: error.message,
          retryCount: retryCount + 1
        }));
      }
    }
  }, [url, JSON.stringify(options), maxRetries]);

  useEffect(() => {
    fetchWithRetry();
  }, [fetchWithRetry]);

  const retry = useCallback(() => {
    fetchWithRetry();
  }, [fetchWithRetry]);

  return { ...state, retry };
}

// 网络状态感知
function useNetworkAwareFetch(url, options = {}) {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  const [state, setState] = useState({
    data: null,
    loading: false,
    error: null
  });

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }

    function handleOffline() {
      setIsOnline(false);
      setState(prev => ({
        ...prev,
        error: '网络连接已断开'
      }));
    }

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  const fetchData = useCallback(async () => {
    if (!isOnline) {
      setState(prev => ({
        ...prev,
        error: '无网络连接'
      }));
      return;
    }

    try {
      setState(prev => ({ ...prev, loading: true, error: null }));
      
      const response = await fetch(url, options);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      const data = await response.json();
      setState({
        data,
        loading: false,
        error: null
      });
    } catch (error) {
      setState(prev => ({
        ...prev,
        loading: false,
        error: error.message
      }));
    }
  }, [url, JSON.stringify(options), isOnline]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  // 网络恢复时自动重试
  useEffect(() => {
    if (isOnline && state.error) {
      fetchData();
    }
  }, [isOnline, state.error, fetchData]);

  return { ...state, isOnline, retry: fetchData };
}

// 断路器模式
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.threshold = threshold;
    this.timeout = timeout;
    this.failureCount = 0;
    this.lastFailureTime = null;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
  }

  async call(apiCall) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('断路器处于开启状态');
      }
    }

    try {
      const result = await apiCall();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();

    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}

// 使用断路器的fetch hook
function useCircuitBreakerFetch(url, options = {}) {
  const circuitBreakerRef = useRef(new CircuitBreaker());
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null
  });

  const fetchData = useCallback(async () => {
    try {
      setState(prev => ({ ...prev, loading: true, error: null }));

      const result = await circuitBreakerRef.current.call(async () => {
        const response = await fetch(url, options);
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }
        return response.json();
      });

      setState({
        data: result,
        loading: false,
        error: null
      });
    } catch (error) {
      setState(prev => ({
        ...prev,
        loading: false,
        error: error.message
      }));
    }
  }, [url, JSON.stringify(options)]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { ...state, retry: fetchData };
}
```

### 错误边界和全局错误处理
```javascript
// API错误边界
class ApiErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // 记录API错误
    console.error('API Error:', error, errorInfo);
    
    // 发送错误报告
    if (error.name === 'ApiError') {
      this.reportApiError(error, errorInfo);
    }
  }

  reportApiError = (error, errorInfo) => {
    // 发送到错误监控服务
    fetch('/api/errors', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        type: 'api_error',
        message: error.message,
        stack: error.stack,
        componentStack: errorInfo.componentStack,
        timestamp: new Date().toISOString()
      })
    }).catch(err => console.error('Failed to report error:', err));
  };

  render() {
    if (this.state.hasError) {
      return (
        <div className="api-error-boundary">
          <h2>数据加载失败</h2>
          <p>很抱歉，无法加载数据。请稍后重试。</p>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            重试
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// 全局错误处理器
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason);
  
  // 如果是API错误，显示用户友好的消息
  if (event.reason?.name === 'ApiError') {
    alert(`操作失败: ${event.reason.message}`);
    event.preventDefault();
  }
});

// 自定义API错误类
class ApiError extends Error {
  constructor(message, status, data = null) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
    this.data = data;
  }
}

// 全局错误处理的fetch函数
async function fetchWithErrorHandling(url, options = {}) {
  try {
    const response = await fetch(url, options);
    
    if (!response.ok) {
      const errorData = await response.json().catch(() => null);
      throw new ApiError(
        errorData?.message || `HTTP ${response.status}`,
        response.status,
        errorData
      );
    }
    
    return response.json();
  } catch (error) {
    if (error instanceof ApiError) {
      throw error;
    }
    
    // 网络错误或其他错误
    throw new ApiError(
      '网络连接失败，请检查网络设置',
      0,
      null
    );
  }
}
```

## 11.5 实际案例：博客系统API集成

### 完整的博客API集成
```javascript
// services/blogService.js
import api from './apiConfig';

class BlogService {
  // 获取文章列表
  async getPosts(params = {}) {
    const { page = 1, limit = 10, category, tag, search } = params;
    
    const queryParams = new URLSearchParams({
      page: page.toString(),
      limit: limit.toString()
    });
    
    if (category) queryParams.append('category', category);
    if (tag) queryParams.append('tag', tag);
    if (search) queryParams.append('search', search);
    
    const response = await api.get(`/posts?${queryParams}`);
    return response.data;
  }

  // 获取单篇文章
  async getPost(id) {
    const response = await api.get(`/posts/${id}`);
    return response.data;
  }

  // 创建文章
  async createPost(postData) {
    const response = await api.post('/posts', postData);
    return response.data;
  }

  // 更新文章
  async updatePost(id, postData) {
    const response = await api.put(`/posts/${id}`, postData);
    return response.data;
  }

  // 删除文章
  async deletePost(id) {
    await api.delete(`/posts/${id}`);
  }

  // 获取分类列表
  async getCategories() {
    const response = await api.get('/categories');
    return response.data;
  }

  // 获取标签列表
  async getTags() {
    const response = await api.get('/tags');
    return response.data;
  }

  // 获取评论
  async getComments(postId) {
    const response = await api.get(`/posts/${postId}/comments`);
    return response.data;
  }

  // 添加评论
  async addComment(postId, commentData) {
    const response = await api.post(`/posts/${postId}/comments`, commentData);
    return response.data;
  }

  // 点赞文章
  async likePost(id) {
    const response = await api.post(`/posts/${id}/like`);
    return response.data;
  }

  // 收藏文章
  async bookmarkPost(id) {
    const response = await api.post(`/posts/${id}/bookmark`);
    return response.data;
  }
}

export default new BlogService();

// components/BlogPost.js
import { useState, useEffect } from 'react';
import blogService from '../services/blogService';

function BlogPost({ postId }) {
  const [post, setPost] = useState(null);
  const [comments, setComments] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [likesCount, setLikesCount] = useState(0);
  const [isLiked, setIsLiked] = useState(false);

  useEffect(() => {
    async function fetchData() {
      try {
        setLoading(true);
        setError(null);

        const [postData, commentsData] = await Promise.all([
          blogService.getPost(postId),
          blogService.getComments(postId)
        ]);

        setPost(postData);
        setComments(commentsData);
        setLikesCount(postData.likesCount);
        setIsLiked(postData.isLikedByUser);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchData();
  }, [postId]);

  const handleLike = async () => {
    try {
      const result = await blogService.likePost(postId);
      setLikesCount(result.likesCount);
      setIsLiked(result.isLiked);
    } catch (err) {
      alert(`点赞失败: ${err.message}`);
    }
  };

  const handleBookmark = async () => {
    try {
      await blogService.bookmarkPost(postId);
      alert('收藏成功！');
    } catch (err) {
      alert(`收藏失败: ${err.message}`);
    }
  };

  if (loading) return <div className="loading">加载中...</div>;
  if (error) return <div className="error">错误: {error}</div>;
  if (!post) return <div>文章不存在</div>;

  return (
    <article className="blog-post">
      <header>
        <h1>{post.title}</h1>
        <div className="meta">
          <span>作者: {post.author}</span>
          <span>发布时间: {new Date(post.createdAt).toLocaleDateString()}</span>
          <span>分类: {post.category}</span>
        </div>
        <div className="tags">
          {post.tags.map(tag => (
            <span key={tag} className="tag">#{tag}</span>
          ))}
        </div>
      </header>

      <div className="content" dangerouslySetInnerHTML={{ __html: post.content }} />

      <footer className="actions">
        <button 
          onClick={handleLike}
          className={`like-btn ${isLiked ? 'liked' : ''}`}
        >
          👍 {likesCount}
        </button>
        <button onClick={handleBookmark} className="bookmark-btn">
          🔖 收藏
        </button>
        <button className="share-btn">
          📤 分享
        </button>
      </footer>

      <CommentsSection comments={comments} postId={postId} />
    </article>
  );
}

// components/CommentsSection.js
function CommentsSection({ comments, postId }) {
  const [commentText, setCommentText] = useState('');
  const [submitting, setSubmitting] = useState(false);
  const [commentsList, setCommentsList] = useState(comments);

  const handleSubmitComment = async (e) => {
    e.preventDefault();
    if (!commentText.trim()) return;

    try {
      setSubmitting(true);
      const newComment = await blogService.addComment(postId, {
        content: commentText.trim()
      });
      
      setCommentsList(prev => [...prev, newComment]);
      setCommentText('');
    } catch (err) {
      alert(`评论失败: ${err.message}`);
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <section className="comments-section">
      <h3>评论 ({commentsList.length})</h3>
      
      <form onSubmit={handleSubmitComment} className="comment-form">
        <textarea
          value={commentText}
          onChange={(e) => setCommentText(e.target.value)}
          placeholder="写下你的评论..."
          rows={4}
        />
        <button type="submit" disabled={submitting || !commentText.trim()}>
          {submitting ? '提交中...' : '发表评论'}
        </button>
      </form>

      <div className="comments-list">
        {commentsList.map(comment => (
          <div key={comment.id} className="comment">
            <div className="comment-header">
              <span className="author">{comment.author}</span>
              <span className="date">
                {new Date(comment.createdAt).toLocaleDateString()}
              </span>
            </div>
            <div className="comment-content">{comment.content}</div>
          </div>
        ))}
      </div>
    </section>
  );
}

// components/BlogList.js
function BlogList() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [currentPage, setCurrentPage] = useState(1);
  const [totalPages, setTotalPages] = useState(1);
  const [filters, setFilters] = useState({
    category: '',
    tag: '',
    search: ''
  });

  const fetchPosts = useCallback(async (page = 1, newFilters = filters) => {
    try {
      setLoading(true);
      setError(null);

      const params = {
        page,
        limit: 10,
        ...newFilters
      };

      const response = await blogService.getPosts(params);
      
      if (page === 1) {
        setPosts(response.data);
      } else {
        setPosts(prev => [...prev, ...response.data]);
      }
      
      setCurrentPage(page);
      setTotalPages(response.totalPages);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [filters]);

  useEffect(() => {
    fetchPosts(1, filters);
  }, [filters, fetchPosts]);

  const handleFilterChange = (newFilters) => {
    setFilters(newFilters);
    setCurrentPage(1);
  };

  const loadMore = () => {
    if (currentPage < totalPages && !loading) {
      fetchPosts(currentPage + 1);
    }
  };

  return (
    <div className="blog-list">
      <BlogFilters filters={filters} onFilterChange={handleFilterChange} />
      
      <div className="posts-grid">
        {posts.map(post => (
          <BlogPostCard key={post.id} post={post} />
        ))}
      </div>

      {loading && <div className="loading">加载中...</div>}
      {error && <div className="error">错误: {error}</div>}
      
      {currentPage < totalPages && !loading && (
        <button onClick={loadMore} className="load-more-btn">
          加载更多
        </button>
      )}
    </div>
  );
}

// components/BlogPostCard.js
function BlogPostCard({ post }) {
  return (
    <div className="blog-post-card">
      {post.featuredImage && (
        <img src={post.featuredImage} alt={post.title} />
      )}
      <div className="card-content">
        <h3>
          <Link to={`/posts/${post.id}`}>{post.title}</Link>
        </h3>
        <p className="excerpt">{post.excerpt}</p>
        <div className="meta">
          <span>{post.author}</span>
          <span>{new Date(post.createdAt).toLocaleDateString()}</span>
          <span>{post.category}</span>
        </div>
        <div className="stats">
          <span>👍 {post.likesCount}</span>
          <span>💬 {post.commentsCount}</span>
          <span>👁 {post.viewsCount}</span>
        </div>
      </div>
    </div>
  );
}
```

## 11.6 总结

### HTTP请求最佳实践
```javascript
// 最佳实践清单
const httpBestPractices = {
  // 请求配置
  configuration: [
    '✅ 使用axios或fetch进行网络请求',
    '✅ 配置请求和响应拦截器',
    '✅ 设置合理的超时时间',
    '✅ 统一错误处理机制',
    '✅ 使用AbortController取消请求'
  ],

  // 状态管理
  stateManagement: [
    '✅ 正确处理loading状态',
    '✅ 优雅处理错误状态',
    '✅ 实现请求缓存机制',
    '✅ 避免重复请求',
    '✅ 清理副作用和内存泄漏'
  ],

  // 用户体验
  userExperience: [
    '✅ 显示加载指示器',
    '✅ 提供重试机制',
    '✅ 实现乐观更新',
    '✅ 处理网络状态变化',
    '✅ 提供离线功能'
  ],

  // 性能优化
  performance: [
    '✅ 实现请求去重',
    '✅ 使用分页和虚拟滚动',
    '✅ 合理使用缓存',
    '✅ 实现预加载',
    '✅ 压缩请求数据'
  ],

  // 安全性
  security: [
    '✅ 验证服务器响应',
    '✅ 防止XSS攻击',
    '✅ 正确处理认证token',
    '✅ 使用HTTPS',
    '✅ 验证用户输入'
  ]
};
```

### 本章重点
1. **HTTP基础**：掌握Fetch API和Axios的使用
2. **状态管理**：正确处理异步请求的各种状态
3. **错误处理**：实现健壮的错误处理和重试机制
4. **性能优化**：缓存、去重、分页等优化技术
5. **实践应用**：通过博客系统学习完整的API集成

### 下章预告
下一章我们将学习测试，包括单元测试、集成测试、端到端测试等内容。

### 练习作业
1. 实现一个完整的REST API客户端
2. 创建一个带缓存和重试的fetch Hook
3. 构建一个实时数据更新的组件

### 常见问题
1. **Q: 何时使用Fetch vs Axios？**
   A: 简单项目使用Fetch，复杂项目推荐Axios（更多功能和更好的浏览器支持）。

2. **Q: 如何处理请求竞态条件？**
   A: 使用AbortController取消过期请求，或者使用请求ID进行去重。

3. **Q: 如何优化大量并发请求？**
   A: 实现请求队列、限制并发数、使用缓存减少重复请求。
