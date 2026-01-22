# 语音条功能 - 快速参考卡

## 🎯 核心概念

**语音条** = 用户输入的文字消息，模拟真实社交软件的语音消息功能

- 🔵 **用户语音**: 蓝色气泡，右对齐
- ⚪ **AI语音**: 灰色气泡，左对齐
- 🎙️ **点击查看**: 显示转文字内容
- 📋 **复制功能**: 一键复制文字

---

## 🚀 快速开始（用户视角）

| 步骤 | 操作 |
|------|------|
| 1 | 打开任意对话 |
| 2 | 点击工具栏 🎙️ 按钮 |
| 3 | 输入文字内容 |
| 4 | 按 Ctrl+Enter 或点击发送 |
| 5 | 点击语音条查看转文字 |
| 6 | 点击复制按钮或关闭 |

---

## 💻 快速开始（开发者视角）

### 初始化
```javascript
VoiceMessageModule.init()  // 自动调用，无需手动
```

### 发送语音条
```javascript
// 用户发送（自动调用）
VoiceMessageModule.sendVoiceMessage()

// AI发送
VoiceMessageModule.sendAIVoiceMessage(convId, '文字内容')
```

### 获取数据
```javascript
// 获取语音消息列表
VoiceMessageModule.getVoiceMessagesForConversation(convId)

// 导出完整转录
VoiceMessageModule.exportVoiceTranscripts(convId)

// 获取单条内容
VoiceMessageModule.getVoiceContent(messageId)
```

### 检查和清理
```javascript
// 检查是否为语音消息
VoiceMessageModule.isVoiceMessage(msg)

// 清空所有数据
VoiceMessageModule.clearVoiceMessages()
```

---

## 📱 UI 快速导航

### 弹窗按钮
```
工具栏：[重回] [表情] [🎙️语音] [拍照] [照片] ...
```

### 弹窗内容
```
┌─────────────────────────┐
│  发送语音条          ✕ │ ← 关闭
├─────────────────────────┤
│ 输入语音内容...（6行）  │
│ 💡 输入内容会作为语音   │
│ 🔤 支持表情和特殊字符   │
├─────────────────────────┤
│  [取消]  [发送]         │
└─────────────────────────┘
```

### 语音气泡（用户）
```
用户语音条 →  ┌────────────┐
              │ 🎙️ 语音条 │
              │ ~~~~ 1秒   │ ← 蓝色气泡
              └────────────┘
              👆 点击显示转文字
```

### 语音气泡（AI）
```
← AI语音条  ┌────────────┐
            │ 语音条 🎙️ │
            │ ~~~~ 1秒   │ ← 灰色气泡
            └────────────┘
            👆 点击显示转文字
```

### 转文字面板
```
┌─────────────────────────┐
│ 语音转文字            ✕│
├─────────────────────────┤
│ 用户输入的原始文字     │
├─────────────────────────┤
│     [复制文字]          │
└─────────────────────────┘
```

---

## 🔧 API 速查表

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `init()` | 无 | 无 | 初始化模块 |
| `openVoiceModal()` | 无 | 无 | 打开输入弹窗 |
| `closeVoiceModal()` | 无 | 无 | 关闭输入弹窗 |
| `sendVoiceMessage()` | 无 | 无 | 发送用户语音 |
| `sendAIVoiceMessage(id, text)` | convId, text | 无 | AI发送语音 |
| `showVoiceTranscript(text, el)` | text, element | 无 | 显示转文字 |
| `getVoiceContent(msgId)` | messageId | string | 获取内容 |
| `isVoiceMessage(msg)` | message | boolean | 检查类型 |
| `getVoiceMessagesForConversation(id)` | convId | array | 获取列表 |
| `exportVoiceTranscripts(id)` | convId | array | 导出转录 |
| `clearVoiceMessages()` | 无 | 无 | 清空数据 |

---

## 📊 数据结构

### 语音消息对象
```javascript
{
    id: "msg_1737482400000_abc123def4",
    type: "voice",                          // ← 类型标记
    content: "用户说的内容",                // ← 文字内容
    sender: "sent",                         // ← 'sent'=用户, 'received'=AI
    senderName: "薯片机用户",
    timestamp: "2026-01-21T12:34:56.000Z",
    conversationId: "conv_xyz789"
}
```

### 转录导出格式
```javascript
[
    {
        id: "msg_...",
        sender: "sent",
        senderName: "薯片机用户",
        timestamp: "2026-01-21T...",
        voiceText: "用户说的内容"
    },
    {
        id: "msg_...",
        sender: "received",
        senderName: "AI",
        timestamp: "2026-01-21T...",
        voiceText: "AI的回复"
    }
]
```

---

## 🎨 样式类参考

| 样式类 | 用途 |
|--------|------|
| `.voice-modal` | 弹窗容器 |
| `.voice-modal-content` | 弹窗内容 |
| `.voice-input` | 输入框 |
| `.voice-bubble` | 语音气泡基础 |
| `.voice-bubble-user` | 用户气泡（蓝色） |
| `.voice-bubble-ai` | AI气泡（灰色） |
| `.voice-transcript` | 转文字面板 |
| `.voice-waveform` | 波形容器 |
| `.wave` | 波形竖条 |

---

## ⌨️ 快捷键

| 快捷键 | 功能 |
|--------|------|
| 点击 🎙️ 按钮 | 打开语音弹窗 |
| Ctrl+Enter | 快速发送（在弹窗中） |
| Esc 或 ✕ | 关闭弹窗 |
| 点击语音气泡 | 显示转文字 |
| 点击 ✕ | 隐藏转文字 |

---

## 🔐 数据持久化

- ✅ 自动保存到 AppState.messages
- ✅ 自动调用 saveToStorage()
- ✅ 刷新页面后保留
- ✅ 离线也能访问

---

## 🐛 调试技巧

```javascript
// 查看当前对话的所有消息
AppState.messages[AppState.currentChat]

// 查看语音消息列表
VoiceMessageModule.getVoiceMessagesForConversation(AppState.currentChat)

// 检查特定消息
const msg = AppState.messages[AppState.currentChat][0]
VoiceMessageModule.isVoiceMessage(msg)

// 清空测试数据
VoiceMessageModule.clearVoiceMessages()

// 重新初始化
location.reload()
```

---

## ✅ 检查清单

### 功能检查
- [ ] 能打开语音条弹窗
- [ ] 能输入文字内容
- [ ] 能发送语音条
- [ ] 语音条显示在对话中
- [ ] 能点击查看转文字
- [ ] 能复制文字内容
- [ ] AI能发送语音条
- [ ] 刷新页面后数据保存

### 样式检查
- [ ] 弹窗视觉美观
- [ ] 用户气泡是蓝色
- [ ] AI气泡是灰色
- [ ] 波形动画正常
- [ ] 转文字面板显示正确
- [ ] 按钮hover状态正常
- [ ] 移动端显示正常

### 性能检查
- [ ] 点击流畅响应
- [ ] 动画无卡顿
- [ ] 多条消息不卡
- [ ] 内存占用合理

---

## 📚 文档链接

| 文档 | 内容 |
|------|------|
| VOICE_MESSAGE_GUIDE.md | 详细API和使用示例 |
| VOICE_MESSAGE_TEST.md | 完整测试清单 |
| VOICE_MESSAGE_SUMMARY.md | 实现总结 |
| VOICE_MESSAGE_INTEGRATION.md | 集成完成报告 |

---

## 🆘 常见问题速查

**Q: 语音条按钮点不了？**  
A: 检查是否打开了对话，查看浏览器控制台

**Q: 发送后没有显示？**  
A: 检查输入框是否有内容，查看AppState是否正确

**Q: 转文字显示不了？**  
A: 点击语音气泡的波形部分，检查控制台错误

**Q: 数据没有保存？**  
A: 检查localStorage是否禁用，查看存储空间

**Q: 样式不正确？**  
A: 硬刷新（Ctrl+Shift+R），检查CSS是否加载

---

## 📞 获取帮助

1. **查阅文档** → 特定文档关键词搜索
2. **浏览器控制台** → 查看错误信息
3. **测试清单** → VOICE_MESSAGE_TEST.md
4. **代码注释** → voice-message.js 中的说明

---

## 版本信息

- **版本**: 1.0
- **发布日期**: 2026年1月21日
- **状态**: 生产就绪 ✅
- **兼容性**: Chrome/Firefox/Safari/移动浏览器

---

**快速参考卡 - 打印版本建议**

可将此卡片保存为PDF或打印，作为快速参考使用。

