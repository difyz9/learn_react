# 第十三章：部署

## 学习目标
- 掌握React应用的构建和打包流程
- 了解各种部署环境和平台
- 学会配置CI/CD自动化部署
- 掌握生产环境优化技巧
- 了解容器化部署策略

## 13.1 构建打包

### Create React App构建
```javascript
// package.json配置
{
  "name": "my-react-app",
  "version": "1.0.0",
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "build:dev": "REACT_APP_ENV=development npm run build",
    "build:staging": "REACT_APP_ENV=staging npm run build",
    "build:prod": "REACT_APP_ENV=production npm run build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "analyze": "npm run build && npx webpack-bundle-analyzer build/static/js/*.js",
    "serve": "npx serve -s build -l 3000"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1"
  }
}

// 环境变量配置
// .env.development
REACT_APP_API_BASE_URL=http://localhost:3001/api
REACT_APP_ENV=development
REACT_APP_DEBUG=true

// .env.staging
REACT_APP_API_BASE_URL=https://staging-api.example.com/api
REACT_APP_ENV=staging
REACT_APP_DEBUG=false

// .env.production
REACT_APP_API_BASE_URL=https://api.example.com/api
REACT_APP_ENV=production
REACT_APP_DEBUG=false

// 环境配置管理
class Config {
  static get apiBaseUrl() {
    return process.env.REACT_APP_API_BASE_URL || 'http://localhost:3001/api';
  }
  
  static get environment() {
    return process.env.REACT_APP_ENV || 'development';
  }
  
  static get isProduction() {
    return this.environment === 'production';
  }
  
  static get isStaging() {
    return this.environment === 'staging';
  }
  
  static get isDevelopment() {
    return this.environment === 'development';
  }
  
  static get enableDebug() {
    return process.env.REACT_APP_DEBUG === 'true';
  }
  
  static get analytics() {
    return {
      googleAnalyticsId: process.env.REACT_APP_GA_ID,
      enableTracking: this.isProduction
    };
  }
  
  static get features() {
    return {
      enableExperimentalFeatures: process.env.REACT_APP_EXPERIMENTAL === 'true',
      enableBetaFeatures: this.isStaging || this.isDevelopment
    };
  }
}

// 使用配置
function App() {
  useEffect(() => {
    if (Config.analytics.enableTracking && Config.analytics.googleAnalyticsId) {
      // 初始化Google Analytics
      initializeAnalytics(Config.analytics.googleAnalyticsId);
    }
    
    if (Config.enableDebug) {
      console.log('App started in debug mode');
      console.log('Environment:', Config.environment);
      console.log('API Base URL:', Config.apiBaseUrl);
    }
  }, []);
  
  return (
    <div className="app">
      {Config.isDevelopment && <DeveloperTools />}
      {Config.features.enableBetaFeatures && <BetaFeatures />}
      <Router>
        <Routes>
          {/* 路由配置 */}
        </Routes>
      </Router>
    </div>
  );
}
```

### 自定义Webpack配置
```javascript
// webpack.config.js (eject后或自定义配置)
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const CompressionPlugin = require('compression-webpack-plugin');

const isProduction = process.env.NODE_ENV === 'production';

module.exports = {
  mode: isProduction ? 'production' : 'development',
  entry: './src/index.js',
  
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: isProduction 
      ? 'static/js/[name].[contenthash:8].js'
      : 'static/js/bundle.js',
    chunkFilename: isProduction
      ? 'static/js/[name].[contenthash:8].chunk.js'
      : 'static/js/[name].chunk.js',
    publicPath: '/',
    clean: true
  },
  
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@hooks': path.resolve(__dirname, 'src/hooks'),
      '@services': path.resolve(__dirname, 'src/services')
    }
  },
  
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              ['@babel/preset-env', { targets: 'defaults' }],
              ['@babel/preset-react', { runtime: 'automatic' }],
              '@babel/preset-typescript'
            ],
            plugins: [
              '@babel/plugin-transform-runtime',
              isProduction && 'babel-plugin-transform-remove-console'
            ].filter(Boolean)
          }
        }
      },
      {
        test: /\.css$/,
        use: [
          isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
          'css-loader',
          'postcss-loader'
        ]
      },
      {
        test: /\.(png|jpe?g|gif|svg|webp)$/,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024 // 8KB
          }
        },
        generator: {
          filename: 'static/media/[name].[hash:8][ext]'
        }
      }
    ]
  },
  
  optimization: {
    minimize: isProduction,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true,
            drop_debugger: true
          }
        }
      }),
      new CssMinimizerPlugin()
    ],
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        },
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          enforce: true
        }
      }
    },
    runtimeChunk: 'single'
  },
  
  plugins: [
    new HtmlWebpackPlugin({
      template: 'public/index.html',
      minify: isProduction ? {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
        useShortDoctype: true,
        removeEmptyAttributes: true,
        removeStyleLinkTypeAttributes: true,
        keepClosingSlash: true,
        minifyJS: true,
        minifyCSS: true,
        minifyURLs: true
      } : false
    }),
    
    isProduction && new MiniCssExtractPlugin({
      filename: 'static/css/[name].[contenthash:8].css',
      chunkFilename: 'static/css/[name].[contenthash:8].chunk.css'
    }),
    
    isProduction && new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 8192,
      minRatio: 0.8
    }),
    
    process.env.ANALYZE && new BundleAnalyzerPlugin()
  ].filter(Boolean),
  
  devServer: {
    port: 3000,
    open: true,
    hot: true,
    historyApiFallback: true,
    compress: true
  }
};

// 构建脚本
// scripts/build.js
const webpack = require('webpack');
const config = require('../webpack.config.js');
const fs = require('fs-extra');
const path = require('path');

async function build() {
  console.log('Creating an optimized production build...');
  
  // 清理构建目录
  await fs.emptyDir(path.resolve(__dirname, '../build'));
  
  return new Promise((resolve, reject) => {
    const compiler = webpack(config);
    
    compiler.run((err, stats) => {
      if (err) {
        console.error('Build failed:', err);
        reject(err);
        return;
      }
      
      if (stats.hasErrors()) {
        console.error('Build completed with errors:');
        stats.toJson().errors.forEach(error => {
          console.error(error);
        });
        reject(new Error('Build failed'));
        return;
      }
      
      console.log('Build completed successfully!');
      console.log(stats.toString({
        chunks: false,
        colors: true,
        modules: false
      }));
      
      resolve();
    });
  });
}

build().catch(error => {
  console.error('Build script failed:', error);
  process.exit(1);
});
```

### 构建优化
```javascript
// 1. Tree Shaking配置
// package.json
{
  "sideEffects": false,
  // 或者指定有副作用的文件
  "sideEffects": [
    "*.css",
    "*.scss",
    "./src/polyfills.js"
  ]
}

// 2. 代码分割策略
// 路由级分割
const HomePage = lazy(() => import('./pages/HomePage'));
const AboutPage = lazy(() => import('./pages/AboutPage'));
const ContactPage = lazy(() => import('./pages/ContactPage'));

// 组件级分割
const HeavyChart = lazy(() => import('./components/HeavyChart'));

// 条件性分割
const AdminPanel = lazy(() => 
  import('./components/AdminPanel').then(module => ({
    default: module.AdminPanel
  }))
);

// 3. 预加载策略
function App() {
  useEffect(() => {
    // 预加载可能用到的路由
    const timer = setTimeout(() => {
      import('./pages/Dashboard');
      import('./pages/Profile');
    }, 2000);
    
    return () => clearTimeout(timer);
  }, []);
  
  return (
    <Router>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route path="/about" element={<AboutPage />} />
          <Route path="/contact" element={<ContactPage />} />
        </Routes>
      </Suspense>
    </Router>
  );
}

// 4. Service Worker配置
// public/sw.js
const CACHE_NAME = 'my-app-v1';
const urlsToCache = [
  '/',
  '/static/js/bundle.js',
  '/static/css/main.css',
  '/manifest.json'
];

self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(urlsToCache))
  );
});

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // 缓存命中则返回缓存
        if (response) {
          return response;
        }
        return fetch(event.request);
      })
  );
});

// 注册Service Worker
// src/serviceWorker.js
export function register() {
  if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
      navigator.serviceWorker.register('/sw.js')
        .then(registration => {
          console.log('SW registered: ', registration);
        })
        .catch(registrationError => {
          console.log('SW registration failed: ', registrationError);
        });
    });
  }
}

// 在index.js中注册
import { register } from './serviceWorker';

ReactDOM.render(<App />, document.getElementById('root'));

// 只在生产环境注册Service Worker
if (process.env.NODE_ENV === 'production') {
  register();
}
```

## 13.2 静态文件托管

### GitHub Pages部署
```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout
      uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run tests
      run: npm test -- --coverage --passWithNoTests
      
    - name: Build
      run: npm run build
      env:
        CI: false
        REACT_APP_API_BASE_URL: ${{ secrets.REACT_APP_API_BASE_URL }}
        
    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      if: github.ref == 'refs/heads/main'
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./build
        cname: your-domain.com
```

```javascript
// 配置basename for GitHub Pages
// package.json
{
  "homepage": "https://username.github.io/repository-name"
}

// 或者在代码中配置
function App() {
  return (
    <Router basename={process.env.PUBLIC_URL}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Router>
  );
}
```

### Netlify部署
```toml
# netlify.toml
[build]
  publish = "build"
  command = "npm run build"

[build.environment]
  NODE_VERSION = "18"
  NPM_VERSION = "8"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/static/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "/*.js"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "/*.css"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

# 环境变量配置
# 在Netlify控制台设置或使用.env文件
REACT_APP_API_BASE_URL=https://api.example.com
REACT_APP_ANALYTICS_ID=GA_TRACKING_ID
```

```javascript
// Netlify Functions (可选)
// netlify/functions/api.js
exports.handler = async (event, context) => {
  const { httpMethod, path, body } = event;
  
  if (httpMethod === 'GET') {
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      },
      body: JSON.stringify({ message: 'Hello from Netlify Functions!' })
    };
  }
  
  return {
    statusCode: 405,
    body: JSON.stringify({ error: 'Method not allowed' })
  };
};

// 在React中使用
async function fetchFromNetlifyFunction() {
  try {
    const response = await fetch('/.netlify/functions/api');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error fetching from Netlify function:', error);
    throw error;
  }
}
```

### Vercel部署
```json
// vercel.json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "build"
      }
    }
  ],
  "routes": [
    {
      "src": "/static/(.*)",
      "headers": {
        "Cache-Control": "public, max-age=31536000, immutable"
      }
    },
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ],
  "env": {
    "REACT_APP_API_BASE_URL": "@api_base_url"
  },
  "build": {
    "env": {
      "REACT_APP_API_BASE_URL": "@api_base_url"
    }
  }
}
```

```javascript
// Vercel API Routes (可选)
// api/hello.js
export default function handler(req, res) {
  if (req.method === 'GET') {
    res.status(200).json({ message: 'Hello from Vercel API!' });
  } else {
    res.status(405).json({ error: 'Method not allowed' });
  }
}

// 在React中使用
async function fetchFromVercelAPI() {
  try {
    const response = await fetch('/api/hello');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error fetching from Vercel API:', error);
    throw error;
  }
}
```

## 13.3 服务器部署

### Nginx配置
```nginx
# /etc/nginx/sites-available/your-app
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    
    # 重定向到HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;
    
    # SSL配置
    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    
    # 网站根目录
    root /var/www/your-app/build;
    index index.html;
    
    # Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;
    
    # 缓存策略
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }
    
    # API代理
    location /api/ {
        proxy_pass http://localhost:3001/api/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
    
    # React Router支持
    location / {
        try_files $uri $uri/ /index.html;
        
        # 安全头部
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Referrer-Policy "strict-origin-when-cross-origin" always;
        add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' https:;" always;
    }
    
    # 健康检查
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
    
    # 错误页面
    error_page 404 /index.html;
    error_page 500 502 503 504 /50x.html;
    
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

### Docker部署
```dockerfile
# Dockerfile
# 多阶段构建
FROM node:18-alpine AS builder

WORKDIR /app

# 复制package.json和package-lock.json
COPY package*.json ./

# 安装依赖
RUN npm ci --only=production && npm cache clean --force

# 复制源代码
COPY . .

# 构建应用
RUN npm run build

# 生产阶段
FROM nginx:alpine

# 复制自定义nginx配置
COPY nginx.conf /etc/nginx/nginx.conf

# 复制构建结果
COPY --from=builder /app/build /usr/share/nginx/html

# 复制SSL证书（如果有）
# COPY ssl/ /etc/nginx/ssl/

# 暴露端口
EXPOSE 80 443

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/health || exit 1

# 启动nginx
CMD ["nginx", "-g", "daemon off;"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "80:80"
      - "443:443"
    environment:
      - NODE_ENV=production
    volumes:
      - ./ssl:/etc/nginx/ssl:ro
      - ./logs:/var/log/nginx
    restart: unless-stopped
    networks:
      - app-network

  # 可选：添加API服务
  api:
    image: your-api-image:latest
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
    restart: unless-stopped
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

# 部署脚本
# deploy.sh
#!/bin/bash

echo "Starting deployment..."

# 构建镜像
docker-compose build --no-cache

# 停止旧容器
docker-compose down

# 启动新容器
docker-compose up -d

# 检查健康状态
echo "Checking health status..."
sleep 10

if curl -f http://localhost/health; then
    echo "Deployment successful!"
else
    echo "Deployment failed!"
    exit 1
fi
```

## 13.4 CI/CD自动化

### GitHub Actions完整流程
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run linting
      run: npm run lint
      
    - name: Run tests
      run: npm test -- --coverage --passWithNoTests
      
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        
  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Build application
      run: npm run build
      env:
        REACT_APP_API_BASE_URL: ${{ secrets.REACT_APP_API_BASE_URL }}
        REACT_APP_GA_ID: ${{ secrets.REACT_APP_GA_ID }}
        
    - name: Setup Docker Buildx
      uses: docker/setup-buildx-action@v2
      
    - name: Login to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
        
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}
          
    - name: Build and push Docker image
      id: build
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
        
  deploy-staging:
    needs: [test, build]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: staging
    
    steps:
    - name: Deploy to staging
      run: |
        echo "Deploying to staging environment..."
        # 实际部署命令
        
  deploy-production:
    needs: [test, build]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
    - name: Deploy to production
      run: |
        echo "Deploying to production environment..."
        # 实际部署命令
        
    - name: Notify deployment
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        channel: '#deployments'
        webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
      if: always()
```

### 蓝绿部署策略
```bash
#!/bin/bash
# blue-green-deploy.sh

set -e

# 配置
BLUE_PORT=3000
GREEN_PORT=3001
HEALTH_CHECK_URL="http://localhost"
NGINX_CONFIG_DIR="/etc/nginx/sites-available"
APP_NAME="my-react-app"

# 获取当前活动环境
get_active_environment() {
    if docker ps --format "table {{.Names}}" | grep -q "${APP_NAME}-blue"; then
        echo "blue"
    else
        echo "green"
    fi
}

# 健康检查
health_check() {
    local port=$1
    local max_attempts=30
    local attempt=1
    
    echo "Performing health check on port $port..."
    
    while [ $attempt -le $max_attempts ]; do
        if curl -f "${HEALTH_CHECK_URL}:${port}/health" >/dev/null 2>&1; then
            echo "Health check passed!"
            return 0
        fi
        
        echo "Attempt $attempt/$max_attempts failed, retrying in 10 seconds..."
        sleep 10
        attempt=$((attempt + 1))
    done
    
    echo "Health check failed after $max_attempts attempts"
    return 1
}

# 切换nginx配置
switch_nginx_config() {
    local target_port=$1
    local config_file="${NGINX_CONFIG_DIR}/${APP_NAME}"
    
    echo "Switching nginx configuration to port $target_port..."
    
    # 更新upstream配置
    sed -i "s/localhost:[0-9]*/localhost:$target_port/g" "$config_file"
    
    # 重新加载nginx配置
    nginx -t && nginx -s reload
    
    echo "Nginx configuration updated successfully"
}

# 主部署逻辑
main() {
    local current_env=$(get_active_environment)
    local target_env
    local target_port
    local current_port
    
    if [ "$current_env" = "blue" ]; then
        target_env="green"
        target_port=$GREEN_PORT
        current_port=$BLUE_PORT
    else
        target_env="blue"
        target_port=$BLUE_PORT
        current_port=$GREEN_PORT
    fi
    
    echo "Current environment: $current_env (port $current_port)"
    echo "Target environment: $target_env (port $target_port)"
    
    # 构建新镜像
    echo "Building new Docker image..."
    docker build -t "${APP_NAME}:latest" .
    
    # 停止目标环境容器（如果存在）
    docker stop "${APP_NAME}-${target_env}" 2>/dev/null || true
    docker rm "${APP_NAME}-${target_env}" 2>/dev/null || true
    
    # 启动新容器
    echo "Starting new container in $target_env environment..."
    docker run -d \
        --name "${APP_NAME}-${target_env}" \
        -p "$target_port:80" \
        --restart unless-stopped \
        "${APP_NAME}:latest"
    
    # 健康检查
    if health_check "$target_port"; then
        # 切换流量
        switch_nginx_config "$target_port"
        
        # 等待一段时间确保切换成功
        echo "Waiting for traffic switch to complete..."
        sleep 30
        
        # 停止旧环境
        echo "Stopping old environment ($current_env)..."
        docker stop "${APP_NAME}-${current_env}" || true
        docker rm "${APP_NAME}-${current_env}" || true
        
        echo "Blue-green deployment completed successfully!"
        echo "Active environment: $target_env (port $target_port)"
    else
        echo "Deployment failed! Rolling back..."
        docker stop "${APP_NAME}-${target_env}" || true
        docker rm "${APP_NAME}-${target_env}" || true
        exit 1
    fi
}

# 执行部署
main "$@"
```

### 滚动更新部署
```yaml
# kubernetes-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-app
  labels:
    app: react-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: react-app
  template:
    metadata:
      labels:
        app: react-app
    spec:
      containers:
      - name: react-app
        image: your-registry/react-app:latest
        ports:
        - containerPort: 80
        env:
        - name: NODE_ENV
          value: "production"
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 250m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: react-app-service
spec:
  selector:
    app: react-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: react-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - your-domain.com
    secretName: react-app-tls
  rules:
  - host: your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: react-app-service
            port:
              number: 80
```

## 13.5 监控和日志

### 应用监控
```javascript
// 错误边界和错误报告
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    this.setState({
      error,
      errorInfo
    });
    
    // 发送错误报告
    this.reportError(error, errorInfo);
  }
  
  reportError = (error, errorInfo) => {
    const errorReport = {
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      url: window.location.href,
      userId: this.getUserId(), // 如果有用户系统
      buildVersion: process.env.REACT_APP_VERSION
    };
    
    // 发送到错误监控服务
    if (window.Sentry) {
      window.Sentry.captureException(error, {
        contexts: {
          react: {
            componentStack: errorInfo.componentStack
          }
        }
      });
    }
    
    // 发送到自定义错误收集服务
    fetch('/api/errors', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(errorReport)
    }).catch(err => {
      console.error('Failed to report error:', err);
    });
  };
  
  getUserId = () => {
    // 获取用户ID的逻辑
    return localStorage.getItem('userId') || 'anonymous';
  };
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>出错了！</h2>
          <p>很抱歉，应用遇到了一个错误。</p>
          {process.env.NODE_ENV === 'development' && (
            <details style={{ whiteSpace: 'pre-wrap' }}>
              <summary>错误详情</summary>
              <p>{this.state.error && this.state.error.toString()}</p>
              <p>{this.state.errorInfo.componentStack}</p>
            </details>
          )}
          <button onClick={() => window.location.reload()}>
            重新加载页面
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// 性能监控
function PerformanceMonitor() {
  useEffect(() => {
    // 监控页面加载性能
    if ('performance' in window) {
      window.addEventListener('load', () => {
        setTimeout(() => {
          const navigation = performance.getEntriesByType('navigation')[0];
          const paint = performance.getEntriesByType('paint');
          
          const performanceData = {
            // 页面加载时间
            loadTime: navigation.loadEventEnd - navigation.loadEventStart,
            // DOM解析时间
            domParseTime: navigation.domContentLoadedEventEnd - navigation.domContentLoadedEventStart,
            // 首次内容绘制
            firstContentfulPaint: paint.find(entry => entry.name === 'first-contentful-paint')?.startTime,
            // 最大内容绘制
            largestContentfulPaint: null,
            // 首次输入延迟
            firstInputDelay: null,
            // 累计布局偏移
            cumulativeLayoutShift: null
          };
          
          // 监控LCP
          new PerformanceObserver((entryList) => {
            const entries = entryList.getEntries();
            const lastEntry = entries[entries.length - 1];
            performanceData.largestContentfulPaint = lastEntry.startTime;
          }).observe({ entryTypes: ['largest-contentful-paint'] });
          
          // 监控FID
          new PerformanceObserver((entryList) => {
            const entries = entryList.getEntries();
            entries.forEach(entry => {
              performanceData.firstInputDelay = entry.processingStart - entry.startTime;
            });
          }).observe({ entryTypes: ['first-input'] });
          
          // 监控CLS
          new PerformanceObserver((entryList) => {
            const entries = entryList.getEntries();
            entries.forEach(entry => {
              if (!entry.hadRecentInput) {
                performanceData.cumulativeLayoutShift += entry.value;
              }
            });
          }).observe({ entryTypes: ['layout-shift'] });
          
          // 发送性能数据
          sendPerformanceData(performanceData);
        }, 0);
      });
    }
  }, []);
  
  return null;
}

function sendPerformanceData(data) {
  fetch('/api/performance', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      ...data,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      url: window.location.href
    })
  }).catch(err => {
    console.error('Failed to send performance data:', err);
  });
}

// 用户行为追踪
function useAnalytics() {
  const trackEvent = useCallback((eventName, properties = {}) => {
    // Google Analytics
    if (window.gtag) {
      window.gtag('event', eventName, properties);
    }
    
    // 自定义分析
    fetch('/api/analytics', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        event: eventName,
        properties,
        timestamp: new Date().toISOString(),
        sessionId: getSessionId(),
        userId: getUserId(),
        url: window.location.href
      })
    }).catch(err => {
      console.error('Failed to track event:', err);
    });
  }, []);
  
  const trackPageView = useCallback((path) => {
    trackEvent('page_view', { path });
  }, [trackEvent]);
  
  return { trackEvent, trackPageView };
}

// 在组件中使用
function App() {
  const { trackPageView } = useAnalytics();
  const location = useLocation();
  
  useEffect(() => {
    trackPageView(location.pathname);
  }, [location.pathname, trackPageView]);
  
  return (
    <ErrorBoundary>
      <PerformanceMonitor />
      {/* 应用内容 */}
    </ErrorBoundary>
  );
}
```

### 日志系统
```javascript
// 统一日志系统
class Logger {
  constructor() {
    this.logLevel = process.env.NODE_ENV === 'production' ? 'warn' : 'debug';
    this.logBuffer = [];
    this.maxBufferSize = 100;
  }
  
  log(level, message, meta = {}) {
    const logEntry = {
      level,
      message,
      meta,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      url: window.location.href,
      sessionId: this.getSessionId()
    };
    
    // 添加到缓冲区
    this.logBuffer.push(logEntry);
    if (this.logBuffer.length > this.maxBufferSize) {
      this.logBuffer.shift();
    }
    
    // 控制台输出
    if (this.shouldLog(level)) {
      console[level](message, meta);
    }
    
    // 发送到服务器（仅限错误和警告）
    if (level === 'error' || level === 'warn') {
      this.sendToServer(logEntry);
    }
  }
  
  debug(message, meta) {
    this.log('debug', message, meta);
  }
  
  info(message, meta) {
    this.log('info', message, meta);
  }
  
  warn(message, meta) {
    this.log('warn', message, meta);
  }
  
  error(message, meta) {
    this.log('error', message, meta);
  }
  
  shouldLog(level) {
    const levels = ['debug', 'info', 'warn', 'error'];
    const currentLevelIndex = levels.indexOf(this.logLevel);
    const messageLevelIndex = levels.indexOf(level);
    return messageLevelIndex >= currentLevelIndex;
  }
  
  sendToServer(logEntry) {
    fetch('/api/logs', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(logEntry)
    }).catch(err => {
      console.error('Failed to send log to server:', err);
    });
  }
  
  getSessionId() {
    let sessionId = sessionStorage.getItem('sessionId');
    if (!sessionId) {
      sessionId = 'session_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
      sessionStorage.setItem('sessionId', sessionId);
    }
    return sessionId;
  }
  
  // 获取所有日志（用于调试）
  getAllLogs() {
    return this.logBuffer;
  }
  
  // 清空日志缓冲区
  clearLogs() {
    this.logBuffer = [];
  }
}

// 创建全局日志实例
const logger = new Logger();

// 导出使用
export default logger;

// React Hook版本
export function useLogger() {
  return {
    debug: logger.debug.bind(logger),
    info: logger.info.bind(logger),
    warn: logger.warn.bind(logger),
    error: logger.error.bind(logger)
  };
}

// 使用示例
function MyComponent() {
  const logger = useLogger();
  
  useEffect(() => {
    logger.info('Component mounted', { componentName: 'MyComponent' });
    
    return () => {
      logger.info('Component unmounted', { componentName: 'MyComponent' });
    };
  }, [logger]);
  
  const handleClick = () => {
    try {
      // 一些操作
      logger.debug('Button clicked', { action: 'user_interaction' });
    } catch (error) {
      logger.error('Button click failed', { error: error.message, stack: error.stack });
    }
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

## 13.6 总结

### 部署清单
```javascript
// 部署前检查清单
const deploymentChecklist = {
  // 代码质量
  codeQuality: [
    '✅ 所有测试通过',
    '✅ 代码审查完成',
    '✅ 没有console.log和debugger',
    '✅ 错误边界已实现',
    '✅ 性能优化完成'
  ],
  
  // 构建配置
  buildConfig: [
    '✅ 环境变量正确配置',
    '✅ 构建优化启用',
    '✅ 代码分割配置',
    '✅ 静态资源压缩',
    '✅ Source map配置'
  ],
  
  // 安全配置
  security: [
    '✅ HTTPS配置',
    '✅ 安全头部设置',
    '✅ API密钥安全存储',
    '✅ 敏感信息清理',
    '✅ CSP策略配置'
  ],
  
  // 监控配置
  monitoring: [
    '✅ 错误监控配置',
    '✅ 性能监控启用',
    '✅ 日志系统配置',
    '✅ 健康检查端点',
    '✅ 分析工具集成'
  ],
  
  // 部署配置
  deployment: [
    '✅ 部署脚本测试',
    '✅ 回滚策略准备',
    '✅ 数据库迁移',
    '✅ 缓存策略配置',
    '✅ 负载均衡配置'
  ]
};
```

### 本章重点
1. **构建优化**：掌握React应用的构建和打包技术
2. **部署策略**：了解各种部署环境和平台的特点
3. **CI/CD流程**：实现自动化的持续集成和部署
4. **监控体系**：建立完善的应用监控和日志系统
5. **安全配置**：确保生产环境的安全性

### 下章预告
下一章我们将学习最佳实践，包括代码规范、项目结构、团队协作等内容。

### 练习作业
1. 设置一个完整的CI/CD流程
2. 部署一个React应用到多个平台
3. 实现蓝绿部署或滚动更新策略

### 常见问题
1. **Q: 如何选择合适的部署平台？**
   A: 根据项目需求、预算、技术栈和团队能力来选择。

2. **Q: 如何处理部署失败？**
   A: 准备回滚策略，监控部署状态，快速定位和解决问题。

3. **Q: 如何优化部署速度？**
   A: 使用缓存、并行构建、增量部署等技术。
