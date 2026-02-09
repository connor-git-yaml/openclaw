---
type: module-spec
version: 1.0
source: src/media/
files_analyzed: 12
loc: 2048
last_updated: 2026-02-09
---

# Media Spec

## 1. Intent (意图)
媒体文件处理模块：
- 音频处理（格式转换、标签读取）
- 图片操作（缩放、格式转换）
- 媒体文件下载和缓存
- MIME 类型检测
- 输入文件预处理（为 LLM 准备附件）
- 媒体 URL 托管

## 2. Interface (接口定义)
- `fetchMedia(url)` — 下载媒体文件
- `resolveMediaHost(config)` — 解析媒体托管 URL
- `processInputFiles(files)` — 预处理输入文件
- `detectMimeType(buffer)` — MIME 类型检测
- 音频/图片操作工具函数

## 3. Business Logic (业务逻辑)
- **媒体下载**: URL 获取 → MIME 检测 → 缓存到本地
- **图片处理**: 缩放到 LLM 支持的尺寸 → 格式转换
- **音频处理**: 格式转换 → 标签提取

## 4. Data Structures (数据结构)
- 媒体常量（支持的格式、大小限制等）

## 5. Constraints (约束)
- 图片处理可能依赖 sharp 或 native 工具
- 音频处理可能依赖 ffmpeg

## 6. Edge Cases (边界条件)
- 不支持的媒体格式
- 超大文件处理
- 网络下载失败重试

## 7. Technical Debt (技术债务)
- 模块较小，职责清晰

## 8. Dependencies (依赖)
- **内部**: config, infra
- **外部**: sharp（可选）, ffmpeg（可选）
