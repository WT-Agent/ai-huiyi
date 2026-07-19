# 网腾无限AI 会议纪要生成 (Nomad Card 网格墙版)

## 项目介绍

**网腾无限AI 会议纪要生成** 是一款真正借鉴 nomads.com (Nomad List) 首页“无限 Card 网格墙与排行榜 (Infinite Card Grid & Ranking Gallery)”核心架构打造的高效会议纪要微应用。该应用基于 Vue 3 + Vite + Vanilla CSS 打造，用户在前台能直接浏览包含 5 维数据胶囊 Pill（效率、决议、协同、时间、风险）与综合 Score 评分徽章的卡片网格墙。点击卡片可下钻展开完整报告与 Action Items 责任表格，并可通过 Web Audio 敲击金属刻章进行 Approved 归档解压。

### 核心特性
- **Nomad List 无限 Card 网格墙**：前台展示全量会议纪要卡片与排行榜，顶部配备多维筛选（最高效率、核心决议、最新生成）与快速搜索栏。
- **卡片 5 维数据胶囊 Pill & Score 徽章**：每张卡片直观陈列效率、决议、协同、时间、风险的胶囊分值与绿/金渐变分值徽章。
- **Approved 会议归档印章 (Sign-off Stamp)**：前端利用 Web Audio API 动态合成金属质感的规整盖章音效，点击印章即可累加 Approved 计数并浮现“Approved +1”渐隐动画。
- **多流派生成与下钻报告**：支持大厂 OKR、九巨擘圆桌、敏捷站会等 5 大流派，生成后新卡片实时插入网格墙头部。
- **右上角常驻分享**：磨砂玻璃悬浮分享按钮，快速呼出微信朋友圈分享指引。
- **并排二维码打赏与联系弹窗**：打赏栏并排展示微信/支付宝支付码，Contact Us 弹窗并排展示微信/钉钉联系码。
- **无下划线自适应弹窗**：条款及隐私弹窗支持 max-height 限高内滚动，而二维码弹窗高度自适应，防止双滚动条。

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
