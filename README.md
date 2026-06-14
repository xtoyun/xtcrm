# XiongTao CRM

雄韬 CRM 管理后台前端。Vue 3 + Vite + Ant Design Vue 4，配合 ThinkPHP 6.1 后端。

---

## 架构概览

```
浏览器                            Nginx / Apache
  │                                   │
  ├── /crm/          ← SPA 入口       │ → public/crm/index.html
  ├── /crm/#/xxx     ← Hash 路由      │    Vue Router 接管
  ├── /crm/config.js ← 运行时配置     │ → public/crm/config.js
  └── /index.php     ← API 请求       │ → PHP (ThinkPHP 6.1)
                        s=/platform     → MppAutoload 路由 → Controller
```

前端和后端部署在同一域名下，`index.php` 走 PHP，其余 `/crm/` 下的静态资源走 Nginx。

## 开发环境

### Node.js

本项目需要 Node.js，**推荐使用 nvm（Node Version Manager）** 管理版本，避免多项目之间的版本冲突。

当前 Node 版本：**v22.16.0**

#### Windows 安装 nvm

下载 [nvm-windows](https://github.com/coreybutler/nvm-windows/releases) 安装包（`nvm-setup.exe`），安装后打开新的终端：

```powershell
# 安装并使用 Node 22
nvm install 22
nvm use 22

# 验证
node -v   # v22.16.0
npm -v    # 10.x
```

#### macOS / Linux 安装 nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
# 重启终端或 source ~/.bashrc

nvm install 22
nvm use 22
```

#### 项目自带版本锁

进入 `mpp/crm/frontend-v2/` 后，`.nvmrc` 文件会自动提示 nvm 切换版本：

```bash
nvm use        # 自动读取 .nvmrc，切换到 v22.16.0
```

## 快速开始

```bash
# 1. 克隆
git clone https://github.com/xtoyun/xtcrm.git
cd xtcrm/mpp/crm/frontend-v2

# 2. 切换到正确的 Node 版本
nvm use

# 3. 安装
npm install

# 4. 开发模式（Vite dev server，端口 9999）
npm run dev
# → http://localhost:9999
# Vite 自动代理 /index.php 到后端（见 vite.config.js 的 server.proxy）

# 5. 构建
npm run build
# → 输出到 public/crm/

管理员账号：admin/123123

数据库/data/xtcrm_data.sql
```

开发时访问 `http://localhost:9999`，登录页 `/#/passport/login`。

## 配置

### 运行时配置 (`public/config.js`)

```js
window.publicConfig = {
  APP_NAME: '客户管理平台',
  BASE_API: '/index.php?s=/platform',   // API 前缀
  uploadImageSize: 2,                    // 图片上传限制 (MB)
  uploadVideoSize: 20,                   // 视频上传限制 (MB)
}
```

`BASE_API` 开发时用相对路径（Vite 代理），生产环境根据实际部署调整。

### Vite 配置 (`vite.config.js`)

```js
export default defineConfig({
  base: '/crm/',                         // 部署路径
  server: {
    port: 9999,
    proxy: {
      '/index.php': { target: 'http://crm.dev.xtocn.com', changeOrigin: true },
      '/uploads':   { target: 'http://crm.dev.xtocn.com', changeOrigin: true },
    },
  },
  build: {
    outDir: '../../../public/crm',       // 输出到 PHP 的 public 目录
  },
})
```

**注意**：改 `proxy.target` 指向你的后端地址。生产构建后，`index.php` 请求不走 Vite，直接由同域 Nginx 转发给 PHP。

## 开发注意事项

### 前后端通信

前端所有 API 请求走 `index.php?s=/platform`：

```js
import { axios as request } from '@/utils/request'

// GET 列表
request({ url: '/crm.customer/lists', method: 'get', params: { page: 1 } })
// POST 提交
request({ url: '/crm.customer/add', method: 'post', data: { name: 'xx' } })
// GET 详情
request({ url: '/crm.customer/detail', method: 'get', params: { id: 1 } })
```

**路由规则**：URL 用点号分隔 `模块.控制器/方法`。子目录也用点号：`crm.admin.user/list`。

### 响应格式

拦截器已解包，`res.data` 直接是业务数据：

```js
request({ url: '/crm.customer/lists' }).then(res => {
  // res.data = { list: [...], total: 100 }   ← 分页
  // res.data = { list: [...] }                ← 全量下拉
  // res.data = { detail: {...} }              ← 单条详情
})
```

**陷阱**：分页列表 `res.data.list` 是对象 `{data: [...], total: N}`；全量下拉 `res.data.list` 是数组 `[...]`。使用前先确认后端是 `paginate()` 还是 `select()`。

### 导入铁律

```js
// ✅ 必须具名导入 — 模块没有 default 导出
import { axios as request } from '@/utils/request'

// ❌ 这行不通 — undefined
import request from '@/utils/request'
```

### 路由注意事项

```js
// 带子菜单的父路由必须加 name，否则子菜单不显示
{ path: '/crm/customer', name: 'customer', component: RouteView, children: [...] }

// 隐藏路由：meta.hidden，不是顶层 hidden
{ path: '/crm/customer/detail', meta: { hidden: true } }

// 当前使用 hash mode：/#/passport/login
```

### Skill（插件）集成

插件前端文件从 `skills/{name}/views-v2/` 复制到 `src/skills/{name}/`：

```
skills/invoice/views-v2/          →  src/skills/invoice/
    routes-v2.js                       routes-v2.js
    invoice/index.vue                  invoice/index.vue
    ...
```

宿主路由一行引入：

```js
import invoiceRoutes from '@skills/invoice/routes-v2'
// children: [...invoiceRoutes]
```

### 权限控制

前端路由 `meta.permission` 控制菜单显隐，后端 `checkAction()` 做 API 级拦截：

```php
// Controller
if (!$this->checkAction('/crm/customer/delete', '删除客户')) return;
```

### 多租户

**这是最容易踩的坑。** 所有数据必须归属 `store_id`。

- Model 查询自动带 `store_id` scope ✅
- `Db::name()` **不自动**带 — 必须手动 `where('store_id', $storeId)` ⚠️
- `Db::table()` **禁止使用** — 硬编码前缀会炸 🔴
- Service 方法必须接收 `$storeId` 参数

### 前端技术栈对比

| 项目 | Vue 版本 | UI | 构建 |
|------|---------|----|------|
| frontend-v2（当前） | Vue 3.5 | Ant Design Vue 4 | Vite 8 |

## AI 架构（五层）

CRM 的 AI 不是单次 API 调用的"外挂"，而是一套从**数据采集 → 分析发现 → 建议生成 → 效果追踪 → 对话执行**的完整闭环。五层自底向上：

```
┌──────────────────────────────────────────────────┐
│  ⑤ Agent 引擎   对话查数据 / 生成草稿 / 执行操作    │
├──────────────────────────────────────────────────┤
│  ④ 反馈闭环     采纳/拒绝 → 追踪 AI 建议效果       │
├──────────────────────────────────────────────────┤
│  ③ 洞察引擎     主动扫描 → 发现机会 → AI 增强文案  │
├──────────────────────────────────────────────────┤
│  ② Service 层   AI 可调用的业务接口（与人共用）     │
├──────────────────────────────────────────────────┤
│  ① 事件流       所有操作写入不可变 event_log       │
└──────────────────────────────────────────────────┘
```

### ① 事件流 — 不可变事实

所有业务操作写入 `crm_event_log` 表，作为 AI 系统的唯一数据源。事件与业务数据在同一个事务中写入，保证原子一致。

**14 种事件类型**：`customer_created/updated/claimed/released`, `follow_up_completed`, `funnel_stage_changed`, `quotation_created/sent/confirmed/rejected/converted`, `order_created/status_changed`, `contract_signed`, `lead_converted`

- `EventService::record()` 不开启自己的事务，参与调用方事务
- `event_log` 只追加不修改，任何 AI 分析都可以重放
- 事务提交后 `handleEvent()` 实时失效已被解决的旧洞察

### ② Service 层 — 人与 AI 共用

Agent 工具调用的不是"AI 专用 API"，而是与前端 Controller 完全相同的 Service 方法。Agent 调用 `getCustomerDetail(42)` 拿到的是和前端页面一模一样的数据。

```php
// Service 统一返回格式
['success' => bool, 'data' => mixed, 'error' => string]
```

### ③ 洞察引擎 — 主动发现

每日凌晨自动扫描，**规则先于 AI**：

**A 级扫描（纯规则，零 API 成本）：**

| 规则 | 输出类型 | 检测逻辑 |
|------|----------|----------|
| `scanFollowupOverdue` | `followup_overdue` | 报价 status=2，days_since > max(avg_reply, 7) |
| `scanRepurchaseWindow` | `repurchase_window` | ≥2 订单，days_since ≥ avg_interval × 0.8 |
| `scanPoolWarning` | `pool_warning` | 公海客户滞留预警 |
| `scanChurnRisk` | `churn_risk` | 上次跟进 > 历史间隔 × 1.5，30天无跟进 |
| `scanDormant` | `dormant` | 90天无任何互动 |

**B 级 AI 增强**：仅对 A 级命中结果调 DeepSeek 生成 30 字跟进建议，存入 `crm_ai_suggestion`。

**事件驱动实时失效**：客户被认领 → `pool_warning` resolved；订单创建 → `repurchase_window` + `churn_risk` + `dormant` resolved；报价成交 → `followup_overdue` + `churn_risk` resolved。

**设计理念**：5 条规则覆盖 80% 的销售管理需求，零 API 费用。只有真正命中的才调 AI。1000 客户的租户通常每天只产生 5-15 条洞察。

### ④ 反馈闭环

工作台以卡片展示洞察（"需关注" / "机会发现"），每条附带 AI 建议。用户可**采纳/拒绝**，追踪采纳率。

- 追踪指标：总建议数 / 采纳数 / 拒绝数 / 采纳率 / 转化率
- 没有反馈闭环的 AI 是一锤子买卖——你不知道建议到底帮到销售没有

### ⑤ Agent 引擎

使用 OpenAI 兼容的 Function Calling 协议，最多 8 轮工具调用循环。

**8 个内置工具**：`searchCustomer`, `getCustomerDetail`, `getOrderList`, `getQuotationList`, `getFollowupHistory`, `getMyStats`, `getDashboard`, `searchProduct`

**技能包工具动态注入**：`SkillPipeline::getAllSkillTools()` 扫描已安装技能包，自动注册其 Function Calling 工具。安装新技能包后 Agent 自动获得新能力，无需改代码。

**完整数据流**：

```
用户操作 → Controller → Service → [业务写入 + EventService::record()]
    │                                    ↓
    │                            crm_event_log（不可变）
    │                                    ↓
    │                     ┌──────────────┴──────────────┐
    │                     ↓                             ↓
    │             实时：handleEvent()          每日：CrmInsight Timer
    │             失效冲突洞察                    ↓
    │                                   InsightService::dailyScan()
    │                                     ├── A 规则扫描（5 条）
    │                                     ├── B AI 增强建议
    │                                     └── 通知生成
    │                                          ↓
    └──────────────────────────→ Dashboard::index()
                                     ├── insights（洞察卡片）
                                     ├── funnel（漏斗图）
                                     ├── kpi / ranking（投影数据）
                                     └── unreadCount（通知铃铛）
```

**六张核心 AI 表**：

| 表 | 用途 | 特点 |
|----|------|------|
| `crm_event_log` | 事件流 | 只追加，不可变 |
| `crm_insight` | 洞察缓存 | 幂等键去重，30天过期 |
| `crm_projection` | 统计快照 | 每日刷新 |
| `crm_ai_suggestion` | AI 建议追踪 | 采纳/拒绝/忽略 |
| `crm_notification` | 站内通知 | 已读/未读 |
| `crm_agent_log` | Agent 会话日志 | 对话 + 工具调用记录 |

详细设计见 `AGENTS.md` 和 `.claude/agents/crm.md`。

## 界面展示

### 登录页

![登录页](docs/登录页.png)

### 工作台首页

仪表盘、洞察卡片、漏斗图、KPI 看板一站式呈现。

![首页](docs/首页.png)

### 客户管理

客户列表、详情、跟进记录、报价/订单关联。

![客户页](docs/客户页.png)

### 销售管理

报价、订单、合同全流程。

![销售页](docs/销售页.png)

### 知识库

产品知识、销售话术、竞品对比、成功案例。

![知识库页](docs/知识库页.png)

### 应用中心

已安装技能包（报价、发票、内容引擎等）。

![应用页](docs/应用页.png)

### 系统设置

用户、角色、部门、菜单权限管理。

![系统设置页](docs/系统设置页.png)

### CRM 设置

租户级参数：公海规则、洞察阈值、跟进提醒等。

![CRM设置页](docs/CRM设置页.png)

### AI 设置

内容引擎 Agent 配置、预设管理、变量定义。

![AI设置页](docs/AI设置页.png)

---

## 联系与赞助

### 联系作者

扫描下方二维码添加微信，备注"雄韬CRM"。

![联系二维码](docs/联系二维码.png)

### 赞助支持

如果这个项目对你有帮助，欢迎请作者喝杯咖啡 ☕

![赞助二维码](docs/赞助二维码.png)

---

## 目录结构

```
mpp/crm/frontend-v2/
├── public/
│   └── config.js              ← 运行时配置
├── src/
│   ├── api/                   ← axios 请求（按模块分文件）
│   ├── components/            ← 公共组件
│   ├── layouts/               ← BasicLayout / UserLayout
│   ├── router/                ← 路由（asyncRoutes + permission guard）
│   ├── skills/                ← 插件前端（从 skills/ 复制进来）
│   ├── stores/                ← Pinia（user, permission）
│   ├── utils/                 ← axios 封装、权限工具
│   └── views/                 ← 页面组件
│       ├── crm/               ← 客户 / 线索 / 报价 / 订单 / 合同
│       ├── passport/          ← 登录
│       ├── manage/            ← 用户 / 角色 / 部门 / 菜单
│       └── index/             ← 工作台
├── index.html
├── vite.config.js
└── package.json
```

## 常见问题

| # | 现象 | 原因 | 处理 |
|---|------|------|------|
| 1 | API 返回 undefined | `import request from` | 用 `import { axios as request } from` |
| 2 | 侧边栏菜单不显示子项 | RouteView 缺 `name` | 加 `name: 'xxx'` |
| 3 | 列表搜索清空后无数据 | 空字符串 `''` 进入 SQL | 过滤 `value !== ''` |
| 4 | Modal 打开 editor 报错 | 编辑器在隐藏 DOM 中初始化 | 用 `v-if` 控制 Modal 内容挂载 |
| 5 | a-modal width 不生效 | scoped 样式穿不透 | 去掉 scoped，加 `wrapClassName` |
| 6 | 按钮全白无样式 | `@ant-design/cssinjs` 未装 | `npm install @ant-design/cssinjs` |
| 7 | 构建后资源 404 | base 路径不匹配 | 确认 `vite.config.js` 的 `base` 值 |
| 8 | 路由跳转地址不对 | hash/history mode 不一致 | 确认 router/index.js 的 mode |

## License

Apache-2.0
