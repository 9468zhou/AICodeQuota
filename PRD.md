# AI 编码额度监控面板 — 产品需求文档（PRD）

**文档版本**: v1.2（全面复查修复：引用编号、托盘旧方案残留、HUD 技术路线统一、数值口径对齐、配置结构补齐、源码事实修正；v1.1 变更：新增智谱账号切换 M10、托盘点阵图标 M1.6 重写、移除 DeepSeek 网页登录）
**创建日期**: 2026-10-09
**整合来源**: `E:\PigBun\Coding-Plan`（Electron + Vue 3） + `E:\PigBun\ZBar`（Tauri 2 + React 19）
**目标产物**: 新建第三个项目 `E:\PigBun\AICodeQuota`

---

## 一、产品定位

### 1.1 一句话定位

跨平台常驻桌面工具，统一监控 AI 编码工具（智谱 Z.ai Coding Plan、DeepSeek、MiniMax、Claude Code、Kimi Code）的额度消耗与花费，菜单栏/托盘一瞥即知。

### 1.2 目标用户

| 用户类型         | 痛点                                                                                       | 本产品价值                                |
| ---------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------- |
| 重度 AI 编码用户 | 同时用多个 AI 编码工具（ZCode/Claude/Cursor/Kimi），额度分散在各客户端内，不知道哪个快用完 | 统一面板聚合，一屏看完所有额度            |
| 多设备用户       | 在公司 Mac + 家里 Windows 都用 Coding Plan，用量数据割裂                                   | 本机统计（本期）+ 预留同步接口            |
| 成本敏感用户     | 不知道每月 AI 编码花多少钱                                                                 | 花费统计 + 价格配置 + models.dev 在线同步 |

### 1.3 竞品定位

- **vs ZBar**：功能对齐（额度监控 + 花费统计 + 会话悬浮窗 + 价格配置 + 周对比），但 UI 全部改为 Vue 3 重写、样式统一到 ZBar 智谱面板规范，增加国际化（简/英）、增加 DeepSeek 服务状态监控
- **vs Coding-Plan**：UI 全面向 ZBar 智谱面板靠拢（玻璃拟态 + 手绘图表 + 极窄字号梯度），平台扩展到 macOS

### 1.4 本期不做（明确边界）

| 不做的功能                      | 原因                                                    |
| ------------------------------- | ------------------------------------------------------- |
| 多设备云同步（Python 服务器）   | 用户筛选时未选，架构上预留`sync` 模块接口但本期不实现 |
| 桌面宠物 / 动态壁纸注入         | 用户筛选时未选，涉及 ZCode 应用 asar 注入，破坏性强     |
| asar 注入类功能（改第三方应用） | 侵入性太强，合规风险                                    |
| 报告导出（日报/周报）           | 用户筛选时未选                                          |
| 项目/会话浏览器                 | 用户未选（会话悬浮窗已覆盖核心场景）                    |
| 模型速度排行（p10/p50/p90）     | 用户未选                                                |
| 并发测试                        | 用户未选                                                |
| 小程序/菜单栏内嵌时钟           | 无明确需求                                              |

---

## 二、已确认的功能范围（来自用户筛选）

### 2.1 必须实现

> **范围说明**：下表为 **v1.0 全量功能范围**。属于 **MVP（托盘优先交付）** 的项已在「MVP」列标注，详见 §十一。
> **重要**：本表「必须实现」≠「MVP 必须」——未标 MVP 的项属 v1.0 交付，不进入 MVP 最小集。

| #   | 功能                           | 来源               | MVP | 备注                                              |
| --- | ------------------------------ | ------------------ | --- | ------------------------------------------------- |
| F1  | 智谱 Z.ai 额度监控             | ZBar + Coding-Plan | ✅  | 双源认证：本地 ZCode 凭证优先 + API Key 兜底       |
| F2  | 智谱账号一键切换               | ZBar               | —   | 快照捕获 + 事务切换 + 自动切换                     |
| F3  | 托盘图标显示额度百分比         | Coding-Plan        | ✅  | 纯代码渲染点阵数字 + 3 色阈值 + 加载动画           |
| F4  | DeepSeek 额度监控              | ZBar               | —   | 仅 API Key（余额型，不进托盘聚合 §1.6.5）          |
| F5  | DeepSeek 服务状态监控          | Coding-Plan        | —   | Flashcat 90 天可用率                              |
| F6  | MiniMax 额度监控               | ZBar + Coding-Plan | ✅  | API Key                                           |
| F7  | [CC] 用量导入                  | ZBar               | ◐   | OAuth 实时额度 ✅ / 会话 Token 导入 —              |
| F8  | Kimi Code 用量导入             | ZBar               | ◐   | 设备码登录 + 额度 ✅ / 会话 Token 导入 —           |
| F9  | 会话 Token 悬浮窗              | ZBar               | —   | 独立窗口，最多 5 会话                              |
| F10 | 周额度对比视图                 | ZBar               | —   | 90 天快照周期切分                                  |
| F11 | 价格配置 + models.dev 在线同步 | ZBar               | —   | 多级模型名匹配（精确/归一/变体渐进）               |
| F12 | 国际化（简体中文 / English）   | Coding-Plan        | —   | 约 360 词条（以最终实现为准对齐）                  |
| F13 | 3 种窗口固定模式               | Coding-Plan        | —   | unpinned / pinned-top / pinned-desktop            |
| F14 | 内存节省模式                   | Coding-Plan        | —   | 隐藏即销毁                                        |
| F15 | 配置加密 + 原子写入 + 6 步迁移 | Coding-Plan        | ◐   | 加密 + 原子写 ✅ / 6 步迁移 —                      |
| F16 | 自动更新                       | ZBar               | —   | tauri-plugin-updater，GitHub 单源（§8.1）         |
| F17 | 网络代理                       | ZBar               | —   | env > 系统代理 > 直连，支持 http/socks5           |

**图例**：✅ = MVP 必须（全量） / ◐ = MVP 部分（详见 §十一 11.2） / — = 非 MVP（v1.0 交付）

### 2.2 UI 规范（硬约束）

**所有 AI 平台的额度/统计面板，样式统一按 ZBar 的智谱额度监控面板实现**（用户提供的截图）。

具体规则：

- 卡片容器：`card-base rounded-2xl`（16px 圆角，半透明玻璃背景 + 1px 细边 + 轻阴影）
- 次级卡 / 小条容器：`rounded-xl`（12px）
- 行 / 槽：`rounded-lg`（8px）
- 主色板：sky（智谱/汇总）/ teal（速度）/ emerald（缓存）/ violet（当前/等级）/ amber（警告）
- **所有数字加 `.num` 类**（SF Mono / JetBrains Mono + `tabular-nums`）
- 字号梯度：8 / 9 / 10 / 11 / 13 / 26px（26px 仅用于 Hero 主数值）
- 透明度任意值：Tailwind v4 支持 `/4` `/6` `/8` `/12` `/45` 等非 5 倍数档
- 主题切换：`document.documentElement.classList.toggle("dark", ...)` + localStorage 持久化
- 图表：手绘 flex div + absolute 定位，**不引入任何图表库**

---

## 三、整体架构设计

### 3.1 技术选型

```
┌─────────────────────────────────────────────────────────┐
│                    Tauri 2 (Rust)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Panel 窗口  │  │ HUD 窗口     │  │ Tray 托盘    │  │
│  │  (Vue 3 SPA) │  │ (原生HTML+DOM)│ │ (纯 Rust)    │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────────┘  │
│         │ invoke          │ 事件推送(zbar式)            │
│  ┌──────▼──────────────────▼──────────────────────────┐ │
│  │              Rust Command 层 (#[tauri::command])     │ │
│  │  fetch_zai_quota / get_deepseek_usage / ...         │ │
│  │  get_pricing / get_stats / ...                      │ │
│  └──────┬──────────┬──────────┬──────────┬────────────┘ │
│         │          │          │          │              │
│  ┌──────▼───┐ ┌────▼─────┐ ┌──▼──────┐ ┌─▼────────────┐ │
│  │ provider │ │ pricing  │ │accounts │ │ config       │ │
│  │ 模块     │ │ 模块     │ │ 模块    │ │ 模块          │ │
│  │(quota.rs │ │(pricing  │ │(多账号) │ │(加密+迁移)   │ │
│  │ deepseek │ │ .rs)    │ │         │ │ (config.rs)  │ │
│  │ minimax  │ │         │ │         │ │              │ │
│  │ claude   │ │         │ │         │ │              │ │
│  │ kimi)    │ │         │ │         │ │              │ │
│  └──────────┘ └─────────┘ └─────────┘ └──────────────┘ │
│         │                                                  │
│  ┌──────▼──────────────────────────────────────────────┐ │
│  │  net_config (HTTP 客户端: ureq + 代理解析)          │ │
│  └──────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
                        │
         ┌──────────────┴──────────────┐
         │                             │
  ┌──────▼────────┐          ┌────────▼────────┐
  │ SQLite 只读    │          │ ~/.aicodequota/ │
  │ ZCode 主库     │          │ 配置文件目录     │
  │ (rusqlite)     │          │                │
  │ Claude JSONL  │          │ config.json     │
  │ Kimi JSONL     │          │ pricing.json    │
  └───────────────┘          │ quota_history   │
                             │   .jsonl        │
                             │ accounts/       │
                             │ credentials/    │
                             └─────────────────┘
```

### 3.2 目录结构规划

```
AICodeQuota/
├── package.json
├── vite.config.ts                    # Vue 3 + Tailwind v4 + 多入口（index.html + hud.html）
├── tsconfig.json
├── index.html                        # panel 入口（含首帧占位，防 WebView 冷启动白屏）
├── hud.html                          # 会话悬浮窗入口（原生 HTML + DOM，非 Vue）
├── src/                              # 渲染层
│   ├── main.ts                       # panel 入口（Vue 3 挂载）
│   ├── hud-main.ts                   # HUD 入口（原生 DOM 操作，从 ZBar 原样搬）
│   ├── App.vue                       # panel 根组件（视图路由）
│   ├── types.ts                      # TS 类型（与 Rust 对齐，附录 A 的 TS 镜像）
│   ├── api.ts                        # invoke 封装
│   ├── utils/
│   │   └── format.ts                 # 格式化工具（从 ZBar format.ts 原样搬）
│   ├── stores/                       # Pinia stores
│   │   ├── data.ts                   # 全局数据缓存 + 轮询（迁自 ZBar DataCache）
│   │   ├── appearance.ts             # 主题/缩放（迁自 ZBar appearance.ts）
│   │   └── settings.ts               # 用户设置（新建）
│   ├── composables/                  # （新建，对应 ZBar hooks）
│   │   ├── useI18n.ts
│   │   ├── useResetDisplay.ts
│   │   └── useNow.ts                 # 组件局部时钟
│   ├── components/
│   │   ├── layout/                   # 布局原语（对应 React layout.tsx）
│   │   │   ├── SectionCard.vue
│   │   │   ├── HeroMetric.vue
│   │   │   ├── PageShell.vue
│   │   │   ├── PageHeader.vue
│   │   │   ├── PillToggle.vue
│   │   │   └── ToolbarButton.vue
│   │   ├── widgets/                  # 数据展示原语（对应 React widgets.tsx）
│   │   │   ├── QuotaBar.vue          # ★ 三列 grid 3.25rem/1fr/3.25rem
│   │   │   ├── ProgressBar.vue
│   │   │   ├── Metric.vue
│   │   │   ├── DetailRow.vue
│   │   │   ├── TrendChart.vue        # ★ 手绘柱状图 + absolute tooltip
│   │   │   └── CompositionBar.vue
│   │   ├── providers/                # 各 AI 平台面板（样式统一）
│   │   │   ├── zai/
│   │   │   │   ├── ZaiQuotaCard.vue       # 5h/周/MCP 额度条
│   │   │   │   ├── ZaiStatsContent.vue    # Hero/趋势/三宫格/Token构成
│   │   │   │   ├── AccountQuotaGroup.vue  # ★ 多账号列表（M10.6）
│   │   │   │   └── SwitchConfirmOverlay.vue # ★ 切换确认浮层（M10.6）
│   │   │   ├── deepseek/
│   │   │   │   ├── DeepSeekQuotaCard.vue
│   │   │   │   ├── DeepSeekStatsContent.vue
│   │   │   │   └── ServiceStatusBar.vue   # ★ 90 天可用率
│   │   │   ├── minimax/
│   │   │   │   └── MiniMaxQuotaCard.vue
│   │   │   ├── claude/
│   │   │   │   └── ClaudePanel.vue
│   │   │   └── kimi/
│   │   │       └── KimiPanel.vue
│   │   └── tabs/
│   │       ├── StatsPanel.vue            # 主容器：头部 + tab + 状态栏
│   │       ├── ComparePanel.vue          # 周额度对比
│   │       ├── PricingPanel.vue          # 价格配置
│   │       └── SettingsPanel.vue         # 设置（含自动切换配置 M10.6）
│   ├── i18n/
│   │   ├── index.ts
│   │   └── locales/
│   │       ├── zh-CN.json               # 从 Coding-Plan shared/locales 裁剪
│   │       └── en-US.json
│   └── styles/
│       └── index.css               # ★ 移植 ZBar 全部自定义 CSS 类
├── src-tauri/
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   ├── capabilities/
│   │   ├── default.json             # panel 窗口
│   │   └── hud.json                 # HUD 窗口
│   ├── icons/
│   └── src/
│       ├── main.rs
│       ├── lib.rs                   # 入口编排：插件注册 + 命令清单挂载 + setup 钩子
│       ├── zcode_crypto.rs          # AES-256-GCM 解密（从 ZBar 搬）
│       ├── quota.rs                 # 智谱额度（双源认证，从 ZBar 搬 + API Key 兜底）
│       ├── quota_history.rs         # 快照 JSONL + 90 天滚动 + 周周期切分
│       ├── deepseek.rs              # DeepSeek 额度（仅 API Key）
│       ├── deepseek_status.rs       # ★ Flashcat 服务状态（从 Coding-Plan 搬）
│       ├── minimax.rs               # MiniMax 额度
│       ├── claude.rs                # Claude Code 导入 + OAuth 额度
│       ├── kimi.rs                  # Kimi Code 导入 + OAuth 额度
│       ├── kimi_oauth.rs            # Kimi 设备码登录
│       ├── pricing.rs               # 价格配置 + models.dev 同步 + 三级匹配
│       ├── accounts.rs              # ★ 智谱多账号快照 + 切换事务 + 自动切换（从 ZBar 搬）
│       ├── provider_credentials.rs  # 通用凭证体系（DeepSeek/MiniMax/Claude/Kimi）
│       ├── db.rs                    # ZCode SQLite 只读查询
│       ├── session_hud.rs           # 会话悬浮窗数据 + 窗口管理
│       ├── config.rs                # ★ 配置管理（加密 + 原子写 + 迁移，从 Coding-Plan 搬）
│       ├── net_config.rs            # HTTP 客户端 + 代理解析
│       ├── tray.rs                  # ★ 托盘点阵数字图标 + 3 色阈值 + 加载动画（从 Coding-Plan 搬）
│       ├── window.rs                # Panel 窗口管理 + 3 种固定模式
│       └── i18n.rs                  # 主进程国际化
└── public/
    └── pricing-defaults.json        # 从 ZBar 搬（编译期嵌入）
```

### 3.3 技术栈选型

| 层          | 技术                    | 版本  | 来源                             |
| ----------- | ----------------------- | ----- | -------------------------------- |
| 桌面框架    | Tauri 2                 | 2.x   | ZBar                             |
| 前端框架    | Vue 3 (Composition API) | 3.5.x | Coding-Plan                      |
| 语言        | TypeScript              | 5.8.x | 两者                             |
| 构建        | Vite                    | 7.x   | 两者                             |
| UI 样式     | Tailwind CSS v4         | 4.x   | ZBar（@theme + 自定义 CSS 类）   |
| 状态管理    | Pinia                   | 3.x   | 迁移自 ZBar 的 DataCache Context |
| 图表        | **无图表库**      | —    | 手绘 flex div（ZBar 方案）       |
| 国际化      | vue-i18n                | 10.x  | Coding-Plan                      |
| Rust HTTP   | ureq 2                  | 2.x   | ZBar                             |
| Rust 加密   | aes-gcm + sha2          | 0.10  | ZBar                             |
| Rust SQLite | rusqlite (bundled)      | 0.32  | ZBar                             |
| Rust 序列化 | serde + serde_json      | 1.x   | ZBar                             |
| 自动更新    | tauri-plugin-updater    | 2.x   | ZBar（GitHub + Gitee 双源）      |

### 3.4 核心依赖（Rust）

```toml
# src-tauri/Cargo.toml
tauri = { version = "2", features = ["macos-private-api", "tray-icon", "image-png"] }
tauri-plugin-positioner = { version = "2", features = ["tray-icon"] }  # 托盘弹出面板定位
tauri-plugin-opener = "2"
tauri-plugin-global-shortcut = "2"
tauri-plugin-clipboard-manager = "2"   # 复制报告/凭证掩码到剪贴板
tauri-plugin-autostart = "2.5.1"
tauri-plugin-updater = "2"
serde = { version = "1", features = ["derive"] }
notify = "6"                    # 配置文件监听（M2.6）
tokio = { version = "1", features = ["fs", "sync", "time"] }  # 异步文件操作（Tauri 2 运行时自带，显式声明供 M2.5 使用）
tauri-plugin-notification = "2"  # 自动切换成功后的系统通知（M10.5）
serde_json = "1"
rusqlite = { version = "0.32", features = ["bundled"] }
ureq = { version = "2", features = ["json", "socks-proxy"] }
aes-gcm = "0.10"
sha2 = "0.10"
base64 = "0.22"
chrono = { version = "0.4", features = ["serde"] }
dirs = "5"
uuid = { version = "1", features = ["v4"] }

[target.'cfg(target_os = "macos")'.dependencies]
objc2 = "0.6"
objc2-app-kit = "0.3"
objc2-foundation = { version = "0.3", features = ["NSProcessInfo"] }

[target.'cfg(windows)'.dependencies]
winreg = "0.52"
```

### 3.5 核心依赖（前端）

```json
{
  "dependencies": {
    "vue": "^3.5.13",
    "vue-i18n": "^10.0.4",
    "pinia": "^3.0.0",
    "@tauri-apps/api": "^2.0.0",
    "@tauri-apps/plugin-global-shortcut": "^2.0.0",
    "@tauri-apps/plugin-updater": "^2.0.0"
  },
  "devDependencies": {
    "vite": "^7.0.5",
    "@vitejs/plugin-vue": "^6.0.0",   # Vite 7 要求 plugin-vue 6.x
    "typescript": "^5.8.3",
    "tailwindcss": "^4.0.11",
    "@tailwindcss/vite": "^4.0.11"
  }
}
```

---

## 四、功能模块划分

### 4.1 模块总览

```
AICodeQuota
├── M1: 窗口管理（Panel / HUD / 托盘 / 3 种固定模式）
├── M2: 配置管理（加密 + 原子写 + 6 步迁移）
├── M3: Provider 适配层（智谱/DeepSeek/MiniMax/Claude/Kimi）
│   ├── M3.1 智谱额度（双源认证）
│   ├── M3.2 DeepSeek 额度（API Key 单模）
│   ├── M3.3 DeepSeek 服务状态（Flashcat）
│   ├── M3.4 MiniMax 额度
│   ├── M3.5 Claude Code 导入 + OAuth
│   └── M3.6 Kimi Code 导入 + OAuth
├── M4: 价格与成本（三级匹配 + models.dev 同步）
├── M5: 额度快照与周对比（90 天 JSONL + 周期切分）
├── M6: 会话悬浮窗（HUD）
├── M7: 国际化（中/英）
├── M8: 自动更新（GitHub + Gitee 双源）
├── M9: 网络代理（env > 系统 > 直连）
└── M10: 智谱多账号（快照 + 切换事务 + 自动切换）
```

---

### M1: 窗口管理

#### 1.1 窗口规格

| 窗口       | Label           | 尺寸                                 | 特性                                                                                                                           |
| ---------- | --------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| 主面板     | `panel`       | 300×500（可调 260~420 × 430~760） | `decorations:false, transparent:true, alwaysOnTop:true, skipTaskbar:true, visible:false`，macOS 用 `windowEffects:popover` |
| 会话悬浮窗 | `session-hud` | 300×自适应（最小 260×160）         | `decorations:false, transparent:true, alwaysOnTop:true, focusable:false`，8 方向 resize                                      |
| 托盘       | —              | 16×16（Windows）/ 22×22（macOS）点阵 | 点阵数字图标显示剩余百分比（两端统一，详见 M1.6），3 色阈值 + 加载动画                                                       |

> 尺寸来源说明：面板默认 300×500 取自 ZBar（Tauri 承接其窗口配置），非 Coding-Plan 的 336×416；两侧用户均可自由调整，高度下限 430。

#### 1.2 3 种窗口固定模式（Coding-Plan 移植）

```typescript
type WindowPinMode = 'unpinned' | 'pinned-top' | 'pinned-desktop'
```

| 模式               | alwaysOnTop | 失焦隐藏                          | 说明                             |
| ------------------ | ----------- | --------------------------------- | -------------------------------- |
| `unpinned`       | ✅          | ✅（Pinned 显示态下）             | 默认。鼠标点击唤起，点击外部消失 |
| `pinned-top`     | ✅          | ❌                                | 固定悬浮层，可编辑内容           |
| `pinned-desktop` | ❌          | ❌                                | 桌面层，可被其他窗口遮挡         |

**切换规则**（从 Coding-Plan 搬）：

```rust
fn apply_always_on_top(pin_mode: WindowPinMode) {
    window.set_always_on_top(pin_mode != WindowPinMode::PinnedDesktop);
}

fn attach_blur_handler(pin_mode: WindowPinMode) {
    if pin_mode != WindowPinMode::Unpinned { return; }
    window.on_window_event(|e| {
        if matches!(e, WindowEvent::Focused(false)) { hide_panel(); }
    });
}
```

**隐藏即重置**（从 Coding-Plan 搬）：`hide_panel()` 时若 pinMode ≠ unpinned，重置为 unpinned 并推送 `window-pinned-state` 事件（pinMode 本身不持久化，仅 `popupPosition`/`popupSize` 记忆）。

#### 1.2a 显示模式状态机（Coding-Plan popup-manager 移植）

固定模式之外，面板还有一个**运行时显示模式**（§5.2 流程依赖）：

```rust
enum DisplayMode {
    Hidden,   // 原生 hide 或销毁（内存节省模式）
    Hover,    // 托盘悬停唤起，鼠标离开 300ms 后自动隐藏
    Pinned,   // 托盘点击/快捷键唤起，点击外部隐藏（pinMode 决定是否真的隐藏）
}
```

| 事件 | Hidden | Hover | Pinned |
| ---- | ------ | ----- | ------ |
| 托盘 mouse-enter（popupTrigger=hover） | → Hover | — | — |
| 托盘 mouse-leave / 鼠标离开面板 | — | 300ms 延迟校验后 → Hidden | — |
| 托盘点击 / 快捷键 | → Pinned | → Pinned（升级可交互） | → Hidden（再点关闭） |
| 窗口 blur | — | — | pinMode=unpinned 时 → Hidden |

悬停隐藏的二次校验：300ms 回调时检查 `displayMode==Hover && 可见 && 光标不在面板 bounds 内`，任一不满足则取消隐藏（防误触）。

#### 1.3 内存节省模式（Coding-Plan 移植）

```rust
fn hide_panel() {
    if memory_saving_mode {
        window.destroy();  // 销毁而非 hide
        panel_window = None;
    } else {
        window.hide();
    }
}

fn show_panel(mode) {
    if panel_window.is_none() {
        create_panel_window();  // 内存节省下每次显示都重建
    }
    // ...
}
```

启动时：非内存节省模式才预创建窗口。

#### 1.4 显示/隐藏时序（macOS popover 效果）

```
用户点击托盘/按快捷键
  → 获取托盘 bounds 或屏幕右下角
  → 计算 x = mon_w - win_w - 4
  → 计算 y（macOS: 25.0 菜单栏下方 / Windows: mon_h - win_h - 48.0 任务栏上方）
  → set_position → show → set_focus
  → ★ 强制 set_size(size+1) → set_size(size) 修复 WKWebView 长期隐藏白屏
```

#### 1.5 Windows 隐藏建窗规避钳宽（从 ZBar 搬）

```rust
#[cfg(not(target_os = "macos"))]
{
    let win = builder.visible(false).build()?;
    let _ = win.set_size(LogicalSize::new(w, h));
    let _ = win.show();
}
#[cfg(target_os = "macos")]
{
    let win = builder.build()?;
    let _ = win.set_background_color(Some(Color(0, 0, 0, 0)));
}
```

**原因**：tao 给所有窗口无条件带 `WS_CAPTION`，创建期间的 `WM_GETMINMAXINFO` 走子类化前的 DefWindowProc，最小跟踪宽度会钳死小窗口。

#### 1.6 托盘图标：额度百分比点阵渲染

**用户要求：托盘图标与 Coding-Plan 一致，图标上直接显示剩余额度百分比。**

实现方式：Rust 侧纯代码生成 16×16 RGBA 点阵图，Tauri 2 用 `tauri::image::Image::new_owned()` 包装后交给 `TrayIconBuilder::icon()`。全程无图片资源文件、无字体依赖。

##### 1.6.1 颜色阈值

```rust
struct ColorThresholds {
    green: u32,   // 默认 50
    yellow: u32,  // 默认 20
}

fn color_by_percent(percent: i32, th: &ColorThresholds) -> TrayColor {
    match percent {
        p if p < 0 => TrayColor::Gray,                              // 无数据
        p if p >= th.green as i32 => TrayColor::Green,             // ≥ 50%
        p if p >= th.yellow as i32 => TrayColor::Yellow,           // 20~50%
        _ => TrayColor::Red,                                       // < 20%
    }
}

enum TrayColor { Green, Yellow, Red, Gray }
// RGB: #22C55E / #F59E0B / #EF4444 / #888888
```

**语义**：显示的是**剩余**百分比（`100 - 已用百分比`），与面板中 `QuotaBar` 的条宽口径一致。

##### 1.6.2 点阵字库

5×7 单色点阵，字符集仅 `0-9` 与 `-`（无数据时显示 `--`）：

```rust
// 每个字符 7 行，每行 5 bit（最高位在左）
// 逐字符与 Coding-Plan tray.ts DIGIT_FONT 位图核对一致
const DIGIT_FONT: [(u8; 7); 11] = [
    [0x0E, 0x11, 0x13, 0x15, 0x19, 0x11, 0x0E], // 0
    [0x04, 0x0C, 0x14, 0x04, 0x04, 0x04, 0x1F], // 1
    [0x0E, 0x11, 0x01, 0x02, 0x04, 0x08, 0x1F], // 2
    [0x0E, 0x11, 0x01, 0x06, 0x01, 0x11, 0x0E], // 3
    [0x02, 0x06, 0x0A, 0x12, 0x1F, 0x02, 0x02], // 4
    [0x1F, 0x10, 0x1E, 0x01, 0x01, 0x11, 0x0E], // 5
    [0x06, 0x08, 0x10, 0x1E, 0x11, 0x11, 0x0E], // 6
    [0x1F, 0x01, 0x02, 0x04, 0x08, 0x08, 0x08], // 7
    [0x0E, 0x11, 0x11, 0x0E, 0x11, 0x11, 0x0E], // 8
    [0x0E, 0x11, 0x11, 0x0F, 0x01, 0x02, 0x0C], // 9
    [0x00, 0x00, 0x00, 0x1F, 0x00, 0x00, 0x00], // -
];
```

##### 1.6.3 渲染算法

```rust
fn render_percent_icon(percent: i32, color: TrayColor) -> tauri::image::Image<'static> {
    const W: usize = 16;
    const H: usize = 16;

    // ① 决定显示内容
    //    注：clamp(0, 999) 为本项目新增保护（Coding-Plan 源码仅 Math.round），
    //    防止脏数据渲染出 4 位数字溢出 16px 画布
    let (text, rgb) = if percent < 0 {
        ("--".to_string(), TRAY_GRAY)
    } else {
        (percent.round().clamp(0, 999).to_string(), color.rgb())
    };

    // ② 计算排版：3 位时字间距为 0，1~2 位为 1
    let char_gap: usize = if text.chars().count() > 2 { 0 } else { 1 };
    let total_w = text.chars().count() * 5 + (text.chars().count() - 1) * char_gap;
    let x0 = (W - total_w) / 2;          // 水平居中
    let y0 = (H - 7) / 2;                // 垂直居中

    // ③ 逐像素填充
    let mut rgba = vec![0u8; W * H * 4];
    for (ci, ch) in text.chars().enumerate() {
        let glyph = glyph_for(ch);       // '-' 或 DIGIT_FONT[c - '0']
        let gx = x0 + ci * (5 + char_gap);
        for (row, bits) in glyph.iter().enumerate() {
            for col in 0..5 {
                if bits & (1 << (4 - col)) != 0 {
                    let idx = ((y0 + row) * W + gx + col) * 4;
                    rgba[idx..idx + 4].copy_from_slice(&[rgb.0, rgb.1, rgb.2, 255]);
                }
            }
        }
    }
    tauri::image::Image::new_owned(rgba, W, H)
}
```

**为什么用 RGBA 而非 PNG**：Tauri 2 的 `Image::new_owned` 直接接受 RGBA 像素缓冲，不需要手写 CRC32 + zlib deflate 的 PNG 编码器（Coding-Plan 用 Electron `nativeImage.createFromBuffer` 才必须自己编码 PNG）。这是移植到 Tauri 后的净简化。

##### 1.6.4 加载动画

刷新进行中显示 6 帧旋转弧，替代百分比数字：

```rust
const LOADING_FRAMES: usize = 6;
const LOADING_INTERVAL_MS: u64 = 180;

fn render_loading_frame(frame_index: usize) -> tauri::image::Image<'static> {
    // 6 帧旋转弧线，每帧转 60°，缺口 60°，双半径描边
    // 颜色 rgb(140, 160, 190)
    // frame_index ∈ [0, 6)，用定时器循环推进
}
```

**互斥规则**：`loading` 为 true 时，`update_display` 不得覆盖图标（与 Coding-Plan 一致）。

**两端统一**（2026-10-09 确认）：macOS 与 Windows 均渲染 6 帧旋转弧（macOS 22px 画布下弧线加粗 1px 绘制）。

##### 1.6.5 百分比来源

```rust
fn calculate_display_percent() -> i32 {
    let entries = /* 当前所有启用账户的 QuotaEntry */;
    // 先过滤无数据账户（remaining = -1 的哨兵不参与 lowest/highest 聚合，
    // 否则单个失效账户会把整个托盘拖成灰色 "--"）
    let valid: Vec<i32> = entries.iter().map(|e| min_remaining(e))
        .filter(|&v| v >= 0).collect();
    match config.tray_display_rule.as_str() {
        // 全部无数据 → -1（灰 "--"）；lowest 取有效值最小，highest 取最大
        "lowest"  => valid.iter().copied().min().unwrap_or(-1),
        "highest" => valid.iter().copied().max().unwrap_or(-1),
        key       => /* "{provider}:{accountId}" 复合键 → 指定账户（允许 -1，显灰提示该账户失效） */,
    }
}

// 单个账户的剩余百分比 = min(所有窗口的剩余)；无任何窗口数据 → -1 哨兵
fn min_remaining(entry: &QuotaEntry) -> i32 {
    entry.windows.iter()
        .filter_map(|w| w.used_percent.map(|u| (100.0 - u).round() as i32))
        .min()
        .unwrap_or(-1)
}
```

`tray_display_rule` 配置项（Coding-Plan 移植，用户确认默认 lowest）：`lowest`（所有账户最低剩余，跨平台聚合）/ `highest`（最高）/ `{provider}:{accountId}`（指定账户）。

**余额型账户的处置**：DeepSeek 等余额型账户不产出百分比窗口（只有 balance），不参与 lowest/highest 聚合——托盘数字始终来自「有百分比窗口的账户」（智谱/MiniMax/Claude/Kimi）。指定复合键指向余额型账户时显示 `--`。

##### 1.6.5a 托盘聚合的空态与部分失败语义（PRD 缺口补全）

§1.6.5 只定义了单值聚合规则，未定义空态与部分失败时的呈现。以下为**本项目定义**（PRD 原文未覆盖，需开发遵循）：

**五态托盘呈现**（`calculate_display_percent()` 的完整返回语义）：

| 态 | 触发条件 | 图标 | tooltip / title |
|----|---------|------|-----------------|
| **正常** | ≥1 个百分比账户有数据 | 点阵数字 + 按剩余着色 | `最低剩余 87%（智谱 个人账号）` |
| **陈旧** | 有数据但 `now - updatedAt > 3×refreshInterval` | 数字 + 降饱和度（视觉降级） | `87%（数据 5 分钟前）` |
| **部分失败** | 部分百分比账户失败、部分正常 | **仍显示正常值**（不用 `--`） | `最低剩余 87%（2/4 平台可用）` |
| **全部失败** | 所有百分比账户均失败 | 灰色 `--` | `数据获取失败（3 分钟前成功）` |
| **未配置** | 无任何已启用账户 | 灰色 `--` | `未配置任何账户，点击添加` |

**关键裁决**：
- **部分失败不降级为 `--`**：托盘数字是「当前最该关注的那个额度」，只要有一个源可用，它就有意义。失败源的数量只在 tooltip 体现。
- **`-1` 哨兵不参与聚合**（§1.6.5 已修）：`lowest`/`highest` 先在「有数据的账户」中取值；全部无数据才回落到 `--`。
- **「未配置」与「全部失败」在托盘上均显示 `--`，但 tooltip 文案不同**——用户必须能区分「我没配」与「配了但连不上」，这是可诊断性的底线。
- **多平台归因**：单个 16px 图标位无法表达「哪个平台」，归因信息只在 tooltip / macOS title / 面板中承载。**托盘本身只承载「危险度」，不承载「来源」**——这是图标物理尺寸的硬约束，不做变通。

##### 1.6.5b 托盘展示规则的配置界面（PRD 缺口补全）

§1.6.5 定义了 `trayDisplayRule` 配置项但未说明其 UI 暴露方式。以下为**本项目定义**：

- **设置页位置**：`SettingsPanel.vue` 的「通用设置」区，独立一行「托盘显示规则」
- **可选值**：`lowest`（最低剩余，默认）/ `highest`（最高剩余）/ 指定账户（下拉，列出所有「有百分比窗口」的账户）
- **默认值来源**：`AppConfig.trayDisplayRule` 默认 `'lowest'`（§2.2）
- **空态**：若无任何百分比账户（如仅配置了 DeepSeek），下拉禁用并提示「暂无百分比型账户」；此时托盘恒显示 `--`，tooltip 为「仅配置了余额型账户，无百分比数据可聚合」

##### 1.6.6 托盘菜单

| 菜单项       | i18n key               | 行为                                                          |
| ------------ | ---------------------- | ------------------------------------------------------------- |
| 显示面板     | `tray.showPanel`       | 左键点击托盘图标等价；菜单内再给一次入口（防 hover 关闭党）   |
| 刷新         | `tray.refresh`         | 触发全量刷新（图标切到 loading 帧，完成后回到百分比）         |
| 设置         | `tray.settings`        | 打开设置面板                                                  |
| 检查更新     | `tray.checkUpdate`     | 手动触发 tauri-plugin-updater                                 |
| 重置窗口位置 | `tray.resetPosition`   | 清空 `popupPosition`，下次显示回到默认位置                    |
| 重置窗口大小 | `tray.resetSize`       | 清空 `popupSize`，恢复默认 300×500（Coding-Plan 同款菜单项）  |
| 退出         | `tray.quit`            | 退出应用（**注意与「关闭面板」的区别，见 M1.7**）             |

**macOS 菜单栏 title**：`tray.title()` 显示剩余百分比数字（如 `87`），每 30s 随轮询刷新；与静态图标互补（点阵在 22px 菜单栏下辨识度差，百分比数字作为主要信息载体）。Windows 托盘不设 title（无此 UI 惯例）。（ZBar 菜单栏 title 的「今日花费 + 总 Token」摘要不做——那依赖统计链路，与 F3 托盘图标需求无关。）

##### 1.6.7 macOS 特有注意事项

```rust
// ① Accessory 模式隐藏 Dock 图标（从 ZBar 搬）
app.set_activation_policy(tauri::ActivationPolicy::Accessory);

// ② 两端统一点阵数字（用户已确认）：
//    Windows: 16×16 逻辑像素
//    macOS:   22×22 逻辑（44×44@2x 物理渲染，Retina 下点阵笔画为 2px 物理宽，可读）
fn build_tray_icon(percent: i32, size: u32) -> tauri::image::Image<'static> {
    render_percent_icon_sized(percent, color_by_percent(percent, &thresholds), size)
    // render_percent_icon_sized = M1.6.3 算法的参数化画布版本
    //（W/H 与居中计算均按 size 缩放；5×7 字形在 22px 下放大 3 倍绘制，保持笔画方正）
}
```

> **已确认（2026-10-09）**：macOS 与 Windows 统一使用点阵数字图标（F3 跨平台一致）；开发期以 44×44@2x 实测校验可读性，若 Retina 实机不可读再回到「macOS 静态图标 + title」备选方案。macOS 同时保留 `tray.title()` 显示同数字（双通道，菜单栏密度高时 title 是更快扫读路径）。

#### 1.7 单实例约束与「关闭面板 / 退出应用」语义（PRD 缺口补全）

PRD 原文未定义这两项，但常驻场景必然遇到。以下为**本项目定义**：

**单实例约束**（新增依赖 `tauri-plugin-single-instance`；**已确认采纳，2026-10-10**）：

| 场景 | 行为 |
|------|------|
| 应用已在运行，用户再次双击图标 | **不启动第二个进程**；唤起已运行实例的面板并聚焦 |
| 应用已注册开机自启，用户手动启动 | 同上（这是自启 + 手动双击的必然场景） |
| 第二实例进程 | 立即退出，把命令行参数转交首实例 |

**为什么必须**（本项目判断）：注册开机自启后用户手动双击 → 必然两个进程 → 两套托盘图标、重复写注册表、**两进程并发原子写（进程内锁跨进程无效）导致 rename 竞争丢更新**。修复成本极低（一个插件 + 二次启动唤起），不做的代价是用户对「为什么有两个图标 / 为什么设置丢了」的直接不信任。

**「关闭面板」vs「退出应用」**：

| 动作 | 语义 | 实现 |
|------|------|------|
| 关闭面板窗口（点 X / 失焦 / Esc） | **仅隐藏**，应用继续常驻 | `WindowEvent::CloseRequested` → `api.prevent_close()` + `window.hide()` |
| 托盘菜单「退出」 | **真正退出进程** | 置退出标志 → `app.exit(0)`（绕过 close 拦截） |

**平台差异**：macOS 关窗不退出是系统惯例（§1.6.7 Accessory 模式）；**Windows 上关窗默认退出进程**，必须显式拦截 `CloseRequested` 改为 hide——否则「常驻」在 Windows 不成立。

#### 1.8 无凭证首启与平台可达性降级（PRD 缺口补全）

PRD 原文 §5.1 默认「存在可加载配置」，未覆盖全空场景；§5.4 仅给 Claude 一句引导。以下为**本项目定义**：

**首次启动空状态**（无任何已启用账户）：

```
面板首屏（非空白页，而是引导卡）:
┌─────────────────────────────────────┐
│  还没有配置任何账户                  │
│                                      │
│  已自动检测到：                      │
│   ✓ 智谱（ZCode 凭证，可直接使用）   │  ← 若检测到本地凭证
│   ✗ Claude（未找到凭证）             │
│                                      │
│  [一键启用智谱]  [手动添加 API Key]  │
└─────────────────────────────────────┘
托盘：灰色 --，tooltip「未配置任何账户，点击添加」
```

**平台可达性前置条件与降级**（每个百分比源的前置不满足时）：

| 源 | 前置条件 | 不满足时的降级与引导 |
|----|---------|---------------------|
| 智谱 | 本机已装并登录 ZCode，或手填 API Key | 检测不到本地凭证 → 引导卡提示「可手填 API Key」；两者皆无 → 该源标 `not_configured` |
| MiniMax | 手填 API Key | 无 Key → `not_configured`，面板显示输入引导 |
| Claude | 本机存在 `.credentials.json`（跨平台）或 macOS 钥匙串 | 未找到 → 面板提示「请先在 [CC] CLI 中执行 /login」 |
| Kimi | 用户完成设备码登录 | 未登录 → 面板显示「登录 Kimi」按钮，走设备码流程（§5.5） |

**统一原则**：**任一源的前置不满足都只影响该源自身**，不阻塞其他源、不阻塞托盘（有数据的源照常聚合，§1.6.5a 部分失败语义）。

---

### M2: 配置管理（Coding-Plan 移植 + Tauri 改写）

#### 2.1 配置文件位置

```
统一目录: ~/.aicodequota/（ZBar 同款模式——pricing/quota_history/accounts/
         provider_credentials 等 Rust 模块均以 home 下固定目录为根，
         原样搬迁移成本最低；Windows 即 %USERPROFILE%\.aicodequota\）
config.json:      ~/.aicodequota/config.json
开发模式:         ~/.aicodequota/config.dev.json（tauri dev 运行时使用）
```

#### 2.2 配置结构

```typescript
interface AppConfig {
  refreshInterval: number;                        // 秒，默认 30，可调 30~1800；控制所有 Provider 的额度轮询周期（§5.1 的错峰首刷 + 周期刷新均以此为准）
  providers: {
    zai:       { accounts: ZaiAccount[] };
    deepseek:  { accounts: DeepSeekAccount[] };
    minimax:   { accounts: MiniMaxAccount[] };
    claude:    { accounts: ClaudeAccount[] };
    kimi:      { accounts: KimiAccount[] };
  };
  display: {
    colorThresholds: { green: number; yellow: number };  // 默认 50/20（唯一来源，M1.6/验收均引用此值）
  };
  autoStart: boolean;                              // 默认 false，经 tauri-plugin-autostart 落地（M1.6.6 菜单不含此项，只在设置页切换）
  memorySavingMode: boolean;                       // 默认 false，仅作用于 panel 窗口（HUD 窗口本身轻量，常驻）
  popupTrigger: 'hover' | 'click';                 // 默认 'hover'；托盘悬停是否唤起面板（M1 显示模式状态机的配置入口）
  language: 'zh-CN' | 'en-US';                    // 默认 zh-CN；优先级：config.language > navigator.language 检测
  theme: 'light' | 'dark' | 'auto';              // 默认 auto；AppConfig.theme 为唯一持久化真源，localStorage 仅作渲染层启动加速缓存
  windowPinMode: WindowPinMode;                   // 默认 unpinned；隐藏时自动重置回 unpinned（不持久化）
  rememberPopupPosition: boolean;                 // 默认 true
  popupPosition?: { x: number; y: number };
  popupSize?: { width: number; height: number };  // 高度记忆（无条件保存，M1.1 尺寸口径）
  autoCheckUpdate: boolean;                       // 默认 true
  trayDisplayRule: string;                        // 默认 'lowest'
  zaiAutoSwitch: ZaiAutoSwitchConfig;             // 智谱账号自动切换配置
  sessionHudEnabled: boolean;                     // 默认 false；为 true 时启动即创建 HUD 窗口（M6）
  sessionHudConfig?: SessionHudConfig;
}

interface DeepSeekAccount {
  id: string;                  // 8 位 hex
  enabled: boolean;
  label: string;
  apiKey?: string;             // 加密存储；当前唯一认证方式（预留 authMode 扩展见 M3.3，未启用前不落该字段）
}

interface ZaiAccount {
  id: string;                  // 8 位 hex（注意：与 M10 AccountSnapshot 的快照 id 是两套标识——
                               //   ZaiAccount 是「本应用内配置项」，快照是「ZCode 凭证文件的整存副本」，
                               //   authMode=local 时二者通过当前登录指纹关联，快照 id 不写入 providers.zai）
  enabled: boolean;
  label: string;
  authMode: 'local' | 'apikey';   // ★ 双源：local=读 ZCode 凭证 / apikey=手动输入
  apiKey?: string;            // 加密存储
  level?: string;
}

interface MiniMaxAccount {
  id: string;
  enabled: boolean;
  label: string;
  apiKey?: string;            // 加密存储
  region?: 'cn' | 'global';   // 默认 cn；global → api.minimax.io，cn/缺省 → api.minimaxi.com（M3.5）
}

interface ClaudeAccount {
  id: string;
  enabled: boolean;
  label: string;
  // Claude 无主动登录流程：凭证从本地读取（.credentials.json 跨平台 / macOS 钥匙串兜底），
  // 本结构仅在多份本地凭证间做启用/标注；oauthToken 不由用户手填
  credentialRef?: string;     // 本地凭证标识（自动发现后回填）
}

interface KimiAccount {
  id: string;
  enabled: boolean;
  label: string;
  region?: 'cn' | 'global';   // 默认 cn；设备码登录端点按 region 选择（M3.7）
  oauthToken?: string;        // 加密存储（设备码登录获得）
  oauthRefreshToken?: string; // 加密存储
  expiresAt?: number;
}

interface ZaiAutoSwitchConfig {
  enabled: boolean;               // 默认 false
  thresholdPercent: number;       // 剩余额度低于此值触发切换，默认 10
  candidateIds: string[];         // 备用账号快照 id 列表（M10 AccountSnapshot.id，按优先级）
}
```

#### 2.3 6 步自动迁移（Coding-Plan 移植）

| # | 迁移逻辑                          | 说明                                                                                            |
| - | --------------------------------- | ----------------------------------------------------------------------------------------------- |
| 1 | 旧单账户 → 多账户                | `provider.apiKey: string` → `provider.accounts: [{id, enabled, apiKey, label}]`            |
| 2 | `opencodego` → `opencode-go` | 本项目首版无存量用户配置，实际不会触发；保留此步仅作 Coding-Plan 迁移逻辑的同构复刻，便于未来承接其用户 |
| 3 | 补齐缺失 provider                 | 遍历**编译期 provider 注册表**（当前为 zai/deepseek/minimax/claude/kimi，来源同 get_available_providers），缺失则 `providers[key] = {accounts:[]}`——注册表驱动而非硬编码，新增 provider 自动覆盖 |
| 4 | 移除`updateInfo` 字段           | 更新状态只在内存，不落盘                                                                        |
| 5 | `autoCheckUpdate` 默认开        | `typeof !== 'boolean'` → 置 `true`                                                         |
| 6 | 移除更新调度字段                  | 删除`autoCheckUpdateInterval` / `lastAutoCheckTime`（改为固定 4h 调度）                     |

**加载失败处理**（从 Coding-Plan 搬）：

| 错误类型                 | 行为                                                        |
| ------------------------ | ----------------------------------------------------------- |
| 文件不存在（ENOENT）     | 创建默认配置                                                |
| JSON 损坏（SyntaxError） | 先备份为`config.corrupt-{YYYYMMDD-HHMMSS}.json`，再建默认 |
| 其他 IO 错误（EACCES）   | 仅在备份成功后建默认，否则抛错终止启动                      |

#### 2.4 加密（Coding-Plan 移植 + Tauri 改写）

**问题**：Electron 的 `safeStorage`（Windows DPAPI / macOS Keychain）在 Tauri 2 中没有直接等价物。

**方案**：使用 `keyring` crate（跨平台凭证存储）或自行实现。

**推荐方案 A（简单）**：保留 `enc:` + base64 前缀格式，但用 **AES-256-GCM + 机器指纹派生密钥** 替代 DPAPI：

```rust
fn credential_secret() -> Vec<u8> {
    // 优先环境变量（测试用）
    if let Ok(s) = std::env::var("AICODEQUOTA_SECRET") {
        if !s.is_empty() { return s.into_bytes(); }
    }
    // 回退：机器指纹
    let platform = if cfg!(target_os="macos") { "darwin" }
                   else if cfg!(target_os="windows") { "win32" }
                   else { "linux" };
    let home = dirs::home_dir().map(|p| p.to_string_lossy().to_string()).unwrap_or_default();
    let username = std::env::var("USER").or_else(|_| std::env::var("USERNAME")).unwrap_or_default();
    format!("aicodequota-fallback:{platform}:{home}:{username}").into_bytes()
}
```

密文格式与 ZCode 一致：`enc:v1:<nonce_b64url>.<tag_b64url>.<cipher_b64url>`，`key = SHA256(secret)`，nonce 12 字节。secret 即「机器指纹派生密钥源」（上下文的 machine_secret / machine_fingerprint / credential_secret 均指此处 `credential_secret()` 的返回值）。

**兼容旧配置**：检测到 `enc:` + base64（Electron DPAPI 格式但非 v1 三段式）→ 尝试 DPAPI 解密（Windows 可行），失败则保留原文。

#### 2.5 原子写入（Coding-Plan 移植）

```rust
async fn save(config: &AppConfig) -> Result<(), String> {
    // 串行化写入队列，防并发竞争同一 tmp 文件
    let run = SAVE_QUEUE.lock().await;
    let tmp_path = format!("{}.tmp", CONFIG_PATH);
    // ① 写临时文件
    let mut file = File::create(&tmp_path).await?;
    file.write_all(content.as_bytes()).await?;
    file.sync_all().await?;     // fsync 落盘
    drop(file);
    // ② rename + Windows EPERM/EBUSY 重试
    for attempt in 1..=3 {
        match tokio::fs::rename(&tmp_path, &CONFIG_PATH).await {
            Ok(_) => break,
            Err(e) if (attempt < 3) && (e.kind() == EPERM || e.kind() == EBUSY) => {
                sleep(100 * attempt).await;
            }
            Err(e) => { let _ = tokio::fs::remove_file(&tmp_path).await; return Err(e.to_string()); }
        }
    }
    Ok(())
}
```

#### 2.6 文件监听去抖（Coding-Plan 移植）

```rust
// 监听 config.json
let _watcher = notify::recommended_watcher(move |res| {
    if let Ok(event) = res {
        let now = Instant::now();
        // 抑制窗口：自身保存 500ms 内的变更忽略
        if now.duration_since(last_self_save) < Duration::from_millis(500) { return; }
        // 去抖：100ms 后的第一次事件才处理
        // ...
    }
});
```

---

### M3: Provider 适配层

#### 3.1 通用 Provider 接口（Rust）

```rust
pub trait Provider {
    /// 返回该 provider 的展示名
    fn name(&self) -> &str;
    /// 查询额度/用量，返回统一的 QuotaEntry
    /// account 参数为 §2.2 五种 Account 结构的枚举封装（Rust enum ProviderAccount），
    /// 各模块按需匹配自己的变体；ZBar 原实现的函数式模块（fetch_quota_entries）
    /// 在外层适配为该 trait
    async fn fetch_quota(&self, account: &ProviderAccount) -> Result<Vec<QuotaEntry>, String>;
}

// 统一输出类型（对应 ZBar 的 provider_quota.rs）
#[derive(Serialize, Clone)]
#[serde(rename_all = "camelCase")]
pub struct QuotaEntry {
    pub account_id: String,
    pub label: String,
    pub status: QuotaStatus,   // ok | expired | error | pending | not_configured
    pub windows: Vec<QuotaWindow>,
    pub balance: Option<Balance>,
    pub plan_name: Option<String>,
    pub message: Option<String>,
    pub updated_at: i64,
}

#[derive(Serialize, Clone)]
#[serde(rename_all = "camelCase")]
pub struct QuotaWindow {
    pub key: String,              // "hour5" | "weekly" | "mcp" | "balance"
    pub title: String,
    pub used_percent: Option<f64>,
    pub used: Option<f64>,
    pub total: Option<f64>,
    pub unit: Option<String>,
    pub resets_at: Option<i64>,
}

#[derive(Serialize, Clone)]
#[serde(rename_all = "camelCase")]
pub struct Balance {
    pub amount: f64,
    pub currency: String,      // "CNY" | "USD"
    pub granted: Option<f64>,
    pub topped_up: Option<f64>,
}
```

#### 3.2 智谱 Z.ai 额度（M3.1）

**双源认证策略**（用户确认：本地 ZCode 凭证优先 + API Key 兜底）：

```
优先级：
1. 本地 ZCode 凭证（authMode = 'local'）
   a. 读 {zcode_v2_dir}/credentials.json
      - key 形态：account-provider:coding-plan:account:<planId>:account:<identity>:api-key
      - 值是 enc:v1: 密文，用 zcode_crypto.rs 解密
   b. 若未命中 → 读 {zcode_v2_dir}/config.json
      - provider map 中优先 "builtin:bigmodel-coding-plan" → "builtin:zai-coding-plan"
      - 两键都缺 → 回退任意 key 含子串 "coding-plan" 且 options.apiKey 非空的 provider
        （`builtin:*-start-plan` 不含该子串，天然排除）
      - apiKey 是明文（config.json 的 apiKey 是明文）
   c. base_url 从 config.json 取（credentials.json 没有端点信息）
2. 手动 API Key（authMode = 'apikey'）
   - 用户在设置页手动输入 Base URL + API Key
```

**API 请求**（从 ZBar 搬）：

```rust
fn base_from_provider_url(url: &str) -> &'static str {
    if url.contains("z.ai") { "https://api.z.ai" } else { "https://open.bigmodel.cn" }
}

fn query_quota(api_key: &str, base_url: &str) -> Result<QuotaEntry, String> {  // QuotaEntry 定义见附录 A.1
    let base = base_from_provider_url(base_url);
    let url = format!("{base}/api/monitor/usage/quota/limit");
    let resp: QuotaResponse = ureq::get(&url)
        .set("Authorization", api_key)  // 裸 token，无 Bearer 前缀
        .timeout(Duration::from_secs(15))
        .call()?.into_json()?;
    // 解析 limits 数组，按 unit+number 拆分为 hour5/weekly/mcp
}
```

**窗口拆分逻辑**（从 ZBar 搬，保持一致）：

- `mcp = limits.find(kind == "TIME_LIMIT")`
- `hour5 = TOKENS_LIMIT && unit==3 && number==5`
- `weekly = TOKENS_LIMIT && unit==6 && number==1`
- 多条匹配时按 `next_reset_time` 升序取最小

**套餐代际识别**（从 Coding-Plan 补充）：

- V1: 仅 5h 额度（TOKENS_LIMIT）
- V2: 5h + 周额度
- V3: 积分制（CREDIT_LIMIT）

**今日增量**（从 ZBar 搬）：

```rust
// 严格按账号分组，今日增量 = 峰值 weekly_pct - 首条 weekly_pct
let delta = peak_pct.saturating_sub(first_pct);
```

**多智谱账号**：详见 M10 模块。

#### 3.3 DeepSeek 额度（M3.2）

**单模认证：仅 API Key**（用户明确要求不做网页登录）。

##### API Key 模式

```rust
const BALANCE_URL: &str = "https://api.deepseek.com/user/balance";
// GET, headers: Authorization: Bearer {api_key}, Accept: application/json
// 响应: {"is_available": bool, "balance_infos": [{"currency":"CNY",
//          "total_balance":"110.00", "granted_balance":"10.00",
//          "topped_up_balance":"100.00"}]}  ← 全是字符串数字
```

**余额映射**：

```rust
// total_balance     → balance.amount
// granted_balance   → balance.granted
// topped_up_balance → balance.topped_up
```

**状态判定**：

| 条件                                 | status    | message                   |
| ------------------------------------ | --------- | ------------------------- |
| HTTP 200 且 `is_available == true`   | `ok`      | —                         |
| HTTP 401 / 403                       | `expired` | "API Key 无效或已过期"    |
| `is_available == false` 且余额非 0   | `ok`      | "余额暂不可用于 API 调用" |
| `is_available == false` 且余额为 0    | `expired` | "余额耗尽，请充值后重试"  |
| 其他网络错误                         | `error`   | 脱敏后的错误文本          |

**移除网页登录带来的简化**：

- 不需要 `web_login.rs` 模块
- 不需要 WebView 登录窗口（`WebviewWindowBuilder` + `eval()` + 独立 `data_directory`）
- 不需要 Token 自动刷新定时器
- 配置结构中 `DeepSeekAccount` 只有 `apiKey` 一个敏感字段

##### 预留扩展点

若后续需要网页登录，接口层已预留：

```rust
// auth_mode 字段预留在配置 schema 中（当前只有 "apikey" 一个合法值）
// 新增 "weblogin" 时只需扩展此分支，不影响调用方
pub enum DeepSeekAuthMode {
    ApiKey, // 当前唯一实现
    // WebLogin,  // 预留：后续如需支持，取消注释并实现 web_login.rs
}
```

#### 3.4 DeepSeek 服务状态（M3.3）

**Flashcat API**（从 Coding-Plan 移植）：

```rust
const PAGE_ID: &str = "6410630422455";
const BASE_URL: &str = "https://status.deepseek.com/api/status-page";
const STATUS_DAYS: usize = 90;
const STATUS_CACHE_TTL_MS: u64 = 5 * 60 * 1000;  // 5 分钟缓存
```

**两个并行请求**：

```
当前状态: {BASE_URL}/{PAGE_ID}/summary/active
90 天历史: {BASE_URL}/{PAGE_ID}/summary/structure?start_at_from_seconds={start}&start_at_to_seconds={end}
```

**响应结构**：

```rust
#[derive(Deserialize)]
struct FlashcatActiveResponse {
    data: Option<FlashcatActiveData>,
}
#[derive(Deserialize)]
struct FlashcatActiveData {
    page: Option<FlashcatPage>,
    active_changes: Vec<ActiveChange>,
}
#[derive(Deserialize)]
struct ActiveChange {
    components: Vec<ActiveComponent>,
}
#[derive(Deserialize)]
struct ActiveComponent {
    component_id: String,
    new_status: String,  // full_outage | partial_outage | degraded | operational
}

#[derive(Deserialize)]
struct FlashcatStructureResponse {
    data: Option<FlashcatStructureData>,
}
#[derive(Deserialize)]
struct FlashcatStructureData {
    component_impacts: Vec<ComponentImpact>,
}
#[derive(Deserialize)]
struct ComponentImpact {
    component_id: String,
    start_at_seconds: i64,
    end_at_seconds: i64,
    status: String,  // full_outage | partial_outage | degraded | maintenance
}
```

**可用率计算**（从 Coding-Plan 搬）：

```rust
fn calc_uptime(days: &[DayStatus], total_minutes: f64, down_minutes: f64) -> f64 {
    let uptime_pct = ((total_minutes - down_minutes) / total_minutes * 100.0).max(0.0);
    (uptime_pct * 100.0).round() / 100.0  // 保留 2 位小数
}
```

**输出类型**（前端展示）：

```typescript
interface DeepSeekServiceComponent {
  id: string;
  name: string;
  status: 'operational' | 'degraded_performance' | 'partial_outage' | 'major_outage';
  days: ('operational' | 'degraded' | 'outage' | 'maintenance')[];  // 长度 90，索引 0 最早
  uptime: number;  // 0~100
}
```

**组件 ID → 友好名映射**（从 Coding-Plan 搬）：

```
01KR3NC9ETZYF436Z8YT1HM047 → i18n('quota.deepseekApiService')
01KR3NC9ETESRRQ4GABE0TGW53 → i18n('quota.deepseekWebService')
```

#### 3.5 MiniMax 额度（M3.4）

**从 Coding-Plan + ZBar 合并**：

```rust
const PRIMARY_PATH: &str = "/v1/token_plan/remains";
const FALLBACK_PATH: &str = "/v1/api/openplatform/coding_plan/remains";

fn host_for_region(region: Option<&str>) -> &'static str {
    match region {
        Some("global") => "api.minimax.io",
        _ => "api.minimaxi.com",
    }
}
```

**区域支持**（来自 ZBar 的 `region: cn | global`）：

```rust
// ZBar: region 字段是 provider_credentials.rs 的通用字段
// MiniMax: global → api.minimax.io / cn → api.minimaxi.com
```

**响应解析**（从 ZBar 搬完整逻辑）：

```rust
// 双信封兼容
// A: {"base_resp":{}, "data":{"model_remains":[...]}}
// B: {"model_remains":[...], "base_resp":{}}

// model_remains 多条时取剩余百分比最低的做主窗口
// 剩余百分比 = *_remaining_percent
// 已用百分比 = 100 - 剩余百分比
// status_code == 1004 或 status_msg 含 "cookie"/"login" 或 HTTP 401/403 → expired
```

#### 3.6 Claude Code 导入（M3.5）

**从 ZBar 搬**（完整移植 claude.rs）：

##### 会话 Token 导入

```
数据源: ~/.claude/projects/**/*.jsonl（项目目录递归，深度受限；文件名即会话 uuid，无固定 sessions/ 层级）
解析: TranscriptLine { timestamp, type, message: { model, usage: {
         input_tokens, output_tokens,
         cache_read_input_tokens, cache_creation_input_tokens }}}
session_id = 文件名（<uuid>.jsonl → <uuid>）
```

##### OAuth 实时额度

```
数据源: https://api.anthropic.com / claude.ai API
认证: OAuth access_token，按以下优先级自动发现（跨平台，ZBar 同款实现）:
  ① 环境变量 AICODEQUOTA_CLAUDE_HOME（指向 .claude 根目录，测试/自定义用）
  ② ~/.claude/.credentials.json（Windows: %USERPROFILE%\.claude\.credentials.json）——全平台通用
  ③ macOS 专用: 登录钥匙串（security find-generic-password -s "[CC]-credentials" -w）
     ★ 仅 #[cfg(target_os = "macos")] 编译；Windows/Linux 走 ①②
  三者任一命中即用；全部未命中 → 面板显示「未找到 Claude 登录凭证（.credentials.json / 钥匙串）」
额度窗口: 5h / 周 / Sonnet周 / Opus周 / 超额
缓存: LIVE_LIMITS_CACHE，成功 60s / 失败 15s TTL
```

##### 流式修订补传（ZBar 核心特性）

**问题**：同一 `message.id` 的中间值可能先入库并被上传，终值稍后覆盖本地行（`id` 不变）。常规 `id > 游标` 查询永远选不出它。

```rust
// 双游标查询：updated_at 时间游标（选修订行）+ id id 游标（分页推进）
pub fn query_revised_since(since_ts: i64, after_id: i64, limit: usize)
    -> Result<Vec<UsageRow>, String> {
    // SELECT * FROM model_usage
    // WHERE updated_at > ?1 AND id > ?2
    // ORDER BY id ASC LIMIT ?3
}
```

服务端以「总量更大者胜」的 upsert 覆盖修正。

#### 3.7 Kimi Code 导入（M3.6）

**从 ZBar 搬**（完整移植 kimi.rs + kimi_oauth.rs）：

##### 会话 Token 导入

```
数据源: ~/.kimi-code/sessions/<sid>/agents/<aid>/wire.jsonl
解析: WireLine { type, model, usage: {
         inputOther, output,
         inputCacheRead, inputCacheCreation }, time }
时间戳: time 是毫秒（直接用）
```

##### 设备码 OAuth 登录

```
1. POST {auth_base}/api/oauth/device_authorization
   （auth_base: region=global → https://auth.kimi.ai，默认/region=cn → https://auth.kimi.com）
   → 返回 { device_code, user_code, verification_uri, interval, expires_in }
2. 用户在浏览器打开 verification_uri 输入 user_code
3. 客户端轮询 POST {auth_base}/api/oauth/token（grant_type: urn:ietf:params:oauth:grant-type:device_code）
   → 成功返回 { access_token, refresh_token, expires_in }
4. 存储 refresh_token，自动刷新 access_token
```

##### 实时额度

```
额度窗口: 5h / 周 / 月 / 加油包
缓存: 同 Claude，LIVE_LIMITS_CACHE TTL 60s/15s
```

---

### M4: 价格与成本

#### 4.1 价格配置（从 ZBar 搬 pricing.rs）

**存储**：`~/.aicodequota/pricing.json`

```json
{
  "usd": {
    "glm-5.3-flash": { "input": 0.0008, "output": 0.002, "cache_read": 0.0002 }
  },
  "hidden": ["some-deleted-model"]
}
```

**模型名匹配链**（从 ZBar 搬；俗称「三级匹配」= L0 归一化预处理 + L1/L2 两级查找 + L3 未命中兜底）：

```
L0 归一: model_id.to_lowercase() → replace('.', '-')
         "claude-sonnet-4.5" ↔ "claude-sonnet-4-5" 视为同一写法

L1 精确/点号归一: map.get(lc) 或 norm_index[normalize_dots(lc)]
                   命中时 reference_id = None（前端不标注来源）

L2 变体渐进去尾: "gpt-5.6-sol" → "gpt-5.6" → "gpt-5"
                   每级同时试原始与点号归一形态
                   命中时 reference_id = Some(命中的 id)（前端标注"参考自 xxx"）

L3 未命中 → missing（未配价警示）
```

**内置参考表**（从 ZBar 搬）：

```
public/pricing-defaults.json（编译期 include_str! 嵌入）
来源: models.dev 白名单 17 家
白名单: anthropic, openai, google, zai, deepseek, moonshotai, moonshotai-cn,
        minimax, xai, mistral, cohere, alibaba, qwen, stepfun, perplexity,
        amazon-bedrock, azure
```

#### 4.2 models.dev 在线同步（从 ZBar 搬）

```rust
const MODELSDEV_URL: &str = "https://models.dev/api.json";
const MODELSDEV_CACHE: &str = "modelsdev-cache.json";

fn sync_modelsdev_pricing() -> Result<ModelsdevSyncResult, String> {
    // 1. 下载 models.dev/api.json（约 5MB）
    // 2. 缓存到 modelsdev-cache.json
    // 3. 提取白名单 provider 的价格
    // 4. 与本地 pricing.json 做 diff
    // 返回 ModelsdevSyncResult { diff, fetched_at, from_cache }
}
```

**成本计算**（从 ZBar 搬）：

```rust
fn compute_cost(usage: &UsageRow, pricing: &PricingConfig) -> f64 {
    let non_cache_input = usage.input_tokens - usage.cache_read_tokens;
    let input_cost = non_cache_input as f64 * pricing.input;
    let output_cost = usage.output_tokens as f64 * pricing.output;
    let cache_cost = usage.cache_read_tokens as f64 * pricing.cache_read;
    input_cost + output_cost + cache_cost
}
```

#### 4.3 汇率（从 ZBar 搬；成功结果写入 ~/.aicodequota/currency.json 缓存 { rate, source, fetched_at }，§7.3 数据表同源）

```rust
const FX_RATE_SOURCES: &[(&str, &str)] = &[
    ("er-api",           "https://open.er-api.com/v6/latest/USD"),
    ("frankfurter",      "https://api.frankfurter.app/latest?from=USD&to=CNY"),
    ("exchangerate-api", "https://api.exchangerate-api.com/v4/latest/USD"),
];

fn fetch_fx_rate() -> Result<(f64, String), String> {
    for (name, url) in FX_RATE_SOURCES {
        if let Ok(rate) = fetch_and_validate(url) {
            if (5.0..=15.0).contains(&rate) { return Ok((rate, name.to_string())); }
        }
    }
    Err("所有汇率源均失败".into())
}
```

---

### M5: 额度快照与周对比

#### 5.1 额度快照存储（quota_history.jsonl，周对比数据源；从 ZBar 搬 quota_history.rs）

> **与 MVP 解耦说明**：快照写入是 `fetch_zai_quota` 成功后的副产物（每次成功查询 append 一条）。
> **MVP 阶段不展示周对比（F10 不做），但仍照常写快照**——理由：①写入成本极低（一次 append）；
> ②避免 MVP→v1.0 升级时历史数据从零开始；③M5.2 的采样前后指纹校验已是 fetch 链路的一部分，
> 去掉写快照反而要额外剥离逻辑。即「采集与展示解耦」：采集始终开，展示按版本灰度。

**文件**：`~/.aicodequota/quota_history.jsonl`

```rust
#[derive(Serialize, Deserialize, Clone)]
pub struct QuotaSnapshot {
    pub ts: i64,                       // 采样毫秒
    pub account: Option<String>,        // 账号指纹，None = 旧版本
    pub level: String,
    pub weekly_pct: u32,
    pub weekly_reset: Option<i64>,
    pub hour5_pct: u32,
    pub mcp_pct: u32,
    pub mcp_used: Option<i64>,
    pub mcp_total: Option<i64>,
}
```

**滚动清理**（90 天，从 ZBar 搬）：

```rust
const RETENTION_MS: i64 = 90 * 86_400_000;
const CLEANUP_INTERVAL: Duration = Duration::from_secs(24 * 3600);

fn maybe_cleanup(path: &PathBuf) {
    // 写入前调用，内部 24h 节流
    // 逐行读，只取 ts 字段判断过期（不严格反序列化）
    // 无 ts 的损坏行保留
    // removed == 0 直接返回
    // 否则写临时文件 → rename 替换
}
```

**同秒防抖**（从 ZBar 搬）：

```rust
fn append_snapshot(snap: &QuotaSnapshot) -> Result<(), String> {
    // 按【账号】回读尾部 2048B 窗口，检查是否同账号同秒
    // 有则跳过，防止 30s 轮询在同秒内重复写入
}
```

#### 5.2 采样时序防账号错配（从 ZBar 搬）

```rust
pub fn fetch_quota() -> Result<QuotaEntry, String> {  // 内部解析中间结构 QuotaParsed 与附录 A.1 字段一致
    // ① 指纹必须在查询【前】取（查询本身最长 15s，期间的切换事务会换凭证文件）
    let account_before = current_fingerprint();
    let result = query_quota()?;
    // ② 复核：查询期间发生切换 → 丢弃采样（最坏丢一条 30s 采样），绝不写错配数据
    if account_before != current_fingerprint() {
        return Ok(result);  // 不写快照
    }
    append_snapshot(&snapshot_of(&result));
    Ok(result)
}
```

#### 5.3 周对比（从 ZBar 搬）

**周期切分**：

```rust
#[derive(Serialize)]
#[serde(rename_all = "camelCase")]
pub struct WeeklyPeriod {
    pub reset_at: i64,
    pub end_at: i64,
    pub is_current: bool,
    pub pct_start: u32,
    pub pct_peak: u32,
    pub pct_end: u32,
    pub sample_count: u32,
}

pub fn split_periods(snaps: &[QuotaSnapshot]) -> Vec<WeeklyPeriod> {
    // 跳变检测: jumped = (cur_reset > prev_reset + 86_400_000)
    // 统一分界: boundary = prev_reset.unwrap_or(snaps[i].ts).min(snaps[i].ts)
    //           相邻周期共用同一边界，区间无缝不重叠
    // 已结束周期 end_at = next_reset
    // 当前周期 end_at = now_ms
}
```

**今日增量**：

```rust
pub fn today_delta() -> Result<(u32, u32), String> {
    let today_start = local_today_start_ms();
    let today_snaps: Vec<_> = load_all()?.into_iter()
        .filter(|s| s.ts >= today_start)
        .collect();
    if today_snaps.is_empty() { return Ok((0, 0)); }
    let peak = today_snaps.iter().map(|s| s.weekly_pct).max().unwrap_or(0);
    let first = today_snaps.first().map(|s| s.weekly_pct).unwrap_or(0);
    Ok((peak.saturating_sub(first), today_snaps.len() as u32))
}
```

---

### M6: 会话悬浮窗（HUD）

#### 6.1 窗口规格（从 ZBar 搬）

| 属性          | 值                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------ |
| Label         | `session-hud`                                                                            |
| 默认尺寸      | 300 × 自适应                                                                              |
| 最小尺寸      | 260 × 160                                                                                 |
| 最大尺寸      | 4096 × 4096                                                                               |
| 特性          | `decorations:false, transparent:true, alwaysOnTop:true, focusable:false, resizable:true` |
| 8 方向 resize | `session_hud_resize_begin/end` 命令                                                      |

#### 6.2 高度自适应公式（从 ZBar 搬）

```rust
const HUD_HEADER_H: f64 = 30.0;        // 头部拖动区
const HUD_ROW_H: f64 = 56.0;          // 会话行（三行制）
const HUD_MORE_H: f64 = 20.0;         // 折叠行
const HUD_TODAY_H: f64 = 20.0;        // 今日合计行
const HUD_MODEL_ROW_H: f64 = 20.0;    // 模型速度行
const HUD_MAX_MODEL_ROWS: usize = 3;
const HUD_LIST_BORDER_H: f64 = 1.0;
const HUD_MODELS_BORDER_H: f64 = 1.0;
const HUD_BOTTOM_PAD: f64 = 8.0;
const HUD_EMPTY_HEIGHT: f64 = 64.0;

fn hud_height(
    visible_rows: usize,
    folded: bool,
    today: bool,
    model_rows: usize,
    font_scale: f64,
) -> f64 {
    let mut h = HUD_HEADER_H;
    h += HUD_LIST_BORDER_H;
    h += visible_rows as f64 * HUD_ROW_H;
    if folded { h += HUD_MORE_H; }
    if today { h += HUD_TODAY_H; }
    h += HUD_MODELS_BORDER_H;
    h += (model_rows.min(HUD_MAX_MODEL_ROWS) as f64) * HUD_MODEL_ROW_H;
    h += HUD_BOTTOM_PAD;
    h * font_scale
}
```

#### 6.3 活跃会话判定（从 ZBar 搬）

**两个信号取并集**：

```sql
-- 信号 1：model_usage（每笔模型请求完成即写，实时性最好）
SELECT session_id, MAX(started_at) FROM (
  SELECT session_id, started_at FROM model_usage
  WHERE started_at >= ?1
  ORDER BY started_at DESC LIMIT 2000     -- MODEL_USAGE_SCAN_ROWS
) GROUP BY session_id ORDER BY 2 DESC LIMIT 24;

-- 信号 2：message（用户发消息即写，覆盖首笔请求未返回的空窗期）
SELECT id, session_id, time_created,
       json_extract(data, '$.role') AS role
FROM (SELECT id, session_id, time_created, data
      FROM message ORDER BY rowid DESC LIMIT 400)  -- MESSAGE_TAIL_ROWS
WHERE time_created >= ?2 ORDER BY time_created DESC, id DESC;
```

**时间窗档位**：

```rust
const HUD_WINDOW_MINUTES_OPTIONS: [i64; 4] = [5, 10, 30, 0];  // 0 = 不限
const DEFAULT_WINDOW_MINUTES: i64 = 10;
const UNLIMITED_FALLBACK_MINUTES: i64 = 24 * 60;  // 不限档兜底
const MAX_VISIBLE_SESSIONS: usize = 5;  // 超出折叠为「还有 N 个会话」只读行（focusable:false 窗口不适合交互展开）
```

#### 6.4 展示内容（从 ZBar 搬）

**会话树**（主会话 + 子代理合并）：

- 子代理活动归并到主会话（`last_active_at` 取 max）
- 子代理永不单独成行（防双计）
- 累计为树内 `model_usage` 合计
- 排序按最近活动倒序

**字段口径**：

```
↑ = Σ max(0, input − cache_read)
↓ = Σ output
⟲ = Σ cache_read
Σ = ↑+↓+⟲
× = model_usage 行数
TTFT = first_token_at − started_at 的加权平均
TPS = output_tokens × 1000 / (completed_at − first_token_at)
```

**样式**（从 ZBar 搬，原生 HTML + DOM 操作，不走 Vue）：

- 全部尺寸用 `calc(Npx * var(--hud-scale))`
- 颜色全部收敛为 `--hud-*` CSS 变量
- 亮色：`--hud-bg: rgb(255,255,255)`（必须完全不透明）
- 暗色：`--hud-bg: rgb(10,14,24)`
- 等宽字体栈：`ui-monospace, SFMono-Regular, Menlo, Consolas`

#### 6.5 轮询节流（从 ZBar 搬）

```rust
const FEED_TICK_MS: u64 = 1000;  // 1s 一拍交替执行 DB 轮询拍与 rollout 速度拍
const HUD_WINDOW_OP_TIMEOUT: Duration = Duration::from_secs(10);
```

---

### M7: 国际化

#### 7.1 翻译文件结构（从 Coding-Plan 搬）

**文件位置**：`src/i18n/locales/zh-CN.json` + `en-US.json`

**Section 划分**（保留 Coding-Plan 的 key 结构，删除不适用功能）：

| Section        | 键数           | 说明                             |
| -------------- | -------------- | -------------------------------- |
| main           | ~50            | 通用 UI（标题、按钮、导航）      |
| settings       | ~80            | 设置页                           |
| quota          | ~90            | 额度相关（含 DeepSeek 服务状态） |
| zaiStats       | ~29            | 智谱统计面板                     |
| overview       | ~14            | 汇总页                           |
| tray           | ~10            | 托盘菜单                         |
| providers      | 5              | provider 显示名（5 个平台）      |
| claude         | ~20            | Claude Code 面板                 |
| accounts       | ~18            | M10 账号列表/切换浮层/自动切换设置 |
| kimi           | ~15            | Kimi Code 面板                   |
| hud            | ~15            | 会话悬浮窗                       |
| pricing        | ~20            | 价格配置                         |
| compare        | ~10            | 周对比                           |
| **合计** | **~360** | 中英完全对齐                     |

**命名规范**：

- `camelCase` 键 + 点分路径引用
- 占位符 `{name}`
- provider 显示名走 `providers.{key}`

#### 7.2 渲染层国际化（vue-i18n）

```typescript
// src/i18n/index.ts
import { createI18n } from 'vue-i18n';
import zhCN from './locales/zh-CN.json';
import enUS from './locales/en-US.json';

const detectLocale = (): string =>
  navigator.language.startsWith('zh') ? 'zh-CN' : 'en-US';

export const i18n = createI18n({
  legacy: false,
  locale: detectLocale(),
  fallbackLocale: 'zh-CN',
  messages: { 'zh-CN': zhCN, 'en-US': enUS },
});
```

#### 7.3 主进程国际化（从 Coding-Plan 搬，零依赖手写）

```rust
// src/i18n.rs
pub fn set_locale(locale: &str);
pub fn t(key: &str, params: Option<&HashMap<String, String>>) -> String;
// key.split('.') 逐层下钻；缺键返回 key 本身
// 参数替换: result.replace(/\{(\w+)\}/g, ...)
```

**消费点**：托盘菜单文案，6 个 key 与 M1.6.6 菜单一一对应——`tray.refresh` / `tray.settings` / `tray.checkUpdate` / `tray.resetPosition` / `tray.resetSize` / `tray.quit`（均 camelCase，实现于 src-tauri/src/i18n.rs）。

**语言切换广播**（从 ZBar 搬）：

```rust
app.emit("aicodequota://appearance-changed", ())  // HUD 等同源窗口监听后重读
```

---

### M8: 自动更新

#### 8.1 更新源（从 ZBar 搬双源方案）

```json
{
  "plugins": {
    "updater": {
      "endpoints": [
        "https://github.com/OWNER/aicodequota/releases/latest/download/latest.json"
      ],
      "pubkey": "..."
    }
  }
}
```

> **已确认（2026-10-09）**：GitHub 单源（Gitee 镜像不做，降低单维护者运维成本，后续需要时再加第二个 endpoint）。**开工前置项**：仓库归属（OWNER 占位）与 updater 签名密钥（`tauri signer generate`）由用户提供后回填。

#### 8.2 更新调度（从 Coding-Plan 搬）

```rust
const AUTO_UPDATE_INITIAL_DELAY_MS: u64 = 30_000;     // 启动 30s 后首次检查
const AUTO_UPDATE_INTERVAL_MS: u64 = 4 * 3600_000;    // 每 4h 检查一次
```

**阶段状态机**：

```
idle → checking → available → downloading → ready
             ↓           ↓
          noUpdate     error
```

---

### M9: 网络代理

**从 ZBar 搬**（三级解析：env > 系统 > 直连）：

```rust
pub fn resolve_proxy() -> Option<String> {
    // ① 环境变量（优先级最高）
    for key in ["HTTPS_PROXY", "https_proxy", "HTTP_PROXY", "http_proxy"] {
        if let Ok(v) = std::env::var(key) {
            if !v.trim().is_empty() { return Some(v.trim().to_string()); }
        }
    }
    // ② 系统代理
    system_proxy()
}

fn system_proxy() -> Option<String> {
    #[cfg(windows)]
    { /* winreg 读 HKCU\...\Internet Settings */ }
    #[cfg(target_os = "macos")]
    { /* scutil --proxy */ }
    #[cfg(not(any(windows, target_os = "macos")))]
    { None }
}
```

**ureq 代理构建**（从 ZBar 搬）：

```rust
pub fn http_agent(timeout_secs: u64) -> Result<ureq::Agent, String> {
    let proxy = load_proxy_config();
    let builder = ureq::AgentBuilder::new()
        .timeout_connect(Duration::from_secs(10))
        .timeout_read(Duration::from_secs(timeout_secs))
        .timeout_write(Duration::from_secs(timeout_secs));
    if proxy.is_empty() { return Ok(builder.build()); }
    // ureq 2 不支持 https:// 代理（无法与代理本身建 TLS）
    if proxy.get(..8).map_or(false, |h| h.eq_ignore_ascii_case("https://")) {
        return Err("暂不支持 https:// 开头的代理地址，请改用 http:// 或 socks5://".into());
    }
    Ok(builder.proxy(ureq::Proxy::new(&proxy)?).build())
}
```

---

### M10: 智谱多账号（快照 + 切换事务 + 自动切换）

**从 ZBar `accounts.rs` 完整移植**（源文件 2256 行；其中切换事务、快照存储、进程控制原样搬，M10.5 自动切换为本项目新增设计——ZBar 只提供了 `expect_fingerprint` 无人值守钩子，无完整自动切换实现）。

**事件常量**：`AUTO_SWITCH_DONE_EVENT = "aicodequota://zai-switch-done"`（手动/自动切换共用，payload `{ auto: bool, switchedTo: String }`；定义于 accounts.rs 顶部）。

#### 10.1 账号快照存储（accounts/<id>.account.json，切换事务数据源）

**目录**：`~/.aicodequota/accounts/`（权限 0700）

**快照文件**：`<id>.account.json`（权限 0600）

```rust
#[derive(Serialize, Deserialize)]
struct AccountSnapshot {
    version: i32,
    id: String,
    fingerprint: String,        // user_id
    display_name: String,
    email: Option<String>,
    created_at: i64,
    updated_at: i64,
    // ★ credentials.json 原文【整串】保存
    //   键集由 ZCode 自行演进，原文回写零损失
    credentials_raw: String,
    // 仅含 key 含 "coding-plan" 的 provider
    config_providers: serde_json::Map<String, serde_json::Value>,
    // 捕获时按固定顺序选中的 provider key
    login_provider: Option<String>,
    // 用户手动重命名过 → 重捕获不覆盖 display_name
    name_locked: bool,
}
```

**备份目录**：`~/.aicodequota/accounts/.last/`（点开头，扫描快照时天然不命中）

#### 10.2 账号指纹提取（从 ZBar 搬）

```rust
fn fingerprint_of_credentials(creds: &Value) -> Option<Fingerprint> {
    // ① 解密 creds["zcodejwttoken"] → JWT payload
    //    user_id = payload.user_id，缺失兜底 payload.sub
    // ② 解密 creds["oauth:bigmodel:user_info"]
    //    email ← JWT 的 email，缺失时用 user_info 的 email
    //    display_name ← user_info.displayName → username → name（首个非空）
    // 任一环节失败返回 None（降级不阻塞）
}
```

#### 10.3 切换事务（9 步，每步失败零改动或回滚）

```rust
pub fn switch_account(id: String, expect_fingerprint: Option<String>)
    -> Result<SwitchOutcome, String> {
    // 持全局 ACCOUNTS_LOCK（OnceLock<Mutex<()>>）
    let _guard = ACCOUNTS_LOCK.lock().unwrap();

    // ① load_snapshot_at(id)                          失败 → 零改动
    let snapshot = load_snapshot_at(&id)?;

    // ② read_live_files()                              失败 → 中止
    let live = read_live_files()?;

    // ②.⑤ expect_fingerprint 校验（无人值守自动切换专用）
    //     现场指纹不符（含读不出）→ Err("登录态已变化，已取消本次自动切换")
    //     ★ 该文案是前端识别"自动切换被取消"的约定，勿改动
    if let Some(expect) = &expect_fingerprint {
        let current = current_fingerprint()?;
        if current.as_deref() != Some(expect.as_str()) {
            return Err("登录态已变化，已取消本次自动切换".into());
        }
    }

    // ③ backup_live(&live) → ~/.aicodequota/accounts/.last/
    backup_live(&live)?;

    // ④ 当前登录已是目标 → Err("该账号已是当前登录账号")
    if live_matches(&snapshot)? {
        return Err("该账号已是当前登录账号".into());
    }

    // ④.⑤ upsert_snapshot_at(live, only_if_exists=true)
    //      刷新"即将被切走账号"的已存在快照（不新建）
    upsert_snapshot_at(&live, true)?;

    // ⑤ quit_zcode()                                   失败 → 中止（此时 ~/.zcode 零改动）
    quit_zcode()?;

    // ⑥ credentials.json 合并写回
    //    ★ 保留现场新增/更新的 account-provider: 套餐凭证键
    //      （否则 ZCode 端套餐失效）
    //    ★ 解析失败时降级为原文整串回写，绝不让合并逻辑使切换失败
    //    失败 → rollback(&live, ...)
    merge_credentials_with_live(&snapshot.credentials_raw, &live.credentials)
        .map_err(|e| rollback(&live, &e))?;

    // ⑦ config.json 合并写：coding-plan provider 逐 key 覆盖，其余配置保留
    //    失败 → rollback
    merge_config_providers(&snapshot.config_providers, &live.config)
        .map_err(|e| rollback(&live, &e))?;

    // ⑧ 重读校验：两文件必须都是合法 JSON
    //    credentials.json 还必须包含快照原文的全部键（合并写回不得丢键）
    //    否则回滚
    verify_writeback(&snapshot, &live)?;

    // ⑨ launch_zcode()
    //    失败不算切换失败，仅置 zcode_relaunched=false
    let relaunched = launch_zcode().is_ok();

    Ok(SwitchOutcome { switched_to: id, zcode_relaunched: relaunched })
}

fn rollback(live: &LiveFiles, msg: &str) -> String {
    // 从 .last/ 恢复两文件
    //   Some(raw) → atomic_write
    //   None      → fs::remove_file
    msg.to_string()
}
```

**路径穿越防护**：

```rust
fn valid_id(id: &str) -> bool {
    // 仅允许 [A-Za-z0-9_-]，防 <id>.account.json 的路径注入
    !id.is_empty() && id.chars().all(|c| c.is_ascii_alphanumeric() || c == '_' || c == '-')
}
```

#### 10.4 ZCode 进程控制（跨平台）

```rust
// macOS：必须用 osascript，禁用 pgrep
//   （ZCode 主进程 argv[0]="ZCode" 是 launchd 直接子进程，pgrep 任何匹配形式都看不到它）
#[cfg(target_os = "macos")]
fn zcode_running() -> bool {
    Command::new("osascript")
        .args(["-e", "application \"ZCode\" is running"])
        .output().map(|o| String::from_utf8_lossy(&o.stdout).trim() == "true")
        .unwrap_or(false)
}

#[cfg(target_os = "macos")]
fn quit_zcode() -> Result<(), String> {
    // osascript quit → 轮询 20×250ms → pkill -x 兜底 → 轮询 12×250ms
}

#[cfg(target_os = "macos")]
fn launch_zcode() -> Result<(), String> {
    Command::new("open").arg("-a").arg("ZCode").spawn()?;
    Ok(())
}

// Windows：tasklist + PowerShell
#[cfg(windows)]
const ZCODE_EXE_NAME: &str = "ZCode";

#[cfg(windows)]
fn zcode_running() -> bool {
    // tasklist /FI "IMAGENAME eq ZCode.exe" /FO CSV /NH
    //   ★ 用 CSV + /NH 避开本地化表头
    let filter = format!("IMAGENAME eq {ZCODE_EXE_NAME}.exe");
    match run_hidden("tasklist", &["/FI", &filter, "/FO", "CSV", "/NH"]) {
        Some(out) if out.status.success() =>
            String::from_utf8_lossy(&out.stdout).to_lowercase()
                .contains(&format!("{}.exe", ZCODE_EXE_NAME.to_lowercase())),
        _ => false,
    }
}

#[cfg(windows)]
fn capture_zcode_exe_path() -> Option<PathBuf> {
    // ★ PowerShell 强制 UTF-8 输出
    //   （管道重定向下 5.1 默认 OEM 码页，中文路径乱码）
    let script = format!(
        "[Console]::OutputEncoding=[System.Text.Encoding]::UTF8; \
         (Get-Process -Name {} -ErrorAction SilentlyContinue \
          | Where-Object {{ $_.Path }} | Select-Object -First 1 -ExpandProperty Path)",
        ZCODE_EXE_NAME
    );
    run_hidden("powershell", &["-NoProfile", "-Command", &script])
}

#[cfg(windows)]
fn zcode_exe_cached() -> Option<PathBuf> {
    // ★ 只信任指向 ZCode.exe 且真实存在的内容
    //   防篡改后借本面板拉起任意程序
    let s = read_cache()?;
    if !s.to_lowercase().ends_with(".exe") { return None; }
    PathBuf::from(s).is_file().then_some(PathBuf::from(s))
}

#[cfg(windows)]
fn run_hidden(program: &str, args: &[&str]) -> Option<std::process::Output> {
    use std::os::windows::process::CommandExt;
    const CREATE_NO_WINDOW: u32 = 0x0800_0000;  // GUI 进程闪控制台黑窗规避
    std::process::Command::new(program)
        .args(args)
        .creation_flags(CREATE_NO_WINDOW)
        .output().ok()
}
```

**安装路径候选**（Windows）：

```rust
fn windows_install_candidates() -> Vec<PathBuf> {
    vec![
        %LOCALAPPDATA%\Programs\ZCode,
        %ProgramFiles%\ZCode,
        %ProgramFiles(x86)%\ZCode,
        C:\Program Files\ZCode,
        C:\Program Files (x86)\ZCode,
    ]
}

fn find_install_root(candidates) -> Option<PathBuf> {
    // 取第一个含 resources\app.asar 的
    //   （Electron 应用安装根判定特征）
}
```

#### 10.5 自动切换

**触发时机**：每次额度轮询（30s）后检查

```rust
fn maybe_auto_switch() -> Result<(), String> {
    let cfg = load_zai_auto_switch_config()?;
    if !cfg.enabled { return Ok(()); }
    if cfg.candidate_ids.is_empty() { return Ok(()); }

    // ① 取当前账号剩余额度最低窗口
    let current = current_fingerprint()?;
    let remaining = min_remaining_of(&current)?;

    // ② 未低于阈值 → 不切换
    if remaining >= cfg.threshold_percent as i32 { return Ok(()); }

    // ③ 按优先级找第一个未耗尽的候选账号
    for candidate_id in &cfg.candidate_ids {
        if candidate_id == &current { continue; }
        let c_remaining = min_remaining_of(candidate_id)?;
        if c_remaining > cfg.threshold_percent as i32 {
            // ④ 传入 expect_fingerprint 做二次校验
            //    （防排队期间用户手动切换被覆盖）
            switch_account(candidate_id.clone(), Some(current.clone()))?;
            emit(AUTO_SWITCH_DONE_EVENT, candidate_id);
            return Ok(());
        }
    }
    Ok(())
}
```

**防抖**：距上次成功切换 < 5 分钟不重复触发（`last_success_switch_at: Instant`，全局单值——防抖窗口必须大于轮询周期 30s 才有意义；切换后首轮无采样数据时跳过判定，防止误触发连环切换）。

#### 10.6 前端交互

**账号列表**（复用 ZBar `AccountQuotaGroup` 样式）：

```
┌─ 全部账号 ─────────────────────┐
│ ● 个人账号   [当前]  Lite  87% │
│ ▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░ │
│ 5h 87% · MCP 12/100 · 今日 +3% │
│                                  │
│ ○ 工作账号   Pro    42%  [切换] │
│ ▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░░ │
│ 5h 42% · MCP 0/100              │
└──────────────────────────────────┘
```

**删除保护**：删除账号快照需二次确认（快照含完整登录凭证原文）；当前登录账号不可删除（先切换到其他账号）。

**切换确认浮层**（ZBar `SwitchConfirmOverlay` 样式）：

```
┌─ 确认切换账号 ─────────────────┐
│ 将切换到「工作账号」            │
│ ZCode 需要重启以生效            │
│                                  │
│ 当前账号的快照会被更新          │
│                                  │
│ [取消]              [确认切换]  │
└──────────────────────────────────┘
```

**自动切换设置**（设置页）：

```
┌─ 自动切换 ─────────────────────┐
│ [✓] 启用自动切换                │
│                                  │
│ 触发阈值                         │
│ 剩余额度低于 [10]% 时切换       │
│                                  │
│ 备用账号（按优先级）             │
│ 1. 工作账号            [移除]    │
│ 2. 测试账号            [移除]    │
│ [+ 添加账号]                    │
└──────────────────────────────────┘
```

#### 10.7 命令清单

M10 相关命令（`list_zai_accounts` / `capture_zai_account` / `switch_zai_account` / `remove_zai_account` / `rename_zai_account` / `get_zai_account_quotas` / `get_zai_auto_switch_config` / `set_zai_auto_switch_config`）的完整签名统一收录在附录 C，此处不重复。

**返回类型**：

```rust
#[derive(Serialize)]
#[serde(rename_all = "camelCase")]
pub struct AccountsState {
    pub current: Option<CurrentAccount>,
    pub accounts: Vec<AccountMeta>,
}

#[derive(Serialize)]
#[serde(rename_all = "camelCase")]
pub struct AccountMeta {
    pub id: String,
    pub display_name: String,
    pub email: Option<String>,
    pub fingerprint: String,
    pub created_at: i64,
    pub is_current: bool,
}

#[derive(Serialize)]
#[serde(rename_all = "camelCase")]
pub struct CurrentAccount {
    pub fingerprint: String,
    pub email: Option<String>,
    pub matched_snapshot_id: Option<String>,
}

#[derive(Serialize)]
#[serde(rename_all = "camelCase")]
pub struct CaptureOutcome {
    pub account: AccountMeta,
    pub updated_existing: bool,
}

#[derive(Serialize)]
#[serde(rename_all = "camelCase")]
pub struct SwitchOutcome {
    pub switched_to: String,
    pub zcode_relaunched: bool,
}
```

#### 10.8 风险与缓解

| 风险                                   | 缓解措施                                                       |
| -------------------------------------- | -------------------------------------------------------------- |
| 切换写到一半失败导致凭证文件损坏      | 9 步事务 + `.last/` 备份 + 任一步失败 `rollback()`              |
| ZCode 未退干净就写文件导致被覆盖        | `quit_zcode()` 内含 2 轮轮询（20×250ms + pkill 兜底 + 12×250ms） |
| 合并写回丢失现场新增的套餐凭证键        | 保留所有现场 `account-provider:` 键；解析失败降级为原文整串回写 |
| 自动切换覆盖用户手动切换                | `expect_fingerprint` 二次校验，现场不符则取消                   |
| 快照文件被篡改导致切换到错误凭证        | `valid_id()` 路径穿越防护 + 指纹校验                             |
| 用户重命名被自动重捕获覆盖              | `name_locked` 标记                                               |

---

## 五、交互流程

### 5.1 首次启动流程

```
1. 应用启动（setup 阶段）
   a. 加载配置（执行 6 步迁移）——先于一切窗口/托盘创建
   b. 预创建 Panel 窗口（visible:false；memorySavingMode=true 时跳过，首次显示时再建，见 M1.3）
   c. sessionHudEnabled=true 时创建 HUD 窗口（见 M6）
   d. 注册全局快捷键（默认 alt+shift+z）
   e. 创建托盘图标（初始灰色 "--"）
2. 渲染层加载
   a. 检测 locale（config.language > navigator.language）→ 设置 i18n
   b. 检测 theme → 应用 dark class
   c. 初始化 Pinia data store
3. 首屏预加载（错峰启动，避免并发请求风暴；周期=refreshInterval，默认 30s）
   t=0ms    加载智谱用量统计（ZCode SQLite，本地无网络）
   t=500ms  加载 Claude/Kimi 会话 Token（本地 JSONL 导入）
   t=1.5s   加载 DeepSeek/MiniMax 额度（网络请求）
   t=2.5s   加载智谱额度（fetch_zai_quota，网络）
   t=3.5s   加载智谱多账号额度 + 今日增量
4. 自动刷新定时器启动（周期均为 refreshInterval 可调；下表为分类节流上限）
   - 统计/智谱额度: refreshInterval（默认 30s）
   - DeepSeek/MiniMax: refreshInterval，最快不低于 60s
   - 服务状态: 5 分钟缓存（缓存命中不发请求）
```

### 5.2 面板显示/隐藏流程

面板有两个正交维度：**显示模式**（Hover/Pinned/Hidden，运行时状态机，见 M1.2a）与**固定模式**（unpinned/pinned-top/pinned-desktop，用户设置，见 M1.2）。

```
用户悬停托盘（popupTrigger = 'hover' 时）
  → show_panel(DisplayMode::Hover)
  → 创建窗口（若无，内存节省模式下重建）或 show 已有窗口
  → apply_always_on_top(pinMode)
  → 发送 aicodequota://window-shown 事件（渲染层收到后触发一次立即刷新）

用户鼠标离开托盘和面板（仅 Hover 显示态）
  → schedule_hide()（延迟 300ms）
  → 回调里二次校验：displayMode == Hover && 面板可见 && 光标不在面板范围内
  → hide_panel()
  → 若 pinMode != unpinned → 重置为 unpinned 并广播 aicodequota://window-pinned-state

用户点击托盘
  → 若已显示 → hide_panel()
  → 否则 → show_panel(DisplayMode::Pinned)（升级为可交互，不随鼠标离开隐藏）

用户按快捷键 alt+shift+z
  → 切换面板显示/隐藏（同点击）
```

### 5.3 智谱账号切换流程

```
用户点击某账号行的「切换」
  → 前端弹出 SwitchConfirmOverlay 确认浮层
     （提示：ZCode 将被退出并重启，未保存的对话上下文不受影响——凭证切换不写会话数据）
  → 用户点「确认切换」
  → 前端调用 switch_zai_account(id, expect_fingerprint=null)
  → Rust 执行 9 步事务（见 M10.3）
     ① 加载目标快照
     ② 读取现场凭证文件
     ③ 备份现场到 accounts/.last/
     ④ 校验不是当前账号
     ⑤ 刷新即将被切走账号的快照
     ⑥ 退出 ZCode（等待进程完全结束）
     ⑦ 合并写回 credentials.json
     ⑧ 合并写回 config.json + 重读校验
     ⑨ 重新启动 ZCode
  → 成功后：发 aicodequota://zai-switch-done 事件（手动/自动切换共用，payload 含 auto: bool）→ 前端刷新账号列表
  → 失败：自动 rollback()，返回脱敏错误
```

**自动切换流程**（每次额度轮询完成后）：

```
额度轮询完成
  → maybe_auto_switch()（本项目新增设计，基于 ZBar expect_fingerprint 钩子）
  → 检查 enabled + candidate_ids 非空
  → 防抖检查：距上次成功切换 < 5 分钟 → 跳过（窗口大于轮询周期，避免形同虚设）
  → 当前账号最低剩余 >= threshold → 跳过
     ★ 切换后首轮若当前账号无采样数据：跳过判定，等下一轮（防止误触发连环切换）
  → 按优先级找第一个剩余 > threshold 的候选
  → switch_zai_account(candidate, expect_fingerprint=current)
     ★ 二次校验：现场指纹变了就取消（用户手动切过了）
  → 成功：发 aicodequota://zai-switch-done 事件（auto: true）
        + 系统通知（tauri-plugin-notification：「已自动切换到 <账号名>」）
```

### 5.4 Claude 凭证读取流程

Claude Code 无主动登录流程（ZBar 同款策略）：凭证从本地读取，跨平台三级发现——① `AICODEQUOTA_CLAUDE_HOME` 环境变量 ② `~/.claude/.credentials.json`（Windows 为 `%USERPROFILE%\.claude\.credentials.json`，**全平台通用**）③ macOS 登录钥匙串（`security find-generic-password -s "Claude Code-credentials" -w`，仅 macOS 编译）。

```
用户打开 Claude 面板
  → Rust 自动发现本地凭证（凭证文件优先，macOS 钥匙串兜底；Windows 走 %USERPROFILE%\.claude\，见 M3.5）
  → 未找到 → 前端显示引导：「请先在 Claude Code CLI 中执行 /login」
  → 找到 → 读取 OAuth access_token（自动刷新，缓存 60s/失败 15s TTL）
  → 查询实时额度（5h/周/Sonnet周/Opus周/超额窗口）
```

### 5.5 Kimi 设备码登录流程

```
用户点击「登录 Kimi」
  → 前端调用 start_kimi_device_auth(region)
  → Rust POST {auth_base}/api/oauth/device_authorization（cn=auth.kimi.com / global=auth.kimi.ai，与 ZBar 一致）
  → 返回 { device_code, user_code, verification_uri, interval }
  → 前端显示 user_code + verification_uri
  → 用户手动打开浏览器输入 user_code
  → 前端每 interval 秒调用 poll_kimi_device_auth(device_code)
  → 轮询直到 authorized
  → 存储 access_token + refresh_token（加密）
  → 前端显示成功
```

### 5.6 周对比视图交互

```
用户点击「对比」tab
  → 前端调用 get_weekly_compare()
  → Rust 加载 quota_history.jsonl
  → split_periods() 切分周期
  → 返回 Vec<WeeklyPeriod>
  → 前端渲染周对比图（手绘 flex 柱 + absolute tooltip，与 TrendChart 同款方案，不引图表库）
```

---

## 六、技术实现方案

### 6.1 目录迁移映射表

| 来源文件                                        | 目标文件                                             | 改动                                         |
| ----------------------------------------------- | ---------------------------------------------------- | -------------------------------------------- |
| `ZBar/src/QuotaPanel.tsx`                     | `src/components/providers/zai/ZaiQuotaCard.vue`    | React → Vue 3，样式原样搬                   |
| `ZBar/src/ZaiStatsContent.tsx`                | `src/components/providers/zai/ZaiStatsContent.vue` | 同上                                         |
| `ZBar/src/StatsPanel.tsx`                     | `src/components/tabs/StatsPanel.vue`               | 同上                                         |
| `ZBar/src/widgets.tsx`                        | `src/components/widgets/*.vue`                     | 拆分为多个 Vue 组件                          |
| `ZBar/src/layout.tsx`                         | `src/components/layout/*.vue`                      | 同上                                         |
| `ZBar/src/index.css`                          | `src/styles/index.css`                             | **原样搬**（自定义 CSS 类 + 暗色覆盖） |
| `ZBar/src/format.ts`                          | `src/utils/format.ts`                              | 原样搬（全部格式化函数）                     |
| `ZBar/src/DataCache.tsx`                      | `src/stores/data.ts`                               | React Context → Pinia store                 |
| `ZBar/src/api.ts`                             | `src/api.ts`                                       | 原样搬（invoke 封装）                        |
| `ZBar/src/appearance.ts`                      | `src/stores/appearance.ts`                         | 原样搬                                       |
| `ZBar/session-hud.html`                       | `hud.html`                                         | 原样搬（原生 HTML + DOM 操作）               |
| `ZBar/src/session-hud-main.ts`                | `src/hud-main.ts`                                  | 原样搬（原生 DOM，不迁 Vue）                 |
| `ZBar/src-tauri/src/quota.rs`                 | `src-tauri/src/quota.rs`                           | 加 API Key 兜底路径                          |
| `ZBar/src-tauri/src/quota_history.rs`         | `src-tauri/src/quota_history.rs`                   | 原样搬                                       |
| `ZBar/src-tauri/src/zcode_crypto.rs`          | `src-tauri/src/zcode_crypto.rs`                    | 原样搬                                       |
| `ZBar/src-tauri/src/accounts.rs`              | `src-tauri/src/accounts.rs`                        | ★ 完整移植：快照+切换事务；自动切换为本项目新增 |
| `ZBar/src-tauri/src/claude.rs`                | `src-tauri/src/claude.rs`                          | 原样搬                                       |
| `ZBar/src-tauri/src/kimi.rs`                  | `src-tauri/src/kimi.rs`                            | 原样搬                                       |
| `ZBar/src-tauri/src/kimi_oauth.rs`            | `src-tauri/src/kimi_oauth.rs`                      | 原样搬                                       |
| `ZBar/src-tauri/src/minimax.rs`               | `src-tauri/src/minimax.rs`                         | 原样搬（region 支持源码已有）                |
| `ZBar/src-tauri/src/deepseek.rs`              | `src-tauri/src/deepseek.rs`                        | 仅 API Key 模式                              |
| `ZBar/src-tauri/src/provider_credentials.rs`  | `src-tauri/src/provider_credentials.rs`            | 原样搬（DeepSeek/MiniMax/Kimi 凭证）         |
| `ZBar/src-tauri/src/provider_quota.rs`        | `src-tauri/src/provider_quota.rs`                  | 原样搬（统一额度查询接口）                   |
| `ZBar/src-tauri/src/db.rs`                    | `src-tauri/src/db.rs`                              | 原样搬（ZCode 主库只读查询）                 |
| `ZBar/src-tauri/src/pricing.rs`               | `src-tauri/src/pricing.rs`                         | 原样搬                                       |
| `ZBar/src-tauri/src/net_config.rs`            | `src-tauri/src/net_config.rs`                      | 原样搬                                       |
| `ZBar/src-tauri/src/session_hud.rs`           | `src-tauri/src/session_hud.rs`                     | 原样搬                                       |
| `ZBar/src-tauri/src/accounts.rs`              | `src-tauri/src/accounts.rs`                        | ★ 完整移植：快照+切换事务+自动切换          |
| `ZBar/public/pricing-defaults.json`           | `public/pricing-defaults.json`                     | 原样搬                                       |
| `Coding-Plan/src/main/config.ts`              | `src-tauri/src/config.rs`                          | TS → Rust，safeStorage → AES-GCM           |
| `Coding-Plan/src/main/popup-manager.ts`       | `src-tauri/src/window.rs`                          | BrowserWindow → WebviewWindow               |
| `Coding-Plan/src/main/tray.ts`                | `src-tauri/src/tray.rs`                            | ★ 点阵字库+PNG 编码→RGBA 直塞，去 zlib     |
| `Coding-Plan/src/providers/deepseek.ts`       | `src-tauri/src/deepseek_status.rs`                 | Flashcat API 移植                            |
| `Coding-Plan/src/providers/minimax.ts`        | 合并入`src-tauri/src/minimax.rs`                   | 合并两版实现                                 |
| `Coding-Plan/src/main/i18n.ts`                | `src-tauri/src/i18n.rs`                            | TS → Rust                                   |
| `Coding-Plan/src/shared/locales/*.json`    | `src/i18n/locales/*.json`                          | 删除不适用功能的 key                         |

### 6.2 Tauri 2 ↔ Electron API 映射

| Electron                           | Tauri 2                                  | 备注                        |
| ---------------------------------- | ---------------------------------------- | --------------------------- |
| `electron-vite` 三段构建         | `vite.config.ts` + `tauri.conf.json` | Vite 构建前端 → Tauri 打包 |
| `BrowserWindow`                  | `tauri::WebviewWindow`                 | 参数映射见 M1.1            |
| `nativeImage.createFromBuffer`   | `tauri::image::Image::new_owned`       | RGBA 直塞,无需 PNG 编码(M1.6.3) |
| `Tray` + `Menu.setContextMenu` | `tauri::tray::TrayIconBuilder`         | 菜单用`tauri::menu`       |
| `app.setLoginItemSettings`       | `tauri-plugin-autostart`               |                             |
| `app.getPath('userData')`        | `dirs::home_dir()/.aicodequota`        | 与 ZBar 同款(M2.1)          |
| `safeStorage` (DPAPI)            | 自实现 AES-GCM                           | 见 M2.4                    |
| `ipcMain.handle`                 | `#[tauri::command]`                    |                             |
| `ipcRenderer.invoke`             | `invoke()` from `@tauri-apps/api`    |                             |
| `fs.watch`                       | `notify` crate                         |                             |
| `electron-updater`               | `tauri-plugin-updater`                 |                             |
| `shell.openExternal`             | `tauri-plugin-opener`                  |                             |
| `webContents.executeJavaScript`  | `WebviewWindow::eval()`                |                             |

### 6.3 密钥管理与安全

**加密存储**（M2.4 详述）：

- `apiKey` / `oauthToken` / `oauthRefreshToken` / 快照中的凭证原文 全部 AES-256-GCM 加密或 0600 权限保护
- 密文格式 `enc:v1:<nonce>.<tag>.<cipher>`，key = SHA256(machine_secret)
- 配置文件权限 mode 0600（Unix），Windows 走 ACL

**凭证文件权限**（从 ZBar 搬）：

```rust
#[cfg(unix)]
fn harden_file(path: &Path) {
    use std::os::unix::fs::PermissionsExt;
    let _ = std::fs::set_permissions(path, std::fs::Permissions::from_mode(0o600));
}
```

**错误脱敏**（从 Coding-Plan 搬）：

```rust
fn safe_error_message(text: &str, max_len: usize) -> String {
    // 脱敏正则（覆盖 Bearer token / sk-xxx / hex / api_key=xxx）
    // 截断到 max_len
}
```

**HTML 转义**：

- Rust 侧不渲染 HTML（只输出 JSON）
- Vue 3 模板默认转义，无需额外处理
- `openExternal` 白名单校验（从 Coding-Plan 搬 `isSafeExternalUrl`）

---

### 6.4 存量导入（Coding-Plan / ZBar → AICodeQuota 迁移工具）

**用户确认（2026-10-09）：做迁移工具**。本人是两个源项目的存量用户，一次性脚本把现有数据平移到新应用，避免重新录入 5 个平台凭证。

**形态**：独立子命令 `aicodequota --import-from <coding-plan|zbar>`（Rust 侧实现，读取 → 转换 → 写入 `~/.aicodequota/`，全程不删除源数据）。

| 导入项 | 来源 | 目标 | 转换规则 |
| ------ | ---- | ---- | -------- |
| 智谱 API Key | Coding-Plan `config.json` → `providers.zhipu.accounts[].apiKey`（`enc:` DPAPI 密文） | `providers.zai.accounts[]`（authMode=apikey） | Windows 上可经 DPAPI 解密（CryptUnprotectData）后重新用本项目 enc:v1 加密；解密失败则跳过并提示手填 |
| DeepSeek API Key | 同上 `providers.deepseek` | `providers.deepseek` | 同上 |
| MiniMax API Key | 同上 `providers.minimax` | `providers.minimax` | 同上 |
| Kimi OAuth Token | ZBar `~/.zbar/credentials/kimi.json`（明文 0600） | `providers.kimi` | 直接读入后 enc:v1 加密落盘 |
| Claude 凭证 | 无需导入 | — | `~/.claude/.credentials.json`（跨平台）或 macOS 钥匙串原地读取，无迁移成本 |
| 价格配置 | ZBar `~/.zbar/pricing.json` | `pricing.json` | 同格式直接复制 |
| 90 天额度快照 | ZBar `~/.zbar/quota_history.jsonl` | `quota_history.jsonl` | 同 schema 直接复制（QuotaSnapshot 结构未改） |
| 智谱账号快照 | ZBar `~/.zbar/accounts/*.account.json` | `accounts/` | 同 schema 直接复制（含凭证原文，保持 0600） |
| 主题/语言/刷新间隔 | Coding-Plan `config.json` 通用段 | `AppConfig` 同名字段 | 字段名映射，缺省用新默认值 |

**安全约束**：迁移工具打印导入清单（脱敏掩码）供确认后落盘；源文件只读，失败不写半截（先全部读入内存校验，再一次写入）。

## 七、边界定义

### 7.1 功能边界

| 边界                | 说明                                                      |
| ------------------- | --------------------------------------------------------- |
| 只读访问            | ZCode SQLite 主库**只读**，绝不写入；凭证文件仅在用户明确点击「切换账号」或已开启的自动切换触发时按 9 步事务写回（含 .last/ 备份与回滚） |
| 不修改第三方应用    | 不注入 ZCode asar；账号切换只改写 ZCode 官方凭证文件（credentials.json / config.json），且有 .last/ 备份 + 回滚 |
| 不上传代码/对话内容 | 本地统计，只上传 token 数（如果有同步功能，本期不上传）   |
| 不修改系统代理      | 只读取代理配置用于 HTTP 请求，不修改系统设置              |
| 不自动登录          | Kimi 需用户手动设备码授权，不模拟用户操作绕过验证         |

### 7.2 技术边界

| 边界          | 说明                                                                        |
| ------------- | --------------------------------------------------------------------------- |
| Rust 版本     | 需要 1.77+（Tauri 2 最低要求）                                              |
| Node 版本     | 需要 20+（Vite 7 要求）                                                     |
| 平台支持      | macOS 12+ / Windows 10 1809+                                                |
| WebView2 依赖 | Windows 需要安装 WebView2 Runtime（Windows 11 内置，Windows 10 需手动安装） |
| SQLite 版本   | rusqlite bundled（内置 SQLite，无需系统依赖）                               |
| 磁盘占用      | ZCode 主库只读，自身配置约 5MB（90 天快照约 38MB）                          |

### 7.3 数据边界

| 数据类型        | 存储位置                          | 保留策略                         |
| --------------- | --------------------------------- | -------------------------------- |
| API Key / Token | `config.json`（AES-GCM 加密）   | 永久，直到用户删除账号           |
| 额度快照        | `quota_history.jsonl`           | 90 天滚动清理                    |
| 价格配置        | `pricing.json`                  | 永久                             |
| models.dev 缓存 | `modelsdev-cache.json`          | 手动刷新或每次同步覆盖           |
| 汇率缓存        | `currency.json`                 | 手动刷新                         |
| Claude 会话数据 | 源自 `~/.claude/projects/**/*.jsonl` | 增量导入本地 `~/.aicodequota/claude.sqlite`（5s 节流 + 流式修订双游标，见 M3.5）；删除该文件即完全清除 |
| Kimi 会话数据   | 源自 `~/.kimi-code/sessions/**/wire.jsonl` | 增量导入本地 `~/.aicodequota/kimi.sqlite`（同 Claude 机制，见 M3.7）；删除即清除 |
| 卸载残留       | `~/.aicodequota/` 整目录（含凭证快照、约 38MB 历史） | 应用卸载不自动清理（Windows NSIS 卸载时提示路径；README 注明手动清理命令——快照含登录凭证原文，建议卸载后删除） |

---

## 八、非功能需求

### 8.1 性能要求

| 指标                           | 目标           | 测量方式                     |
| ------------------------------ | -------------- | ---------------------------- |
| 冷启动到面板可交互             | < 1.5s         | 从进程启动到 Vue 挂载完成    |
| 额度查询响应时间               | < 500ms（P95） | 从 invoke 到 UI 更新         |
| 30s 轮询对 CPU 的影响          | < 5%（空闲时） | 任务管理器观察               |
| 内存占用（常驻模式）           | < 150MB        | 任务管理器观察               |
| 内存占用（内存节省模式，空闲） | < 50MB         | 任务管理器观察               |
| 托盘图标响应延迟               | < 100ms        | 从刷新完成到图标更新         |
| 90 天快照加载                  | < 200ms        | 从文件读取到 UI 渲染         |
| HUD 高度自适应响应             | < 50ms         | 从会话列表变化到窗口尺寸调整 |
| models.dev 同步（5MB JSON）    | < 5s           | 网络良好情况下               |

### 8.2 可靠性要求

| 要求                 | 实现方式                                                   |
| -------------------- | ---------------------------------------------------------- |
| 单平台故障不影响其他 | 每个 Provider 独立超时（15s），失败标记 error 但不阻塞聚合 |
| 网络波动容忍         | 线性退避重试（1x/2x/3x 基准 + 随机抖动；瞬时网络错误 ×1.5，最多 3 次重试，仅 GET） |
| 凭证失效优雅降级     | 显示"凭证失效"提示 + 引导重新登录，其他平台不受影响        |
| 快照数据完整性       | 同秒防抖 + 残行修补 + 单次原子写                           |
| 配置写入原子性       | 临时文件 + fsync + rename，Windows EPERM/EBUSY 重试        |
| 账号切换防错配       | 查询前后各取一次指纹，不一致则丢弃采样                     |
| 崩溃后可恢复         | 配置损坏自动备份 + 重建默认，快照残行自动修补              |
| WebView 白屏修复     | 显示时强制 set_size(size+1) → set_size(size)              |
| Windows 小窗口钳宽   | 隐藏建窗 → set_size → show 规避 tao 的 WS_CAPTION 缺陷   |

### 8.3 安全性要求

| 要求               | 实现方式                                       |
| ------------------ | ---------------------------------------------- |
| 凭证加密存储       | AES-256-GCM，key = SHA256(machine_fingerprint) |
| 配置文件权限       | mode 0600（Unix），Windows ACL                 |
| 只读访问第三方数据 | ZCode SQLite 用`PRAGMA query_only = ON`      |
| 不泄露敏感信息     | 错误脱敏正则（Bearer token / API key / hex）   |
| 外链白名单         | openExternal 前校验 hostname                   |
| 凭证文件隔离       | 各账号快照/凭证独立文件 + 0600 权限，互不污染  |
| 不存储明文密码     | 所有凭证字段加密后落盘                         |

### 8.4 可维护性要求

| 要求            | 实现方式                                                        |
| --------------- | --------------------------------------------------------------- |
| Provider 可插拔 | `trait Provider` 接口，新增平台只需实现接口                   |
| 模块化          | 每个 Provider 独立文件，M3.1~M3.6 分离                          |
| 单一职责        | 窗口管理、配置管理、Provider 适配、价格计算、快照存储各独立模块 |
| 可测试性        | Rust 侧单元测试覆盖率 ≥60%（价格匹配、快照切分、额度解析）     |
| i18n 完整       | 中英词条严格对齐，CI 检查                                       |
| 代码规范        | Rust:`cargo clippy` + `cargo fmt`；TS: ESLint + Prettier    |
| 无 TODO 遗留    | 所有未实现功能已在 §1.4 明确列出，不留 TODO                    |

### 8.5 国际化要求

| 要求         | 实现方式                                                      |
| ------------ | ------------------------------------------------------------- |
| 完整双语     | 简体中文 + English，所有 UI 文案国际化                        |
| 主进程支持   | 托盘菜单等无法用 vue-i18n 的场景用 Rust 手写 t()              |
| 格式化本地化 | 数字（千分位）、日期（相对时间）、货币（¥/$）跟随 locale     |
| 切换实时生效 | 切换后广播事件，HUD 等同源窗口同步更新                        |
| 持久化       | localStorage`aicodequota-theme`，配置文件 `language` 字段 |

### 8.6 兼容性要求

| 平台             | 最低版本                       |
| ---------------- | ------------------------------ |
| macOS            | 12.0 (Monterey)                |
| Windows          | 10 1809（October 2018 Update） |
| WebView2 Runtime | Windows 10 需要手动安装最新版  |
| Rust             | 1.77+                          |
| Node.js          | 20+                            |

---

## 九、开发阶段划分

**总工期**：Phase 1~7 合计约 **17.5~22.5 天**（3~5 + 3~4 + 3~4 + 2~3 + 2 + 2 + 2.5）。各 Phase 串行推进，Phase 2 起每阶段依赖前一阶段的骨架/数据层。

### Phase 1: 基础设施（3~5 天）

**目标**：项目骨架 + 构建配置 + Rust 基础模块

**任务**：

- [ ] 初始化 Tauri 2 + Vue 3 + Vite 项目
- [ ] 配置 Tailwind v4 + 移植 ZBar 的 `index.css`
- [ ] 实现 `config.rs`（加密 + 原子写 + 6 步迁移）
- [ ] 实现 `net_config.rs`（HTTP + 代理）
- [ ] 实现 `zcode_crypto.rs`（AES-GCM 解密）
- [ ] 实现 `tray.rs`（★ 点阵数字图标 + 3 色阈值 + 加载动画 + 菜单）
- [ ] 实现 `window.rs`（Panel 窗口 + 3 种固定模式）
- [ ] 实现 `i18n.rs`（主进程国际化）
- [ ] 实现基础 Vue 组件（SectionCard / HeroMetric / PageShell）

**交付标准**：

- `pnpm tauri dev` 可启动
- 托盘图标显示数字百分比（加载动画先行；Phase 1 末用 mock 数据驱动变色验证渲染管线，真实额度数据 Phase 2 接入）
- 点击托盘弹出空面板
- 配置文件读写正常

### Phase 2: 智谱 Z.ai 额度（3~4 天）

**目标**：完成第一个 Provider 的完整链路

**任务**：

- [ ] 移植 `zcode_crypto.rs` + `quota.rs`（双源认证）
- [ ] 实现 `quota_history.rs`（快照 + 滚动 + 今日增量）
- [ ] 实现 `accounts.rs` 第一步（快照存储 + 指纹提取 + 额度聚合；切换事务留 Phase 3）
- [ ] 移植 `QuotaPanel.tsx` → `ZaiQuotaCard.vue`
- [ ] 移植 `ZaiStatsContent.tsx` → `ZaiStatsContent.vue`
- [ ] 实现智谱统计（ZCode SQLite 只读查询）
- [ ] 实现价格配置基础（多级匹配 + 成本计算）
- [ ] 实现 `get_stats` / `get_trend` / `compute_cost` 命令

**交付标准**：

- 智谱面板显示 5h/周/MCP 额度条
- Hero 卡片显示总花费、总 Token、请求次数
- 趋势图正确渲染
- Token 构成正确显示

### Phase 3: DeepSeek + MiniMax + 智谱账号切换（3~4 天）

**目标**：完成 API Key 认证的 Provider + 账号切换事务

**任务**：

- [ ] 实现 `deepseek.rs`（仅 API Key 模式）
- [ ] 实现 `minimax.rs`（合并 Coding-Plan + ZBar 实现，含 region 支持）
- [ ] 实现 `deepseek_status.rs`（Flashcat API）
- [ ] 移植 `DeepSeekServiceStatus.vue` → `ServiceStatusBar.vue`
- [ ] 实现各 Provider 的 Vue 面板（样式统一到智谱面板规范）
- [ ] ★ 移植 `accounts.rs` 完整版（快照存储 + 指纹提取 + 9 步切换事务）
- [ ] 实现 ZCode 进程控制（macOS osascript / Windows tasklist）
- [ ] 实现账号列表 UI（AccountQuotaGroup 样式）+ 切换确认浮层
- [ ] 实现自动切换（阈值触发 + expect_fingerprint 二次校验）

**交付标准**：

- DeepSeek API Key 余额查询正常
- DeepSeek 服务状态 90 天可用率正确显示
- MiniMax 额度正确显示（区域选择 cn/global）
- 智谱账号可捕获、切换、删除、重命名
- 切换失败可回滚（手动杀 ZCode 进程模拟）
- 自动切换在额度低于阈值时触发

### Phase 4: Claude + Kimi（2~3 天）

**目标**：完成 OAuth 认证的 Agent

**任务**：

- [ ] 移植 `claude.rs`（会话导入 + OAuth 额度 + 流式修订）
- [ ] 移植 `kimi.rs` + `kimi_oauth.rs`
- [ ] 实现 Claude 凭证读取（跨平台三级发现：环境变量 / .credentials.json / macOS 钥匙串）
- [ ] 实现 Kimi 设备码登录流程
- [ ] 实现各 Provider 的 Vue 面板

**交付标准**：

- Claude Code 显示会话 Token + 5h/周额度
- Kimi Code 显示会话 Token + 5h/周/月额度
- 流式修订游标语义正确（同一 message.id 终值覆盖本地 SQLite 旧行，「总量更大者胜」upsert 可本地验证；服务端补传链路本期不实现，属 §1.4 边界）

### Phase 5: 会话悬浮窗（2 天）

**目标**：独立窗口实时显示活跃会话

**任务**：

- [ ] 移植 `session_hud.html` + `session_hud-main.ts`
- [ ] 实现 `session_hud.rs`（活跃会话判定 + 高度自适应）
- [ ] 实现 8 方向 resize
- [ ] 实现 HUD 窗口创建（Windows 隐藏建窗）
- [ ] 实现主题同步（跟随主面板）

**交付标准**：

- HUD 窗口独立显示，最小化不抢焦点
- 活跃会话实时更新（1s 轮询）
- 高度随会话数自适应
- 8 方向拖拽调整大小正常

### Phase 6: 周对比 + 价格配置（2 天）

**目标**：历史分析与价格管理

**任务**：

- [ ] 实现 `get_weekly_compare` 命令
- [ ] 实现周对比图（手绘 flex 柱，复用 TrendChart 方案）
- [ ] 移植 `PricingPanel.tsx` → `PricingPanel.vue`
- [ ] 实现 models.dev 在线同步
- [ ] 实现汇率获取与缓存

**交付标准**：

- 周对比视图正确切分周期
- 价格配置可增删改
- models.dev 同步显示 diff 并可应用
- 汇率实时获取

### Phase 7: 国际化 + 打磨（2.5 天，含迁移工具 0.5 天）

**目标**：完善双语支持 + 修 bug

**任务**：

- [ ] 整理 i18n 词条（中英对齐）
- [ ] 实现语言切换（主进程 + 渲染层 + HUD）
- [ ] 实现数字/日期/货币本地化
- [ ] 实现自动更新（双源）
- [ ] 全平台测试（macOS + Windows）
- [ ] 性能优化与内存泄漏排查
- [ ] 实现存量导入工具（§6.4：--import-from coding-plan/zbar，含 DPAPI 解密转换）
- [ ] 编写 README

**交付标准**：

- 中英文切换无遗漏
- 自动更新可用
- 两个平台稳定运行

---

## 十、验收标准

### 10.1 功能完整性

- [ ] 5 个 AI 平台（智谱 / DeepSeek / MiniMax / Claude / Kimi）额度监控正常
- [ ] 智谱支持本地凭证 + API Key 双源认证
- [ ] DeepSeek API Key 余额查询正常
- [ ] DeepSeek 服务状态 90 天可用率正确显示
- [ ] 智谱账号一键切换（捕获/切换/删除/重命名），切换失败可回滚
- [ ] 智谱账号自动切换（阈值触发 + 指纹二次校验 + 5 分钟防抖）
- [ ] 托盘显示额度百分比——两端统一点阵数字图标（3 色阈值 + 加载动画；macOS 22px/44px@2x 实测可读，见 M1.6.7）
- [ ] 会话悬浮窗独立工作，最多 5 会话
- [ ] 周对比视图正确切分周期
- [ ] 价格配置多级模型名匹配正确（精确/归一/变体渐进）
- [ ] models.dev 在线同步可用
- [ ] 国际化完整（中英对齐，约 360 词条）
- [ ] 3 种窗口固定模式正常切换
- [ ] 内存节省模式正常销毁/重建窗口
- [ ] 配置加密生效（落盘文件中 apiKey 为 `enc:v1:` 密文）+ 原子写（kill 进程后 config.json 不损坏）+ 6 步迁移全过（F15）
- [ ] 自动更新可检查/下载/安装（GitHub 源；Gitee 源可选验证）（F16）
- [ ] 网络代理生效（设置 http/socks5 代理后额度请求经代理发出）（F17）
- [ ] 存量导入：Coding-Plan 三平台 API Key + ZBar 价格/快照/账号快照完整迁入且源数据不动（§6.4）

### 10.2 稳定性

- [ ] macOS 12+ 稳定运行，无崩溃、无白屏
- [ ] Windows 10/11 稳定运行，无崩溃、无白屏
- [ ] 默认 30s 轮询对 CPU 影响 < 5%
- [ ] 内存占用（常驻）< 150MB
- [ ] 网络断开后恢复自动重连
- [ ] 配置损坏自动恢复（备份为 .corrupt-{时间戳}.json 后重建默认）
- [ ] 凭证失效优雅降级（显示提示，不崩溃）
- [ ] 账号切换中途失败（杀进程/文件锁）后 ZCode 凭证可从 .last/ 备份恢复

### 10.3 代码质量

- [ ] `cargo clippy` 无警告
- [ ] `cargo fmt` 格式化完成
- [ ] Rust 单元测试覆盖率 ≥ 60%
- [ ] TypeScript strict 模式无类型错误
- [ ] ESLint + Prettier 通过
- [ ] 无 TODO/FIXME 遗留（§1.4 已列明不做的功能不算 TODO）

### 10.4 文档

- [ ] README.md（安装说明 + 使用指南）
- [ ] 架构文档（模块划分 + 数据流图）
- [ ] API 文档（Tauri commands + Provider 接口）

### 10.5 验收前提与测试数据依赖（PRD 缺口补全）

PRD 原文未定义验收所需的前置数据条件。以下为**本项目定义**——**验收必须在明确的测试数据前提下进行**，否则「显示 0%」与「显示正确值」无法区分：

| 验收项 | 前置数据条件 | 无数据时的替代验收 |
|--------|------------|------------------|
| F1 智谱额度 | 本机已装并登录 ZCode（`~/.zcode/v2/credentials.json` 存在） | 或手填有效 API Key |
| F6 MiniMax 额度 | 有效 MiniMax API Key | 用无效 Key 验证 `expired` 分支 |
| F7 Claude 额度 | `~/.claude/.credentials.json` 存在（或 macOS 钥匙串已登录） | 验证「未找到凭证」引导文案 |
| F8 Kimi 额度 | 已完成设备码登录 | 验证登录按钮与设备码流程 |
| 托盘聚合 | ≥1 个百分比源可用 | **用 mock 数据源**（§5.1 已定义 CQB_MOCK 类似机制）验证五态渲染 |
| 部分失败语义 | 需人为构造（如断网 + 保留一个源） | 用 mock 注入「部分失败」响应 |
| 陈旧值降级 | 需等待 `3×refreshInterval` 或 mock 时间 | mock 注入 `updatedAt` 为过去时刻 |

**关键要求**：**必须提供 mock 数据源**，使「托盘五态（正常/陈旧/部分失败/全部失败/未配置）」与「各平台面板渲染」能在**无真实凭证、无真实网络**的条件下完整验证。这是 MVP 可交付性的前提——否则开发机无法复现失败路径。

### 10.6 平台验证范围（PRD 缺口补全）

PRD 原文 §1.6.6/§1.6.7/§5.2/§5.4 均为 macOS 路径，Windows 等价路径未全部给出。以下为**本项目定义**：

| 平台 | 验证要求 | 说明 |
|------|---------|------|
| **Windows 10/11** | **主验证平台**（完整验收） | 开发机即 Windows，所有验收项必须在此平台实测通过 |
| **macOS 12+** | **次级验证**（能编译、关键路径抽验） | 若开发机无 macOS 环境，`tray.title()`、Accessory 模式、钥匙串读取、popover 效果**降级为「代码就绪、未实机验证」**，并在 README 标注 |

**必须补齐的 Windows 等价路径**（PRD 原文只给 macOS 的）：

| 功能 | macOS（PRD 原文） | Windows（本项目补充） |
|------|------------------|---------------------|
| 托盘图标 | 22×22 / 44×44@2x | 16×16 |
| 菜单栏 title | `tray.title()` 显示百分比 | **无 title**（Windows 托盘无此惯例）→ 数字只在图标内 |
| 隐藏 Dock | `ActivationPolicy::Accessory` | 不适用 |
| 面板定位 | y = 25.0（菜单栏下方） | y = `mon_h - win_h - 48.0`（任务栏上方） |
| Claude 凭证 | 钥匙串 / `.credentials.json` | `%USERPROFILE%\.claude\.credentials.json` |
| 代理读取 | `scutil --proxy` | `winreg` 读 `HKCU\...\Internet Settings` |
| 关窗行为 | 默认不退出（Accessory） | **默认退出进程，必须拦截 `CloseRequested` 改为 hide**（M1.7） |

**Win11 托盘溢出区提示**：Windows 11 默认把新托盘图标收进溢出区（需用户手动「钉住」才常驻可见）。README 必须写明这一步——否则用户会认为「装完没反应」。

---

### 10.7 遗留 PRD 内部矛盾（需澄清）

| # | 矛盾点 | 处置 |
|---|--------|------|
| 1 | §2.1 F16 写「GitHub + Gitee 双源」，§8.1 写「Gitee 镜像不做」 | **已修**：§2.1 改为「GitHub 单源（§8.1）」 |
| 2 | §8.1 OWNER 占位 + updater 签名密钥未提供 | **开工前置项**，由用户提供（不阻塞 MVP，F16 不在 MVP 内） |

---

## 十一、MVP 分层与交付范围（本项目新增）

> **背景**：PRD §2.1 把 F1~F17 全部标为「必须实现」，§九 Phase 1~7（17.5~22.5 天）是 v1.0 全量工期，§十 是 v1.0 全量验收。**本章定义 MVP 分层**——MVP 只规定**交付顺序**，不改变 §2.1 的最终功能范围。

### 11.1 MVP 定义与判据

**MVP = 让「托盘上能显示多平台额度」跑通的最小范围。**

判据（唯一）：**去掉它，「托盘显示多平台额度」这条链路是否失效。**

### 11.2 MVP 范围（6 条 F 项，含 3 条部分）

| F 编号 | 项 | MVP 判定 | 范围收窄 |
|--------|----|---------|---------|
| **F1** | 智谱 Z.ai 额度监控 | 必须 | 全量（双源认证） |
| **F3** | 托盘图标显示额度百分比 | 必须 | 全量（点阵 + 3 色 + 动画） |
| **F6** | MiniMax 额度监控 | 必须 | 全量（API Key + region） |
| **F7** | [CC] 用量导入 | **部分** | 只做 OAuth 实时额度；**不做**会话 Token 导入 |
| **F8** | Kimi Code 用量导入 | **部分** | 只做设备码登录 + 额度；**不做**会话 Token 导入 |
| **F15** | 配置加密 + 原子写入 + 6 步迁移 | **部分** | 做加密 + 原子写；**不做** 6 步迁移（首版不触发） |

**MVP 托盘百分比来源 = 4 个**（智谱 / MiniMax / Claude / Kimi），满足「多平台」定义（≥2）。

### 11.3 MVP 不做的 F 项（12 条）

以下各项**属 §2.1「本期必须实现」**（不是 §1.4「本期根本不做」），只是不进入 MVP 交付集：

| F 编号 | 项 | 剔除依据 |
|--------|----|---------|
| F2 | 智谱账号一键切换 | §1.6.5 单账户即可产出托盘数字；§7.1 属第三方写操作，旁挂 |
| F4 | DeepSeek 额度监控 | §1.6.5「余额型账户不产出百分比窗口、不参与聚合」（**已确认接受，2026-10-10**） |
| F5 | DeepSeek 服务状态 | 服务可用率非额度百分比，不进 §1.6.5 聚合 |
| F9 | 会话 Token 悬浮窗 | M6 独立窗口，与托盘是两条链路 |
| F10 | 周额度对比 | M5.3 历史分析，不产 usedPercent |
| F11 | 价格配置 + models.dev | M4 输出是金额，托盘不涉及价格 |
| F12 | 国际化 | 托盘只有数字；§8.5 为全量口径 |
| F13 | 3 种窗口固定模式 | M1.2 窗口行为，与额度拉取正交 |
| F14 | 内存节省模式 | M1.3 资源优化，非功能必需 |
| F16 | 自动更新 | M8 发布运维能力；前置项未就绪 |
| F17 | 网络代理 | M9 以直连兜底；不在 §1.6.5 来源清单内 |
| F15 子项 | 6 步迁移 | M2.3「首版无存量用户配置，实际不会触发」 |

### 11.4 MVP 交付顺序（依赖驱动）

```
第 1 步  基础设施：项目骨架 + 配置持久化（F15 加密+原子写）+ net_config HTTP 客户端
         + 单实例（M1.7）→ 交付物：可启动、可读写配置
第 2 步  第一个数据源：F1 智谱额度（双源认证）+ 面板窗口（M1.1）→ 交付物：能拉到真实数据
第 3 步  托盘：F3（点阵图标 + 3 色阈值 + 五态语义 §1.6.5a）→ 交付物：托盘显示真实数字
第 4 步  多平台：F6 MiniMax + F7 Claude 额度 + F8 Kimi 额度 → 交付物：多平台聚合成立
第 5 步  容错与空态：§1.6.5a 五态 + M1.8 无凭证首启 + §10.5 mock 数据源 → 交付物：可完整验收
```

**预计工期**：**5~6 天**（对比 §九 全量 17.5~22.5 天）。

### 11.5 MVP 验收标准（可客观判定）

区别于 §10.1 的「功能存在性」断言，MVP 验收必须是**可观测断言**：

- [ ] **零配置出数**：本机已登录 ZCode 时，装上后托盘直接显示智谱额度数字（无需任何输入）
- [ ] **颜色阈值**：数字随剩余额度变色（≥50% 绿 / ≥20% 黄 / <20% 红），阈值来自 `display.colorThresholds`
- [ ] **点开可解释**：点击托盘 → 面板按平台列出各额度窗口的已用/总量/重置时间
- [ ] **多平台成立**：至少 2 个百分比源（智谱 + MiniMax/Claude/Kimi 任一）同时显示
- [ ] **断网降级**：拔网线 → 托盘保留上次值并标注时间戳；超过 `3×refreshInterval` → 视觉降级（陈旧态）
- [ ] **单源失败隔离**：故意让一个源凭证失效 → 该源显示 `expired`，**其他源与托盘数字不受影响**
- [ ] **未配置态**：清空所有配置 → 托盘显示 `--` + tooltip「未配置任何账户」，面板显示引导卡（M1.8）
- [ ] **常驻语义**：macOS 关窗后进程仍在；Windows 关窗后进程仍在（M1.7 拦截生效）
- [ ] **单实例**：双击图标两次 → 只有一个托盘图标，第二次唤起已有面板
- [ ] **24 小时稳定**：连续运行 24h，内存无持续增长

### 11.6 MVP 与全量的关系

```
MVP（5~6 天）                v1.0 全量（17.5~22.5 天）
├── F1 智谱额度              ├── F1 全量
├── F3 托盘（全量）           ├── F2 账号切换（M10）
├── F6 MiniMax               ├── F4/F5 DeepSeek + 服务状态
├── F7 Claude 额度            ├── F7/F8 会话 Token 导入
├── F8 Kimi 额度              ├── F9 会话悬浮窗
├── F15 加密+原子写           ├── F10 周对比
└── M1.7/M1.8 缺口补全        ├── F11 价格配置
                              ├── F12 国际化
                              ├── F13/F14 窗口模式
                              ├── F15 6 步迁移
                              ├── F16 自动更新
                              └── F17 网络代理
```

**关键**：MVP 是**删除功能类别**（不是把 17 项排成优先级队列）；MVP 完成后，剩余 F 项按 §九 Phase 顺序继续。

---

## 附录 A：核心数据结构

### A.1 统一额度输出（Rust → TypeScript）

> 本附录为全文档类型的**唯一权威定义**；M3.1 的 Rust trait 签名是本表的 Rust 镜像，实现时两处保持字段一致，修改以本附录为准。

```typescript
interface QuotaEntry {
  accountId: string;
  label: string;
  status: 'ok' | 'expired' | 'error' | 'pending' | 'not_configured';
  // pending：查询进行中（首轮或手动刷新占位）；not_configured：该 provider 无任何启用账户
  windows: QuotaWindow[];
  balance: Balance | null;
  planName: string | null;
  message: string | null;
  updatedAt: number;  // 毫秒
}

interface QuotaWindow {
  key: string;              // 智谱: hour5/weekly/mcp；DeepSeek: balance；MiniMax: interval/weekly；
                            // Claude: hour5/weekly/sonnetWeekly/opusWeekly/overage；Kimi: hour5/weekly/monthly/boostPack
  title: string;
  usedPercent: number | null;  // 0~100
  used: number | null;
  total: number | null;
  unit: string | null;
  resetsAt: number | null;  // 毫秒
}

interface Balance {
  amount: number;
  currency: string;      // "CNY" | "USD"
  granted: number | null;
  toppedUp: number | null;
}
```

### A.2 统计数据（ZCode SQLite → 前端）

```typescript
interface Stats {
  totalTokens: number;
  inputTokens: number;
  outputTokens: number;
  cacheReadTokens: number;
  cacheWriteTokens: number;
  reasoningTokens: number;
  requests: number;
  totalCost: number;      // 按 pricing.json 计算
  currentModel: CurrentModel | null;
  byModel: ModelStat[];
}

interface ModelStat {
  modelId: string;
  requests: number;
  inputTokens: number;
  outputTokens: number;
  cacheReadTokens: number;
  avgTps: number | null;  // 平均输出速度 tokens/s
  cost: number;
}

interface CurrentModel {
  modelId: string;
  lastUsedAt: number;  // 毫秒
}
```

### A.3 趋势数据

```typescript
interface TrendBucket {
  ts: number;          // 桶起始时间戳（毫秒）
  cost: number;
  tokens: number;
  requests: number;
}
```

### A.4 周对比数据

```typescript
interface WeeklyPeriod {
  resetAt: number;     // 周期开始（毫秒）
  endAt: number;       // 周期结束（毫秒）
  isCurrent: boolean;
  pctStart: number;    // 起始已用百分比
  pctPeak: number;     // 峰值已用百分比
  pctEnd: number;      // 结束已用百分比
  sampleCount: number;
}
```

---

## 附录 B：i18n 词条结构示例

```json
{
  "main": {
    "appName": "AI 编码额度",
    "refresh": "刷新",
    "settings": "设置",
    "close": "关闭",
    "loading": "加载中..."
  },
  "quota": {
    "fiveHour": "5 小时",
    "weekly": "每周",
    "mcpUsage": "MCP 用量",
    "resetIn": "{time} 后重置",
    "resetAt": "{date} 重置",
    "todayDelta": "今日 +{pct}%",
    "deepseekApiService": "API 服务",
    "deepseekWebService": "网页对话服务",
    "statusOperational": "正常",
    "statusDegraded": "性能下降",
    "statusPartialOutage": "部分中断",
    "statusMajorOutage": "完全中断",
    "uptime": "90 天可用率 {pct}%"
  },
  "providers": {
    "zai": "智谱 Z.ai",
    "deepseek": "DeepSeek",
    "minimax": "MiniMax",
    "claude": "Claude Code",
    "kimi": "Kimi Code"
  }
}
```

---

## 附录 C：Rust 命令清单

> 命令名全文档以此清单为唯一权威源（M3/M4/M6/M10 章节中的内联签名若与此不一致，以此为准）。
> 引用的类型定义：`QuotaEntry/QuotaWindow/Balance/Stats/ModelStat/CurrentModel/TrendBucket/WeeklyPeriod` 见附录 A；
> `SessionHudConfig/KimiDeviceAuthInfo/KimiDevicePollResult/ClaudeSnapshot/KimiSnapshot/PricingDiff/ModelsdevSyncResult/AccountsState/CaptureOutcome/SwitchOutcome/AccountMeta/ZaiAutoSwitchConfig/UpdateInfo/StatsRequest/TrendRequest/ClaudeUsageRequest/KimiUsageRequest/CostResult` 在对应模块源文件中定义（移植 ZBar 时随文件带入），附录不重复展开。

```rust
// 统计（ZCode SQLite 只读）
#[tauri::command] async fn get_stats(req: StatsRequest) -> Result<Stats, String>
#[tauri::command] async fn get_trend(req: TrendRequest) -> Result<Vec<TrendBucket>, String>
#[tauri::command] async fn compute_cost(req: StatsRequest) -> Result<CostResult, String>
// 注：不做模型速度排行（§1.4 边界）；HUD 内的每会话 TPS/TTFT 由 session_hud.rs 内部计算、
// 经事件推送，不暴露独立命令

// 智谱额度 + 多账号（fetch_zai_quota 查当前登录账号；get_zai_account_quotas 查全部快照账号——
// 前者每次成功后写快照采样，后者只读聚合）
#[tauri::command] async fn fetch_zai_quota() -> Result<Vec<QuotaEntry>, String>
#[tauri::command] async fn get_zai_today_delta() -> Result<(u32, u32), String>  // 当前登录账号的(今日增量,采样数)；内部走多账号 today_deltas 后取当前指纹（M5.3）
#[tauri::command] async fn get_weekly_compare() -> Result<Vec<WeeklyPeriod>, String>
#[tauri::command] async fn list_zai_accounts() -> Result<AccountsState, String>
#[tauri::command] async fn capture_zai_account() -> Result<CaptureOutcome, String>
#[tauri::command] async fn switch_zai_account(id: String, expect_fingerprint: Option<String>) -> Result<SwitchOutcome, String>
#[tauri::command] async fn remove_zai_account(id: String) -> Result<(), String>
#[tauri::command] async fn rename_zai_account(id: String, name: String) -> Result<AccountMeta, String>
#[tauri::command] async fn get_zai_account_quotas() -> Result<Vec<QuotaEntry>, String>
#[tauri::command] fn get_zai_auto_switch_config() -> Result<ZaiAutoSwitchConfig, String>
#[tauri::command] fn set_zai_auto_switch_config(cfg: ZaiAutoSwitchConfig) -> Result<(), String>

// DeepSeek
#[tauri::command] async fn get_deepseek_usage(account_id: String) -> Result<QuotaEntry, String>
#[tauri::command] async fn get_deepseek_service_status() -> Result<Vec<DeepSeekServiceComponent>, String>

// MiniMax
#[tauri::command] async fn get_minimax_usage(account_id: String) -> Result<QuotaEntry, String>

// Claude（凭证自动发现，无登录命令）
#[tauri::command] async fn get_claude_usage(req: ClaudeUsageRequest) -> Result<ClaudeSnapshot, String>

// Kimi（设备码 OAuth）
#[tauri::command] async fn get_kimi_usage(req: KimiUsageRequest) -> Result<KimiSnapshot, String>
#[tauri::command] async fn start_kimi_device_auth(region: Option<String>) -> Result<KimiDeviceAuthInfo, String>
#[tauri::command] async fn poll_kimi_device_auth(session_id: String) -> Result<KimiDevicePollResult, String>

// 价格（check_pricing_updates：与内置参考表 diff，标注新增/变动/缺失，不写盘；
// sync_modelsdev_pricing：拉取 models.dev 生成 diff + 缓存，同样不直接写盘，用户确认后 set_pricing 落盘）
#[tauri::command] fn get_pricing() -> Result<PricingConfig, String>
#[tauri::command] fn set_pricing(config: PricingConfig) -> Result<(), String>
#[tauri::command] async fn check_pricing_updates() -> Result<PricingDiff, String>
#[tauri::command] async fn sync_modelsdev_pricing() -> Result<ModelsdevSyncResult, String>
#[tauri::command] async fn fetch_fx_rate() -> Result<(f64, String), String>  // (USD→CNY, 来源名)，成功写 currency.json 缓存（M4.3）

// 配置（get_available_providers：返回编译期 5 个 provider key，供设置页渲染列表）
#[tauri::command] fn get_config() -> Result<AppConfig, String>
#[tauri::command] async fn update_config(updates: Partial<AppConfig>) -> Result<(), String>
#[tauri::command] fn get_available_providers() -> Result<Vec<String>, String>

// 窗口
#[tauri::command] fn set_window_pin_mode(mode: WindowPinMode) -> Result<(), String>
#[tauri::command] fn get_window_pin_mode() -> Result<WindowPinMode, String>

// HUD（打开无独立命令：sessionHudEnabled=true 时启动创建；运行中改配置经 set_session_hud_config 即时生效并热推事件；
//      会话数据不走 invoke——由 session_hud.rs 后台线程 1s 轮询后 emit "aicodequota://session-hud-usage" 推送，
//      配置热推走 "aicodequota://session-hud-params"）
#[tauri::command] fn get_session_hud_config() -> Result<SessionHudConfig, String>
#[tauri::command] async fn set_session_hud_config(config: SessionHudConfig) -> Result<SessionHudConfig, String>
#[tauri::command] fn set_session_hud_font_scale(scale: f64) -> Result<SessionHudConfig, String>
#[tauri::command] async fn session_hud_resize_begin() -> Result<(), String>   // 拖拽开始：冻结高度自适应
#[tauri::command] async fn session_hud_resize_end() -> Result<(), String>     // 拖拽结束：恢复自适应
#[tauri::command] async fn close_session_hud() -> Result<(), String>          // 关闭并置 sessionHudEnabled=false

// 通用
#[tauri::command] async fn check_update() -> Result<Option<UpdateInfo>, String>
#[tauri::command] fn get_app_version() -> Result<String, String>
#[tauri::command] fn set_locale(locale: String) -> Result<(), String>
```

---

**文档结束**
