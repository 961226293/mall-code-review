# 输出模板

写入 `CR/{yyyyMMdd}_CR.md` 时，使用以下结构：

```markdown
# {yyyyMMdd} Code Review

## Review Summary

- Review scope: current git changes / specified diff range
- Files reviewed: X
- Lines changed: Y
- Overall assessment: APPROVE / REQUEST_CHANGES / COMMENT
- Risk summary: high / medium / low

## Findings

### P0 - 阻塞问题
- [path:line] 标题
  - 问题描述
  - 风险影响
  - 建议修法

### P1 - 高优先级问题
- ...

### P2 - 中优先级问题
- ...

### P3 - 低优先级建议
- ...

## 新增/修改的类

- `com.xxx.AaaController` - 新增
- `com.xxx.BbbServiceImpl` - 修改

## 修改文件概览

- `mall-admin-web/.../AaaController.java` - 接口入口调整
- `mall-admin-core/.../BbbServiceImpl.java` - 业务逻辑调整

## 代码修改主要目的总结

- 模块/功能 A：本次修改主要在解决什么问题
- 模块/功能 B：本次修改主要在增强什么能力

## Removal / Iteration Plan

- Safe delete now:
- Follow-up plan:

## Additional Checks

- ARCHITECTURE.md constraints checked:
- CR-STANDARD.md constraints checked / skipped if missing:
- Team standard checks checked:
- Not verified:

## 修复执行结果

- 已修复：
- 未修复：
- 不修复原因：

## Repair Options

- 1. 修复全部问题（只改本地文件，不提交 git）
- 2. 只修复 P0 / P1
- 3. 只修复指定问题
- 4. 仅输出 review，不改代码
```

如果没有发现问题，也保留同样结构，并明确写出：

- 检查了什么
- 没有验证什么
- 剩余风险是什么
