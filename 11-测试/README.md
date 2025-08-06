# 第十一章：测试

## 学习目标
- 理解React应用测试的重要性和测试类型
- 掌握Jest和React Testing Library的使用
- 学会编写单元测试、集成测试和端到端测试
- 了解测试驱动开发(TDD)和测试最佳实践
- 能够构建完整的测试体系

## 11.1 测试基础概念

### 为什么需要测试？
1. **保证代码质量**：确保功能按预期工作
2. **防止回归**：避免新代码破坏现有功能
3. **文档作用**：测试即文档，说明代码预期行为
4. **重构信心**：有测试保障的重构更安全
5. **提高开发效率**：早期发现问题，降低修复成本

### 测试类型金字塔
```
       E2E Tests        <- 少量，昂贵，慢
      /          \
    Integration Tests   <- 中等数量，中等成本
   /              \
  Unit Tests             <- 大量，便宜，快
```

### React测试环境搭建
```bash
# Create React App已经配置好了Jest和React Testing Library
npx create-react-app my-app

# 额外安装的测试工具
npm install --save-dev @testing-library/jest-dom
npm install --save-dev @testing-library/user-event
npm install --save-dev msw  # Mock Service Worker
```

### 测试配置
```javascript
// src/setupTests.js
import '@testing-library/jest-dom';

// 全局测试配置
global.ResizeObserver = class ResizeObserver {
  observe() {}
  unobserve() {}
  disconnect() {}
};

// Mock localStorage
const localStorageMock = {
  getItem: jest.fn(),
  setItem: jest.fn(),
  removeItem: jest.fn(),
  clear: jest.fn(),
};
global.localStorage = localStorageMock;

// Mock fetch
global.fetch = jest.fn();
```

## 11.2 单元测试

### 组件渲染测试
```javascript
// src/components/Button.js
import React from 'react';

function Button({ 
  children, 
  onClick, 
  disabled = false, 
  variant = 'primary',
  size = 'medium',
  ...props 
}) {
  const className = `btn btn-${variant} btn-${size}`;
  
  return (
    <button
      className={className}
      onClick={onClick}
      disabled={disabled}
      {...props}
    >
      {children}
    </button>
  );
}

export default Button;

// src/components/__tests__/Button.test.js
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Button from '../Button';

describe('Button Component', () => {
  test('renders button with children', () => {
    render(<Button>Click me</Button>);
    
    const button = screen.getByRole('button', { name: /click me/i });
    expect(button).toBeInTheDocument();
  });
  
  test('applies correct CSS classes', () => {
    render(<Button variant="secondary" size="large">Test</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toHaveClass('btn', 'btn-secondary', 'btn-large');
  });
  
  test('handles click events', async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  test('is disabled when disabled prop is true', () => {
    render(<Button disabled>Disabled</Button>);
    
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
  });
  
  test('does not call onClick when disabled', async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick} disabled>Disabled</Button>);
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(handleClick).not.toHaveBeenCalled();
  });
  
  test('forwards additional props', () => {
    render(<Button data-testid="custom-button" id="btn-1">Test</Button>);
    
    const button = screen.getByTestId('custom-button');
    expect(button).toHaveAttribute('id', 'btn-1');
  });
});
```

### Hooks测试
```javascript
// src/hooks/useCounter.js
import { useState, useCallback } from 'react';

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  const decrement = useCallback(() => {
    setCount(prev => prev - 1);
  }, []);
  
  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);
  
  const setValue = useCallback((value) => {
    setCount(value);
  }, []);
  
  return {
    count,
    increment,
    decrement,
    reset,
    setValue
  };
}

export default useCounter;

// src/hooks/__tests__/useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import useCounter from '../useCounter';

describe('useCounter Hook', () => {
  test('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
  });
  
  test('should initialize with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    expect(result.current.count).toBe(10);
  });
  
  test('should increment count', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
  
  test('should decrement count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });
  
  test('should reset to initial value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
  
  test('should set specific value', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.setValue(25);
    });
    
    expect(result.current.count).toBe(25);
  });
  
  test('should maintain function identity', () => {
    const { result, rerender } = renderHook(() => useCounter());
    
    const firstIncrement = result.current.increment;
    
    rerender();
    
    expect(result.current.increment).toBe(firstIncrement);
  });
});
```

### 异步操作测试
```javascript
// src/services/api.js
export const fetchUser = async (userId) => {
  const response = await fetch(`/api/users/${userId}`);
  
  if (!response.ok) {
    throw new Error(`Failed to fetch user: ${response.status}`);
  }
  
  return response.json();
};

export const createUser = async (userData) => {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(userData),
  });
  
  if (!response.ok) {
    throw new Error(`Failed to create user: ${response.status}`);
  }
  
  return response.json();
};

// src/components/UserProfile.js
import React, { useState, useEffect } from 'react';
import { fetchUser } from '../services/api';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const loadUser = async () => {
      try {
        setLoading(true);
        setError(null);
        const userData = await fetchUser(userId);
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    if (userId) {
      loadUser();
    }
  }, [userId]);
  
  if (loading) {
    return <div data-testid="loading">Loading...</div>;
  }
  
  if (error) {
    return <div data-testid="error">Error: {error}</div>;
  }
  
  if (!user) {
    return <div data-testid="no-user">No user found</div>;
  }
  
  return (
    <div data-testid="user-profile">
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      <p>Role: {user.role}</p>
    </div>
  );
}

export default UserProfile;

// src/components/__tests__/UserProfile.test.js
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import UserProfile from '../UserProfile';
import { fetchUser } from '../../services/api';

// Mock API module
jest.mock('../../services/api');
const mockFetchUser = fetchUser as jest.MockedFunction<typeof fetchUser>;

describe('UserProfile Component', () => {
  beforeEach(() => {
    mockFetchUser.mockClear();
  });
  
  test('shows loading state initially', () => {
    mockFetchUser.mockImplementation(() => new Promise(() => {})); // Never resolves
    
    render(<UserProfile userId="1" />);
    
    expect(screen.getByTestId('loading')).toBeInTheDocument();
  });
  
  test('displays user data when fetch succeeds', async () => {
    const mockUser = {
      id: '1',
      name: 'John Doe',
      email: 'john@example.com',
      role: 'admin'
    };
    
    mockFetchUser.mockResolvedValue(mockUser);
    
    render(<UserProfile userId="1" />);
    
    await waitFor(() => {
      expect(screen.getByTestId('user-profile')).toBeInTheDocument();
    });
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('Email: john@example.com')).toBeInTheDocument();
    expect(screen.getByText('Role: admin')).toBeInTheDocument();
  });
  
  test('displays error when fetch fails', async () => {
    const errorMessage = 'Failed to fetch user: 404';
    mockFetchUser.mockRejectedValue(new Error(errorMessage));
    
    render(<UserProfile userId="1" />);
    
    await waitFor(() => {
      expect(screen.getByTestId('error')).toBeInTheDocument();
    });
    
    expect(screen.getByText(`Error: ${errorMessage}`)).toBeInTheDocument();
  });
  
  test('does not fetch when userId is not provided', () => {
    render(<UserProfile userId={null} />);
    
    expect(mockFetchUser).not.toHaveBeenCalled();
    expect(screen.getByTestId('no-user')).toBeInTheDocument();
  });
  
  test('refetches when userId changes', async () => {
    const { rerender } = render(<UserProfile userId="1" />);
    
    expect(mockFetchUser).toHaveBeenCalledWith('1');
    
    rerender(<UserProfile userId="2" />);
    
    expect(mockFetchUser).toHaveBeenCalledWith('2');
    expect(mockFetchUser).toHaveBeenCalledTimes(2);
  });
});
```

## 11.3 集成测试

### 表单集成测试
```javascript
// src/components/LoginForm.js
import React, { useState } from 'react';

function LoginForm({ onSubmit, loading = false }) {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState({});
  
  const validateForm = () => {
    const newErrors = {};
    
    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Invalid email format';
    }
    
    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 6) {
      newErrors.password = 'Password must be at least 6 characters';
    }
    
    return newErrors;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const formErrors = validateForm();
    setErrors(formErrors);
    
    if (Object.keys(formErrors).length === 0) {
      onSubmit(formData);
    }
  };
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };
  
  return (
    <form onSubmit={handleSubmit} data-testid="login-form">
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <div id="email-error" role="alert" className="error">
            {errors.email}
          </div>
        )}
      </div>
      
      <div>
        <label htmlFor="password">Password:</label>
        <input
          id="password"
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
          aria-invalid={!!errors.password}
          aria-describedby={errors.password ? 'password-error' : undefined}
        />
        {errors.password && (
          <div id="password-error" role="alert" className="error">
            {errors.password}
          </div>
        )}
      </div>
      
      <button type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}

export default LoginForm;

// src/components/__tests__/LoginForm.test.js
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import LoginForm from '../LoginForm';

describe('LoginForm Integration', () => {
  test('successful form submission flow', async () => {
    const mockOnSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    // Fill out the form
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    
    // Submit the form
    await user.click(screen.getByRole('button', { name: /login/i }));
    
    // Verify submission
    expect(mockOnSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123'
    });
  });
  
  test('validation error flow', async () => {
    const mockOnSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    // Submit without filling the form
    await user.click(screen.getByRole('button', { name: /login/i }));
    
    // Check validation errors
    expect(screen.getByRole('alert', { name: /email is required/i })).toBeInTheDocument();
    expect(screen.getByRole('alert', { name: /password is required/i })).toBeInTheDocument();
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });
  
  test('error clearing flow', async () => {
    const mockOnSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    // Trigger validation errors
    await user.click(screen.getByRole('button', { name: /login/i }));
    
    expect(screen.getByRole('alert', { name: /email is required/i })).toBeInTheDocument();
    
    // Start typing to clear error
    await user.type(screen.getByLabelText(/email/i), 't');
    
    expect(screen.queryByRole('alert', { name: /email is required/i })).not.toBeInTheDocument();
  });
  
  test('loading state', () => {
    render(<LoginForm onSubmit={jest.fn()} loading={true} />);
    
    const submitButton = screen.getByRole('button');
    expect(submitButton).toHaveTextContent('Logging in...');
    expect(submitButton).toBeDisabled();
  });
  
  test('email format validation', async () => {
    const mockOnSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    await user.type(screen.getByLabelText(/email/i), 'invalid-email');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /login/i }));
    
    expect(screen.getByRole('alert', { name: /invalid email format/i })).toBeInTheDocument();
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });
  
  test('password length validation', async () => {
    const mockOnSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), '123');
    await user.click(screen.getByRole('button', { name: /login/i }));
    
    expect(screen.getByRole('alert', { name: /password must be at least 6 characters/i })).toBeInTheDocument();
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });
});
```

### Context和Provider测试
```javascript
// src/contexts/AuthContext.js
import React, { createContext, useContext, useReducer } from 'react';

const AuthContext = createContext();

const authReducer = (state, action) => {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, loading: true, error: null };
    case 'LOGIN_SUCCESS':
      return { ...state, loading: false, user: action.payload, error: null };
    case 'LOGIN_FAILURE':
      return { ...state, loading: false, error: action.payload };
    case 'LOGOUT':
      return { ...state, user: null, error: null };
    default:
      return state;
  }
};

export function AuthProvider({ children }) {
  const [state, dispatch] = useReducer(authReducer, {
    user: null,
    loading: false,
    error: null
  });
  
  const login = async (credentials) => {
    dispatch({ type: 'LOGIN_START' });
    
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      });
      
      if (!response.ok) {
        throw new Error('Login failed');
      }
      
      const user = await response.json();
      dispatch({ type: 'LOGIN_SUCCESS', payload: user });
    } catch (error) {
      dispatch({ type: 'LOGIN_FAILURE', payload: error.message });
    }
  };
  
  const logout = () => {
    dispatch({ type: 'LOGOUT' });
  };
  
  return (
    <AuthContext.Provider value={{ ...state, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};

// src/contexts/__tests__/AuthContext.test.js
import React from 'react';
import { render, screen, act, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { AuthProvider, useAuth } from '../AuthContext';

// Mock fetch
global.fetch = jest.fn();

// Test component that uses the auth context
function TestComponent() {
  const { user, loading, error, login, logout } = useAuth();
  
  return (
    <div>
      <div data-testid="user">{user ? user.name : 'Not logged in'}</div>
      <div data-testid="loading">{loading ? 'Loading' : 'Not loading'}</div>
      <div data-testid="error">{error || 'No error'}</div>
      
      <button onClick={() => login({ email: 'test@example.com', password: 'password' })}>
        Login
      </button>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

describe('AuthContext', () => {
  beforeEach(() => {
    fetch.mockClear();
  });
  
  test('provides initial state', () => {
    render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );
    
    expect(screen.getByTestId('user')).toHaveTextContent('Not logged in');
    expect(screen.getByTestId('loading')).toHaveTextContent('Not loading');
    expect(screen.getByTestId('error')).toHaveTextContent('No error');
  });
  
  test('handles successful login', async () => {
    const mockUser = { id: 1, name: 'John Doe', email: 'john@example.com' };
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockUser
    });
    
    const user = userEvent.setup();
    
    render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );
    
    await user.click(screen.getByText('Login'));
    
    // Check loading state
    expect(screen.getByTestId('loading')).toHaveTextContent('Loading');
    
    // Wait for login to complete
    await waitFor(() => {
      expect(screen.getByTestId('user')).toHaveTextContent('John Doe');
    });
    
    expect(screen.getByTestId('loading')).toHaveTextContent('Not loading');
    expect(screen.getByTestId('error')).toHaveTextContent('No error');
  });
  
  test('handles login failure', async () => {
    fetch.mockResolvedValueOnce({
      ok: false,
      status: 401
    });
    
    const user = userEvent.setup();
    
    render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );
    
    await user.click(screen.getByText('Login'));
    
    await waitFor(() => {
      expect(screen.getByTestId('error')).toHaveTextContent('Login failed');
    });
    
    expect(screen.getByTestId('user')).toHaveTextContent('Not logged in');
    expect(screen.getByTestId('loading')).toHaveTextContent('Not loading');
  });
  
  test('handles logout', async () => {
    const mockUser = { id: 1, name: 'John Doe', email: 'john@example.com' };
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockUser
    });
    
    const user = userEvent.setup();
    
    render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );
    
    // Login first
    await user.click(screen.getByText('Login'));
    await waitFor(() => {
      expect(screen.getByTestId('user')).toHaveTextContent('John Doe');
    });
    
    // Then logout
    await user.click(screen.getByText('Logout'));
    
    expect(screen.getByTestId('user')).toHaveTextContent('Not logged in');
  });
  
  test('throws error when used outside provider', () => {
    // Suppress console.error for this test
    const originalError = console.error;
    console.error = jest.fn();
    
    expect(() => {
      render(<TestComponent />);
    }).toThrow('useAuth must be used within AuthProvider');
    
    console.error = originalError;
  });
});
```

## 11.4 端到端测试

### Cypress基础
```javascript
// 安装Cypress
// npm install --save-dev cypress

// cypress/integration/auth.spec.js
describe('Authentication Flow', () => {
  beforeEach(() => {
    // 访问应用首页
    cy.visit('/');
  });
  
  it('should allow user to login successfully', () => {
    // 点击登录链接
    cy.get('[data-testid="login-link"]').click();
    
    // 填写登录表单
    cy.get('[data-testid="email-input"]').type('user@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    
    // 点击登录按钮
    cy.get('[data-testid="login-button"]').click();
    
    // 验证登录成功
    cy.url().should('include', '/dashboard');
    cy.get('[data-testid="user-name"]').should('contain', 'John Doe');
    cy.get('[data-testid="logout-button"]').should('be.visible');
  });
  
  it('should show error for invalid credentials', () => {
    cy.get('[data-testid="login-link"]').click();
    
    cy.get('[data-testid="email-input"]').type('invalid@example.com');
    cy.get('[data-testid="password-input"]').type('wrongpassword');
    cy.get('[data-testid="login-button"]').click();
    
    // 验证错误消息
    cy.get('[data-testid="error-message"]')
      .should('be.visible')
      .and('contain', 'Invalid credentials');
    
    // 确保还在登录页面
    cy.url().should('include', '/login');
  });
  
  it('should validate form fields', () => {
    cy.get('[data-testid="login-link"]').click();
    
    // 尝试提交空表单
    cy.get('[data-testid="login-button"]').click();
    
    // 验证验证消息
    cy.get('[data-testid="email-error"]')
      .should('be.visible')
      .and('contain', 'Email is required');
    
    cy.get('[data-testid="password-error"]')
      .should('be.visible')
      .and('contain', 'Password is required');
  });
});

// cypress/integration/todo.spec.js
describe('Todo Application', () => {
  beforeEach(() => {
    // 模拟已登录状态
    cy.login('user@example.com', 'password123');
    cy.visit('/todos');
  });
  
  it('should create a new todo', () => {
    const todoText = 'Learn Cypress testing';
    
    cy.get('[data-testid="todo-input"]').type(todoText);
    cy.get('[data-testid="add-todo-button"]').click();
    
    // 验证新待办事项出现在列表中
    cy.get('[data-testid="todo-item"]')
      .should('contain', todoText)
      .and('not.have.class', 'completed');
  });
  
  it('should mark todo as completed', () => {
    // 首先创建一个待办事项
    cy.get('[data-testid="todo-input"]').type('Complete this task');
    cy.get('[data-testid="add-todo-button"]').click();
    
    // 点击复选框标记为完成
    cy.get('[data-testid="todo-checkbox"]').first().click();
    
    // 验证状态变化
    cy.get('[data-testid="todo-item"]')
      .first()
      .should('have.class', 'completed');
  });
  
  it('should delete a todo', () => {
    // 创建待办事项
    cy.get('[data-testid="todo-input"]').type('Delete this task');
    cy.get('[data-testid="add-todo-button"]').click();
    
    // 删除待办事项
    cy.get('[data-testid="delete-todo-button"]').first().click();
    
    // 确认删除对话框
    cy.get('[data-testid="confirm-delete"]').click();
    
    // 验证待办事项被删除
    cy.get('[data-testid="todo-item"]').should('not.exist');
  });
  
  it('should filter todos', () => {
    // 创建多个待办事项
    cy.get('[data-testid="todo-input"]').type('Active task');
    cy.get('[data-testid="add-todo-button"]').click();
    
    cy.get('[data-testid="todo-input"]').type('Completed task');
    cy.get('[data-testid="add-todo-button"]').click();
    
    // 标记一个为完成
    cy.get('[data-testid="todo-checkbox"]').last().click();
    
    // 过滤显示只有活跃的
    cy.get('[data-testid="filter-active"]').click();
    cy.get('[data-testid="todo-item"]').should('have.length', 1);
    cy.get('[data-testid="todo-item"]').should('contain', 'Active task');
    
    // 过滤显示只有完成的
    cy.get('[data-testid="filter-completed"]').click();
    cy.get('[data-testid="todo-item"]').should('have.length', 1);
    cy.get('[data-testid="todo-item"]').should('contain', 'Completed task');
  });
});

// cypress/support/commands.js
Cypress.Commands.add('login', (email, password) => {
  cy.request({
    method: 'POST',
    url: '/api/login',
    body: { email, password }
  }).then((response) => {
    window.localStorage.setItem('token', response.body.token);
  });
});

Cypress.Commands.add('createTodo', (text) => {
  cy.request({
    method: 'POST',
    url: '/api/todos',
    headers: {
      Authorization: `Bearer ${window.localStorage.getItem('token')}`
    },
    body: { text }
  });
});
```

## 11.5 Mock和Test Doubles

### API Mocking with MSW
```javascript
// src/mocks/handlers.js
import { rest } from 'msw';

export const handlers = [
  // 登录API
  rest.post('/api/login', (req, res, ctx) => {
    const { email, password } = req.body;
    
    if (email === 'user@example.com' && password === 'password123') {
      return res(
        ctx.status(200),
        ctx.json({
          id: 1,
          name: 'John Doe',
          email: 'user@example.com',
          token: 'fake-jwt-token'
        })
      );
    }
    
    return res(
      ctx.status(401),
      ctx.json({ message: 'Invalid credentials' })
    );
  }),
  
  // 用户信息API
  rest.get('/api/users/:userId', (req, res, ctx) => {
    const { userId } = req.params;
    
    if (userId === '1') {
      return res(
        ctx.status(200),
        ctx.json({
          id: 1,
          name: 'John Doe',
          email: 'john@example.com',
          role: 'admin'
        })
      );
    }
    
    return res(
      ctx.status(404),
      ctx.json({ message: 'User not found' })
    );
  }),
  
  // 待办事项API
  rest.get('/api/todos', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json([
        { id: 1, text: 'Learn React', completed: false },
        { id: 2, text: 'Write tests', completed: true },
        { id: 3, text: 'Deploy app', completed: false }
      ])
    );
  }),
  
  rest.post('/api/todos', (req, res, ctx) => {
    const { text } = req.body;
    
    return res(
      ctx.status(201),
      ctx.json({
        id: Date.now(),
        text,
        completed: false
      })
    );
  }),
  
  // 模拟网络错误
  rest.get('/api/error', (req, res, ctx) => {
    return res.networkError('Failed to connect');
  }),
  
  // 模拟延迟响应
  rest.get('/api/slow', (req, res, ctx) => {
    return res(
      ctx.delay(2000),
      ctx.status(200),
      ctx.json({ message: 'Slow response' })
    );
  })
];

// src/mocks/server.js
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// src/setupTests.js
import { server } from './mocks/server';

// 启动mock服务器
beforeAll(() => server.listen());

// 每个测试后重置handlers
afterEach(() => server.resetHandlers());

// 测试完成后关闭服务器
afterAll(() => server.close());
```

### 组件Mock
```javascript
// src/components/__tests__/Dashboard.test.js
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import Dashboard from '../Dashboard';

// Mock子组件
jest.mock('../UserProfile', () => {
  return function MockUserProfile({ userId }) {
    return <div data-testid="user-profile">User Profile for {userId}</div>;
  };
});

jest.mock('../TodoList', () => {
  return function MockTodoList() {
    return <div data-testid="todo-list">Todo List</div>;
  };
});

jest.mock('../Analytics', () => {
  return function MockAnalytics() {
    return <div data-testid="analytics">Analytics</div>;
  };
});

describe('Dashboard Component', () => {
  test('renders all dashboard sections', async () => {
    render(<Dashboard userId="1" />);
    
    await waitFor(() => {
      expect(screen.getByTestId('user-profile')).toBeInTheDocument();
      expect(screen.getByTestId('todo-list')).toBeInTheDocument();
      expect(screen.getByTestId('analytics')).toBeInTheDocument();
    });
    
    expect(screen.getByTestId('user-profile')).toHaveTextContent('User Profile for 1');
  });
});
```

## 11.6 测试最佳实践

### 测试组织和结构
```javascript
// 测试文件组织结构
/*
src/
  components/
    Button/
      Button.js
      Button.test.js
      Button.stories.js
    Form/
      Form.js
      Form.test.js
      __tests__/
        FormValidation.test.js
        FormSubmission.test.js
  hooks/
    __tests__/
      useCounter.test.js
      useLocalStorage.test.js
  utils/
    __tests__/
      formatters.test.js
      validators.test.js
  __tests__/
    App.test.js
    integration/
      LoginFlow.test.js
      CheckoutFlow.test.js
*/

// 测试辅助工具
// src/test-utils/render.js
import React from 'react';
import { render as rtlRender } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { AuthProvider } from '../contexts/AuthContext';
import { ThemeProvider } from '../contexts/ThemeContext';

function render(ui, options = {}) {
  const {
    initialEntries = ['/'],
    user = null,
    theme = 'light',
    ...renderOptions
  } = options;

  function Wrapper({ children }) {
    return (
      <BrowserRouter>
        <ThemeProvider initialTheme={theme}>
          <AuthProvider initialUser={user}>
            {children}
          </AuthProvider>
        </ThemeProvider>
      </BrowserRouter>
    );
  }

  return rtlRender(ui, { wrapper: Wrapper, ...renderOptions });
}

// 重新导出所有RTL工具
export * from '@testing-library/react';
export { render };

// 测试数据工厂
// src/test-utils/factories.js
export const createUser = (overrides = {}) => ({
  id: 1,
  name: 'Test User',
  email: 'test@example.com',
  role: 'user',
  createdAt: new Date().toISOString(),
  ...overrides
});

export const createTodo = (overrides = {}) => ({
  id: 1,
  text: 'Test todo',
  completed: false,
  userId: 1,
  createdAt: new Date().toISOString(),
  ...overrides
});

export const createPost = (overrides = {}) => ({
  id: 1,
  title: 'Test Post',
  content: 'This is a test post content',
  authorId: 1,
  tags: ['test'],
  publishedAt: new Date().toISOString(),
  ...overrides
});

// 使用工厂函数的测试
import { render, screen } from '../test-utils/render';
import { createUser, createTodo } from '../test-utils/factories';

test('displays user todos', () => {
  const user = createUser({ name: 'Alice' });
  const todos = [
    createTodo({ text: 'First todo' }),
    createTodo({ text: 'Second todo', completed: true })
  ];
  
  render(<TodoList todos={todos} />, { user });
  
  expect(screen.getByText('First todo')).toBeInTheDocument();
  expect(screen.getByText('Second todo')).toBeInTheDocument();
});
```

### 测试覆盖率和质量
```json
// package.json
{
  "scripts": {
    "test": "react-scripts test",
    "test:coverage": "react-scripts test --coverage --watchAll=false",
    "test:ci": "react-scripts test --coverage --watchAll=false --silent"
  },
  "jest": {
    "collectCoverageFrom": [
      "src/**/*.{js,jsx}",
      "!src/index.js",
      "!src/reportWebVitals.js",
      "!src/**/*.stories.{js,jsx}",
      "!src/test-utils/**"
    ],
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

### TDD示例
```javascript
// 测试驱动开发示例：计算器组件

// 第一步：写测试
describe('Calculator', () => {
  test('displays 0 initially', () => {
    render(<Calculator />);
    expect(screen.getByDisplayValue('0')).toBeInTheDocument();
  });
  
  test('displays number when clicked', async () => {
    const user = userEvent.setup();
    render(<Calculator />);
    
    await user.click(screen.getByText('5'));
    expect(screen.getByDisplayValue('5')).toBeInTheDocument();
  });
  
  test('performs addition', async () => {
    const user = userEvent.setup();
    render(<Calculator />);
    
    await user.click(screen.getByText('5'));
    await user.click(screen.getByText('+'));
    await user.click(screen.getByText('3'));
    await user.click(screen.getByText('='));
    
    expect(screen.getByDisplayValue('8')).toBeInTheDocument();
  });
});

// 第二步：实现最小代码让测试通过
function Calculator() {
  const [display, setDisplay] = useState('0');
  const [operation, setOperation] = useState(null);
  const [previousValue, setPreviousValue] = useState(null);
  
  const handleNumber = (num) => {
    setDisplay(display === '0' ? num : display + num);
  };
  
  const handleOperation = (op) => {
    setOperation(op);
    setPreviousValue(display);
    setDisplay('0');
  };
  
  const calculate = () => {
    const prev = parseFloat(previousValue);
    const current = parseFloat(display);
    
    let result;
    switch (operation) {
      case '+':
        result = prev + current;
        break;
      default:
        return;
    }
    
    setDisplay(result.toString());
    setOperation(null);
    setPreviousValue(null);
  };
  
  return (
    <div>
      <input type="text" value={display} readOnly />
      <div>
        {[1, 2, 3, 4, 5, 6, 7, 8, 9, 0].map(num => (
          <button key={num} onClick={() => handleNumber(num.toString())}>
            {num}
          </button>
        ))}
        <button onClick={() => handleOperation('+')}>+</button>
        <button onClick={calculate}>=</button>
      </div>
    </div>
  );
}

// 第三步：重构代码，保持测试通过
// 第四步：添加更多测试和功能...
```

## 11.7 总结

### 本章重点
1. **测试类型**：单元测试、集成测试、端到端测试
2. **测试工具**：Jest、React Testing Library、Cypress、MSW
3. **测试策略**：测试金字塔、TDD、Mock策略
4. **最佳实践**：测试组织、覆盖率、可维护性

### 测试策略
- **70%单元测试**：快速、可靠、易维护
- **20%集成测试**：测试组件协作
- **10%端到端测试**：测试用户流程

### 测试原则
1. **测试行为而非实现**
2. **保持测试简单和专注**
3. **测试应该是可靠和可重复的**
4. **好的测试名称描述测试场景**
5. **避免测试实现细节**

### 下章预告
下一章我们将学习性能优化，包括渲染优化、代码分割、内存管理等内容。

### 练习作业
1. 为一个电商购物车组件编写完整的测试套件
2. 使用TDD方法开发一个表单验证库
3. 编写端到端测试覆盖用户注册和登录流程

### 常见问题
1. **Q: 应该测试什么，不测试什么？**
   A: 测试公共API和用户行为，不测试实现细节和第三方库。

2. **Q: 如何处理异步操作的测试？**
   A: 使用waitFor、findBy查询和适当的断言等待。

3. **Q: Mock应该用到什么程度？**
   A: Mock外部依赖和副作用，但不要过度Mock导致测试脱离现实。
