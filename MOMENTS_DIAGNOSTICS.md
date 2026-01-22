# 🔍 朋友圈数据同步诊断指南

## 诊断步骤

### 问题 1: 转发不显示消息页面的角色

在浏览器 Console 中运行以下代码：

```javascript
// 检查 conversations 是否存在
const appState = momentsManager.getAppState();
console.log('AppState.conversations:', appState?.conversations);
console.log('conversations 数量:', appState?.conversations?.length);

// 检查 getFriends 返回什么
console.log('getFriends():', momentsManager.getFriends());

// 手动测试转发函数
// 点击一个朋友圈的转发按钮，应该显示 conversations 列表而不是 friends 列表
```

**预期结果**: 
- conversations 应该显示消息页面中的所有对话
- getFriends() 应该返回 AppState.friends（用于角色选择）
- 转发时的 prompt 应该显示的是 conversations 中的名字

---

### 问题 2: 用户昵称不同步

在浏览器 Console 中运行：

```javascript
// 检查 getUserName 返回什么
console.log('getUserName():', momentsManager.getUserName());

// 检查侧边栏昵称
console.log('display-name:', document.getElementById('display-name')?.textContent);
console.log('user-name:', document.querySelector('.user-name')?.textContent);

// 发送朋友圈后，检查侧边栏是否被更新
// 预期: 侧边栏的昵称应该和朋友圈页面一致
```

**预期结果**:
- getUserName() 返回侧边栏的昵称
- 发送朋友圈后，侧边栏昵称不变（因为已经是一致的）
- 如果在朋友圈页面修改昵称，侧边栏会同步更新

---

### 问题 3: 头像显示

在浏览器 Console 中运行：

```javascript
// 检查 getUserAvatar 返回什么
console.log('getUserAvatar():', momentsManager.getUserAvatar());

// 检查侧边栏头像
const cardAvatar = document.getElementById('card-avatar')?.querySelector('img');
console.log('card-avatar:', cardAvatar?.src);

// 检查发布的朋友圈的头像
const feedItems = document.querySelectorAll('.feed-item');
feedItems.forEach((item, index) => {
  const avatar = item.querySelector('.feed-avatar');
  console.log(`朋友圈 ${index}:`, avatar?.src || '无头像');
});

// 预期: 用户发的朋友圈应该有侧边栏的头像
//      角色发的朋友圈应该有角色的头像
```

**预期结果**:
- 用户发的朋友圈显示侧边栏头像
- 角色发的朋友圈显示角色头像
- 没有头像的不显示 img 标签

---

### 问题 4: AppState 同步问题

```javascript
// 检查 window.AppState 是否存在
console.log('window.AppState:', window.AppState);
console.log('window.parent.AppState:', window.parent?.AppState);

// 如果显示 undefined，说明 AppState 没有被正确加载

// 检查 localStorage 中的缓存
const cached = localStorage.getItem('cachedAppState');
console.log('localStorage.cachedAppState:', cached ? JSON.parse(cached) : null);
```

---

## 常见问题排查

### 1. 转发时显示"没有加入的角色可以转发"

**可能原因**:
- AppState.conversations 为空
- conversations 没有正确传递到 forwardMoment

**排查**:
```javascript
// 检查是否有对话
const appState = momentsManager.getAppState();
if (!appState?.conversations?.length) {
  console.error('❌ conversations 为空！');
  // 需要在消息页面添加一些角色
} else {
  console.log('✅ conversations 存在:', appState.conversations);
}
```

### 2. 转发时显示错误的好友列表

**可能原因**:
- 仍然使用 getFriends() 而不是 conversations
- 函数没有正确读取 AppState

**排查**:
```javascript
// 在转发时打印日志
// 编辑 moments.js 中的 forwardMoment 函数，在第 1501 行之后添加:
console.log('转发时的 conversations:', conversations);
```

### 3. 用户昵称不保存

**可能原因**:
- syncNameToSidebar 没有正确访问真实的 AppState
- AppState.user 不存在

**排查**:
```javascript
// 检查 AppState.user
const appState = momentsManager.getAppState();
console.log('AppState.user:', appState?.user);

// 如果为 null，说明需要初始化用户对象
if (!appState?.user) {
  console.error('❌ AppState.user 不存在！');
}
```

### 4. 头像不显示

**可能原因**:
- getUserAvatar() 返回空字符串
- 侧边栏没有头像
- 头像 URL 无效

**排查**:
```javascript
// 检查侧边栏是否有头像
const cardAvatar = document.getElementById('card-avatar');
const img = cardAvatar?.querySelector('img');
console.log('侧边栏头像 URL:', img?.src);

// 如果没有，需要在侧边栏上传或设置头像
```

---

## 修复步骤

如果上述诊断发现问题，请按以下步骤修复：

### 修复 1: 如果 conversations 为空
1. 打开消息页面
2. 点击任何对话（添加角色）
3. 确保 AppState.conversations 中有数据

### 修复 2: 如果 AppState.user 不存在
1. 查看 app.js 中的 AppState 初始化
2. 确保 AppState.user 被创建

### 修复 3: 如果侧边栏没有昵称
1. 在侧边栏修改用户昵称
2. 确保 display-name 元素有内容

### 修复 4: 如果侧边栏没有头像
1. 在侧边栏上传或设置头像
2. 确保 card-avatar 中有 img 元素

---

## 完整诊断脚本

将以下代码复制到浏览器 Console 中一次性诊断所有问题：

```javascript
(function() {
  console.log('🔍 开始诊断朋友圈数据同步...\n');
  
  // 检查 1: conversations
  const appState = momentsManager.getAppState();
  console.log('1️⃣ Conversations 检查:');
  console.log('  - 存在?', !!appState?.conversations);
  console.log('  - 数量:', appState?.conversations?.length || 0);
  console.log('  - 内容:', appState?.conversations || '❌ 空');
  
  // 检查 2: 用户昵称
  console.log('\n2️⃣ 用户昵称检查:');
  const userName = momentsManager.getUserName();
  const displayName = document.getElementById('display-name')?.textContent;
  console.log('  - getUserName():', userName);
  console.log('  - 侧边栏昵称:', displayName);
  console.log('  - 同步?', userName === displayName ? '✅' : '❌');
  
  // 检查 3: 用户头像
  console.log('\n3️⃣ 用户头像检查:');
  const userAvatar = momentsManager.getUserAvatar();
  const sidebarAvatar = document.getElementById('card-avatar')?.querySelector('img')?.src;
  console.log('  - getUserAvatar():', userAvatar || '(空)');
  console.log('  - 侧边栏头像:', sidebarAvatar || '(无)');
  console.log('  - 同步?', userAvatar === sidebarAvatar ? '✅' : '❌');
  
  // 检查 4: AppState
  console.log('\n4️⃣ AppState 检查:');
  console.log('  - window.AppState 存在?', !!window.AppState);
  console.log('  - window.parent.AppState 存在?', !!window.parent?.AppState);
  console.log('  - AppState.user 存在?', !!appState?.user);
  
  // 检查 5: 朋友圈项目
  console.log('\n5️⃣ 朋友圈项目检查:');
  const moments = document.querySelectorAll('.feed-item');
  console.log('  - 朋友圈数量:', moments.length);
  moments.forEach((item, i) => {
    const author = item.querySelector('.feed-username')?.textContent;
    const hasAvatar = !!item.querySelector('.feed-avatar');
    console.log(`  - 朋友圈 ${i+1}: "${author}", 有头像? ${hasAvatar ? '✅' : '❌'}`);
  });
  
  console.log('\n✅ 诊断完成！');
})();
```

运行这个脚本，截图结果，告诉我输出是什么，我就能快速定位问题！

