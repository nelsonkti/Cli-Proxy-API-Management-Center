# Cli-Proxy-API-Management-Center - 项目架构与知识体系

> 本文档是 CLIProxyAPI 前端管理中心的完整知识库，涵盖技术栈、项目结构、页面路由、组件体系、状态管理、API 集成、样式系统等方面，可作为后续开发和维护的参考手册。

---

## 目录

- [1. 项目概述](#1-项目概述)
- [2. 技术栈](#2-技术栈)
- [3. 项目结构](#3-项目结构)
- [4. 路由与页面](#4-路由与页面)
- [5. 组件体系](#5-组件体系)
- [6. 状态管理](#6-状态管理)
- [7. API 集成层](#7-api-集成层)
- [8. 认证体系](#8-认证体系)
- [9. 样式系统](#9-样式系统)
- [10. 国际化](#10-国际化)
- [11. 构建与部署](#11-构建与部署)
- [12. 与后端接口对照](#12-与后端接口对照)
- [13. 关键文件索引](#13-关键文件索引)

---

## 1. 项目概述

**Cli-Proxy-API-Management-Center** 是 CLIProxyAPI 后端网关的 Web 管理界面（Management Center），以 React SPA 形式提供可视化操作面板，用于管理 AI 代理网关的配置、认证、配额、插件、日志等所有功能。

### 核心能力

| 能力 | 说明 |
|------|------|
| **仪表盘** | 概览所有 AI 提供商状态、用量统计 |
| **AI 提供商管理** | 管理 Gemini/Claude/Codex/OpenAI/Vertex 等 API Key |
| **认证文件管理** | OAuth Token 文件上传/下载/启禁/模型关联 |
| **OAuth 授权** | 可视化发起各提供商 OAuth 授权流程 |
| **配额管理** | 查看/重置各提供商配额 |
| **配置编辑** | YAML 源码编辑 + 可视化分块编辑 + Diff 对比 |
| **日志查看** | 请求日志、错误日志查询 |
| **插件管理** | 插件列表/安装/配置/商店 |
| **模型别名映射** | 可视化模型映射关系图 |
| **系统信息** | 服务器版本、运行状态 |
| **多主题** | auto/light/white/dark 四种主题 |
| **多语言** | zh-CN / zh-TW / en / ru 四种语言 |

### 部署特点

整个前端应用构建为**单个 HTML 文件**（`management.html`），所有 JS/CSS/资源全部内联，由后端 Go 服务器直接提供服务，无需独立部署。

---

## 2. 技术栈

### 核心框架

| 类别 | 技术 | 版本 | 说明 |
|------|------|------|------|
| **框架** | React | ^19.2.1 | 最新 React 19 |
| **语言** | TypeScript | ^6.0.3 | 严格类型检查 |
| **构建工具** | Vite 8 | ^8.0.10 | Rolldown 引擎 |
| **包管理** | Bun | 1.3.14 | 替代 npm/yarn |
| **路由** | react-router-dom | ^7.10.1 | Hash Router |
| **状态管理** | Zustand | ^5.0.9 | 轻量级状态管理 |
| **HTTP 客户端** | Axios | 1.15.2 | API 请求层 |
| **动画** | Motion (Framer Motion) | ^12.34.3 | 页面转场动画 |
| **代码编辑器** | CodeMirror 6 (@uiw/react-codemirror) | ^4.25.3 | YAML 编辑器 |
| **国际化** | i18next + react-i18next | ^26.2.0 / ^17.0.8 | 多语言支持 |
| **YAML** | yaml + @codemirror/lang-yaml | - | YAML 解析与编辑 |
| **Lint** | ESLint 9 + typescript-eslint | - | 代码规范 |
| **格式化** | Prettier 3 | ^3.7.4 | 代码格式化 |

### 构建特性

- **单文件输出**: `vite-plugin-singlefile` 将整个应用打包为一个 HTML 文件
- **无代码拆分**: `codeSplitting: false`，`assetsInlineLimit: 100000000`
- **Rolldown 引擎**: Vite 8 的新一代打包引擎
- **版本注入**: `__APP_VERSION__` 全局变量在构建时注入
- **路径别名**: `@` → `./src`

---

## 3. 项目结构

```
Cli-Proxy-API-Management-Center/
├── .github/workflows/release.yml     # CI/CD: tag 触发构建 & GitHub Release
├── package.json                       # 依赖、脚本
├── tsconfig.json                      # TS 基础配置
├── tsconfig.app.json                  # TS 应用配置 (ES2022, bundler)
├── tsconfig.node.json                 # TS Node 配置
├── vite.config.ts                     # Vite 构建配置 (77行)
├── eslint.config.js                   # ESLint 扁平配置
├── .prettierrc                        # Prettier 配置
├── index.html                         # 入口 HTML
│
└── src/
    ├── main.tsx                        # 应用入口
    ├── App.tsx                         # 根组件，路由注册
    ├── vite-env.d.ts                   # Vite 环境类型声明
    │
    ├── assets/                         # 静态资源
    │   ├── logoInline.ts               # Logo base64 内联
    │   └── icons/                      # 提供商品牌 SVG/PNG 图标 (18个)
    │       ├── antigravity.svg, claude.svg, codex.svg, deepseek.svg,
    │       ├── gemini.svg, glm.svg, grok.svg, kimi.svg, minimax.svg,
    │       ├── openai.svg, qwen.svg, vertex.svg, apikey-fun.png, ...
    │
    ├── components/                     # 可复用组件
    │   ├── common/                     # 全局共享组件
    │   │   ├── ConfirmationModal.tsx   # 全局确认弹窗
    │   │   ├── NotificationContainer.tsx # Toast 通知容器
    │   │   ├── PageTransition.tsx      # 页面转场动画
    │   │   ├── SecondaryScreenShell.tsx # 二级页面外壳
    │   │   └── SplashScreen.tsx        # 启动画面
    │   │
    │   ├── config/                     # 配置编辑器
    │   │   ├── ConfigSection.tsx       # 配置区块
    │   │   ├── ConfigSourceEditor.tsx  # CodeMirror YAML 编辑器
    │   │   ├── DiffModal.tsx           # 配置 Diff 对比弹窗
    │   │   ├── VisualConfigEditor.tsx  # 可视化配置编辑器
    │   │   ├── VisualConfigEditorBlocks.tsx # 配置块组件
    │   │   └── configSearchIndex.ts    # 配置搜索索引
    │   │
    │   ├── layout/
    │   │   └── MainLayout.tsx          # 主布局 (侧边栏 + 头部 + 内容)
    │   │
    │   ├── modelAlias/                 # 模型别名映射图
    │   │   ├── ModelMappingDiagram.tsx # 映射图主组件
    │   │   ├── ModelMappingDiagramColumns.tsx
    │   │   ├── ModelMappingDiagramContextMenu.tsx
    │   │   ├── ModelMappingDiagramModals.tsx
    │   │   └── ModelMappingDiagramTypes.ts
    │   │
    │   ├── providers/                  # 提供商共享组件
    │   │   ├── ProviderStatusBar.tsx   # 提供商状态条
    │   │   ├── hooks/useProviderRecentRequests.ts
    │   │   └── utils.ts
    │   │
    │   ├── quota/                      # 配额展示组件
    │   │   ├── QuotaCard.tsx           # 配额卡片
    │   │   ├── QuotaSection.tsx        # 配额区块
    │   │   └── quotaConfigs.ts         # 配额配置定义
    │   │
    │   └── ui/                         # 基础 UI 组件（设计系统）
    │       ├── AutocompleteInput.tsx   # 自动完成输入框
    │       ├── Button.tsx              # 按钮（多变体、多尺寸、loading）
    │       ├── Card.tsx                # 卡片容器
    │       ├── Collapsible/            # 折叠/展开组件
    │       ├── EmptyState.tsx          # 空状态占位
    │       ├── Input.tsx               # 输入框（label, hint, right element）
    │       ├── LoadingSpinner.tsx      # 加载动画
    │       ├── Modal.tsx               # 模态弹窗
    │       ├── Select.tsx              # 下拉选择
    │       ├── SelectionCheckbox.tsx   # 选择复选框
    │       ├── Sheet/                  # 侧滑面板
    │       ├── Skeleton/               # 骨架屏
    │       ├── Table/                  # 数据表格
    │       ├── ToggleSwitch.tsx        # 开关切换
    │       ├── icons.tsx               # 30+ SVG 图标组件
    │       └── scrollLock.ts           # 弹窗滚动锁定
    │
    ├── features/                       # 功能模块（领域组织）
    │   ├── authFiles/                  # 认证文件管理
    │   │   ├── components/             # AuthFileCard, AuthFileModelsModal 等
    │   │   ├── hooks/                  # useAuthFilesData, useAuthFilesModels
    │   │   ├── constants.ts, uiState.ts
    │   │
    │   ├── plugins/                    # 插件系统
    │   │   ├── PluginsPage.tsx         # 插件管理页
    │   │   ├── PluginStorePage.tsx     # 插件商店页
    │   │   ├── PluginResourcePage.tsx  # 插件资源页
    │   │   └── pluginPolling.ts        # 插件状态轮询
    │   │
    │   └── providers/                  # AI 提供商管理
    │       ├── components/             # ProviderCategoryList, ProviderHeaderCard
    │       ├── sheets/                 # ProviderSheet, BaseProviderForm
    │       ├── adapters.ts             # 数据适配器
    │       ├── brandLogos.ts           # 品牌 Logo 映射
    │       ├── descriptors.ts          # 提供商描述器
    │       └── useProviderWorkbench.ts # 工作台 Hook
    │
    ├── hooks/                          # 全局自定义 Hooks
    │   ├── useActionBarHeightVar.ts    # 操作栏高度 CSS 变量
    │   ├── useApiKeysForModels.ts      # 模型-API Key 关联
    │   ├── useEdgeSwipeBack.ts         # 边缘滑动返回手势
    │   ├── useHeaderRefresh.ts         # 头部刷新按钮
    │   ├── useInterval.ts             # 定时器 Hook
    │   ├── useLocalStorage.ts         # localStorage Hook
    │   ├── useMediaQuery.ts           # 媒体查询 Hook
    │   ├── useUnsavedChangesGuard.ts  # 未保存变更守卫
    │   └── useVisualConfig.ts         # 可视化配置 Hook
    │
    ├── i18n/                           # 国际化
    │   ├── index.ts                    # i18next 配置
    │   └── locales/
    │       ├── zh-CN.json              # 简体中文（默认/回退）
    │       ├── zh-TW.json              # 繁体中文
    │       ├── en.json                 # 英文
    │       └── ru.json                 # 俄语
    │
    ├── pages/                          # 页面组件
    │   ├── DashboardPage.tsx           # 仪表盘
    │   ├── LoginPage.tsx               # 登录页
    │   ├── ConfigPage.tsx              # 配置管理
    │   ├── AuthFilesPage.tsx           # 认证文件管理
    │   ├── AuthFilesOAuthExcludedEditPage.tsx   # OAuth 排除模型编辑
    │   ├── AuthFilesOAuthModelAliasEditPage.tsx # OAuth 模型别名编辑
    │   ├── OAuthPage.tsx               # OAuth 授权
    │   ├── QuotaPage.tsx               # 配额管理
    │   ├── LogsPage.tsx                # 日志查看
    │   ├── SystemPage.tsx              # 系统信息
    │   ├── PlaceholderPage.tsx         # 占位页面
    │   └── hooks/                      # 页面级 Hooks
    │       ├── logParsing.ts           # 日志解析
    │       ├── logTypes.ts             # 日志类型定义
    │       ├── useLogFilters.ts        # 日志过滤器
    │       └── useLogScroller.ts       # 日志滚动控制
    │
    ├── router/                         # 路由配置
    │   ├── MainRoutes.tsx              # 路由定义
    │   └── ProtectedRoute.tsx          # 认证守卫组件
    │
    ├── services/                       # 服务层
    │   ├── api/                        # API 客户端 + 端点模块
    │   │   ├── client.ts               # Axios ApiClient 单例
    │   │   ├── config.ts               # 配置 API
    │   │   ├── configFile.ts           # 配置文件 API
    │   │   ├── providers.ts            # 提供商 API (525行)
    │   │   ├── authFiles.ts            # 认证文件 API (487行)
    │   │   ├── logs.ts                 # 日志 API (242行)
    │   │   ├── models.ts               # 模型 API
    │   │   ├── oauth.ts                # OAuth API
    │   │   ├── plugins.ts              # 插件 API
    │   │   ├── apiKeyUsage.ts          # 用量统计 API
    │   │   ├── apiKeys.ts              # API Key 管理
    │   │   ├── antigravitySubscription.ts # Antigravity 订阅
    │   │   ├── apiCall.ts              # API 测试调用
    │   │   ├── version.ts              # 版本检测
    │   │   ├── vertex.ts               # Vertex AI API
    │   │   └── transformers.ts         # 响应数据转换
    │   └── storage/
    │       └── secureStorage.ts        # 混淆 localStorage 封装
    │
    ├── stores/                         # Zustand 状态仓库
    │   ├── useAuthStore.ts             # 认证状态 (289行)
    │   ├── useConfigStore.ts           # 配置缓存 (289行)
    │   ├── useThemeStore.ts            # 主题状态 (114行)
    │   ├── useLanguageStore.ts         # 语言状态 (55行)
    │   ├── useNotificationStore.ts     # 通知状态 (106行)
    │   ├── useModelsStore.ts           # 模型缓存 (85行)
    │   └── useQuotaStore.ts            # 配额缓存 (71行)
    │
    ├── styles/                         # 全局 SCSS 架构
    │   ├── variables.scss              # 设计令牌 (颜色/间距/圆角/阴影/断点)
    │   ├── themes.scss                 # 主题 CSS 变量 (light/dark/white)
    │   ├── mixins.scss                 # SCSS Mixins
    │   ├── reset.scss                  # CSS Reset
    │   ├── global.scss                 # 全局样式、滚动条、工具类
    │   ├── layout.scss                 # 应用外壳布局
    │   └── components.scss             # 共享组件样式 (.btn, .card, .modal)
    │
    ├── types/                          # TypeScript 类型定义
    │   ├── api.ts                      # API 响应类型
    │   ├── auth.ts                     # 认证类型
    │   ├── authFile.ts                 # 认证文件类型
    │   ├── common.ts                   # 通用类型
    │   ├── config.ts                   # 配置类型
    │   ├── log.ts                      # 日志类型
    │   ├── oauth.ts                    # OAuth 类型
    │   ├── plugin.ts                   # 插件类型
    │   ├── provider.ts                 # 提供商类型
    │   ├── quota.ts                    # 配额类型
    │   ├── visualConfig.ts             # 可视化配置类型
    │   └── style.d.ts                  # SCSS 模块类型声明
    │
    └── utils/                          # 工具函数
        ├── authIndex.ts                # 认证索引
        ├── clipboard.ts                # 剪贴板操作
        ├── compare.ts                  # 比较工具
        ├── connection.ts               # 连接检测
        ├── constants.ts                # 常量定义
        ├── download.ts                 # 文件下载
        ├── encryption.ts               # 数据混淆（非加密）
        ├── format.ts                   # 格式化工具
        ├── headers.ts                  # HTTP 头部工具
        ├── helpers.ts                  # 通用助手
        ├── language.ts                 # 语言检测
        ├── models.ts                   # 模型工具
        ├── providerKeys.ts             # 提供商密钥工具
        ├── recentRequests.ts           # 最近请求
        ├── routeParams.ts              # 路由参数
        ├── timestamp.ts                # 时间戳工具
        ├── validation.ts               # 表单验证
        └── quota/                      # 配额专用工具
            ├── builders.ts, constants.ts, formatters.ts
            ├── parsers.ts, resetCredits.ts
            ├── resolvers.ts, validators.ts
```

---

## 4. 路由与页面

### 路由类型

使用 **Hash Router** (`createHashRouter`)，所有路由以 `/#/` 开头，无需服务器端路由配置。

### 路由树

```
RootShell (NotificationContainer + ConfirmationModal + Outlet)
│
├── /login                          → LoginPage (公开)
│
└── /*                              → ProtectedRoute → MainLayout
    │
    ├── / (/dashboard)              → DashboardPage 仪表盘
    │
    ├── /quick-start                → ProvidersWorkbenchPage (fixedBrand="apikeyFun")
    │
    ├── /ai-providers               → ProvidersWorkbenchPage AI 提供商管理
    │
    ├── /auth-files                 → AuthFilesPage 认证文件管理
    │   ├── /oauth-excluded         → AuthFilesOAuthExcludedEditPage
    │   └── /oauth-model-alias      → AuthFilesOAuthModelAliasEditPage
    │
    ├── /oauth                      → OAuthPage OAuth 授权
    │
    ├── /quota                      → QuotaPage 配额管理
    │
    ├── /plugins (条件路由)          → PluginsPage 插件管理
    ├── /plugin-store (条件路由)     → PluginStorePage 插件商店
    ├── /plugin-pages/:id/:index    → PluginResourcePage 插件页面
    │
    ├── /config                     → ConfigPage 配置管理
    │
    ├── /logs                       → LogsPage 日志查看
    │
    ├── /system                     → SystemPage 系统信息
    │
    ├── /settings                   → 重定向 → /config
    ├── /api-keys                   → 重定向 → /config
    │
    └── * (兜底)                    → 重定向 → /
```

### 侧边栏导航分组

| 分组 | 菜单项 | 说明 |
|------|--------|------|
| **Operate 运营** | Dashboard, Quick Start | 概览与快速开始 |
| **Gateway 网关** | AI Providers, Auth Files, OAuth, Quick Start | 提供商与认证管理 |
| **Observe 观测** | Quota, Logs | 配额与日志 |
| **Control 控制** | Config, Plugins, Plugin Store, System | 配置与系统 |
| **Plugin Pages** | 动态生成 | 已安装插件的自定义页面 |

### 条件路由

- 插件相关路由仅在服务器支持插件功能时可用（通过响应头 `X-Supports-Plugin` 检测）
- Quick Start 在 `apikey.fun` 未配置时显示在 Operate 分组，配置后移至 Gateway 分组

---

## 5. 组件体系

### 组件层级

```
App
└── RouterProvider
    └── RootShell
        ├── NotificationContainer    ← Toast 通知
        ├── ConfirmationModal        ← 全局确认弹窗
        └── Outlet
            ├── LoginPage
            └── ProtectedRoute
                └── MainLayout
                    ├── Header       ← 刷新/语言/主题/登出
                    ├── Sidebar      ← 导航分组/插件页面
                    └── Content
                        └── PageTransition (Motion动画)
                            └── 当前页面组件
```

### 基础 UI 组件 (`components/ui/`)

| 组件 | 功能 | 特点 |
|------|------|------|
| `Button` | 按钮 | 多变体(ghost/primary)、多尺寸、loading 状态 |
| `Input` | 输入框 | label、hint、right element 插槽 |
| `Select` | 下拉选择 | 选项列表 |
| `AutocompleteInput` | 自动完成输入 | 建议列表 |
| `Modal` | 模态弹窗 | 遮罩层、可关闭 |
| `Sheet` | 侧滑面板 | 详情展示 |
| `Card` | 卡片容器 | 内容卡片 |
| `Table` | 数据表格 | 表头/行/列 |
| `Collapsible` | 折叠组件 | 展开/收起 |
| `Skeleton` | 骨架屏 | 加载占位 |
| `ToggleSwitch` | 开关 | 布尔切换 |
| `SelectionCheckbox` | 复选框 | 选择样式 |
| `EmptyState` | 空状态 | 占位提示 |
| `LoadingSpinner` | 加载器 | 旋转动画 |
| `icons` | 图标集 | 30+ SVG 图标 React 组件 |

### 功能模块组件

#### 提供商管理 (`features/providers/`)
- `ProviderCategoryList` — 按分类展示提供商
- `ProviderHeaderCard` — 提供商头部卡片
- `ProviderResourceTable` — 资源表格
- `ProviderResourcePanel` — 资源面板
- `ProviderStatusBar` — 状态条
- `ProviderSheet` — 侧滑编辑面板
- `BaseProviderForm` — 提供商表单基类

#### 认证文件 (`features/authFiles/`)
- `AuthFileCard` — 认证文件卡片
- `AuthFileModelsModal` — 模型关联弹窗
- `AuthFileQuotaSection` — 配额区段
- `QuotaProgressBar` — 配额进度条
- `OAuthExcludedCard` — OAuth 排除模型卡片
- `OAuthModelAliasCard` — 模型别名卡片

#### 配置编辑 (`components/config/`)
- `ConfigSourceEditor` — CodeMirror YAML 源码编辑器
- `VisualConfigEditor` — 可视化配置编辑器
- `VisualConfigEditorBlocks` — 配置块组件
- `DiffModal` — 配置变更 Diff 对比弹窗

#### 模型映射 (`components/modelAlias/`)
- `ModelMappingDiagram` — 可视化模型映射关系图
- `ModelMappingDiagramColumns` — 映射列
- `ModelMappingDiagramContextMenu` — 右键菜单
- `ModelMappingDiagramModals` — 编辑弹窗

---

## 6. 状态管理

### Zustand Store 架构

项目使用 **Zustand 5** 管理全局状态，共 7 个 Store：

| Store | 文件 | 行数 | 持久化 | 说明 |
|-------|------|------|--------|------|
| `useAuthStore` | `useAuthStore.ts` | 289 | **是** (混淆 localStorage) | 登录/登出、会话恢复、服务器版本/运行时检测 |
| `useConfigStore` | `useConfigStore.ts` | 289 | 否 (内存) | 服务器配置缓存，TTL 过期，区段级缓存，请求去重 |
| `useThemeStore` | `useThemeStore.ts` | 114 | **是** (localStorage) | 主题切换 (auto/light/white/dark)，系统偏好检测 |
| `useLanguageStore` | `useLanguageStore.ts` | 55 | **是** (localStorage) | 多语言切换，与 i18next 同步 |
| `useNotificationStore` | `useNotificationStore.ts` | 106 | 否 | Toast 通知 (自动消失)、确认弹窗 |
| `useModelsStore` | `useModelsStore.ts` | 85 | 否 | 模型列表缓存，TTL 过期，按 API Base 分区 |
| `useQuotaStore` | `useQuotaStore.ts` | 71 | 否 | 5 种提供商配额数据缓存 |

### 关键设计模式

#### 请求去重 (`useConfigStore`)
```typescript
// 多个组件同时请求配置时，共享同一个 HTTP 请求
// 避免重复请求 (lines 125-129)
```

#### 混淆存储 (`useAuthStore`)
```typescript
// 使用 ObfuscatedStorage 替代明文 localStorage
// 通过 createJSONStorage 适配器包装
// 注意: 这是数据混淆，非加密
```

#### 全局事件通信
```typescript
// API Client 拦截器 → window.dispatchEvent(CustomEvent)
// useAuthStore 监听全局事件:
//   - 'unauthorized'       → 自动登出
//   - 'server-version-update' → 更新版本信息
//   - 'server-plugin-support-update' → 更新插件支持状态
```

### 数据流

```
用户操作 → 页面组件 → API Service → Axios → 后端
                                              │
后端响应 → Axios 拦截器 → window.dispatchEvent │
    │                                         │
    ▼                                         ▼
Store 更新 ← 页面 Hooks ← 组件 re-render   Store 监听器
```

---

## 7. API 集成层

### API 客户端架构 (`services/api/client.ts`)

**单例 `ApiClient` 类**，封装 Axios：

```typescript
class ApiClient {
  setConfig({ apiBase, managementKey, timeout })

  // 请求拦截器
  //   - 设置 baseURL
  //   - 注入 Authorization: Bearer <key>
  //   - 规范化 Gemini 端点路径

  // 响应拦截器
  //   - 读取 version/build-date/plugin-support 响应头
  //   - 派发 CustomEvent 到 window
  //   - 401 → 派发 'unauthorized' → 自动登出

  // 方法
  get, post, put, patch, delete, getRaw, postForm, requestRaw
}
```

### API 服务模块

| 模块 | 文件 | 行数 | 主要端点 |
|------|------|------|----------|
| `configApi` | `config.ts` | - | `GET/PUT /config`, `PUT /debug`, `PUT /proxy-url`, `PUT /request-retry`, `PUT /quota-exceeded/*`, `PUT /logging-to-file`, `PUT /ws-auth`, `PUT /routing/strategy` |
| `providersApi` | `providers.ts` | 525 | `GET/PUT/DELETE /gemini-api-key`, `/codex-api-key`, `/claude-api-key`, `/vertex-api-key`, `/openai-compatibility` (含 PATCH 禁用) |
| `authFilesApi` | `authFiles.ts` | 487 | `GET /auth-files`, `PATCH /auth-files/status`, `POST /auth-files` (上传), `DELETE /auth-files`, `GET/PUT/PATCH/DELETE /oauth-excluded-models`, `GET/PATCH/DELETE /oauth-model-alias` |
| `logsApi` | `logs.ts` | 242 | `GET/DELETE /logs`, `GET /request-error-logs`, `GET /request-log-by-id/:id` |
| `modelsApi` | `models.ts` | - | 模型列表 |
| `oauthApi` | `oauth.ts` | - | OAuth 配置 |
| `pluginsApi` | `plugins.ts` | - | 插件管理 |
| `versionApi` | `version.ts` | - | 运行时类型检测 (`cpa` / `home`) |
| `vertexApi` | `vertex.ts` | - | Google Vertex AI |
| `apiKeyUsageApi` | `apiKeyUsage.ts` | - | 用量统计 |
| `apiKeysApi` | `apiKeys.ts` | - | API Key 管理 |
| `configFileApi` | `configFile.ts` | - | 配置文件操作 |
| `antigravitySubscriptionApi` | `antigravitySubscription.ts` | - | Antigravity 订阅配额 |

### 响应转换 (`transformers.ts`)

- snake_case → camelCase 字段名转换
- 字段重命名
- 默认值填充

---

## 8. 认证体系

### 登录流程

```
LoginPage 登录页
    │
    ├─ 自动检测: detectApiBaseFromLocation() 从当前 URL 推导 API Base
    │
    ├─ 会话恢复: restoreSession()
    │    └─ 读取混淆 localStorage → 自动 login()
    │
    └─ 手动登录:
         │  用户输入 managementKey + 可选 apiBase
         │
         ▼
    useAuthStore.login()
         │
         ├─ connectionStatus: 'connecting'
         ├─ apiClient.setConfig({ apiBase, managementKey })
         ├─ configApi.getConfig()  ← 连通性测试
         ├─ 检测运行时类型 (cpa / home)
         │
         ├─ 成功 → isAuthenticated: true, 保存凭证
         └─ 失败 → connectionStatus: 'error'
```

### 认证方式

- **Bearer Token**: `Authorization: Bearer <managementKey>` 每个请求
- **非 JWT/Session**: 使用预共享的管理密钥
- **混淆存储**: `ObfuscatedStorageService` 封装 localStorage（非加密，仅混淆）

### 路由守卫

```typescript
// ProtectedRoute.tsx
if (!isAuthenticated) {
  if (hasStoredCredentials) {
    checkAuth()  // 尝试恢复会话
  }
  redirect('/login', { state: { from: location } })
}
```

### 自动登出

```
401 响应 → ApiClient 错误拦截器
    → window.dispatchEvent('unauthorized')
    → useAuthStore 监听器
    → logout()
    → 重定向 /login
```

---

## 9. 样式系统

### 方案: SCSS Modules + 全局 SCSS + CSS 自定义属性

**无 CSS 框架**（不使用 Tailwind/Bootstrap 等），完全自定义设计系统。

### 文件架构

| 文件 | 说明 |
|------|------|
| `variables.scss` | 设计令牌: 颜色、间距(4-48px)、圆角(4-full)、阴影、断点(768/1024/1280)、z-index 层级、字体 |
| `themes.scss` | 主题 CSS 变量: `--bg-primary`, `--bg-secondary`, `--text-primary`, `--border-color` 等 |
| `mixins.scss` | SCSS Mixins: `flex-center`, `text-ellipsis`, 响应式辅助 |
| `reset.scss` | CSS Reset |
| `global.scss` | 全局样式、滚动条、工具类、淡入淡出转场 |
| `layout.scss` | 应用外壳: `.app-shell`, `.main-header`, `.sidebar`, `.content` |
| `components.scss` | 共享样式: `.btn`, `.card`, `.modal`, `.badge` |

### 主题系统

| 主题 | 说明 | 背景色 |
|------|------|--------|
| `auto` | 跟随系统 `prefers-color-scheme` | 动态 |
| `light` | 暖色调浅色 | `#faf9f5` |
| `white` | 纯白变体 | `#ffffff` |
| `dark` | 深色模式 | `#151412` |

**切换机制**: `document.documentElement.setAttribute('data-theme', ...)` → CSS 变量切换

### 组件样式模式

```scss
// 每个组件搭配 .module.scss 文件
// Vite 配置自动注入 variables.scss
// camelCase 命名约定: styles.className
```

### 响应式设计

- **移动优先**: 断点 768px (mobile/tablet)、1024px (tablet/desktop)
- **侧边栏**: 桌面端可折叠，移动端覆盖式抽屉 + 背景遮罩
- **手势支持**: `useEdgeSwipeBack` Hook 支持边缘滑动返回

---

## 10. 国际化

### 配置

- **框架**: i18next + react-i18next
- **回退语言**: zh-CN (简体中文)
- **支持语言**:

| 语言代码 | 语言 | 文件 |
|----------|------|------|
| `zh-CN` | 简体中文 | `locales/zh-CN.json` |
| `zh-TW` | 繁体中文 | `locales/zh-TW.json` |
| `en` | 英文 | `locales/en.json` |
| `ru` | 俄语 | `locales/ru.json` |

### 语言检测

自动从浏览器/系统检测初始语言 (`getInitialLanguage()`)，用户可在头部切换，选择持久化到 localStorage。

---

## 11. 构建与部署

### 构建命令

```bash
bun install          # 安装依赖
bun run dev          # 开发服务器
bun run build        # tsc + vite build → dist/index.html
bun run preview      # 预览构建结果
bun run lint         # ESLint 检查
bun run format       # Prettier 格式化
bun run type-check   # TypeScript 类型检查
```

### CI/CD 流水线 (`.github/workflows/release.yml`)

```
git tag v* 推送
    │
    ▼
Ubuntu latest, Node 24, Bun 1.3.14
    │
    ├─ bun install --frozen-lockfile
    ├─ bun run build (VERSION=git tag)
    ├─ 重命名 dist/index.html → dist/management.html
    ├─ 自动生成 git log 发布说明
    └─ GitHub Release (softprops/action-gh-release@v1)
         └─ 附件: management.html
```

### 部署模型

```
management.html (单文件，全部内联)
    │
    ▼
CLIProxyAPI Go 后端
    │
    ├─ 嵌入 Go 二进制 (managementasset/)
    └─ 或作为静态文件 alongside 提供
         │
         └─ GET /management.html → 返回 SPA
              │
              └─ Hash Router 处理所有前端路由
```

### 测试状态

**当前无测试** — 没有测试文件、测试运行器或测试脚本。

---

## 12. 与后端接口对照

### 前端 API 模块 → 后端端点映射

| 前端模块 | 后端路由组 | 说明 |
|----------|-----------|------|
| `configApi` | `/v0/management/config*` | 配置管理 |
| `providersApi` | `/v0/management/{gemini,claude,codex,vertex}-api-key`, `/openai-compatibility` | 提供商密钥 |
| `authFilesApi` | `/v0/management/auth-files*`, `/oauth-excluded-models`, `/oauth-model-alias` | 认证文件 |
| `logsApi` | `/v0/management/logs*`, `/request-error-logs*` | 日志 |
| `oauthApi` | `/v0/management/{anthropic,codex,antigravity,kimi,xai}-auth-url` | OAuth URL |
| `pluginsApi` | `/v0/management/plugins*`, `/plugin-store*` | 插件 |
| `apiKeyUsageApi` | `/v0/management/api-key-usage`, `/usage-queue` | 用量 |
| `apiKeysApi` | `/v0/management/api-keys` | API Key 管理 |
| `versionApi` | `GET /` (响应头) | 版本检测 |

### 认证方式

| 前端 | 后端 |
|------|------|
| `Authorization: Bearer <managementKey>` | `X-MANAGEMENT-KEY` 或 `MANAGEMENT_PASSWORD` |
| 401 → 自动登出 | `managementAvailabilityMiddleware` + `mgmt.Middleware()` |

### 响应头通信

| 响应头 | 前端事件 | 说明 |
|--------|----------|------|
| `X-Server-Version` | `server-version-update` | 服务器版本 |
| `X-Build-Date` | (同事件) | 构建日期 |
| `X-Supports-Plugin` | `server-plugin-support-update` | 插件支持状态 |

---

## 13. 关键文件索引

| 文件 | 行数 | 说明 |
|------|------|------|
| `src/App.tsx` | ~35 | 根组件，Hash Router 注册 |
| `src/components/layout/MainLayout.tsx` | ~625 | 主布局，侧边栏/头部/内容 |
| `src/services/api/client.ts` | ~204 | API 客户端单例，拦截器 |
| `src/stores/useAuthStore.ts` | 289 | 认证状态管理 |
| `src/stores/useConfigStore.ts` | 289 | 配置缓存管理 |
| `src/services/api/providers.ts` | 525 | 提供商 API（最大的服务模块） |
| `src/services/api/authFiles.ts` | 487 | 认证文件 API |
| `src/services/api/logs.ts` | 242 | 日志 API |
| `vite.config.ts` | 77 | 构建配置（单文件输出） |
| `src/styles/variables.scss` | - | 设计令牌定义 |
| `src/styles/themes.scss` | - | 主题变量定义 |
| `src/i18n/index.ts` | - | i18next 配置 |

---

> **文档版本**: v1.0  
> **最后更新**: 2026-06-26  
> **适用版本**: React 19 + Vite 8 + TypeScript 6
