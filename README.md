# PDF 文本统计工具

一个简洁高效的 PDF 文本内容统计分析工具，支持中英文混合文本的精确统计。

## 功能特性

- **极速解析** - 基于 PDF.js 的客户端解析，无需服务器支持
- **全面统计** - 支持中文字符、英文单词、标点符号等详细分类统计
- **简约界面** - 现代化设计，专注核心功能，操作直观
- **响应式设计** - 完美适配桌面端和移动端设备
- **离线可用** - 纯前端实现，上传文件后即可离线使用
- **实时处理** - 即时显示统计结果，无需等待

## 统计内容

### 主要统计
- **中文字符** - 包含汉字及扩展字符集
- **英文单词** - 连续英文字母组成的单词
- **数字** - 0-9 数字字符
- **总字符数** - 不包含空格的总字符

### 详细统计
- **英文字母** - 所有英文字母字符总数
- **中文标点** - ，。！？、；：""''（）【】《》…—～·
- **英文标点** - ,.!?;:'"()[]{}<>/\@#$%^&*-_+=|`~
- **空白字符** - 包括空格和制表符
- **换行符** - \n 字符统计

## 技术栈

- **前端框架**: Vue 3 (Composition API)
- **开发语言**: TypeScript
- **构建工具**: Vite 7
- **PDF 处理**: PDF.js 5.4
- **代码规范**: ESLint
- **包管理**: npm / bun

## 快速开始

### 环境要求

- Node.js >= 20.19.0 或 >= 22.12.0
- npm >= 9 或 bun >= 1

### 安装依赖

```bash
npm install
```

或使用 bun：

```bash
bun install
```

### 开发模式

```bash
npm run dev
```

访问 [http://localhost:5174](http://localhost:5174) 查看应用。

### 构建生产版本

```bash
npm run build
```

构建产物位于 `dist` 目录。

### 预览生产版本

```bash
npm run preview
```

## 使用方法

1. **打开网页** - 在浏览器中访问应用
2. **上传 PDF** - 拖拽 PDF 文件到上传区域，或点击"选择文件"按钮
3. **查看结果** - 文件解析完成后，统计结果将自动显示

## 项目结构

```
pdf_static/
├── src/
│   ├── main.ts          # 应用入口
│   ├── App.vue          # 主组件
│   └── vite-env.d.ts    # Vite 类型声明
├── public/              # 静态资源
├── dist/                # 构建输出
├── index.html           # HTML 模板
├── package.json         # 项目配置
├── tsconfig.json        # TypeScript 配置
├── vite.config.ts       # Vite 配置
└── README.md           # 项目文档
```

## 代码规范

项目使用 ESLint 进行代码规范检查：

```bash
npm run lint
```

运行 TypeScript 类型检查：

```bash
npm run type-check
```

## 故障排除

### 常见问题

**问：PDF 文件无法解析？**
答：请确保 PDF 文件未加密且未损坏。某些扫描版 PDF 可能不包含可提取的文本。

**问：统计结果不准确？**
答：PDF.js 的文本提取基于 PDF 的内部结构，特殊格式或复杂布局可能影响结果准确性。

**问：大文件处理缓慢？**
答：文件解析在浏览器中进行，建议文件大小不超过 50MB。

**问：中文统计异常？**
答：工具支持完整的 Unicode 中文字符集，包括 CJK 扩展区。

### 浏览器兼容性

- Chrome >= 88
- Firefox >= 87
- Safari >= 14
- Edge >= 88

## 贡献

欢迎提交 Issue 和 Pull Request。

1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开 Pull Request

## 更新日志

### v1.0.0 (2025-01-25)
- 初始版本发布
- 全新简约界面设计
- 完整的 PDF 文本统计功能
- 响应式布局支持
- 客户端解析，无需服务器
