---
type: module-spec
version: 1.0
source: src/security/
files_analyzed: 8
loc: 4028
last_updated: 2026-02-09
---

# Security Spec

## 1. Intent (意图)
安全审计和加固模块：
- 全面的安全审计报告生成
- 文件系统权限检查
- 配置中的敏感信息检测
- 外部内容安全扫描
- Skill 代码安全扫描
- 插件信任评估
- 攻击面分析

## 2. Interface (接口定义)
- `runSecurityAudit(cfg)` → `SecurityAuditReport` — 运行完整安全审计
- `SecurityAuditFinding` — 审计发现（checkId, severity, title, detail, remediation）
- `SecurityAuditSeverity` = `"info" | "warn" | "critical"`
- `inspectPathPermissions(path)` — 检查文件系统权限
- Skill scanner, external content scanner

## 3. Business Logic (业务逻辑)
- **审计流程**: 收集各类检查项 → 评估严重性 → 生成报告
- **检查项**: 配置中的明文密钥、文件权限、Gateway TLS、暴露的端口、Hook 加固、模型卫生、小模型风险
- **深度检查**: Gateway 连接探测、浏览器配置检查

## 4. Data Structures (数据结构)
- `SecurityAuditReport` — 审计报告（summary, findings, deep checks）
- `SecurityAuditFinding` — 单个发现
- `SecurityAuditSummary` — 汇总统计

## 5. Constraints (约束)
- 审计为只读操作，不修改系统
- fix 命令可自动修复部分问题

## 6. Edge Cases (边界条件)
- Gateway 不可达时的降级检查
- 权限检查在不同 OS 上的差异

## 7. Technical Debt (技术债务)
- audit-extra.ts 可能包含过多检查逻辑

## 8. Dependencies (依赖)
- **内部**: config, gateway, browser, channels, infra, pairing
- **外部**: 无
