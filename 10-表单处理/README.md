# 第十章：表单处理

## 学习目标
- 理解受控组件和非受控组件的区别
- 掌握表单验证的各种方法
- 学会使用第三方表单库
- 能够处理复杂的表单场景
- 了解表单性能优化技巧

## 10.1 表单基础概念

### 受控组件 vs 非受控组件

#### 受控组件
```javascript
import React, { useState } from 'react';

function ControlledForm() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    confirmPassword: '',
    age: '',
    gender: '',
    hobbies: [],
    bio: '',
    agreeTerms: false
  });
  
  const handleInputChange = (e) => {
    const { name, value, type, checked } = e.target;
    
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };
  
  const handleHobbyChange = (e) => {
    const { value, checked } = e.target;
    
    setFormData(prev => ({
      ...prev,
      hobbies: checked 
        ? [...prev.hobbies, value]
        : prev.hobbies.filter(hobby => hobby !== value)
    }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('表单数据:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>用户名:</label>
        <input
          type="text"
          name="username"
          value={formData.username}
          onChange={handleInputChange}
          required
        />
      </div>
      
      <div>
        <label>邮箱:</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleInputChange}
          required
        />
      </div>
      
      <div>
        <label>密码:</label>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleInputChange}
          required
        />
      </div>
      
      <div>
        <label>确认密码:</label>
        <input
          type="password"
          name="confirmPassword"
          value={formData.confirmPassword}
          onChange={handleInputChange}
          required
        />
      </div>
      
      <div>
        <label>年龄:</label>
        <input
          type="number"
          name="age"
          value={formData.age}
          onChange={handleInputChange}
          min="1"
          max="150"
        />
      </div>
      
      <div>
        <label>性别:</label>
        <select
          name="gender"
          value={formData.gender}
          onChange={handleInputChange}
        >
          <option value="">请选择</option>
          <option value="male">男</option>
          <option value="female">女</option>
          <option value="other">其他</option>
        </select>
      </div>
      
      <div>
        <label>兴趣爱好:</label>
        {['读书', '运动', '音乐', '旅行', '编程'].map(hobby => (
          <label key={hobby}>
            <input
              type="checkbox"
              value={hobby}
              checked={formData.hobbies.includes(hobby)}
              onChange={handleHobbyChange}
            />
            {hobby}
          </label>
        ))}
      </div>
      
      <div>
        <label>个人简介:</label>
        <textarea
          name="bio"
          value={formData.bio}
          onChange={handleInputChange}
          rows="4"
        />
      </div>
      
      <div>
        <label>
          <input
            type="checkbox"
            name="agreeTerms"
            checked={formData.agreeTerms}
            onChange={handleInputChange}
            required
          />
          我同意服务条款
        </label>
      </div>
      
      <button type="submit">提交</button>
    </form>
  );
}
```

#### 非受控组件
```javascript
import React, { useRef } from 'react';

function UncontrolledForm() {
  const formRef = useRef();
  const usernameRef = useRef();
  const emailRef = useRef();
  const passwordRef = useRef();
  const fileRef = useRef();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // 使用FormData获取表单数据
    const formData = new FormData(formRef.current);
    const data = Object.fromEntries(formData.entries());
    
    // 获取文件
    const file = fileRef.current.files[0];
    
    console.log('表单数据:', data);
    console.log('上传的文件:', file);
    
    // 直接访问DOM元素
    console.log('用户名:', usernameRef.current.value);
    console.log('邮箱:', emailRef.current.value);
  };
  
  const resetForm = () => {
    formRef.current.reset();
  };
  
  return (
    <form ref={formRef} onSubmit={handleSubmit}>
      <div>
        <label>用户名:</label>
        <input
          ref={usernameRef}
          type="text"
          name="username"
          defaultValue=""
          required
        />
      </div>
      
      <div>
        <label>邮箱:</label>
        <input
          ref={emailRef}
          type="email"
          name="email"
          defaultValue=""
          required
        />
      </div>
      
      <div>
        <label>密码:</label>
        <input
          ref={passwordRef}
          type="password"
          name="password"
          defaultValue=""
          required
        />
      </div>
      
      <div>
        <label>头像:</label>
        <input
          ref={fileRef}
          type="file"
          name="avatar"
          accept="image/*"
        />
      </div>
      
      <div>
        <button type="submit">提交</button>
        <button type="button" onClick={resetForm}>重置</button>
      </div>
    </form>
  );
}
```

## 10.2 表单验证

### 原生HTML验证
```javascript
function NativeValidationForm() {
  const [errors, setErrors] = useState({});
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const form = e.target;
    
    // 检查表单有效性
    if (!form.checkValidity()) {
      // 收集验证错误
      const formErrors = {};
      const invalidInputs = form.querySelectorAll(':invalid');
      
      invalidInputs.forEach(input => {
        formErrors[input.name] = input.validationMessage;
      });
      
      setErrors(formErrors);
      return;
    }
    
    setErrors({});
    // 提交表单
    const formData = new FormData(form);
    console.log('表单提交:', Object.fromEntries(formData));
  };
  
  return (
    <form onSubmit={handleSubmit} noValidate>
      <div>
        <label>用户名 (3-20个字符):</label>
        <input
          type="text"
          name="username"
          minLength="3"
          maxLength="20"
          required
          pattern="[a-zA-Z0-9_]+"
          title="只能包含字母、数字和下划线"
        />
        {errors.username && <span className="error">{errors.username}</span>}
      </div>
      
      <div>
        <label>邮箱:</label>
        <input
          type="email"
          name="email"
          required
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <label>年龄 (18-100岁):</label>
        <input
          type="number"
          name="age"
          min="18"
          max="100"
          required
        />
        {errors.age && <span className="error">{errors.age}</span>}
      </div>
      
      <div>
        <label>网站 (可选):</label>
        <input
          type="url"
          name="website"
          placeholder="https://example.com"
        />
        {errors.website && <span className="error">{errors.website}</span>}
      </div>
      
      <button type="submit">提交</button>
    </form>
  );
}
```

### 自定义验证
```javascript
function CustomValidationForm() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    confirmPassword: '',
    phone: ''
  });
  
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  // 验证规则
  const validationRules = {
    username: [
      { test: value => value.length >= 3, message: '用户名至少3个字符' },
      { test: value => value.length <= 20, message: '用户名不能超过20个字符' },
      { test: value => /^[a-zA-Z0-9_]+$/.test(value), message: '用户名只能包含字母、数字和下划线' }
    ],
    email: [
      { test: value => /\S+@\S+\.\S+/.test(value), message: '请输入有效的邮箱地址' }
    ],
    password: [
      { test: value => value.length >= 8, message: '密码至少8个字符' },
      { test: value => /(?=.*[a-z])/.test(value), message: '密码必须包含小写字母' },
      { test: value => /(?=.*[A-Z])/.test(value), message: '密码必须包含大写字母' },
      { test: value => /(?=.*\d)/.test(value), message: '密码必须包含数字' },
      { test: value => /(?=.*[@$!%*?&])/.test(value), message: '密码必须包含特殊字符' }
    ],
    confirmPassword: [
      { test: value => value === formData.password, message: '两次输入的密码不一致' }
    ],
    phone: [
      { test: value => !value || /^1[3-9]\d{9}$/.test(value), message: '请输入有效的手机号码' }
    ]
  };
  
  // 验证单个字段
  const validateField = (name, value) => {
    const rules = validationRules[name];
    if (!rules) return '';
    
    for (const rule of rules) {
      if (!rule.test(value)) {
        return rule.message;
      }
    }
    return '';
  };
  
  // 验证所有字段
  const validateForm = () => {
    const newErrors = {};
    Object.keys(formData).forEach(name => {
      const error = validateField(name, formData[name]);
      if (error) {
        newErrors[name] = error;
      }
    });
    return newErrors;
  };
  
  const handleInputChange = (e) => {
    const { name, value } = e.target;
    
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    
    // 实时验证
    if (touched[name]) {
      const error = validateField(name, value);
      setErrors(prev => ({
        ...prev,
        [name]: error
      }));
    }
  };
  
  const handleBlur = (e) => {
    const { name, value } = e.target;
    
    setTouched(prev => ({
      ...prev,
      [name]: true
    }));
    
    const error = validateField(name, value);
    setErrors(prev => ({
      ...prev,
      [name]: error
    }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // 标记所有字段为已触摸
    const allTouched = Object.keys(formData).reduce((acc, key) => {
      acc[key] = true;
      return acc;
    }, {});
    setTouched(allTouched);
    
    // 验证表单
    const formErrors = validateForm();
    setErrors(formErrors);
    
    if (Object.keys(formErrors).length === 0) {
      console.log('表单提交:', formData);
      alert('注册成功！');
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="custom-form">
      <div className="form-group">
        <label>用户名 *</label>
        <input
          type="text"
          name="username"
          value={formData.username}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.username ? 'error' : ''}
          required
        />
        {touched.username && errors.username && (
          <span className="error-message">{errors.username}</span>
        )}
      </div>
      
      <div className="form-group">
        <label>邮箱 *</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.email ? 'error' : ''}
          required
        />
        {touched.email && errors.email && (
          <span className="error-message">{errors.email}</span>
        )}
      </div>
      
      <div className="form-group">
        <label>密码 *</label>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.password ? 'error' : ''}
          required
        />
        {touched.password && errors.password && (
          <span className="error-message">{errors.password}</span>
        )}
        <div className="password-strength">
          <PasswordStrengthIndicator password={formData.password} />
        </div>
      </div>
      
      <div className="form-group">
        <label>确认密码 *</label>
        <input
          type="password"
          name="confirmPassword"
          value={formData.confirmPassword}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.confirmPassword ? 'error' : ''}
          required
        />
        {touched.confirmPassword && errors.confirmPassword && (
          <span className="error-message">{errors.confirmPassword}</span>
        )}
      </div>
      
      <div className="form-group">
        <label>手机号码 (可选)</label>
        <input
          type="tel"
          name="phone"
          value={formData.phone}
          onChange={handleInputChange}
          onBlur={handleBlur}
          className={errors.phone ? 'error' : ''}
          placeholder="13888888888"
        />
        {touched.phone && errors.phone && (
          <span className="error-message">{errors.phone}</span>
        )}
      </div>
      
      <button 
        type="submit" 
        className="submit-button"
        disabled={Object.keys(errors).some(key => errors[key])}
      >
        注册
      </button>
    </form>
  );
}

// 密码强度指示器
function PasswordStrengthIndicator({ password }) {
  const getStrength = (password) => {
    let score = 0;
    if (password.length >= 8) score++;
    if (/[a-z]/.test(password)) score++;
    if (/[A-Z]/.test(password)) score++;
    if (/\d/.test(password)) score++;
    if (/[@$!%*?&]/.test(password)) score++;
    return score;
  };
  
  const strength = getStrength(password);
  const levels = ['很弱', '弱', '一般', '强', '很强'];
  const colors = ['#ff4757', '#ff6b81', '#ffa726', '#66bb6a', '#4caf50'];
  
  return (
    <div className="password-strength-indicator">
      <div className="strength-bar">
        {[1, 2, 3, 4, 5].map(level => (
          <div
            key={level}
            className={`bar ${level <= strength ? 'active' : ''}`}
            style={{ backgroundColor: level <= strength ? colors[strength - 1] : '#e0e0e0' }}
          />
        ))}
      </div>
      <span className="strength-text">
        {password && levels[strength - 1]}
      </span>
    </div>
  );
}
```

## 10.3 表单Hook封装

### 通用表单Hook
```javascript
import { useState, useCallback } from 'react';

function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const setValue = useCallback((name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
    
    // 如果字段已被触摸，立即验证
    if (touched[name] && validate) {
      const fieldError = validate({ ...values, [name]: value })[name];
      setErrors(prev => ({ ...prev, [name]: fieldError }));
    }
  }, [values, touched, validate]);
  
  const setFieldTouched = useCallback((name, isTouched = true) => {
    setTouched(prev => ({ ...prev, [name]: isTouched }));
    
    if (isTouched && validate) {
      const fieldError = validate(values)[name];
      setErrors(prev => ({ ...prev, [name]: fieldError }));
    }
  }, [values, validate]);
  
  const setFieldError = useCallback((name, error) => {
    setErrors(prev => ({ ...prev, [name]: error }));
  }, []);
  
  const handleChange = useCallback((e) => {
    const { name, value, type, checked } = e.target;
    const fieldValue = type === 'checkbox' ? checked : value;
    setValue(name, fieldValue);
  }, [setValue]);
  
  const handleBlur = useCallback((e) => {
    const { name } = e.target;
    setFieldTouched(name, true);
  }, [setFieldTouched]);
  
  const validateForm = useCallback(() => {
    if (!validate) return {};
    return validate(values);
  }, [validate, values]);
  
  const handleSubmit = useCallback(async (onSubmit) => {
    setIsSubmitting(true);
    
    // 标记所有字段为已触摸
    const allTouched = Object.keys(values).reduce((acc, key) => {
      acc[key] = true;
      return acc;
    }, {});
    setTouched(allTouched);
    
    // 验证表单
    const formErrors = validateForm();
    setErrors(formErrors);
    
    const hasErrors = Object.values(formErrors).some(error => error);
    
    if (!hasErrors) {
      try {
        await onSubmit(values);
      } catch (error) {
        console.error('表单提交失败:', error);
      }
    }
    
    setIsSubmitting(false);
    return !hasErrors;
  }, [values, validateForm]);
  
  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  }, [initialValues]);
  
  const getFieldProps = useCallback((name) => ({
    name,
    value: values[name] || '',
    onChange: handleChange,
    onBlur: handleBlur
  }), [values, handleChange, handleBlur]);
  
  const getFieldMeta = useCallback((name) => ({
    error: errors[name],
    touched: touched[name],
    valid: !errors[name]
  }), [errors, touched]);
  
  return {
    values,
    errors,
    touched,
    isSubmitting,
    setValue,
    setFieldTouched,
    setFieldError,
    handleChange,
    handleBlur,
    handleSubmit,
    reset,
    getFieldProps,
    getFieldMeta,
    isValid: Object.keys(errors).length === 0
  };
}

// 使用示例
function LoginForm() {
  const initialValues = {
    email: '',
    password: '',
    rememberMe: false
  };
  
  const validate = (values) => {
    const errors = {};
    
    if (!values.email) {
      errors.email = '请输入邮箱';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      errors.email = '邮箱格式不正确';
    }
    
    if (!values.password) {
      errors.password = '请输入密码';
    } else if (values.password.length < 6) {
      errors.password = '密码至少6个字符';
    }
    
    return errors;
  };
  
  const form = useForm(initialValues, validate);
  
  const onSubmit = async (values) => {
    console.log('登录数据:', values);
    
    // 模拟API调用
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    alert('登录成功！');
    form.reset();
  };
  
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      form.handleSubmit(onSubmit);
    }}>
      <div className="form-field">
        <label>邮箱:</label>
        <input
          {...form.getFieldProps('email')}
          type="email"
          className={form.getFieldMeta('email').error ? 'error' : ''}
        />
        <ErrorMessage name="email" form={form} />
      </div>
      
      <div className="form-field">
        <label>密码:</label>
        <input
          {...form.getFieldProps('password')}
          type="password"
          className={form.getFieldMeta('password').error ? 'error' : ''}
        />
        <ErrorMessage name="password" form={form} />
      </div>
      
      <div className="form-field">
        <label>
          <input
            {...form.getFieldProps('rememberMe')}
            type="checkbox"
            checked={form.values.rememberMe}
          />
          记住我
        </label>
      </div>
      
      <button 
        type="submit" 
        disabled={form.isSubmitting || !form.isValid}
      >
        {form.isSubmitting ? '登录中...' : '登录'}
      </button>
    </form>
  );
}

// 错误消息组件
function ErrorMessage({ name, form }) {
  const meta = form.getFieldMeta(name);
  
  if (!meta.touched || !meta.error) {
    return null;
  }
  
  return <span className="error-message">{meta.error}</span>;
}
```

## 10.4 第三方表单库

### React Hook Form
```javascript
// 安装: npm install react-hook-form
import { useForm, Controller } from 'react-hook-form';

function ReactHookFormExample() {
  const {
    register,
    handleSubmit,
    control,
    formState: { errors, isSubmitting },
    watch,
    reset,
    setValue,
    getValues
  } = useForm({
    defaultValues: {
      username: '',
      email: '',
      password: '',
      confirmPassword: '',
      age: '',
      gender: '',
      skills: [],
      bio: ''
    },
    mode: 'onChange' // 实时验证
  });
  
  const password = watch('password');
  
  const onSubmit = async (data) => {
    console.log('表单数据:', data);
    
    // 模拟API调用
    await new Promise(resolve => setTimeout(resolve, 2000));
    
    alert('提交成功！');
    reset();
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>用户名:</label>
        <input
          {...register('username', {
            required: '用户名不能为空',
            minLength: {
              value: 3,
              message: '用户名至少3个字符'
            },
            maxLength: {
              value: 20,
              message: '用户名不能超过20个字符'
            },
            pattern: {
              value: /^[a-zA-Z0-9_]+$/,
              message: '用户名只能包含字母、数字和下划线'
            }
          })}
        />
        {errors.username && (
          <span className="error">{errors.username.message}</span>
        )}
      </div>
      
      <div>
        <label>邮箱:</label>
        <input
          type="email"
          {...register('email', {
            required: '邮箱不能为空',
            pattern: {
              value: /\S+@\S+\.\S+/,
              message: '邮箱格式不正确'
            }
          })}
        />
        {errors.email && (
          <span className="error">{errors.email.message}</span>
        )}
      </div>
      
      <div>
        <label>密码:</label>
        <input
          type="password"
          {...register('password', {
            required: '密码不能为空',
            minLength: {
              value: 8,
              message: '密码至少8个字符'
            },
            validate: {
              hasLowerCase: value => 
                /[a-z]/.test(value) || '密码必须包含小写字母',
              hasUpperCase: value => 
                /[A-Z]/.test(value) || '密码必须包含大写字母',
              hasNumber: value => 
                /\d/.test(value) || '密码必须包含数字',
              hasSpecialChar: value => 
                /[@$!%*?&]/.test(value) || '密码必须包含特殊字符'
            }
          })}
        />
        {errors.password && (
          <span className="error">{errors.password.message}</span>
        )}
      </div>
      
      <div>
        <label>确认密码:</label>
        <input
          type="password"
          {...register('confirmPassword', {
            required: '请确认密码',
            validate: value => 
              value === password || '两次输入的密码不一致'
          })}
        />
        {errors.confirmPassword && (
          <span className="error">{errors.confirmPassword.message}</span>
        )}
      </div>
      
      <div>
        <label>年龄:</label>
        <input
          type="number"
          {...register('age', {
            min: {
              value: 18,
              message: '年龄不能小于18岁'
            },
            max: {
              value: 100,
              message: '年龄不能大于100岁'
            }
          })}
        />
        {errors.age && (
          <span className="error">{errors.age.message}</span>
        )}
      </div>
      
      <div>
        <label>性别:</label>
        <Controller
          name="gender"
          control={control}
          rules={{ required: '请选择性别' }}
          render={({ field }) => (
            <select {...field}>
              <option value="">请选择</option>
              <option value="male">男</option>
              <option value="female">女</option>
              <option value="other">其他</option>
            </select>
          )}
        />
        {errors.gender && (
          <span className="error">{errors.gender.message}</span>
        )}
      </div>
      
      <div>
        <label>技能:</label>
        <Controller
          name="skills"
          control={control}
          render={({ field }) => (
            <div>
              {['JavaScript', 'React', 'Node.js', 'Python', 'Java'].map(skill => (
                <label key={skill}>
                  <input
                    type="checkbox"
                    value={skill}
                    checked={field.value.includes(skill)}
                    onChange={(e) => {
                      if (e.target.checked) {
                        field.onChange([...field.value, skill]);
                      } else {
                        field.onChange(field.value.filter(s => s !== skill));
                      }
                    }}
                  />
                  {skill}
                </label>
              ))}
            </div>
          )}
        />
      </div>
      
      <div>
        <label>个人简介:</label>
        <textarea
          {...register('bio', {
            maxLength: {
              value: 500,
              message: '个人简介不能超过500个字符'
            }
          })}
          rows="4"
        />
        {errors.bio && (
          <span className="error">{errors.bio.message}</span>
        )}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '提交中...' : '提交'}
      </button>
    </form>
  );
}
```

### Formik
```javascript
// 安装: npm install formik yup
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

// 验证模式
const validationSchema = Yup.object({
  username: Yup.string()
    .min(3, '用户名至少3个字符')
    .max(20, '用户名不能超过20个字符')
    .matches(/^[a-zA-Z0-9_]+$/, '用户名只能包含字母、数字和下划线')
    .required('用户名不能为空'),
  
  email: Yup.string()
    .email('邮箱格式不正确')
    .required('邮箱不能为空'),
  
  password: Yup.string()
    .min(8, '密码至少8个字符')
    .matches(/[a-z]/, '密码必须包含小写字母')
    .matches(/[A-Z]/, '密码必须包含大写字母')
    .matches(/\d/, '密码必须包含数字')
    .matches(/[@$!%*?&]/, '密码必须包含特殊字符')
    .required('密码不能为空'),
  
  confirmPassword: Yup.string()
    .oneOf([Yup.ref('password')], '两次输入的密码不一致')
    .required('请确认密码'),
  
  age: Yup.number()
    .min(18, '年龄不能小于18岁')
    .max(100, '年龄不能大于100岁')
    .nullable(),
  
  gender: Yup.string()
    .required('请选择性别'),
  
  skills: Yup.array()
    .min(1, '请至少选择一项技能'),
  
  terms: Yup.boolean()
    .oneOf([true], '请同意服务条款')
});

function FormikExample() {
  const initialValues = {
    username: '',
    email: '',
    password: '',
    confirmPassword: '',
    age: '',
    gender: '',
    skills: [],
    bio: '',
    terms: false
  };
  
  const handleSubmit = async (values, { setSubmitting, resetForm }) => {
    console.log('表单数据:', values);
    
    try {
      // 模拟API调用
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      alert('提交成功！');
      resetForm();
    } catch (error) {
      console.error('提交失败:', error);
    } finally {
      setSubmitting(false);
    }
  };
  
  return (
    <Formik
      initialValues={initialValues}
      validationSchema={validationSchema}
      onSubmit={handleSubmit}
    >
      {({ isSubmitting, values, setFieldValue }) => (
        <Form>
          <div className="form-field">
            <label>用户名:</label>
            <Field
              name="username"
              type="text"
            />
            <ErrorMessage name="username" component="div" className="error" />
          </div>
          
          <div className="form-field">
            <label>邮箱:</label>
            <Field
              name="email"
              type="email"
            />
            <ErrorMessage name="email" component="div" className="error" />
          </div>
          
          <div className="form-field">
            <label>密码:</label>
            <Field
              name="password"
              type="password"
            />
            <ErrorMessage name="password" component="div" className="error" />
          </div>
          
          <div className="form-field">
            <label>确认密码:</label>
            <Field
              name="confirmPassword"
              type="password"
            />
            <ErrorMessage name="confirmPassword" component="div" className="error" />
          </div>
          
          <div className="form-field">
            <label>年龄:</label>
            <Field
              name="age"
              type="number"
            />
            <ErrorMessage name="age" component="div" className="error" />
          </div>
          
          <div className="form-field">
            <label>性别:</label>
            <Field name="gender" as="select">
              <option value="">请选择</option>
              <option value="male">男</option>
              <option value="female">女</option>
              <option value="other">其他</option>
            </Field>
            <ErrorMessage name="gender" component="div" className="error" />
          </div>
          
          <div className="form-field">
            <label>技能:</label>
            <div>
              {['JavaScript', 'React', 'Node.js', 'Python', 'Java'].map(skill => (
                <label key={skill}>
                  <Field
                    name="skills"
                    type="checkbox"
                    value={skill}
                    checked={values.skills.includes(skill)}
                    onChange={(e) => {
                      if (e.target.checked) {
                        setFieldValue('skills', [...values.skills, skill]);
                      } else {
                        setFieldValue('skills', values.skills.filter(s => s !== skill));
                      }
                    }}
                  />
                  {skill}
                </label>
              ))}
            </div>
            <ErrorMessage name="skills" component="div" className="error" />
          </div>
          
          <div className="form-field">
            <label>个人简介:</label>
            <Field
              name="bio"
              as="textarea"
              rows="4"
            />
            <ErrorMessage name="bio" component="div" className="error" />
          </div>
          
          <div className="form-field">
            <label>
              <Field
                name="terms"
                type="checkbox"
              />
              我同意服务条款
            </label>
            <ErrorMessage name="terms" component="div" className="error" />
          </div>
          
          <button type="submit" disabled={isSubmitting}>
            {isSubmitting ? '提交中...' : '提交'}
          </button>
        </Form>
      )}
    </Formik>
  );
}
```

## 10.5 复杂表单场景

### 动态表单
```javascript
function DynamicForm() {
  const [formConfig, setFormConfig] = useState([
    {
      id: 'name',
      type: 'text',
      label: '姓名',
      required: true,
      value: ''
    },
    {
      id: 'email',
      type: 'email',
      label: '邮箱',
      required: true,
      value: ''
    }
  ]);
  
  const [formData, setFormData] = useState({});
  
  const fieldTypes = {
    text: { label: '文本输入', component: TextInput },
    email: { label: '邮箱输入', component: EmailInput },
    number: { label: '数字输入', component: NumberInput },
    select: { label: '下拉选择', component: SelectInput },
    textarea: { label: '多行文本', component: TextareaInput },
    checkbox: { label: '复选框', component: CheckboxInput },
    radio: { label: '单选框', component: RadioInput },
    file: { label: '文件上传', component: FileInput }
  };
  
  const addField = (type) => {
    const newField = {
      id: `field_${Date.now()}`,
      type,
      label: `新${fieldTypes[type].label}`,
      required: false,
      value: type === 'checkbox' ? false : '',
      ...(type === 'select' || type === 'radio' ? { options: ['选项1', '选项2'] } : {})
    };
    
    setFormConfig(prev => [...prev, newField]);
  };
  
  const updateField = (fieldId, updates) => {
    setFormConfig(prev => 
      prev.map(field => 
        field.id === fieldId ? { ...field, ...updates } : field
      )
    );
  };
  
  const removeField = (fieldId) => {
    setFormConfig(prev => prev.filter(field => field.id !== fieldId));
  };
  
  const moveField = (fieldId, direction) => {
    setFormConfig(prev => {
      const index = prev.findIndex(field => field.id === fieldId);
      const newIndex = direction === 'up' ? index - 1 : index + 1;
      
      if (newIndex < 0 || newIndex >= prev.length) return prev;
      
      const newConfig = [...prev];
      [newConfig[index], newConfig[newIndex]] = [newConfig[newIndex], newConfig[index]];
      return newConfig;
    });
  };
  
  const handleFieldChange = (fieldId, value) => {
    setFormData(prev => ({ ...prev, [fieldId]: value }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('表单配置:', formConfig);
    console.log('表单数据:', formData);
  };
  
  return (
    <div className="dynamic-form-builder">
      <div className="form-builder">
        <h3>表单构建器</h3>
        
        <div className="field-types">
          <h4>添加字段:</h4>
          {Object.entries(fieldTypes).map(([type, config]) => (
            <button
              key={type}
              onClick={() => addField(type)}
              className="add-field-btn"
            >
              + {config.label}
            </button>
          ))}
        </div>
        
        <div className="form-preview">
          <h4>表单预览:</h4>
          <form onSubmit={handleSubmit}>
            {formConfig.map((field) => {
              const Component = fieldTypes[field.type].component;
              return (
                <div key={field.id} className="form-field-wrapper">
                  <div className="field-controls">
                    <button
                      type="button"
                      onClick={() => moveField(field.id, 'up')}
                      disabled={formConfig[0].id === field.id}
                    >
                      ↑
                    </button>
                    <button
                      type="button"
                      onClick={() => moveField(field.id, 'down')}
                      disabled={formConfig[formConfig.length - 1].id === field.id}
                    >
                      ↓
                    </button>
                    <button
                      type="button"
                      onClick={() => removeField(field.id)}
                    >
                      删除
                    </button>
                  </div>
                  
                  <FieldEditor
                    field={field}
                    onUpdate={(updates) => updateField(field.id, updates)}
                  />
                  
                  <Component
                    field={field}
                    value={formData[field.id] || field.value}
                    onChange={(value) => handleFieldChange(field.id, value)}
                  />
                </div>
              );
            })}
            
            <button type="submit">提交表单</button>
          </form>
        </div>
      </div>
    </div>
  );
}

// 字段编辑器
function FieldEditor({ field, onUpdate }) {
  const [isEditing, setIsEditing] = useState(false);
  
  if (!isEditing) {
    return (
      <button onClick={() => setIsEditing(true)} className="edit-field-btn">
        编辑字段
      </button>
    );
  }
  
  return (
    <div className="field-editor">
      <input
        type="text"
        value={field.label}
        onChange={(e) => onUpdate({ label: e.target.value })}
        placeholder="字段标签"
      />
      
      <label>
        <input
          type="checkbox"
          checked={field.required}
          onChange={(e) => onUpdate({ required: e.target.checked })}
        />
        必填
      </label>
      
      {(field.type === 'select' || field.type === 'radio') && (
        <div>
          <label>选项 (每行一个):</label>
          <textarea
            value={field.options.join('\n')}
            onChange={(e) => onUpdate({ 
              options: e.target.value.split('\n').filter(Boolean) 
            })}
          />
        </div>
      )}
      
      <button onClick={() => setIsEditing(false)}>完成编辑</button>
    </div>
  );
}

// 各种输入组件
function TextInput({ field, value, onChange }) {
  return (
    <div className="form-field">
      <label>
        {field.label}
        {field.required && <span className="required">*</span>}
      </label>
      <input
        type="text"
        value={value}
        onChange={(e) => onChange(e.target.value)}
        required={field.required}
      />
    </div>
  );
}

function SelectInput({ field, value, onChange }) {
  return (
    <div className="form-field">
      <label>
        {field.label}
        {field.required && <span className="required">*</span>}
      </label>
      <select
        value={value}
        onChange={(e) => onChange(e.target.value)}
        required={field.required}
      >
        <option value="">请选择</option>
        {field.options.map(option => (
          <option key={option} value={option}>{option}</option>
        ))}
      </select>
    </div>
  );
}

// ... 其他输入组件类似实现
```

### 文件上传表单
```javascript
function FileUploadForm() {
  const [files, setFiles] = useState([]);
  const [uploadProgress, setUploadProgress] = useState({});
  const [uploadedFiles, setUploadedFiles] = useState([]);
  
  const handleFileSelect = (e) => {
    const selectedFiles = Array.from(e.target.files);
    
    // 验证文件
    const validFiles = selectedFiles.filter(file => {
      const isValidType = ['image/jpeg', 'image/png', 'application/pdf'].includes(file.type);
      const isValidSize = file.size <= 10 * 1024 * 1024; // 10MB
      
      if (!isValidType) {
        alert(`${file.name}: 不支持的文件类型`);
        return false;
      }
      
      if (!isValidSize) {
        alert(`${file.name}: 文件大小不能超过10MB`);
        return false;
      }
      
      return true;
    });
    
    setFiles(prev => [...prev, ...validFiles]);
  };
  
  const removeFile = (index) => {
    setFiles(prev => prev.filter((_, i) => i !== index));
  };
  
  const uploadFile = async (file, index) => {
    const formData = new FormData();
    formData.append('file', file);
    
    try {
      const xhr = new XMLHttpRequest();
      
      // 监听上传进度
      xhr.upload.addEventListener('progress', (e) => {
        if (e.lengthComputable) {
          const progress = Math.round((e.loaded / e.total) * 100);
          setUploadProgress(prev => ({ ...prev, [index]: progress }));
        }
      });
      
      return new Promise((resolve, reject) => {
        xhr.addEventListener('load', () => {
          if (xhr.status === 200) {
            const response = JSON.parse(xhr.responseText);
            resolve(response);
          } else {
            reject(new Error('上传失败'));
          }
        });
        
        xhr.addEventListener('error', () => {
          reject(new Error('网络错误'));
        });
        
        xhr.open('POST', '/api/upload');
        xhr.send(formData);
      });
    } catch (error) {
      console.error('上传失败:', error);
      throw error;
    }
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (files.length === 0) {
      alert('请选择要上传的文件');
      return;
    }
    
    try {
      // 并发上传所有文件
      const uploadPromises = files.map((file, index) => uploadFile(file, index));
      const results = await Promise.all(uploadPromises);
      
      setUploadedFiles(results);
      setFiles([]);
      setUploadProgress({});
      
      alert('所有文件上传成功！');
    } catch (error) {
      alert('部分文件上传失败，请重试');
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="file-upload-form">
      <div className="upload-area">
        <input
          type="file"
          multiple
          accept="image/*,.pdf"
          onChange={handleFileSelect}
          id="file-input"
          style={{ display: 'none' }}
        />
        
        <label htmlFor="file-input" className="upload-button">
          <div className="upload-icon">📁</div>
          <div>点击选择文件或拖拽到此处</div>
          <div className="upload-hint">
            支持 JPG, PNG, PDF 格式，单个文件不超过 10MB
          </div>
        </label>
      </div>
      
      {files.length > 0 && (
        <div className="file-list">
          <h4>待上传文件:</h4>
          {files.map((file, index) => (
            <div key={index} className="file-item">
              <div className="file-info">
                <span className="file-name">{file.name}</span>
                <span className="file-size">
                  {(file.size / 1024 / 1024).toFixed(2)} MB
                </span>
              </div>
              
              {uploadProgress[index] !== undefined && (
                <div className="progress-bar">
                  <div 
                    className="progress-fill"
                    style={{ width: `${uploadProgress[index]}%` }}
                  />
                  <span className="progress-text">
                    {uploadProgress[index]}%
                  </span>
                </div>
              )}
              
              <button
                type="button"
                onClick={() => removeFile(index)}
                className="remove-file-btn"
              >
                删除
              </button>
            </div>
          ))}
        </div>
      )}
      
      {uploadedFiles.length > 0 && (
        <div className="uploaded-files">
          <h4>已上传文件:</h4>
          {uploadedFiles.map((file, index) => (
            <div key={index} className="uploaded-file-item">
              <span>{file.filename}</span>
              <a href={file.url} target="_blank" rel="noopener noreferrer">
                查看
              </a>
            </div>
          ))}
        </div>
      )}
      
      <button 
        type="submit" 
        disabled={files.length === 0}
        className="submit-button"
      >
        上传所有文件
      </button>
    </form>
  );
}
```

## 10.6 表单性能优化

### 防抖和节流
```javascript
import { useMemo, useCallback, useRef } from 'react';

// 防抖Hook
function useDebounce(callback, delay) {
  const timeoutRef = useRef(null);
  
  return useCallback((...args) => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    timeoutRef.current = setTimeout(() => {
      callback(...args);
    }, delay);
  }, [callback, delay]);
}

// 节流Hook
function useThrottle(callback, delay) {
  const lastCallRef = useRef(0);
  
  return useCallback((...args) => {
    const now = Date.now();
    if (now - lastCallRef.current >= delay) {
      lastCallRef.current = now;
      callback(...args);
    }
  }, [callback, delay]);
}

// 优化的搜索表单
function OptimizedSearchForm() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  
  // 防抖搜索
  const debouncedSearch = useDebounce(async (searchQuery) => {
    if (!searchQuery.trim()) {
      setResults([]);
      return;
    }
    
    setLoading(true);
    try {
      const response = await fetch(`/api/search?q=${encodeURIComponent(searchQuery)}`);
      const data = await response.json();
      setResults(data.results);
    } catch (error) {
      console.error('搜索失败:', error);
    } finally {
      setLoading(false);
    }
  }, 300);
  
  const handleInputChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    debouncedSearch(value);
  };
  
  return (
    <div className="search-form">
      <input
        type="text"
        value={query}
        onChange={handleInputChange}
        placeholder="搜索..."
      />
      
      {loading && <div>搜索中...</div>}
      
      <div className="search-results">
        {results.map(result => (
          <div key={result.id} className="search-result">
            {result.title}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 大表单优化
```javascript
import { memo, useMemo } from 'react';

// 优化的表单字段组件
const OptimizedFormField = memo(({ field, value, onChange, error }) => {
  console.log(`渲染字段: ${field.name}`);
  
  const handleChange = useCallback((e) => {
    onChange(field.name, e.target.value);
  }, [field.name, onChange]);
  
  return (
    <div className="form-field">
      <label>{field.label}</label>
      <input
        type={field.type}
        value={value}
        onChange={handleChange}
        className={error ? 'error' : ''}
      />
      {error && <span className="error-message">{error}</span>}
    </div>
  );
});

// 大表单组件
function LargeForm() {
  const [formData, setFormData] = useState({});
  const [errors, setErrors] = useState({});
  
  // 使用useMemo优化字段配置
  const fields = useMemo(() => [
    { name: 'field1', label: '字段1', type: 'text' },
    { name: 'field2', label: '字段2', type: 'text' },
    // ... 更多字段
  ], []);
  
  // 优化的onChange处理器
  const handleFieldChange = useCallback((fieldName, value) => {
    setFormData(prev => ({
      ...prev,
      [fieldName]: value
    }));
    
    // 清除该字段的错误
    if (errors[fieldName]) {
      setErrors(prev => ({
        ...prev,
        [fieldName]: null
      }));
    }
  }, [errors]);
  
  // 分组渲染大表单
  const fieldGroups = useMemo(() => {
    const groupSize = 10;
    const groups = [];
    
    for (let i = 0; i < fields.length; i += groupSize) {
      groups.push(fields.slice(i, i + groupSize));
    }
    
    return groups;
  }, [fields]);
  
  return (
    <form className="large-form">
      {fieldGroups.map((group, groupIndex) => (
        <div key={groupIndex} className="field-group">
          <h3>字段组 {groupIndex + 1}</h3>
          {group.map(field => (
            <OptimizedFormField
              key={field.name}
              field={field}
              value={formData[field.name] || ''}
              onChange={handleFieldChange}
              error={errors[field.name]}
            />
          ))}
        </div>
      ))}
    </form>
  );
}
```

## 10.7 总结

### 本章重点
1. **表单基础**：受控组件vs非受控组件的选择
2. **表单验证**：原生验证、自定义验证、第三方库
3. **表单库**：React Hook Form和Formik的使用
4. **复杂场景**：动态表单、文件上传、大表单优化
5. **性能优化**：防抖节流、组件优化、渲染优化

### 最佳实践
- 优先使用受控组件保持数据流清晰
- 合理选择验证时机（实时vs提交时）
- 使用成熟的表单库减少重复代码
- 注意表单性能，避免不必要的重渲染
- 提供良好的用户体验和错误提示

### 下章预告
下一章我们将学习测试，包括单元测试、集成测试、端到端测试等内容。

### 练习作业
1. 使用React Hook Form构建一个完整的用户注册表单
2. 实现一个支持多步骤的向导式表单
3. 创建一个动态表单构建器，支持拖拽排序

### 常见问题
1. **Q: 什么时候使用受控组件vs非受控组件？**
   A: 大多数情况下使用受控组件，只有在简单表单或集成第三方库时考虑非受控组件。

2. **Q: 如何处理大表单的性能问题？**
   A: 使用memo、分组渲染、虚拟滚动、懒加载等技术优化。

3. **Q: React Hook Form vs Formik怎么选？**
   A: React Hook Form性能更好，API更简单；Formik功能更丰富，生态更完善。
