# 运行说明

## Cursor 导入

1. 打开 Cursor → Settings → Rules
2. 点击 **Import from GitHub** 或 **Add Rule**
3. 输入本仓库地址，Cursor 会自动识别 `.cursor/rules/` 下的 `.mdc` 规则文件
4. 选择需要的规则导入即可

## 手动使用

将 `.cursor/rules/` 目录复制到目标项目根目录下：

```bash
cp -r .cursor/rules/ /path/to/your-project/.cursor/rules/
```

## 通义灵码

将 `.lingma/rules/` 目录复制到目标项目根目录下，通义灵码会自动加载。

## 注意事项

- 规则按 `globs` 模式匹配文件，仅在编辑对应类型文件时生效
- `alwaysApply: true` 的规则对所有文件生效
- 可在 Cursor Settings → Rules 中查看和管理已导入的规则
