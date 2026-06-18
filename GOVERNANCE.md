# Governance

XTCRM 采用**单一维护者 + 社区贡献**模型，随着社区成长逐步向多维护者演进。

## 角色与职责

| 角色 | 权限 | 职责 |
|------|------|------|
| **Maintainer** | 仓库写入权、合并 PR、发布版本 | 代码审查、架构决策、安全响应、社区方向 |
| **Contributor** | Fork + PR | 提交代码、文档、Bug 报告、功能建议 |
| **Community** | Issue、Discussion | 使用反馈、功能请求、文档纠错 |

## 当前 Maintainer

- **xtoyun** — 项目作者，拥有最终决策权

## 成为 Contributor

1. Fork 仓库，在分支上提交修改
2. 遵循 [CONTRIBUTING.md](CONTRIBUTING.md) 规范
3. 提交 Pull Request，描述修改动机和内容
4. 通过代码审查后合并

**持续贡献者**（3 次以上被合并的实质性 PR）可被提名为 Maintainer。

## 成为 Maintainer

提名需满足以下条件：

- 至少 3 次实质性 PR 被合并
- 参与过代码审查（Review 他人的 PR）
- 在 Issue/Discussion 中提供了有价值的帮助
- 理解并认同项目的架构理念和技术方向

提名由现有 Maintainer 发起，经现有 Maintainer 讨论后决定。Maintainer 的添加和移除记录在本文档中。

## 决策流程

### 日常决策（默认：Lazy Consensus）

- 小改动（文档修正、Bug fix、格式调整）：PR 提交后，Maintainer 审查通过即合并
- 不需要事先讨论，沉默即同意

### 重大变更（需公开讨论）

以下类型的变更**必须**在 Issue 中发起讨论，至少保留 **7 天** 的评论期：

- 架构级改动（如更换框架、引入新的核心依赖）
- Breaking Change（API 不兼容、数据库迁移需人工介入）
- 许可证变更
- 新增/移除 Maintainer
- 项目加入/退出基金会托管

讨论结束后，由 Maintainer 综合各方意见做出最终决策。如有分歧，以项目作者（xtoyun）为准。

### 冲突解决

- 技术争议以代码和数据说话（性能 benchmark、可维护性评估）
- 无法达成共识时，项目作者拥有最终裁决权
- 所有决策记录在对应的 Issue/PR 中，确保可追溯

## 行为准则

所有参与者必须遵守 [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)。违反者可能被移除参与资格。

## 商标与品牌

- "XTCRM" 项目名和 Logo 归项目作者所有
- 衍生作品如使用相同名称，需明确标注与原项目的区别（如 "XTCRM Community Edition"）
- 禁止使用项目名称进行未经授权的商业认证或背书

## 基金会托管

当项目满足以下条件时，将考虑移入中立基金会（如开放原子、Apache 或 Linux Foundation）：

1. 至少 3 名活跃 Maintainer（且来自不同组织）
2. 稳定的用户基础和社区活跃度
3. 明确的版本路线图和长期维护承诺
4. 完整的开源合规基础设施（License / DCO / SECURITY / 依赖审计）

加入基金会的目的是将项目的商标和关键资产置于中立法律实体之下，确保项目不受单一个人或企业控制。届时，治理模型将按基金会要求调整。

## 历史记录

| 日期 | 事件 |
|------|------|
| 2025-06 | 项目首次公开发布，采用单一 Maintainer 模型 |
