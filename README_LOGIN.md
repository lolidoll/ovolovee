# Discord 登录系统集成指南

## 📋 概述

已为您的网站集成了一个完整的 Discord OAuth2 登录系统，具有以下特性：

- ✨ **黑白灰简约设计** + 猫耳装饰动画
- 🔐 **安全的 OAuth2 认证** 流程
- 💾 **本地持久化存储** - 用户登录后无需重复登录
- 📱 **完全响应式** - 在所有设备上完美工作
- 🎯 **CSRF 防护** - 使用状态码验证

---

## 🚀 快速开始

### 1️⃣ 创建 Discord 应用

#### 步骤：
1. 访问 [Discord Developer Portal](https://discord.com/developers/applications)
2. 点击 **"New Application"** 按钮
3. 输入应用名称（例如："薯片机"）
4. 点击 **"Create"**

### 2️⃣ 获取 CLIENT_ID

1. 在应用页面左侧菜单选择 **"OAuth2"**
2. 在 **"CLIENT ID"** 字段找到您的 ID
3. 复制这个 ID

### 3️⃣ 配置重定向 URI

1. 在 OAuth2 页面找到 **"Redirects"** 部分
2. 点击 **"Add Redirect"**
3. 输入您的重定向 URI：
   - **开发环境**：`http://localhost:8000/login.html`
   - **生产环境**：`https://yourdomain.com/login.html`
4. 点击 **"Save Changes"**

### 4️⃣ 配置登录系统

打开 `login.js` 文件，找到这一行（约第 7 行）：

```javascript
CLIENT_ID: 'YOUR_DISCORD_CLIENT_ID',
```

替换为您的实际 CLIENT_ID：

```javascript
CLIENT_ID: '1234567890123456789',
```

---

## 🔧 文件结构

```
项目根目录/
├── login.html          ← 登录页面（黑白灰简约设计 + 猫耳）
├── login.css           ← 登录页面样式
├── login.js            ← 登录认证管理器（核心逻辑）
├── index.html          ← 主应用页面（已集成登录检查）
├── app.js              ← 主应用脚本
├── LOGIN_GUIDE.js      ← 详细配置指南
└── README_LOGIN.md     ← 此文件
```

---

## 🎨 设计特性

### 登录页面样式

- **颜色方案**：黑、白、灰（高级简约）
- **猫耳装饰**：带动画效果的可爱猫耳
- **按钮**：Discord 蓝色梯度，具有悬停和活动效果
- **响应式**：在手机、平板和桌面上完美工作
- **深色模式**：自动适配系统深色模式

### 动画效果

- 🐱 猫耳轻微摇晃动画
- 📥 登录卡片上滑进入动画
- 🎯 Logo 浮动效果
- ✨ 按钮光泽效果

---

## 🔐 工作流程

```
用户访问网站
    ↓
检查本地登录状态
    ↓
    ├─ 已登录 → 直接进入应用
    │
    └─ 未登录 → 显示登录页面
         ↓
      用户点击 Discord 登录
         ↓
      重定向到 Discord 授权页面
         ↓
      用户在 Discord 授权
         ↓
      重定向回登录页面（带授权码）
         ↓
      交换授权码获取 Token
         ↓
      获取用户信息
         ↓
      保存到本地存储
         ↓
      重定向到主应用
```

---

## 💾 本地存储

用户登录后，以下信息存储在浏览器本地存储中：

| 键名 | 说明 |
|------|------|
| `discord_auth_token` | 用户的 Discord 访问令牌 |
| `discord_user_data` | 用户基本信息（JSON） |
| `discord_token_expiry` | Token 过期时间戳 |

### 安全说明

- 令牌存储在 **localStorage**（可通过开发者工具查看）
- 强烈建议在生产环境使用 **HTTPS**
- 考虑使用后端 API 来中介 token 交换

---

## 🛠️ 后端集成（推荐）

为了更高的安全性，建议使用后端 API 处理 token 交换。

### Node.js/Express 示例

```bash
# 安装依赖
npm install express axios cors body-parser
```

创建 `backend/auth.js`：

```javascript
const express = require('express');
const axios = require('axios');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

const DISCORD_CLIENT_ID = 'YOUR_CLIENT_ID';
const DISCORD_CLIENT_SECRET = 'YOUR_CLIENT_SECRET';
const REDIRECT_URI = 'http://localhost:8000/login.html';

app.post('/api/auth/discord/callback', async (req, res) => {
    try {
        const { code } = req.body;
        
        // 交换授权码获取 token
        const tokenResponse = await axios.post(
            'https://discord.com/api/v10/oauth2/token',
            {
                client_id: DISCORD_CLIENT_ID,
                client_secret: DISCORD_CLIENT_SECRET,
                grant_type: 'authorization_code',
                code: code,
                redirect_uri: REDIRECT_URI
            }
        );
        
        const { access_token } = tokenResponse.data;
        
        // 获取用户信息
        const userResponse = await axios.get(
            'https://discord.com/api/v10/users/@me',
            {
                headers: {
                    'Authorization': `Bearer ${access_token}`
                }
            }
        );
        
        return res.json({
            access_token: access_token,
            expires_in: 604800,
            user: userResponse.data
        });
        
    } catch (error) {
        console.error('OAuth 错误:', error);
        return res.status(500).json({ error: '认证失败' });
    }
});

app.listen(3000, () => {
    console.log('认证服务器运行在 http://localhost:3000');
});
```

然后在 `login.js` 中更新：

```javascript
TOKEN_ENDPOINT: 'http://localhost:3000/api/auth/discord/callback',
```

---

## 📱 在应用中使用用户信息

### 检查登录状态

```javascript
if (authManager.isUserLoggedIn()) {
    const user = authManager.getCurrentUser();
    console.log('用户:', user);
    // {
    //   id: '...',
    //   username: '...',
    //   email: '...',
    //   avatar: '...'
    // }
}
```

### 在 API 请求中使用 Token

```javascript
const token = authManager.getAuthToken();

fetch('/api/data', {
    headers: {
        'Authorization': `Bearer ${token}`
    }
})
.then(res => res.json())
.then(data => console.log(data));
```

### 添加登出功能

```javascript
// 在 app.js 或相关脚本中
document.getElementById('logout-btn').addEventListener('click', () => {
    if (confirm('确定要退出登录吗？')) {
        authManager.logout();
    }
});
```

---

## 🔧 高级配置

### 修改授权作用域

在 `login.js` 中修改 `SCOPES` 数组：

```javascript
SCOPES: ['identify', 'email']  // 基础信息
// 可选作用域：
// 'identify' - 基础用户信息
// 'email' - 用户邮箱
// 'guilds' - 用户加入的服务器列表
// 'dm_channels.read' - 读取 DM 频道
```

### 自定义重定向 URI

```javascript
REDIRECT_URI: 'https://yourdomain.com/auth/callback.html'
```

### 修改页面文案

在 `login.html` 中修改：

```html
<h1 class="login-title">您的应用名称</h1>
<p class="login-subtitle">使用 Discord 登录</p>
```

---

## 🧪 测试

### 本地测试

1. 启动本地服务器：
```bash
# Python 3
python -m http.server 8000

# 或 Node.js (http-server)
npx http-server -p 8000
```

2. 访问 `http://localhost:8000/login.html`

3. 点击 Discord 登录按钮

4. 在 Discord 授权

5. 应该自动重定向回应用

### 检查浏览器存储

1. 打开开发者工具（F12）
2. 进入 **Application** → **Local Storage**
3. 应该看到 `discord_auth_token` 等键

---

## ❌ 常见问题排解

### ❌ "请配置 Discord CLIENT_ID"

**原因**：未设置 CLIENT_ID 或设置为 `YOUR_DISCORD_CLIENT_ID`

**解决**：
1. 确认已在 Discord Developer Portal 创建应用
2. 复制正确的 CLIENT_ID
3. 在 `login.js` 中更新 `CLIENT_ID`

### ❌ 无法重定向到 Discord

**原因**：重定向 URI 不匹配

**解决**：
1. 检查 Discord 应用中注册的重定向 URI
2. 确保与 `login.js` 中的 `REDIRECT_URI` 完全匹配
3. 确保使用了正确的协议（http/https）

### ❌ 登录成功但获取用户信息失败

**原因**：后端 API 未实现或网络错误

**解决**：
1. 检查后端 `/api/auth/discord/callback` 是否正确实现
2. 查看浏览器控制台的错误信息
3. 确保网络连接正常

### ❌ Token 总是过期

**原因**：Token 过期时间设置过短或系统时间不同步

**解决**：
1. 增加后端返回的 `expires_in` 值
2. 检查系统时间是否正确
3. 实现 token 刷新机制

---

## 🔐 安全最佳实践

✅ **推荐做法**

- 使用 HTTPS（生产环境必需）
- 在后端处理 token 交换
- 定期检查 token 过期时间
- 实现 token 刷新机制
- 使用 CSRF 防护（已内置）

❌ **不推荐做法**

- 在前端暴露 `CLIENT_SECRET`
- 使用 HTTP 传输敏感信息
- 将 token 存储在 sessionStorage
- 跳过状态码验证

---

## 📝 修改记录

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-01-22 | 初始版本发布 |

---

## 🆘 获取帮助

### Discord Developer Documentation
- [OAuth2 文档](https://discord.com/developers/docs/topics/oauth2)
- [API 参考](https://discord.com/developers/docs/reference)

### 调试提示

1. 打开浏览器控制台（F12）
2. 查看 Console 和 Network 标签的错误
3. 检查 Local Storage 中的存储数据
4. 启用 Discord 应用的日志记录

---

## 📄 许可证

此代码为您专属定制，可自由修改使用。

---

**祝您使用愉快！** 🎉

如有任何问题，请参考 `LOGIN_GUIDE.js` 中的详细配置说明。
