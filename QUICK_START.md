# 🚀 Discord 登录系统 - 快速部署指南

## 📦 文件清单

已创建的文件：

```
✅ login.html              - 登录页面（黑白灰简约设计 + 猫耳装饰）
✅ login.css               - 登录样式（含响应式设计 + 深色模式）
✅ login.js                - Discord OAuth2 认证管理器
✅ auth-server.js          - 后端认证服务（Node.js/Express）
✅ .env.example            - 环境变量示例
✅ README_LOGIN.md         - 完整使用指南
✅ LOGIN_GUIDE.js          - 详细配置指南
✅ LOGIN_INTEGRATION_HELPER.js - 集成代码片段
✅ QUICK_START.md          - 此文件
```

---

## ⚡ 3 分钟快速开始

### 步骤 1: 创建 Discord 应用（2 分钟）

1. 访问 👉 https://discord.com/developers/applications
2. 点击 **"New Application"**，输入名称
3. 选择 **"OAuth2"** → 找到 **"CLIENT ID"** → 复制
4. 添加重定向 URI:
   - 开发: `http://localhost:8000/login.html`
   - 生产: `https://yourdomain.com/login.html`
5. 保存 CLIENT ID

### 步骤 2: 配置本地登录 (1 分钟)

打开 `login.js`，找到第 7 行：

```javascript
CLIENT_ID: 'YOUR_DISCORD_CLIENT_ID',
```

替换为您的 CLIENT ID：

```javascript
CLIENT_ID: '1234567890123456789',
```

### 步骤 3: 启动本地服务器

```bash
# Python 3
python -m http.server 8000

# 或 Node.js
npx http-server -p 8000
```

访问 👉 http://localhost:8000/login.html

**完成！** 🎉

---

## 🔐 完整部署清单

### 本地开发环境

- [ ] 获取 Discord CLIENT_ID
- [ ] 在 `login.js` 中更新 CLIENT_ID
- [ ] 启动本地服务器
- [ ] 测试登录流程
- [ ] 检查浏览器 LocalStorage 中的 token

### 生产环境部署

- [ ] 获取 Discord CLIENT_ID 和 CLIENT_SECRET
- [ ] 部署后端认证服务
- [ ] 在 Discord 应用中注册生产重定向 URI
- [ ] 在 `login.js` 中更新 TOKEN_ENDPOINT
- [ ] 更新所有配置使用 HTTPS
- [ ] 在后端配置环境变量
- [ ] 测试完整登录流程
- [ ] 监控后端日志

### 可选（推荐）

- [ ] 实现后端 `/api/auth/verify` 端点
- [ ] 实现 Token 刷新机制
- [ ] 添加登出功能
- [ ] 实现用户数据同步到数据库
- [ ] 设置监控和告警

---

## 🔧 核心配置参考

### login.js 中的关键配置

```javascript
// Discord OAuth 配置
CLIENT_ID: '你的CLIENT_ID',                    // ⚠️ 必须配置
REDIRECT_URI: this.getRedirectUri(),           // 自动生成
AUTHORIZE_URL: 'https://discord.com/api/oauth2/authorize',
TOKEN_ENDPOINT: '/api/auth/discord/callback',  // 后端 API
SCOPES: ['identify', 'email']                  // OAuth 权限
```

### 后端配置（auth-server.js）

```javascript
// 环境变量
DISCORD_CLIENT_ID: process.env.DISCORD_CLIENT_ID
DISCORD_CLIENT_SECRET: process.env.DISCORD_CLIENT_SECRET
DISCORD_REDIRECT_URI: process.env.DISCORD_REDIRECT_URI
PORT: process.env.PORT || 3000
```

---

## 🌐 浏览器 LocalStorage 数据结构

用户登录后，以下数据存储在浏览器中：

```javascript
// discord_auth_token - 访问令牌
localStorage.getItem('discord_auth_token')
// → "接收到的Discord access token"

// discord_user_data - 用户信息（JSON）
JSON.parse(localStorage.getItem('discord_user_data'))
// → {
//      id: "用户ID",
//      username: "用户名",
//      email: "邮箱",
//      avatar: "头像哈希",
//      discriminator: "0000"
//    }

// discord_token_expiry - 过期时间（时间戳）
localStorage.getItem('discord_token_expiry')
// → "1705974234567"
```

---

## 📱 登录流程图

```
访问 login.html
    ↓
检查本地 localStorage 中的 token
    ↓
    ├─ Token 存在且未过期
    │  └─ 重定向到 index.html ✅
    │
    └─ Token 不存在或已过期
       └─ 显示登录页面
          ↓
       用户点击 "使用 Discord 登录"
          ↓
       重定向到 Discord 授权
          ↓
       用户在 Discord 授权
          ↓
       重定向回 login.html?code=...&state=...
          ↓
       用户端与后端交换 token
          ↓
       保存 token 到 localStorage
          ↓
       重定向到 index.html ✅
```

---

## 🚀 部署到服务器

### 使用 Node.js + Express

```bash
# 1. 创建项目目录
mkdir my-app && cd my-app

# 2. 初始化 npm
npm init -y

# 3. 安装依赖
npm install express axios cors body-parser dotenv

# 4. 复制 auth-server.js
cp auth-server.js .

# 5. 创建 .env 文件（基于 .env.example）
cp .env.example .env
# 编辑 .env，填入您的 Discord 凭证

# 6. 启动服务器
node auth-server.js
```

### 使用 Nginx + 静态文件

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    
    root /var/www/html;
    index index.html;
    
    # 静态文件
    location ~ \.(html|css|js|json|png|jpg|gif)$ {
        expires 1d;
    }
    
    # API 代理到后端
    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # 单页应用路由
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### 使用 Docker

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "auth-server.js"]
```

```bash
# 构建镜像
docker build -t my-app .

# 运行容器
docker run -d \
  -p 3000:3000 \
  -e DISCORD_CLIENT_ID=YOUR_ID \
  -e DISCORD_CLIENT_SECRET=YOUR_SECRET \
  -e DISCORD_REDIRECT_URI=https://yourdomain.com/login.html \
  my-app
```

---

## 🧪 测试清单

- [ ] 访问 login.html 显示登录页面
- [ ] 点击登录按钮重定向到 Discord
- [ ] Discord 授权后返回应用
- [ ] 自动重定向到 index.html
- [ ] 检查浏览器 LocalStorage 有 token
- [ ] 刷新页面不需要重新登录
- [ ] 在另一个设备登录，需要重新授权
- [ ] 检查手机上的登录页面显示正常（响应式）

---

## 🆘 常见问题

### Q: 访问 login.html 后什么都没有发生

**A:** 
- 检查 CLIENT_ID 是否正确配置
- 打开浏览器控制台（F12）查看错误信息
- 检查网络连接

### Q: Discord 回调后一直在加载中

**A:**
- 检查 TOKEN_ENDPOINT 是否正确
- 检查后端 `/api/auth/discord/callback` 是否正常工作
- 查看后端日志了解详细错误

### Q: 重定向 URI 不匹配的错误

**A:**
- 确保 Discord 应用中注册的 URI 与代码中的完全相同
- 检查协议（http vs https）
- 检查端口号和路径

### Q: Token 总是过期

**A:**
- 检查系统时间是否正确
- 增加后端返回的 expires_in 时间
- 考虑实现 token 刷新机制

---

## 📚 资源链接

- 🔗 Discord Developer Portal: https://discord.com/developers/applications
- 📖 OAuth2 文档: https://discord.com/developers/docs/topics/oauth2
- 🛡️ 安全最佳实践: https://discord.com/developers/docs/reference

---

## 📞 获取帮助

1. 查看 `README_LOGIN.md` - 完整使用指南
2. 查看 `LOGIN_GUIDE.js` - 详细配置说明
3. 查看 `LOGIN_INTEGRATION_HELPER.js` - 代码示例
4. 查看浏览器控制台的错误信息
5. 检查后端日志（如果使用了后端）

---

## ✨ 功能特性总结

### 前端
- ✅ 黑白灰简约设计
- ✅ 猫耳装饰（带动画）
- ✅ 响应式布局
- ✅ 深色模式支持
- ✅ 本地 token 存储
- ✅ 自动重定向

### 后端（可选）
- ✅ OAuth2 token 交换
- ✅ 用户信息获取
- ✅ Token 验证
- ✅ Token 刷新
- ✅ 登出支持
- ✅ 错误处理

---

**祝您使用愉快！** 🎉

最后更新：2026-01-22
