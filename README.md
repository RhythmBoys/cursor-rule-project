# Cursor Rule Project

Cursor / 通义灵码 共享规则集，为 Java、Python 等项目提供统一的编码规范和最佳实践。

## 规则列表

### 通用规则

| 规则文件 | 说明 |
|---------|------|
| project-check-rules | 项目结构与文档检查 |
| lingma-rules | 通义灵码规则兼容 |
| interview-review-rules | 面试复盘：Review 代码生成技术说明和面试题 |

### Java 规则

| 规则文件 | 说明 |
|---------|------|
| java-code-optimization-rules | 代码优化（接口方法简写等） |
| java-comment-check-rules | 注释规范检查 |
| java-null-check-rules | 空值检查统一使用 Objects |
| java-jdk-prefer-rules | 优先使用 JDK 内置方法 |
| spring-web-layered-rules | Spring Web 分层架构规范 |
| database-design-rules | 数据库设计规范 |

### Python 规则

| 规则文件 | 说明 |
|---------|------|
| python-project-rules | 依赖管理、项目结构、编码规范 |

## 项目结构

```
├── .cursor/rules/       # Cursor 规则文件（.mdc）
├── .lingma/rules/       # 通义灵码规则文件
├── a-doc/               # 详细文档
│   ├── interview/       # 面试复盘文档
│   ├── install.md       # 运行说明
│   ├── version.md       # 版本记录
│   └── *.md             # 各规则详细说明
└── README.md            # 项目总览
```

## 使用方式

详见 [运行说明](a-doc/install.md)。
