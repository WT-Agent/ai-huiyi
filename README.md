# 网腾无限AI 会议模板与框架生成

## 项目介绍

**网腾无限AI 会议模板与框架生成** 是一款根据用户开会意图智能定制专业【会议议程与讨论框架】的效率微应用。该应用基于 Vue 3 + Vite + Vanilla CSS 打造，开会意图输入框与五维框架滑块直接置于首页中心区。用户输入开会意图与目标后，AI 将精算并生成包含分钟级议程、破局提问句式、表决规则与防跑题指南的专业模板，并支持一键打印（导出 PDF）、一键下载 `.md` 文件与在首页下方的 Nomad List 热门会议模板库网格墙中进行下钻与载入。

### 核心特性
- **首页中心意图输入与快捷标签**：开会意图输入框、快捷标签（新项目启动会、季度 OKR 复盘、部门预算对齐、绩效考核面谈、创投路演预演）与五维框架滑块直接呈现在首页中心区。
- **五维会议框架侧重滑块**：结构化程度 (Structure)、目标聚焦度 (Goal Focus)、时间规划力 (Timeline)、角色分工明确度 (Role)、风险预案严密度 (Risk Prep) 的无缝打分。
- **一键打印与导出 PDF (Print & Export PDF)**：内置专用的 `@media print` 打印 CSS 规则，点击“打印 / 导出 PDF”直接调起浏览器高清打印窗口导出 PDF 模板。
- **一键下载模板 (.md)**：点击一键导出完整的 `.md` Markdown 格式会议模板与框架。
- **Approved 会议模板归档印章 (Sign-off Stamp)**：前端利用 Web Audio API 动态合成金属刻章音效，点击印章即可累加 Approved 计数并浮现“Template Approved +1”渐隐动画。
- **Nomad List 热门会议模板排行榜**：首页下方呈现全量热门与历史会议模板网格墙，每张卡片直观陈列推荐 Score 徽章与 5 维数据胶囊 Pill，支持一键“载入此框架”回填。
- **右上角常驻炫彩分享**：极光紫粉渐变悬浮分享按钮，快速呼出微信朋友圈分享指引。
- **并排二维码打赏与联系弹窗**：打赏栏并排展示微信/支付宝支付码，Contact Us 弹窗并排展示微信/钉钉联系码。

## 快速启动

### 1. 克隆项目
```bash
git clone https://github.com/WT-Agent/ai-huiyi.git
cd ai-huiyi
```

### 2. 安装依赖
项目强制使用 pnpm 作为包管理器：
```bash
pnpm install
```

### 3. 配置本地开发环境
复制并配置环境变量：
```bash
cp .env.example .env
```
在 `.env` 中填入您的 API 密钥：
- `DEEPSEEK_API_KEY`: 您的 DeepSeek 开发者 API 密钥（用于文本生成任务）

### 4. 开发与构建
启动本地开发服务（支持本地反向代理中转，防止前端密钥泄露）：
```bash
pnpm dev
```
打包生产静态资源（打包输出在 `dist/` 目录中，支持零成本部署于 GitHub Pages、Vercel 或 OSS 容器）：
```bash
pnpm build
```

## 联系我们

如果您在使用过程中有任何问题、建议或商务合作，可以通过微信或钉钉联系我们。

## 打赏支持

如果本项目对您有帮助，欢迎打赏作者喝杯咖啡。您的支持是项目持续优化与更新的动力。

| 微信支付 | 支付宝 |
| :---: | :---: |
| <img src="asset/tenpay.png" width="180" alt="微信支付" /> | <img src="asset/alipay.png" width="180" alt="支付宝" /> |

## 版权与许可

本项目基于 MIT License 开源协议。

Copyright (c) 2026. All rights reserved.
