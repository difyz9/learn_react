# 第二章：JSX 语法

## 学习目标
- 掌握JSX语法的基本规则和使用方法
- 理解JSX与HTML的区别
- 学会在JSX中使用JavaScript表达式
- 掌握条件渲染和列表渲染技巧

## 2.1 JSX 简介

### 什么是JSX？
JSX（JavaScript XML）是React的语法扩展，允许我们在JavaScript代码中编写类似HTML的标记。它是语法糖，最终会被编译成React.createElement调用。

```javascript
// JSX语法
const element = <h1>Hello, world!</h1>;

// 编译后的JavaScript
const element = React.createElement('h1', null, 'Hello, world!');
```

### 为什么使用JSX？
1. **直观易读**：类似HTML的语法更容易理解
2. **功能强大**：可以在标记中嵌入JavaScript表达式
3. **类型安全**：编译时可以捕获错误
4. **性能优化**：编译时进行优化

## 2.2 JSX 基本语法

### 基本规则

#### 1. 必须有一个根元素
```javascript
// ✅ 正确 - 单个根元素
const element = (
  <div>
    <h1>标题</h1>
    <p>段落</p>
  </div>
);

// ✅ 正确 - 使用Fragment
const element = (
  <React.Fragment>
    <h1>标题</h1>
    <p>段落</p>
  </React.Fragment>
);

// ✅ 正确 - Fragment简写
const element = (
  <>
    <h1>标题</h1>
    <p>段落</p>
  </>
);

// ❌ 错误 - 多个根元素
const element = (
  <h1>标题</h1>
  <p>段落</p>
);
```

#### 2. 标签必须闭合
```javascript
// ✅ 正确
<input type="text" />
<img src="photo.jpg" alt="photo" />

// ❌ 错误
<input type="text">
<img src="photo.jpg" alt="photo">
```

#### 3. 属性命名使用camelCase
```javascript
// ✅ 正确
<button onClick={handleClick} className="btn">
  点击我
</button>

// ❌ 错误
<button onclick={handleClick} class="btn">
  点击我
</button>
```

### JSX与HTML的差异

| HTML | JSX | 说明 |
|------|-----|------|
| class | className | class是JavaScript保留字 |
| for | htmlFor | for是JavaScript保留字 |
| onclick | onClick | 事件名使用camelCase |
| tabindex | tabIndex | 属性名使用camelCase |
| readonly | readOnly | 属性名使用camelCase |

## 2.3 JSX 表达式

### 在JSX中嵌入JavaScript表达式
使用花括号`{}`在JSX中嵌入任何有效的JavaScript表达式。

```javascript
function Welcome() {
  const name = 'React学习者';
  const age = 25;
  const isStudent = true;
  
  return (
    <div>
      <h1>欢迎，{name}！</h1>
      <p>年龄：{age}</p>
      <p>身份：{isStudent ? '学生' : '非学生'}</p>
      <p>明年：{age + 1}岁</p>
      <p>当前时间：{new Date().toLocaleString()}</p>
    </div>
  );
}
```

### 表达式的类型

#### 1. 基本数据类型
```javascript
function DataTypes() {
  const str = 'Hello';
  const num = 42;
  const bool = true;
  const nullValue = null;
  const undefinedValue = undefined;
  
  return (
    <div>
      <p>字符串：{str}</p>
      <p>数字：{num}</p>
      {/* 布尔值不会渲染 */}
      <p>布尔值：{bool.toString()}</p>
      {/* null和undefined不会渲染 */}
      <p>null：{nullValue || '无值'}</p>
      <p>undefined：{undefinedValue || '无值'}</p>
    </div>
  );
}
```

#### 2. 数组和对象
```javascript
function ArrayAndObject() {
  const numbers = [1, 2, 3, 4, 5];
  const user = {
    name: '张三',
    email: 'zhangsan@example.com'
  };
  
  return (
    <div>
      {/* 数组会被连接成字符串 */}
      <p>数组：{numbers}</p>
      
      {/* 对象不能直接渲染，需要访问属性 */}
      <p>用户名：{user.name}</p>
      <p>邮箱：{user.email}</p>
      
      {/* 渲染对象的JSON表示 */}
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </div>
  );
}
```

#### 3. 函数调用和方法调用
```javascript
function FunctionCalls() {
  const formatDate = (date) => {
    return date.toLocaleDateString('zh-CN');
  };
  
  const mathOperation = (a, b) => a + b;
  
  return (
    <div>
      <p>今天：{formatDate(new Date())}</p>
      <p>计算结果：{mathOperation(10, 20)}</p>
      <p>随机数：{Math.floor(Math.random() * 100)}</p>
      <p>字符串长度：{'Hello World'.length}</p>
    </div>
  );
}
```

## 2.4 JSX 属性

### 字符串属性
```javascript
function StringAttributes() {
  return (
    <div>
      <img src="logo.png" alt="Logo" />
      <a href="https://reactjs.org" target="_blank" rel="noopener noreferrer">
        React官网
      </a>
    </div>
  );
}
```

### 表达式属性
```javascript
function ExpressionAttributes() {
  const imageUrl = 'https://via.placeholder.com/150';
  const linkText = 'React官方文档';
  const isDisabled = false;
  
  return (
    <div>
      <img src={imageUrl} alt="占位图" width={150} height={150} />
      <button disabled={isDisabled} onClick={() => alert('点击了！')}>
        {linkText}
      </button>
    </div>
  );
}
```

### 样式属性
```javascript
function StyleAttributes() {
  const primaryColor = '#007bff';
  const fontSize = 16;
  
  const styles = {
    container: {
      backgroundColor: '#f8f9fa',
      padding: '20px',
      borderRadius: '8px'
    },
    title: {
      color: primaryColor,
      fontSize: `${fontSize + 4}px`,
      textAlign: 'center'
    }
  };
  
  return (
    <div style={styles.container}>
      <h2 style={styles.title}>样式示例</h2>
      <p style={{ color: 'green', fontWeight: 'bold' }}>
        内联样式文本
      </p>
    </div>
  );
}
```

## 2.5 条件渲染

### 使用条件运算符
```javascript
function ConditionalRendering() {
  const isLoggedIn = true;
  const user = { name: '张三', role: 'admin' };
  
  return (
    <div>
      {/* 三元运算符 */}
      <h1>
        {isLoggedIn ? `欢迎回来，${user.name}！` : '请先登录'}
      </h1>
      
      {/* 逻辑与运算符 */}
      {isLoggedIn && (
        <div>
          <p>用户角色：{user.role}</p>
          {user.role === 'admin' && <button>管理面板</button>}
        </div>
      )}
      
      {/* 逻辑或运算符 */}
      <p>{user.name || '游客'}</p>
    </div>
  );
}
```

### 使用函数进行条件渲染
```javascript
function ConditionalRenderingWithFunction() {
  const userStatus = 'premium'; // 'guest', 'basic', 'premium'
  
  const renderUserBadge = (status) => {
    switch (status) {
      case 'premium':
        return <span className="badge badge-gold">💎 高级用户</span>;
      case 'basic':
        return <span className="badge badge-silver">⭐ 普通用户</span>;
      default:
        return <span className="badge badge-gray">👤 访客</span>;
    }
  };
  
  const renderFeatures = (status) => {
    const features = {
      guest: ['基本浏览'],
      basic: ['基本浏览', '评论功能'],
      premium: ['基本浏览', '评论功能', '高级搜索', '无广告']
    };
    
    return (
      <ul>
        {features[status]?.map((feature, index) => (
          <li key={index}>{feature}</li>
        ))}
      </ul>
    );
  };
  
  return (
    <div>
      <h2>用户权限</h2>
      {renderUserBadge(userStatus)}
      <h3>可用功能：</h3>
      {renderFeatures(userStatus)}
    </div>
  );
}
```

## 2.6 列表渲染

### 基本列表渲染
```javascript
function BasicListRendering() {
  const fruits = ['苹果', '香蕉', '橙子', '葡萄'];
  const numbers = [1, 2, 3, 4, 5];
  
  return (
    <div>
      <h3>水果列表</h3>
      <ul>
        {fruits.map((fruit, index) => (
          <li key={index}>{fruit}</li>
        ))}
      </ul>
      
      <h3>数字列表</h3>
      <ol>
        {numbers.map(number => (
          <li key={number}>{number} 的平方是 {number * number}</li>
        ))}
      </ol>
    </div>
  );
}
```

### 渲染对象数组
```javascript
function ObjectListRendering() {
  const users = [
    { id: 1, name: '张三', email: 'zhangsan@example.com', age: 25 },
    { id: 2, name: '李四', email: 'lisi@example.com', age: 30 },
    { id: 3, name: '王五', email: 'wangwu@example.com', age: 28 }
  ];
  
  return (
    <div>
      <h3>用户列表</h3>
      <div className="user-grid">
        {users.map(user => (
          <div key={user.id} className="user-card">
            <h4>{user.name}</h4>
            <p>年龄：{user.age}</p>
            <p>邮箱：{user.email}</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Key的重要性
```javascript
function KeyImportance() {
  const [items, setItems] = useState([
    { id: 1, text: '学习React' },
    { id: 2, text: '学习JSX' },
    { id: 3, text: '构建项目' }
  ]);
  
  const addItem = () => {
    const newItem = {
      id: Date.now(),
      text: `新任务 ${items.length + 1}`
    };
    setItems([newItem, ...items]);
  };
  
  const deleteItem = (id) => {
    setItems(items.filter(item => item.id !== id));
  };
  
  return (
    <div>
      <h3>待办事项</h3>
      <button onClick={addItem}>添加任务</button>
      
      {/* ✅ 正确 - 使用唯一的id作为key */}
      <ul>
        {items.map(item => (
          <li key={item.id}>
            <span>{item.text}</span>
            <button onClick={() => deleteItem(item.id)}>删除</button>
          </li>
        ))}
      </ul>
      
      {/* ❌ 错误 - 使用数组索引作为key（在动态列表中） */}
      {/* <ul>
        {items.map((item, index) => (
          <li key={index}>
            <span>{item.text}</span>
            <button onClick={() => deleteItem(item.id)}>删除</button>
          </li>
        ))}
      </ul> */}
    </div>
  );
}
```

## 2.7 实践案例：个人信息展示组件

让我们创建一个综合性的个人信息展示组件，应用本章所学的所有概念。

### 创建 PersonProfile.js
```javascript
import React, { useState } from 'react';
import './PersonProfile.css';

function PersonProfile() {
  const [person] = useState({
    name: '李小明',
    age: 28,
    email: 'lixiaoming@example.com',
    avatar: 'https://via.placeholder.com/100',
    bio: '全栈开发工程师，热爱技术，喜欢学习新知识。',
    skills: ['JavaScript', 'React', 'Node.js', 'Python', 'SQL'],
    education: [
      {
        id: 1,
        school: '北京大学',
        degree: '计算机科学学士',
        year: '2015-2019'
      },
      {
        id: 2,
        school: '清华大学',
        degree: '软件工程硕士',
        year: '2019-2021'
      }
    ],
    experience: [
      {
        id: 1,
        company: '腾讯',
        position: '前端开发工程师',
        duration: '2021-2023',
        description: '负责React项目开发和维护'
      },
      {
        id: 2,
        company: '阿里巴巴',
        position: '全栈开发工程师',
        duration: '2023-至今',
        description: '负责全栈项目架构设计和开发'
      }
    ],
    isOnline: true,
    lastActive: new Date()
  });
  
  const [activeTab, setActiveTab] = useState('about');
  
  // 技能等级映射
  const getSkillLevel = (skill) => {
    const levels = {
      'JavaScript': 'expert',
      'React': 'expert',
      'Node.js': 'intermediate',
      'Python': 'intermediate',
      'SQL': 'beginner'
    };
    return levels[skill] || 'beginner';
  };
  
  // 获取技能颜色
  const getSkillColor = (level) => {
    const colors = {
      expert: '#28a745',
      intermediate: '#ffc107',
      beginner: '#6c757d'
    };
    return colors[level];
  };
  
  // 渲染技能标签
  const renderSkills = () => {
    return person.skills.map((skill, index) => {
      const level = getSkillLevel(skill);
      const color = getSkillColor(level);
      
      return (
        <span
          key={index}
          className="skill-tag"
          style={{ backgroundColor: color }}
        >
          {skill}
          <small className="skill-level">({level})</small>
        </span>
      );
    });
  };
  
  // 渲染教育经历
  const renderEducation = () => {
    return person.education.map(edu => (
      <div key={edu.id} className="timeline-item">
        <div className="timeline-marker"></div>
        <div className="timeline-content">
          <h4>{edu.school}</h4>
          <p className="degree">{edu.degree}</p>
          <p className="year">{edu.year}</p>
        </div>
      </div>
    ));
  };
  
  // 渲染工作经历
  const renderExperience = () => {
    return person.experience.map(exp => (
      <div key={exp.id} className="timeline-item">
        <div className="timeline-marker"></div>
        <div className="timeline-content">
          <h4>{exp.company}</h4>
          <p className="position">{exp.position}</p>
          <p className="duration">{exp.duration}</p>
          <p className="description">{exp.description}</p>
        </div>
      </div>
    ));
  };
  
  // 渲染标签页内容
  const renderTabContent = () => {
    switch (activeTab) {
      case 'about':
        return (
          <div className="tab-content">
            <h3>关于我</h3>
            <p>{person.bio}</p>
            <h4>技能专长</h4>
            <div className="skills-container">
              {renderSkills()}
            </div>
          </div>
        );
      case 'education':
        return (
          <div className="tab-content">
            <h3>教育背景</h3>
            <div className="timeline">
              {renderEducation()}
            </div>
          </div>
        );
      case 'experience':
        return (
          <div className="tab-content">
            <h3>工作经历</h3>
            <div className="timeline">
              {renderExperience()}
            </div>
          </div>
        );
      default:
        return null;
    }
  };
  
  return (
    <div className="person-profile">
      {/* 个人信息头部 */}
      <div className="profile-header">
        <div className="avatar-container">
          <img src={person.avatar} alt={`${person.name}的头像`} />
          {person.isOnline && <div className="online-indicator"></div>}
        </div>
        <div className="basic-info">
          <h2>{person.name}</h2>
          <p className="age">年龄：{person.age}岁</p>
          <p className="email">
            <span>📧 </span>
            <a href={`mailto:${person.email}`}>{person.email}</a>
          </p>
          {person.isOnline ? (
            <p className="status online">🟢 在线</p>
          ) : (
            <p className="status offline">
              ⚫ 离线（最后活跃：{person.lastActive.toLocaleString()}）
            </p>
          )}
        </div>
      </div>
      
      {/* 标签页导航 */}
      <div className="tab-navigation">
        {['about', 'education', 'experience'].map(tab => (
          <button
            key={tab}
            className={`tab-button ${activeTab === tab ? 'active' : ''}`}
            onClick={() => setActiveTab(tab)}
          >
            {tab === 'about' && '关于'}
            {tab === 'education' && '教育'}
            {tab === 'experience' && '经历'}
          </button>
        ))}
      </div>
      
      {/* 标签页内容 */}
      {renderTabContent()}
    </div>
  );
}

export default PersonProfile;
```

### 创建 PersonProfile.css
```css
.person-profile {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.profile-header {
  display: flex;
  align-items: center;
  padding: 20px 0;
  border-bottom: 2px solid #f0f0f0;
  margin-bottom: 20px;
}

.avatar-container {
  position: relative;
  margin-right: 20px;
}

.avatar-container img {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  border: 3px solid #007bff;
}

.online-indicator {
  position: absolute;
  bottom: 5px;
  right: 5px;
  width: 20px;
  height: 20px;
  background: #28a745;
  border: 3px solid white;
  border-radius: 50%;
}

.basic-info h2 {
  margin: 0 0 10px 0;
  color: #333;
}

.basic-info p {
  margin: 5px 0;
  color: #666;
}

.basic-info .email a {
  color: #007bff;
  text-decoration: none;
}

.basic-info .email a:hover {
  text-decoration: underline;
}

.status.online {
  color: #28a745;
  font-weight: bold;
}

.status.offline {
  color: #6c757d;
}

.tab-navigation {
  display: flex;
  margin-bottom: 20px;
  border-bottom: 1px solid #ddd;
}

.tab-button {
  padding: 12px 24px;
  border: none;
  background: none;
  cursor: pointer;
  font-size: 16px;
  color: #666;
  border-bottom: 2px solid transparent;
  transition: all 0.3s ease;
}

.tab-button:hover {
  color: #007bff;
}

.tab-button.active {
  color: #007bff;
  border-bottom-color: #007bff;
  font-weight: bold;
}

.tab-content {
  min-height: 300px;
}

.tab-content h3 {
  color: #333;
  margin-bottom: 20px;
}

.tab-content h4 {
  color: #555;
  margin: 20px 0 10px 0;
}

.skills-container {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 15px;
}

.skill-tag {
  display: inline-block;
  padding: 8px 12px;
  color: white;
  border-radius: 20px;
  font-size: 14px;
  font-weight: bold;
}

.skill-level {
  opacity: 0.8;
  font-weight: normal;
  margin-left: 5px;
}

.timeline {
  position: relative;
  padding-left: 30px;
}

.timeline::before {
  content: '';
  position: absolute;
  left: 15px;
  top: 0;
  bottom: 0;
  width: 2px;
  background: #ddd;
}

.timeline-item {
  position: relative;
  margin-bottom: 30px;
}

.timeline-marker {
  position: absolute;
  left: -37px;
  top: 0;
  width: 12px;
  height: 12px;
  background: #007bff;
  border: 3px solid white;
  border-radius: 50%;
  box-shadow: 0 0 0 3px #ddd;
}

.timeline-content {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  border-left: 4px solid #007bff;
}

.timeline-content h4 {
  margin: 0 0 10px 0;
  color: #333;
}

.degree, .position {
  font-weight: bold;
  color: #007bff;
  margin: 5px 0;
}

.year, .duration {
  color: #666;
  font-size: 14px;
  margin: 5px 0;
}

.description {
  color: #555;
  margin: 10px 0 0 0;
}

/* 响应式设计 */
@media (max-width: 600px) {
  .profile-header {
    flex-direction: column;
    text-align: center;
  }
  
  .avatar-container {
    margin-right: 0;
    margin-bottom: 20px;
  }
  
  .tab-navigation {
    flex-direction: column;
  }
  
  .tab-button {
    text-align: left;
    border-bottom: 1px solid #eee;
  }
  
  .tab-button.active {
    border-bottom-color: #007bff;
  }
}
```

## 2.8 JSX 最佳实践

### 1. 保持JSX简洁
```javascript
// ✅ 好的做法
function UserCard({ user }) {
  const formatDate = (date) => date.toLocaleDateString();
  
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>注册时间：{formatDate(user.registerDate)}</p>
    </div>
  );
}

// ❌ 避免的做法
function UserCard({ user }) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>
        注册时间：{new Date(user.registerDate).toLocaleDateString('zh-CN', {
          year: 'numeric',
          month: 'long',
          day: 'numeric'
        })}
      </p>
    </div>
  );
}
```

### 2. 合理使用条件渲染
```javascript
// ✅ 好的做法
function MessageList({ messages, isLoading, error }) {
  if (isLoading) return <div>加载中...</div>;
  if (error) return <div>错误：{error.message}</div>;
  if (!messages.length) return <div>暂无消息</div>;
  
  return (
    <ul>
      {messages.map(message => (
        <li key={message.id}>{message.content}</li>
      ))}
    </ul>
  );
}

// ❌ 避免的做法
function MessageList({ messages, isLoading, error }) {
  return (
    <div>
      {isLoading ? (
        <div>加载中...</div>
      ) : error ? (
        <div>错误：{error.message}</div>
      ) : !messages.length ? (
        <div>暂无消息</div>
      ) : (
        <ul>
          {messages.map(message => (
            <li key={message.id}>{message.content}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

### 3. 正确使用key
```javascript
// ✅ 好的做法 - 使用稳定且唯一的标识符
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
}

// ⚠️ 谨慎使用 - 仅在列表项顺序不变时使用索引
function StaticList({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>
          {item}
        </li>
      ))}
    </ul>
  );
}
```

## 2.9 总结

### 本章重点
1. **JSX语法**：类HTML的JavaScript语法扩展
2. **基本规则**：单根元素、标签闭合、属性命名
3. **表达式嵌入**：使用`{}`嵌入JavaScript表达式
4. **条件渲染**：三元运算符、逻辑运算符、函数条件
5. **列表渲染**：map方法、key属性的重要性

### 关键概念理解
- **声明式UI**：描述界面应该是什么样子
- **表达式 vs 语句**：JSX中只能使用表达式
- **Key的作用**：帮助React识别列表项的变化

### 下章预告
下一章我们将学习React组件开发，包括函数组件和类组件的创建、组件的组合和通信等核心概念。

### 练习作业
1. 创建一个学生成绩管理组件，包含学生列表、成绩统计等功能
2. 实现一个简单的购物车界面，支持商品添加、删除、计算总价
3. 制作一个个人简历组件，包含多个标签页展示不同信息

### 常见问题
1. **Q: 为什么不能在JSX中使用if语句？**
   A: JSX中只能使用表达式，if是语句。可以使用三元运算符或在外部处理逻辑。

2. **Q: 什么时候使用Fragment？**
   A: 当需要返回多个元素但不想添加额外DOM节点时使用。

3. **Q: 为什么要使用key？**
   A: Key帮助React识别哪些列表项发生了变化，提高渲染性能和保持组件状态。
