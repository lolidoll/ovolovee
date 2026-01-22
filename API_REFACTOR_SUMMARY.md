# API 架构重新规划 - 改动总结

## 【核心改动】

### 1. 主API 与 心声生成
**旧架构：**
- 主API：仅生成AI回复
- 副API：单独调用生成心声（双击头像触发）
- 流程：主API完成 → 等待200ms → 副API提取心声

**新架构：**
- 主API：同时生成AI回复 + 心声数据（在系统提示词中要求）
- 心声提取：在 `appendSingleAssistantMessage()` 中自动提取
- 副API：改为通用功能API（翻译、总结等）
- 流程：主API完成 → 自动从响应中提取心声

### 2. 副API 功能重新定义
**旧功能：** 只用于心声生成  
**新功能：** 使用动态提示词支持多种功能

| 功能 | 提示词类型 | 调用函数 |
|------|----------|---------|
| 翻译为英文 | `translateEnglish` | `translateTextViaSecondaryAPI()` |
| 翻译为中文 | `translateChinese` | `translateTextViaSecondaryAPI()` |
| 自动总结 | `summarize` | `summarizeTextViaSecondaryAPI()` |
| 手动总结 | `summarize` | `summarizeContextWithAPI()` |

### 3. 新增函数

#### 核心提取函数
- `extractMindStateFromText(text)` - 从AI响应中提取心声数据

#### 副API动态提示词函数
- `callSecondaryAPIWithDynamicPrompt(content, promptType, onSuccess, onError)` - 通用副API调用
- `translateTextViaSecondaryAPI(text, targetLanguage, onSuccess, onError)` - 翻译
- `summarizeTextViaSecondaryAPI(text, onSuccess, onError)` - 总结
- `summarizeConversationViaSecondaryAPI(convId, onSuccess, onError)` - 对话总结

### 4. 修改函数

| 函数名 | 改动内容 |
|--------|---------|
| `appendSingleAssistantMessage()` | 添加心声提取和存储逻辑 |
| `handleDoubleClickAvatar()` | 移除副API调用，改为仅调用主API |
| `translateToEnglishViaAPI()` | 改用 `translateTextViaSecondaryAPI()` |
| `translateToChineseViaAPI()` | 新增并使用动态提示词 |
| `translateMessageViaSecondaryAPI()` | 改用新的翻译函数 |
| `summarizeContextWithAPI()` | 完全重写，改用新架构 |

---

## 【数据存储】

### 心声数据结构
心声数据保存在 `conversation.mindStates` 数组中：

```javascript
{
    timestamp: '2026-01-21T...',
    messageId: 'msg_...',
    affinity: 80,              // 好感度
    affinityChange: 5,         // 好感度变化
    affinityReason: '理解',    // 好感度原因
    outfit: '...',            // 穿搭
    mood: '...',              // 心情
    action: '...',            // 动作
    thought: '...',           // 心声
    badThought: '...'         // 坏心思
}
```

---

## 【兼容性保证】

### ✅ 不影响的功能
1. **消息发送接收** - 完全未改动
2. **消息编辑撤回** - 完全未改动
3. **表情包系统** - 完全未改动
4. **多选消息** - 完全未改动
5. **引用回复** - 完全未改动
6. **会话管理** - 完全未改动
7. **用户信息** - 完全未改动
8. **主题系统** - 完全未改动
9. **钱包系统** - 完全未改动
10. **通知系统** - 完全未改动

### ✅ 改进的功能
1. **心声生成** - 更快（不需要额外API调用等待）
2. **翻译功能** - 使用动态提示词，支持自定义
3. **总结功能** - 使用动态提示词，支持自定义

### ⚠️ 需要注意的地方
1. **心声生成** - 现在必须在主API系统提示词中要求 `【心声】` 格式
2. **副API配置** - 副API现在是可选的（不影响主AI交互）
3. **动态提示词** - 可在设置中自定义各功能的提示词

---

## 【测试清单】

### 主要功能测试
- [ ] 发送消息后主AI能正常回复
- [ ] 双击头像后AI回复中能提取心声
- [ ] 翻译消息功能正常工作
- [ ] 手动总结对话功能正常工作
- [ ] 自动总结功能正常工作（如配置）

### 数据持久化测试
- [ ] 心声数据被正确保存到 LocalStorage/IndexDB
- [ ] 刷新页面后心声数据能被正确加载
- [ ] 心声页面能正确显示历史记录

### 副功能测试
- [ ] 翻译为英文
- [ ] 翻译为中文
- [ ] 消息复制、删除、撤回
- [ ] 消息多选功能
- [ ] 表情包发送

### 向后兼容测试
- [ ] 旧数据能正确加载（无 mindStates 的 conversation）
- [ ] 不配置副API时主API仍正常工作
- [ ] 不配置主API时翻译/总结仍可用

---

## 【系统提示词变化】

### 主API系统提示词 (collectConversationForApi)
- ✅ 已增加心声格式要求在末尾
- ✅ 使用 `【心声】` 标记表示心声数据开始
- ✅ 格式必须为: `【心声】穿搭：... 心情：... ...`

### 副API提示词 (secondaryPrompts)
- ✅ `translateEnglish` - 翻译为英文
- ✅ `translateChinese` - 翻译为中文
- ✅ `summarize` - 总结内容
- ✅ `mindState` - 保留用于向后兼容

---

## 【API 调用流程】

### 场景1: 双击头像发送消息
```
双击头像
  ↓
callApiWithConversation() [主API]
  ↓
AI 生成回复 + 心声数据（在系统提示中要求）
  ↓
appendAssistantMessage()
  ↓
extractMindStateFromText() 提取心声
  ↓
保存消息和心声数据到 conversation
  ↓
UI 渲染（心声已提取，仅显示AI回复）
```

### 场景2: 翻译消息
```
点击翻译按钮
  ↓
translateMessageViaSecondaryAPI()
  ↓
translateTextViaSecondaryAPI() [副API，动态提示词]
  ↓
callSecondaryAPIWithDynamicPrompt(..., 'translateEnglish' or 'translateChinese')
  ↓
翻译结果保存到消息对象
  ↓
UI 渲染翻译结果
```

### 场景3: 手动总结
```
点击手动总结
  ↓
summarizeContextWithAPI()
  ↓
summarizeTextViaSecondaryAPI() [副API，动态提示词]
  ↓
callSecondaryAPIWithDynamicPrompt(..., 'summarize')
  ↓
总结结果保存到 conversation.summaries
  ↓
UI 显示总结
```

---

## 【配置说明】

### 主API 配置（必需）
- 端点 (endpoint)
- API 密钥 (apiKey)
- 模型 (selectedModel)
- 系统提示词 (prompts)

### 副API 配置（可选）
- 端点 (secondaryEndpoint) - 可选
- API 密钥 (secondaryApiKey) - 可选
- 模型 (secondarySelectedModel) - 可选
- 动态提示词 (secondaryPrompts) - 可选，有预设

### 心声设置
- 不再需要单独配置，由主API系统提示词控制

---

## 【故障排查】

### 问题：心声无法提取
**可能原因：**
- 主API系统提示词中缺少 `【心声】` 格式要求
- AI 回复格式与期望不符
- extractMindStateFromText() 提取逻辑需调整

**解决方案：**
1. 检查系统提示词是否包含心声格式要求
2. 查看浏览器控制台日志，查找 "未找到【心声】标记" 的提示
3. 根据 AI 实际返回格式调整正则表达式

### 问题：翻译/总结无法工作
**可能原因：**
- 副API 未配置
- 副API 提示词为空或不正确
- 副API 请求超时

**解决方案：**
1. 检查副API端点、密钥、模型是否正确配置
2. 尝试手动修改副API提示词
3. 查看浏览器控制台日志，检查副API请求错误

### 问题：数据无法保存
**可能原因：**
- IndexDB 或 LocalStorage 已满
- 浏览器隐私模式
- 存储权限被拒绝

**解决方案：**
1. 清除浏览器缓存
2. 检查浏览器设置
3. 尝试切换到非隐私模式

---

## 【开发建议】

### 如何自定义提示词
副API 提示词可在代码中修改：
```javascript
AppState.apiSettings.secondaryPrompts.summarize = "你的自定义总结提示词";
```

### 如何添加新的副API功能
1. 在 `secondaryPrompts` 中添加新的提示词
2. 创建新的函数调用 `callSecondaryAPIWithDynamicPrompt(..., '新功能')`
3. 在需要的地方调用新函数

### 如何调整心声提取逻辑
修改 `extractMindStateFromText()` 函数中的正则表达式匹配逻辑。

---

## 【总结】

✅ **新架构的优势：**
1. 心声生成更快（不需要等待副API）
2. 副API 功能更灵活（支持多种功能）
3. 系统提示词可动态配置
4. 不影响其他功能的正常运行
5. 向后兼容旧数据

⚠️ **需要注意：**
1. 主API 系统提示词必须包含心声格式要求
2. 副API 现在用于通用功能而非心声生成
3. 旧的 `generateCharacterMindStateViaSecondaryAPI` 函数已不再使用

