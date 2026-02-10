---
type: skill-spec
version: 1.0
source: skills/openai-image-gen/
skill_name: openai-image-gen
files_analyzed: 3
loc: ~500
last_updated: 2026-02-10
---

# OpenAI Image Generation Skill Specification

## 1. Intent (意图)
OpenAI Image Generation Skill 通过 OpenAI Images API 提供 AI 图像生成功能：
- 批量图像生成和管理
- 多种 OpenAI 模型支持（DALL-E 2/3, GPT-Image系列）
- 随机提示词生成和结构化prompt管理
- 本地图片库和HTML画廊生成
- 高质量图像输出和格式控制
- 创意工作流和批量处理

该技能为创意工作、设计原型、内容生成等场景提供强大的AI图像生成能力。

## 2. Interface (接口定义)
### Core Generation Commands
```bash
# 基础图像生成
python3 scripts/gen.py --prompt "prompt text" --count 4

# 模型和质量选择
python3 scripts/gen.py --model dall-e-3 --quality hd --size 1792x1024

# 批量生成和输出控制
python3 scripts/gen.py --count 16 --out-dir ./custom/path --format webp

# 风格和背景控制
python3 scripts/gen.py --style vivid --background transparent
```

### Supported Models
- **DALL-E 3**: 最新高质量模型，单次生成，多种尺寸
- **DALL-E 2**: 经典模型，支持批量生成
- **GPT-Image-1/1.5**: 新一代模型，平衡质量和速度
- **Custom Models**: 支持自定义模型配置

### Configuration Options
```yaml
openai_image_gen:
  enabled: true
  api_key: "${OPENAI_API_KEY}"
  default_model: "dall-e-3"
  default_size: "1024x1024"
  default_quality: "standard"  # standard, hd
  default_style: "vivid"       # vivid, natural
  output_dir: "~/Projects/tmp/openai-image-gen"
  gallery_enabled: true
  max_concurrent: 5
  timeout_seconds: 300
```

### Gallery Features
- **HTML Gallery**: 自动生成交互式图片库
- **Metadata Display**: 显示提示词、模型、参数等信息
- **Preview Support**: 缩略图和全尺寸预览
- **Download Links**: 直接下载原始图片
- **Batch Organization**: 按生成批次组织展示

## 3. Business Logic (业务逻辑)
### 图像生成流程
1. **提示词处理**: 解析和优化输入提示词
2. **参数验证**: 验证模型支持的尺寸、质量等参数
3. **API调用**: 并发调用OpenAI API生成图像
4. **文件管理**: 下载并本地保存图像文件
5. **元数据存储**: 记录生成参数和元信息
6. **画廊更新**: 更新HTML画廊展示新图像

### 随机提示词生成
1. **模板系统**: 预定义的提示词模板和变量
2. **随机采样**: 从不同类别中随机选择元素
3. **风格控制**: 艺术风格、颜色、构图等控制选项
4. **质量增强**: 自动添加质量提升关键词
5. **去重机制**: 避免生成重复的提示词

### 批量处理优化
1. **并发控制**: 合理控制并发数避免API限制
2. **错误重试**: 自动重试失败的生成请求
3. **进度跟踪**: 实时显示批量生成进度
4. **资源管理**: 磁盘空间检查和清理机制
5. **成本控制**: 生成成本估算和预算控制

### 输出管理
1. **文件命名**: 智能文件命名包含时间戳和参数
2. **目录结构**: 按日期和批次组织文件结构
3. **格式转换**: 支持PNG、WEBP、JPEG格式输出
4. **压缩优化**: 可选的图像压缩和优化
5. **备份机制**: 重要图像的自动备份

## 4. Data Structures (数据结构)
### Generation Request
```typescript
interface ImageGenerationRequest {
  prompt: string;
  model: "dall-e-2" | "dall-e-3" | "gpt-image-1" | "gpt-image-1.5";
  size: "256x256" | "512x512" | "1024x1024" | "1792x1024" | "1024x1792";
  quality?: "standard" | "hd";
  style?: "vivid" | "natural";
  n?: number;  // 生成数量
  response_format?: "url" | "b64_json";
  user?: string;  // 用户标识
}
```

### Generation Result
```typescript
interface ImageGenerationResult {
  id: string;
  prompt: string;
  model: string;
  parameters: GenerationParameters;
  images: GeneratedImage[];
  created_at: Date;
  cost: number;
  duration: number;
  status: "success" | "error" | "partial";
}

interface GeneratedImage {
  url: string;
  local_path: string;
  filename: string;
  size_bytes: number;
  dimensions: { width: number; height: number };
  format: string;
}
```

### Gallery Metadata
```typescript
interface GalleryMetadata {
  title: string;
  description?: string;
  created_at: Date;
  total_images: number;
  total_cost: number;
  models_used: string[];
  batches: GalleryBatch[];
}

interface GalleryBatch {
  id: string;
  timestamp: Date;
  prompt: string;
  parameters: GenerationParameters;
  images: string[];  // 文件路径列表
}
```

## 5. Constraints (约束)
### OpenAI API 限制
- **Rate Limits**: 每分钟请求数限制（模型相关）
- **Image Limits**: DALL-E 3每次只能生成1张图
- **Size Restrictions**: 不同模型支持的尺寸限制
- **Content Policy**: OpenAI内容政策限制
- **Cost Limits**: API调用成本控制

### 系统资源限制
- **Disk Space**: 本地存储空间要求（高质量图像较大）
- **Network Bandwidth**: 图像下载带宽要求
- **Memory Usage**: 批量处理时的内存使用
- **Processing Time**: 大批量生成的时间成本
- **Concurrent Requests**: 并发请求数控制

### 质量和格式限制
- **Image Quality**: 不同质量设置的文件大小差异
- **Format Support**: 支持的输出格式限制
- **Resolution Limits**: 最大分辨率限制
- **Compression**: 压缩质量和文件大小平衡

## 6. Edge Cases (边界条件)
### API 调用异常
- **Rate Limit Exceeded**: 触发速率限制的处理
- **Content Rejection**: 违反内容政策的提示词处理
- **API Outages**: OpenAI服务中断的降级处理
- **Authentication Failure**: API密钥无效或过期
- **Model Unavailable**: 特定模型临时不可用

### 文件系统问题
- **Disk Full**: 磁盘空间不足的处理
- **Permission Denied**: 文件写入权限问题
- **Path Too Long**: 文件路径过长的处理
- **Invalid Characters**: 文件名包含非法字符
- **Concurrent Access**: 多进程同时访问文件

### 生成质量问题
- **Poor Quality Results**: 生成结果质量不佳的重试机制
- **Prompt Misinterpretation**: 提示词理解偏差的处理
- **Inconsistent Style**: 批量生成风格不一致的问题
- **Generation Failures**: 部分图像生成失败的处理

### 网络和下载问题
- **Download Failures**: 图像下载失败的重试
- **Corrupted Files**: 下载文件损坏的检测
- **Slow Network**: 网络慢导致的超时处理
- **Large Files**: 超大图像文件的处理

## 7. Technical Debt (技术债务)
### 当前问题
- **Error Handling**: 错误处理不够细致，缺乏分类处理
- **Configuration**: 配置选项分散，缺乏统一管理
- **Logging**: 日志记录不完整，调试困难
- **Testing**: 缺乏自动化测试，特别是集成测试

### 架构问题
- **Script-based**: 基于脚本的架构，缺乏模块化设计
- **State Management**: 缺乏状态管理，重复操作效率低
- **Code Reuse**: 代码复用性差，功能重复实现
- **Documentation**: 内部文档不足，维护困难

### 功能缺失
- **Image Editing**: 不支持图像编辑和变体生成
- **Batch Management**: 批次管理功能简单
- **Analytics**: 缺乏生成统计和分析功能
- **Integration**: 与其他技能集成度低

### 性能问题
- **Memory Usage**: 大批量处理时内存使用过高
- **Disk I/O**: 频繁磁盘操作影响性能
- **Network Efficiency**: 网络请求效率有待优化
- **Caching**: 缺乏智能缓存机制

## 8. Dependencies (依赖)
### 外部依赖
- **OpenAI API**: OpenAI Platform API密钥和访问
- **Python 3**: Python运行环境 (>= 3.8)
- **openai**: OpenAI Python客户端库
- **requests**: HTTP请求库
- **Pillow**: 图像处理库（可选，用于格式转换）

### 系统依赖
- **Network**: 稳定的互联网连接
- **Storage**: 足够的本地存储空间
- **File System**: 文件系统读写权限
- **Environment Variables**: OPENAI_API_KEY环境变量

### OpenClaw内部依赖
- **config**: 技能配置管理
- **logging**: 操作日志记录
- **security**: API密钥安全存储
- **utils**: 文件处理和工具函数

### 可选依赖
- **Web Server**: 本地HTTP服务器（用于画廊展示）
- **Image Optimization**: 图像优化工具
- **Cloud Storage**: 云存储同步（备份用途）
- **Database**: 元数据存储（高级用途）

---

## 常用操作示例

### 基础图像生成
```bash
# 单张高质量图像
python3 scripts/gen.py --model dall-e-3 --quality hd --prompt "futuristic city skyline"

# 批量生成（DALL-E 2）
python3 scripts/gen.py --model dall-e-2 --count 8 --prompt "abstract art patterns"
```

### 高级配置
```bash
# 自定义尺寸和风格
python3 scripts/gen.py --size 1792x1024 --style natural --background transparent

# 指定输出目录
python3 scripts/gen.py --out-dir ./project/assets --format webp
```

### 随机批量生成
```bash
# 使用随机提示词模板
python3 scripts/gen.py --random --count 12 --theme "nature"

# 查看生成的画廊
open ~/Projects/tmp/openai-image-gen-*/index.html
```

该技能为创意工作者、设计师和开发者提供了强大的AI图像生成能力，支持从概念设计到批量内容生成的各种需求。