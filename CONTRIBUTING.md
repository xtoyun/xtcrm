# Contributing to XTCRM

感谢你对 XTCRM 的关注！本文档帮助你了解如何参与贡献。

## 沟通渠道

- **Bug 报告 / 功能建议**：通过 [GitHub Issues](https://github.com/xtoyun/xtcrm/issues) 提交
- **使用讨论 / 求助**：通过 [GitHub Discussions](https://github.com/xtoyun/xtcrm/discussions) 发帖
- **安全漏洞**：请勿公开提交 Issue，直接联系 maintainer（见 [SECURITY.md](SECURITY.md)）

## 开发流程

### 1. 环境搭建

```bash
# Fork + Clone
git clone https://github.com/YOUR_USERNAME/xtcrm.git
cd xtcrm

# 后端：PHP 7.4 + MySQL + Redis（推荐宝塔面板搭建）
# 详见 README.md「PHP（宝塔面板）」章节

# 前端：Node 22 + Vite
cd mpp/crm/frontend-v2
nvm use
npm install
npm run dev   # → http://localhost:9999
```

### 2. 选择 Issue

- 从 [Issues](https://github.com/xtoyun/xtcrm/issues) 中挑选感兴趣的
- 在 Issue 下留言说明你正在处理，避免多人重复劳动
- 不确定从哪开始？搜索 `good first issue` 标签

### 3. 创建分支

```bash
git checkout -b feat/my-feature     # 新功能
git checkout -b fix/my-bug          # Bug 修复
git checkout -b docs/some-update    # 文档更新
```

### 4. 编码规范

#### PHP

- 遵循 PSR-12 编码风格
- Controller 继承 `app\platform\backend\BaseController`
- Model 继承 `cores\BaseModel`，`protected $name` 设无前缀表名
- Service 继承 `app\platform\BaseService`，所有方法必须显式传递 `$storeId`
- **铁律**：`Db::name()` 必须手动 `where('store_id', $storeId)`；禁止 `Db::table()`

#### Vue 3 / JavaScript

- 使用具名导入：`import { axios as request } from '@/utils/request'`
- 带子菜单的 RouteView 必须添加 `name`
- 隐藏路由在 `meta` 中设置 `hidden: true`
- 详细规范见 [CLAUDE.md](CLAUDE.md)

### 5. 数据库迁移

新增或修改表结构，SQL 文件放在对应模块的 `migrations/` 目录：

```
mpp/crm/migrations/NNN_版本_描述.sql
```

命名规则：序号递增（取当前最大序号 +1），版本号与下一版本一致。

### 6. 提交信息

遵循 Conventional Commits：

```
feat(module): 新增客户导入功能
fix(quotation): 修复报价金额精度丢失
docs(readme): 补充宝塔部署说明
refactor(customer): 提取公海规则到独立方法
```

### 7. Pull Request

1. 确保 PR 针对 `main` 分支
2. PR 标题遵循 Conventional Commits
3. PR 描述应包含：
   - **做了什么** — 一句话概述
   - **为什么做** — 解决了什么问题
   - **如何测试** — 测试步骤和预期结果
   - **Breaking Changes** — 如有，必须标注
4. 确保与最新 `main` 分支无冲突

### 8. 代码审查

- 至少一位 Maintainer 审查通过后才能合并
- CI 检查（如有）必须通过
- 审查意见请客观、专业、对事不对人

## 社区贡献

以下贡献同样宝贵：

- 完善文档、修正错别字
- 编写或翻译教程
- 回答社区提问
- 报告 Bug 并附上复现步骤
- 提案新功能（先开 Issue 讨论）

## 许可

贡献的代码默认以 Apache License 2.0 授权。提交 PR 即表示你同意此条款。
