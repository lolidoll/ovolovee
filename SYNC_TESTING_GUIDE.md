# 实时同步测试指南

## 测试环境准备

1. 打开浏览器开发者工具（F12）
2. 打开 Console 标签页
3. 访问应用页面

## 测试场景 1️⃣: 消息页面 → 好友页面同步

### 步骤
1. 在 **消息页面** 看到好友列表
2. 在 **好友页面** 看到相同的好友列表
3. 预期: ✅ 两个页面显示的好友应该完全相同

### 验证方式
```javascript
// 在 Console 中运行
console.log('当前好友数:', AppState.friends.length);
AppState.friends.forEach(f => console.log(f.name));
```

## 测试场景 2️⃣: 朋友圈页面数据同步

### 步骤
1. 点击 **朋友圈** 标签
2. 查看 **"选择角色"** 下拉框
3. 预期: ✅ 应该显示所有好友（来自 AppState.friends）

### 验证方式
```javascript
// 在 Console 中运行
if (window.momentsManager) {
    console.log('朋友圈中的好友:', momentsManager.getFriends().map(f => f.name));
    console.log('朋友圈中的分组:', momentsManager.getFriendGroups().map(g => g.name));
}
```

## 测试场景 3️⃣: 好友数据实时同步

### 前提条件
- 需要能够在应用中添加或删除好友的功能
- 如果应用不支持，可以在 Console 中手动修改

### 步骤

#### 方式 A: 手动测试（通过 Console）
```javascript
// 1. 打开消息页面，记住当前好友数
console.log('初始好友数:', AppState.friends.length);

// 2. 在 Console 中添加一个好友
AppState.friends.push({
    id: 'test-friend-' + Date.now(),
    name: '测试朋友 ' + new Date().toLocaleTimeString(),
    avatar: 'https://via.placeholder.com/50',
    group: 'group_default',
    online: true
});

// 3. 等待 500ms 左右，观察 UI 变化
setTimeout(() => {
    console.log('500ms 后好友数:', AppState.friends.length);
}, 500);
```

#### 方式 B: 功能测试（如果应用支持）
1. 在好友页面添加/删除好友
2. 立即切换到消息页面
3. 立即切换回好友页面
4. 预期: ✅ 每次都看到最新数据，不应该有延迟

### 预期结果
- 添加好友后 **500ms 内** 消息页面应该显示新好友
- 删除好友后 **500ms 内** 所有页面都应该移除该好友
- 切换页面时应该总是看到最新的数据

## 测试场景 4️⃣: 分组数据同步

### 步骤
1. 在好友页面查看当前分组
2. 点击朋友圈页面
3. 查看 **"选择分组"** 下拉框
4. 预期: ✅ 应该显示相同的分组

### 验证方式
```javascript
// 在 Console 中运行
console.log('AppState 分组:', AppState.friendGroups.map(g => ({ id: g.id, name: g.name })));

// 如果在朋友圈页面
if (window.momentsManager) {
    console.log('moments 分组:', momentsManager.getFriendGroups().map(g => ({ id: g.id, name: g.name })));
}
```

## 测试场景 5️⃣: 页面切换时的数据刷新

### 步骤
1. 打开任何页面
2. 在 Console 中添加/修改好友数据
3. 点击另一个页面标签
4. 再点击第三个页面标签
5. 预期: ✅ 每个页面都应该显示最新数据

### 验证代码
```javascript
// 添加一个带时间戳的好友，便于追踪
const newFriend = {
    id: 'test-' + Date.now(),
    name: '同步测试 ' + new Date().toLocaleTimeString(),
    avatar: 'https://via.placeholder.com/50',
    group: 'group_default'
};

AppState.friends.push(newFriend);
console.log('✅ 已添加好友:', newFriend.name);
console.log('🔄 现在切换页面，应该在 500ms 内看到这个好友');
```

## 同步监听器状态检查

### 检查 setupDataSyncListener 是否正常工作

```javascript
// 方式1: 查看 Console 日志
// 当数据变化时，应该看到:
// "检测到好友数量变化，更新UI"
// "检测到分组数量变化，更新UI"

// 方式2: 手动验证
setInterval(() => {
    console.log('当前时间戳:', new Date().toLocaleTimeString(), '| 好友数:', AppState.friends.length);
}, 1000);

// 在这个输出期间修改 AppState.friends，应该看到日志输出
```

### 检查 monitorFriendsAndGroupsChanges 是否正常工作

```javascript
// 在朋友圈页面运行
// 查看 Console 中是否有类似的日志:
// "朋友圈: 检测到好友数据变化"
// "朋友圈: 检测到分组数据变化"

// 如果没看到日志，可能是 moments 页面没有正确初始化
console.log('检查 monitorFriendsAndGroupsChanges 是否运行...');
```

## 性能检查

### 检查 CPU 使用率
1. 打开 Chrome DevTools → Performance 标签
2. 点击 Record 按钮
3. 等待 5 秒钟，让轮询运行多次
4. 停止录制
5. 预期: ✅ 每 500ms 应该看到一个小的 setInterval 回调，CPU 占用应该 < 5%

### 检查内存泄漏
1. 打开 Chrome DevTools → Memory 标签
2. 拍取初始堆快照
3. 等待 30 秒钟（轮询运行 60 次）
4. 拍取第二个堆快照
5. 预期: ✅ 内存增长应该很小（< 1MB）

## 常见问题排查

### 问题 1: 在好友页面看不到好友

**可能原因**:
- AppState.friends 为空
- renderFriends() 函数有错误

**排查步骤**:
```javascript
console.log('AppState.friends 内容:', AppState.friends);
console.log('好友数量:', AppState.friends.length);

// 如果为空，查看初始化
if (AppState.friends.length === 0) {
    console.log('❌ 好友列表为空！检查 loadFromStorage() 初始化');
}
```

### 问题 2: 修改数据后没有看到 UI 更新

**可能原因**:
- setupDataSyncListener 没有启动
- renderFriends/renderConversations 函数有错误
- 页面 DOM 元素不存在

**排查步骤**:
```javascript
// 检查是否有 setInterval 在运行
console.log('Window 中的 setInterval 数量:', window);

// 手动触发渲染
if (typeof renderFriends === 'function') {
    renderFriends();
    console.log('✅ 已手动调用 renderFriends()');
}

if (typeof renderConversations === 'function') {
    renderConversations();
    console.log('✅ 已手动调用 renderConversations()');
}
```

### 问题 3: 朋友圈页面的 selectbox 为空

**可能原因**:
- initCharacterSelect 或 initGroupSelect 没有被调用
- getFriends 或 getFriendGroups 返回空数组

**排查步骤**:
```javascript
// 检查 moments 函数是否存在
console.log('initCharacterSelect 函数存在?', typeof initCharacterSelect === 'function');
console.log('initGroupSelect 函数存在?', typeof initGroupSelect === 'function');

// 手动调用初始化
if (typeof initCharacterSelect === 'function') {
    initCharacterSelect();
    console.log('✅ 已手动调用 initCharacterSelect()');
}

if (typeof initGroupSelect === 'function') {
    initGroupSelect();
    console.log('✅ 已手动调用 initGroupSelect()');
}

// 检查数据源
console.log('moments 中的好友数:', momentsManager.getFriends().length);
console.log('moments 中的分组数:', momentsManager.getFriendGroups().length);
```

## 完整测试脚本

```javascript
// 这个脚本可以一次性验证整个同步系统

(function() {
    console.log('========== 开始完整同步测试 ==========');
    
    // 1. 检查初始状态
    console.log('\n📊 初始状态:');
    console.log('  好友数:', AppState.friends.length);
    console.log('  分组数:', AppState.friendGroups.length);
    
    // 2. 添加测试好友
    const testFriendId = 'test-' + Date.now();
    const testFriend = {
        id: testFriendId,
        name: '✅ 测试好友 ' + new Date().toLocaleTimeString(),
        avatar: 'https://via.placeholder.com/50',
        group: 'group_default',
        online: true
    };
    
    AppState.friends.push(testFriend);
    console.log('\n➕ 已添加测试好友:', testFriend.name);
    
    // 3. 等待同步发生
    console.log('\n⏳ 等待 600ms 让同步机制运行...');
    
    setTimeout(() => {
        console.log('\n✅ 同步检查结果:');
        
        // 检查数据
        const hasFriend = AppState.friends.find(f => f.id === testFriendId);
        console.log('  ✅ 好友仍在 AppState 中:', !!hasFriend);
        
        // 检查 moments 中的数据
        if (typeof momentsManager !== 'undefined' && typeof momentsManager.getFriends === 'function') {
            const momentsFriends = momentsManager.getFriends();
            const hasInMoments = momentsFriends.find(f => f.id === testFriendId);
            console.log('  ✅ 好友在 moments 中:', !!hasInMoments);
        }
        
        // 清理测试数据
        AppState.friends = AppState.friends.filter(f => f.id !== testFriendId);
        console.log('\n🧹 已清理测试数据');
        
        console.log('\n========== 测试完成 ==========');
    }, 600);
})();
```

## 提交报告

测试完成后，请记录以下内容：

- ✅ / ❌ 消息页面和好友页面数据一致
- ✅ / ❌ 朋友圈页面 selectbox 显示所有好友
- ✅ / ❌ 添加好友后 500ms 内所有页面更新
- ✅ / ❌ 删除好友后 500ms 内所有页面更新
- ✅ / ❌ 切换页面时总是看到最新数据
- ✅ / ❌ 没有明显的 CPU 或内存增长

