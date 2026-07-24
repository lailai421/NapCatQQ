# 代码风格与约定

- 语言：TypeScript，ES module。
- ESLint 使用 `neostandard({ ts: true, semi: true })`，强制分号。
- 未使用变量报错，但以下划线开头的参数、变量、catch 错误可忽略。
- camelcase 规则关闭，允许下划线命名。
- 尾随逗号使用 `always-multiline`。
- 现有代码多采用类式 action、接口类型声明、命名导出与路径别名 `@/...`。