# 快速问题排查清单

## 问题-原因-解决方案 快速对照表

### 问题1：转发朋友圈时说"没有好友可以转发"
```
原因：AppState.friends 为空数组
解决方案：✅ 已在app.js中添加初始化示例好友
预期结果：打开朋友圈转发时应该显示5个好友
```

### 问题2：好友分组不一致
```
原因：
  - 好友页面：AppState.friendGroups 只有1个默认分组
  - 发朋友圈页面：降级显示默认的2个分组
解决方案：✅ 已在app.js中添加4个分组初始化
预期结果：两个地方都显示相同的4个分组
```

### 问题3：所有头像都是空的/默认的
```
原因：
  - 优先级错误：先查AppState.user.avatar（为空）
  - 才降级到侧边栏的真实头像
解决方案：✅ 修改了getUserAvatar()的优先级
新逻辑：侧边栏头像 > profileAvatar > AppState > 默认值
预期结果：朋友圈头像总是显示侧边栏实际头像
```

### 问题4：用户名不是侧边栏的昵称
```
原因：
  - initProfileData()优先从AppState读（AppState.user.name为默认值）
  - 没有及时从侧边栏同步最新的昵称
解决方案：✅ 重写了initProfileData()和getUserName()
新逻辑：侧边栏name > profileName > AppState > 默认值
预期结果：打开朋友圈时显示侧边栏实际昵称
```

## 快速验证步骤（2分钟）

### 验证1：好友数据
```javascript
// 在浏览器console执行：
console.log('好友数量:', window.AppState?.friends?.length);
console.log('分组数量:', window.AppState?.friendGroups?.length);
console.log('好友详情:', window.AppState?.friends);
console.log('分组详情:', window.AppState?.friendGroups);

// 预期结果：
// 好友数量: 5
// 分组数量: 4
```

### 验证2：转发功能
```
1. 打开朋友圈页面
2. 发布一条朋友圈
3. 点击转发按钮
4. 应该看到弹出框显示5个好友
```

### 验证3：头像同步
```
1. 在侧边栏点击头像，上传新头像
2. 打开朋友圈页面
3. 查看：
   - 个人资料卡片的头像是否更新 ✓
   - 评论框的头像是否更新 ✓
```

### 验证4：用户名同步
```
1. 在侧边栏点击用户名，修改昵称
2. 打开朋友圈页面
3. 验证个人资料中显示的是新的昵称 ✓
```

## 调试信息查看

### 如何查看系统日志
```
1. 打开浏览器 F12 → Console标签
2. 查看以下日志：
   - "已初始化示例好友数据" → 好友初始化成功
   - "已初始化示例好友分组" → 分组初始化成功
   - "renderMoments出错" → 朋友圈渲染有错
   - 其他错误信息 → 数据加载问题
```

### 如何清除缓存重新初始化
```javascript
// 在浏览器console执行：
localStorage.clear();  // 清除localStorage
// 或者：
localStorage.removeItem('shupianjAppState');  // 只清除应用数据
// 然后刷新页面
location.reload();
```

## 修复前后对比

| 功能 | 修复前 | 修复后 |
|------|-------|-------|
| 好友列表 | ❌ 空列表 | ✅ 5个示例好友 |
| 好友分组 | ❌ 只有默认分组 | ✅ 4个分组 |
| 转发功能 | ❌ 无好友可选 | ✅ 显示5个好友 |
| 头像同步 | ❌ 显示默认头像 | ✅ 显示侧边栏头像 |
| 用户名同步 | ❌ 显示默认用户名 | ✅ 显示侧边栏昵称 |
| 朋友圈角色 | ❌ 无角色可选 | ✅ 显示5个好友作为角色 |

## 常见问题回答

### Q1: 为什么初始化数据后还是看不到好友？
```
A: 可能需要清除缓存重新加载
  1. F12 → Application → Clear site data
  2. 或执行 localStorage.clear()
  3. 刷新页面 F5
```

### Q2: 头像为什么还是默认的？
```
A: 检查以下问题：
  1. 侧边栏是否真的上传了新头像？
  2. 打开朋友圈时是否有错误？（查看console）
  3. 是否清除了缓存？（执行localStorage.clear()）
```

### Q3: 分组为什么在好友页面显示但发朋友圈不显示？
```
A: 这个问题已修复
  - 之前：好友页面用AppState.friendGroups
         发朋友圈用硬编码的默认2个分组
  - 现在：都使用AppState.friendGroups（4个分组）
```

### Q4: 修改昵称后为什么朋友圈没有同步？
```
A: 需要重新打开朋友圈页面
  - initProfileData()在页面加载时运行一次
  - 如果已打开朋友圈，改昵称后需要刷新朋友圈页面
  - monitorAvatarChanges()每2秒会监听一次（用于头像）
```

## 修复检查清单

- [x] app.js: 添加示例好友初始化（~5个好友）
- [x] app.js: 添加示例分组初始化（~4个分组）
- [x] moments.js: 修改getUserAvatar()优先级
- [x] moments.js: 修改getUserName()优先级
- [x] moments.js: 重写initProfileData()
- [x] 所有修改通过语法检查
- [ ] 在浏览器中验证功能（待用户确认）
- [ ] 测试所有数据同步场景（待用户确认）

---

**最后更新**：2026-01-22
**修复状态**：✅ 代码级别完成，⏳ 等待测试验证
