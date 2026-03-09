# 技术规格文档

## 1. 技术栈选型

### 1.1 前端技术栈

| 技术 | 选型 | 说明 |
|-----|------|------|
| 开发框架 | 微信小程序原生 | 官方支持，性能最优 |
| 备选方案 | uni-app | 跨平台，可发布到多端 |
| UI 组件库 | Vant Weapp | 有赞出品，组件丰富 |
| 状态管理 | 小程序全局数据 + 本地存储 | 轻量级，满足需求 |
| 构建工具 | 微信开发者工具 | 官方 IDE |

### 1.2 后端技术栈

| 技术 | 选型 | 说明 |
|-----|------|------|
| 云开发 | 微信云开发 | 免运维，与小程序深度集成 |
| 数据库 | 云数据库 (MongoDB) | 文档型，灵活扩展 |
| 云函数 | Node.js | 处理业务逻辑 |
| 存储 | 云存储 | 存放物品图片 |
| 消息推送 | 微信模板消息 | 过期提醒推送 |

### 1.3 技术选型理由

**选择微信云开发的理由：**
- ✅ 免运维，降低开发和运营成本
- ✅ 与微信小程序深度集成，开发效率高
- ✅ 免费额度足够初期使用
- ✅ 自动扩缩容，应对用户增长
- ✅ 内置数据库、存储、云函数，一站式解决方案

**备选方案：**
- 如需独立部署：Node.js + Express + MongoDB + 阿里云/腾讯云

---

## 2. 系统架构

### 2.1 整体架构

```
┌─────────────────────────────────────────────────────────┐
│                    微信小程序客户端                        │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │ 物品管理 │ │ 分类管理 │ │ 过期提醒 │ │ 统计分析 │       │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘       │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                    微信云开发平台                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   云函数     │  │  云数据库    │  │   云存储     │     │
│  │ (业务逻辑)   │  │ (数据存储)   │  │ (图片存储)   │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│  ┌─────────────┐  ┌─────────────┐                       │
│  │  模板消息    │  │   云调用     │                       │
│  │ (消息推送)   │  │ (微信 API)   │                       │
│  └─────────────┘  └─────────────┘                       │
└─────────────────────────────────────────────────────────┘
```

### 2.2 目录结构

```
family-inventory-miniprogram/
├── cloud/                      # 云函数目录
│   ├── getItems/              # 获取物品列表
│   ├── addItem/               # 添加物品
│   ├── updateItem/            # 更新物品
│   ├── deleteItem/            # 删除物品
│   ├── checkExpire/           # 检查过期（定时触发）
│   └── sendReminder/          # 发送提醒
├── miniprogram/               # 小程序目录
│   ├── pages/                 # 页面目录
│   │   ├── index/             # 首页
│   │   ├── item-list/         # 物品列表
│   │   ├── item-detail/       # 物品详情
│   │   ├── item-add/          # 添加物品
│   │   ├── category/          # 分类管理
│   │   ├── location/          # 位置管理
│   │   ├── reminder/          # 提醒中心
│   │   ├── search/            # 搜索页面
│   │   └── stats/             # 统计页面
│   ├── components/            # 组件目录
│   │   ├── item-card/         # 物品卡片组件
│   │   ├── category-picker/   # 分类选择器
│   │   ├── location-picker/   # 位置选择器
│   │   └── date-picker/       # 日期选择器
│   ├── utils/                 # 工具函数
│   │   ├── api.js             # API 封装
│   │   ├── util.js            # 通用工具
│   │   └── constants.js       # 常量配置
│   ├── images/                # 图片资源
│   ├── styles/                # 样式文件
│   ├── app.js                 # 小程序入口
│   ├── app.json               # 小程序配置
│   └── project.config.json    # 项目配置
├── README.md
└── package.json
```

---

## 3. 数据库设计

### 3.1 集合设计

#### 3.1.1 items 集合
```javascript
{
  _id: ObjectId,
  _openid: string,              // 用户 openid（云开发自动索引）
  name: string,                 // 物品名称
  categoryId: string,           // 分类 ID
  categoryName: string,         // 分类名称（冗余，方便查询）
  quantity: number,             // 数量
  unit: string,                 // 单位
  locationId: string,           // 位置 ID
  locationPath: string,         // 位置路径（冗余）
  purchaseDate: Date,           // 购买日期
  expireDate: Date,             // 过期日期
  imageUrl: string,             // 图片 URL
  remark: string,               // 备注
  isExpired: boolean,           // 是否已过期
  isWarning: boolean,           // 是否即将过期
  createdAt: Date,
  updatedAt: Date
}
```

#### 3.1.2 categories 集合
```javascript
{
  _id: ObjectId,
  _openid: string,              // 用户 openid（null 为系统分类）
  name: string,                 // 分类名称
  icon: string,                 // 图标
  parentId: string,             // 父分类 ID
  sort: number,                 // 排序
  isSystem: boolean,            // 是否系统分类
  createdAt: Date
}
```

#### 3.1.3 locations 集合
```javascript
{
  _id: ObjectId,
  _openid: string,              // 用户 openid（null 为系统位置）
  name: string,                 // 位置名称
  path: string,                 // 完整路径
  parentId: string,             // 父位置 ID
  level: number,                // 层级
  sort: number,                 // 排序
  isSystem: boolean,            // 是否系统位置
  createdAt: Date
}
```

#### 3.1.4 reminders 集合
```javascript
{
  _id: ObjectId,
  _openid: string,
  itemId: string,               // 物品 ID
  itemName: string,             // 物品名称（冗余）
  expireDate: Date,             // 过期日期
  remindDate: Date,             // 提醒日期
  remindType: string,           // 提醒类型：warning/expired
  daysLeft: number,             // 剩余天数
  isRead: boolean,              // 是否已读
  isHandled: boolean,           // 是否已处理
  createdAt: Date
}
```

#### 3.1.5 consumption_records 集合
```javascript
{
  _id: ObjectId,
  _openid: string,
  itemId: string,
  itemName: string,
  quantity: number,
  date: Date,
  remark: string,
  createdAt: Date
}
```

### 3.2 索引设计

```javascript
// items 集合索引
db.items.createIndex({ _openid: 1, categoryId: 1 })
db.items.createIndex({ _openid: 1, locationId: 1 })
db.items.createIndex({ _openid: 1, expireDate: 1 })
db.items.createIndex({ _openid: 1, isExpired: 1 })
db.items.createIndex({ _openid: 1, name: 'text' })

// reminders 集合索引
db.reminders.createIndex({ _openid: 1, isRead: 1 })
db.reminders.createIndex({ _openid: 1, remindDate: 1 })
```

---

## 4. 云函数设计

### 4.1 云函数列表

| 函数名 | 功能 | 触发方式 |
|-------|------|---------|
| getItems | 获取物品列表 | HTTP 调用 |
| addItem | 添加物品 | HTTP 调用 |
| updateItem | 更新物品 | HTTP 调用 |
| deleteItem | 删除物品 | HTTP 调用 |
| getCategories | 获取分类列表 | HTTP 调用 |
| addCategory | 添加分类 | HTTP 调用 |
| getLocations | 获取位置列表 | HTTP 调用 |
| addLocation | 添加位置 | HTTP 调用 |
| checkExpire | 检查过期物品 | 定时触发（每日） |
| sendReminder | 发送提醒消息 | HTTP 调用/定时触发 |
| getStats | 获取统计数据 | HTTP 调用 |

### 4.2 云函数示例：checkExpire

```javascript
// cloud/checkExpire/index.js
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });
const db = cloud.database();
const _ = db.command;

exports.main = async (event, context) => {
  const { OPENID } = cloud.getWXContext();
  
  // 获取所有用户
  const users = await db.collection('users').get();
  
  for (const user of users.data) {
    const openid = user._openid;
    const today = new Date();
    const warningDate = new Date();
    warningDate.setDate(today.getDate() + 7);
    
    // 查找即将过期的物品（7 天内）
    const warningItems = await db.collection('items')
      .where({
        _openid: openid,
        expireDate: _.lte(warningDate).gt(today),
        isWarning: false
      })
      .get();
    
    // 查找已过期的物品
    const expiredItems = await db.collection('items')
      .where({
        _openid: openid,
        expireDate: _.lte(today),
        isExpired: false
      })
      .get();
    
    // 更新物品状态并创建提醒
    for (const item of warningItems.data) {
      await db.collection('items').doc(item._id).update({
        data: { isWarning: true }
      });
      await db.collection('reminders').add({
        data: {
          _openid: openid,
          itemId: item._id,
          itemName: item.name,
          expireDate: item.expireDate,
          remindDate: today,
          remindType: 'warning',
          daysLeft: Math.ceil((item.expireDate - today) / 86400000),
          isRead: false,
          isHandled: false
        }
      });
    }
    
    for (const item of expiredItems.data) {
      await db.collection('items').doc(item._id).update({
        data: { isExpired: true }
      });
      await db.collection('reminders').add({
        data: {
          _openid: openid,
          itemId: item._id,
          itemName: item.name,
          expireDate: item.expireDate,
          remindDate: today,
          remindType: 'expired',
          daysLeft: 0,
          isRead: false,
          isHandled: false
        }
      });
    }
  }
  
  return { success: true };
};
```

---

## 5. 定时任务

### 5.1 过期检查任务

```javascript
// config.json (云函数配置)
{
  "triggers": [
    {
      "name": "checkExpireTimer",
      "type": "timer",
      "config": "0 0 9 * * * *"  // 每天上午 9 点执行
    }
  ]
}
```

### 5.2 提醒推送任务

```javascript
// config.json
{
  "triggers": [
    {
      "name": "sendReminderTimer",
      "type": "timer",
      "config": "0 30 9 * * * *"  // 每天上午 9:30 执行
    }
  ]
}
```

---

## 6. 接口设计

### 6.1 物品相关接口

| 接口 | 方法 | 参数 | 返回 |
|-----|------|-----|-----|
| /api/items | GET | page, pageSize, categoryId, locationId, keyword | items, total |
| /api/items | POST | name, categoryId, quantity, locationId, expireDate... | item |
| /api/items/:id | GET | - | item |
| /api/items/:id | PUT | 更新字段 | item |
| /api/items/:id | DELETE | - | success |

### 6.2 分类相关接口

| 接口 | 方法 | 参数 | 返回 |
|-----|------|-----|-----|
| /api/categories | GET | - | categories |
| /api/categories | POST | name, icon, parentId | category |
| /api/categories/:id | DELETE | - | success |

### 6.3 位置相关接口

| 接口 | 方法 | 参数 | 返回 |
|-----|------|-----|-----|
| /api/locations | GET | - | locations |
| /api/locations | POST | name, parentId | location |
| /api/locations/:id | DELETE | - | success |

### 6.4 提醒相关接口

| 接口 | 方法 | 参数 | 返回 |
|-----|------|-----|-----|
| /api/reminders | GET | isRead | reminders |
| /api/reminders/:id/read | PUT | - | success |
| /api/reminders/:id/handle | PUT | handleType | success |

---

## 7. 安全设计

### 7.1 数据权限
- 云数据库规则：用户只能访问自己的数据（通过_openid 隔离）
- 云函数中验证用户身份

### 7.2 数据安全
- 敏感数据加密存储
- 图片存储使用云存储，设置访问权限
- 接口调用频率限制

### 7.3 家庭共享权限
- 邀请码机制
- 权限分级（管理员/普通成员）
- 数据变更日志

---

## 8. 性能优化

### 8.1 前端优化
- 分页加载，避免一次性加载大量数据
- 图片懒加载
- 本地缓存常用数据
- 使用虚拟列表优化长列表

### 8.2 后端优化
- 数据库索引优化
- 云函数冷启动优化
- 数据冗余减少关联查询

### 8.3 缓存策略
- 分类和位置数据本地缓存
- 首页数据预加载
- 搜索关键词缓存

---

## 9. 开发环境

### 9.1 开发工具
- 微信开发者工具（最新版）
- Node.js >= 14.0.0
- npm >= 6.0.0

### 9.2 开发流程
1. 克隆项目
2. 安装依赖：`npm install`
3. 配置云开发环境
4. 本地调试
5. 上传云函数
6. 真机测试
7. 提交审核

### 9.3 代码规范
- ESLint + Prettier
- 遵循微信小程序开发规范
- 提交信息规范

---

## 10. 部署与发布

### 10.1 测试环境
- 云开发环境：test
- 测试账号准备

### 10.2 生产环境
- 云开发环境：prod
- 小程序 AppID 配置
- 模板消息配置

### 10.3 发布流程
1. 代码审核
2. 上传小程序代码
3. 提交微信审核
4. 审核通过后发布

---

## 11. 监控与日志

### 11.1 云函数日志
- 启用云函数日志
- 错误告警配置

### 11.2 小程序统计
- 启用微信小程序统计
- 关键事件埋点

### 11.3 异常监控
- 云函数错误监控
- 小程序异常捕获
