# 技术说明 — Cursor Rule Project

## 技术栈总览

| 类别 | 技术 |
|------|------|
| 规则引擎 | Cursor Rules（`.mdc` 格式）、通义灵码（`.lingma/rules`） |
| 目标语言 | Java（Spring Boot + MyBatis）、Python、SQL |
| 规则格式 | MDC = YAML Frontmatter + Markdown 正文 |
| 版本控制 | Git + GitHub |

## 目录结构说明

```
cursor-rule-project/
├── .cursor/                        # Cursor IDE 配置根目录
│   ├── rules/                      # 核心：所有 .mdc 规则文件
│   └── skills/                     # Cursor Skills 扩展
├── .lingma/rules/                  # 通义灵码规则（兼容层）
├── a-doc/                          # 项目文档集中目录
│   ├── interview/                  # 面试复盘文档
│   │   ├── technical.md            # 本文件：技术说明
│   │   └── readme.md               # 面试准备：技术难点 + 面试题
│   ├── install.md                  # 安装/导入说明
│   ├── version.md                  # 版本迭代记录
│   └── *.md                        # 各规则的详细说明文档
└── README.md                       # 项目总览入口
```

## 每个文件的作用

### .cursor/rules/ — 规则文件

| 文件 | 类型 | 作用 |
|------|------|------|
| `project-check-rules.mdc` | 通用 | 检查项目是否包含必要文档（README、install、version），强制文档归入 `a-doc/` |
| `lingma-rules.mdc` | 通用 | 兼容层：启动时扫描 `.lingma/rules/` 并加载其中规则 |
| `interview-review-rules.mdc` | 通用 | 面试复盘：Review 代码后自动生成技术说明和面试题文档 |
| `java-code-optimization-rules.mdc` | Java | 接口方法省略 `public`、`default` 方法、注解接口等简写规范 |
| `java-comment-check-rules.mdc` | Java | 禁止 `@data` 等非标准注释，规范 JavaDoc 格式 |
| `java-null-check-rules.mdc` | Java | 统一使用 `Objects.isNull()` / `Objects.nonNull()` / `Objects.requireNonNull()` |
| `java-jdk-prefer-rules.mdc` | Java | JDK 已有等价方法时禁止引入三方库（附对照表） |
| `spring-web-layered-rules.mdc` | Java | Controller→Service→Mapper 分层职责，DTO/Entity 严格分离，统一响应格式仅限 Controller |
| `database-design-rules.mdc` | SQL | 表/列/索引命名、字段类型选择、软删除、utf8mb4 等规范 |
| `python-project-rules.mdc` | Python | 依赖文件必须存在、`.gitignore` 规范、PEP 8 导入顺序、f-string、异常处理 |

### a-doc/ — 文档文件

| 文件 | 作用 |
|------|------|
| `install.md` | Cursor 导入步骤、手动复制方式、通义灵码使用说明 |
| `version.md` | 版本变更记录，按 `vX.Y.Z (日期)` + 变更列表格式 |
| `database-design-rules.md` | 数据库设计规则的详细版（含完整建表 SQL 示例） |
| `java-code-optimization-rules.md` | Java 代码优化规则详细版 |
| `java-comment-check-rules.md` | Java 注释检查规则详细版 |
| `spring-web-layered-rules.md` | Spring 分层架构规则详细版（含 DTO/Entity/Result 完整代码） |

## 核心设计分析

### MDC 规则文件格式

每个 `.mdc` 文件由两部分构成：

```
---
description: 规则的一句话描述
globs: "**/*.java"          # 文件匹配模式，为空则需手动触发
alwaysApply: false          # true = 始终生效，false = 按 globs 或手动触发
---

# 规则正文（Markdown）
```

- `globs` 控制规则在哪些文件编辑时自动激活
- `alwaysApply: true` 的规则（如 project-check、lingma）对所有操作生效
- 正文使用 Markdown 格式，AI 可直接理解并执行

### 规则分层架构

```
alwaysApply 层（全局生效）
├── project-check-rules     → 项目结构守卫
├── lingma-rules            → 跨工具兼容
└── interview-review-rules  → 按需触发

Java 层（编辑 .java 时生效）
├── java-code-optimization  → 代码风格
├── java-comment-check      → 注释质量
├── java-null-check         → 空值安全
├── java-jdk-prefer         → 依赖治理
├── spring-web-layered      → 架构约束
└── database-design         → 数据层规范

Python 层（编辑 .py 时生效）
└── python-project          → 项目规范 + 编码规范
```

### 双工具兼容设计

项目同时支持 Cursor 和通义灵码两种 AI 编码工具：
- Cursor 读取 `.cursor/rules/*.mdc`
- 通义灵码读取 `.lingma/rules/*`
- `lingma-rules.mdc` 作为桥接层，确保 Cursor 也能读取通义灵码规则
- 冲突时以 `.cursor/rules/` 为准
