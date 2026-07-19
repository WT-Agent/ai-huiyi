# NetTeng Infinite AI Meeting Summary & Action Items Generator (Nomad Card Grid Edition)

## Project Introduction

**NetTeng Infinite AI Meeting Summary & Action Items Generator** is an interactive productivity micro-application built directly on the core architectural style of nomads.com (Nomad List)—the **Infinite Card Grid & Ranking Gallery**. Built on Vue 3 + Vite + Vanilla CSS, users can explore a rich front-page grid of meeting cards featuring 5-dimensional metric pills (Efficiency, Action Clarity, Alignment, Time Control, and Risk Index) along with overall Score badges. Clicking any card expands a detailed report modal containing full Markdown minutes and Action Items tables, accompanied by a metallic Web Audio Approved sign-off stamp.

### Key Features
- **Nomad List Infinite Card Grid**: Front-page card gallery and leaderboard with multi-metric filter pills (Top Score, Action-Packed, Recent) and quick search.
- **5-Dimensional Metric Pills & Score Badges**: Each card displays score pills for Efficiency, Clarity, Alignment, Time, and Risk alongside score badges.
- **Approved Sign-Off Stamp**: Local sound synthesis using the Web Audio API (metallic corporate stamp click). Clicking the stamp increments the Approved count and triggers a floating "Approved +1" animation.
- **Multi-Style Generation & Detail Expansion**: Supports 5 narrative styles (Big-Tech OKR, 9-Figure Roundtable, Agile Standup, Formal Minutes, Brainstorming). Generated cards insert into the top of the grid in real-time.
- **Floating Share Button**: A sleek glassmorphism share button at the top-right corner to invoke WeChat moments sharing guidance.
- **Side-by-Side QR Codes**: Parallel WeChat and Alipay payment codes in the donation section, and WeChat and DingTalk codes in the Contact Us modal.
- **Adaptive Modal Views**: Terms and privacy modals support vertical scrolling with max-height limits, while QR code modals adjust height automatically to prevent nested scrollbars.

## Quick Start

### 1. Clone the Project
```bash
git clone https://github.com/WT-Agent/ai-huiyi.git
cd ai-huiyi
```

### 2. Install Dependencies
This project enforces pnpm as the package manager:
```bash
pnpm install
```

### 3. Local Environment Configuration
Copy and configure the environment variables:
```bash
cp .env.example .env
```
Fill in your API key in the `.env` file:
- `DEEPSEEK_API_KEY`: Your DeepSeek developer API key (used for text generation tasks)

### 4. Development & Build
Start the local development server (with reverse proxy support to prevent API key leaks):
```bash
pnpm dev
```
Build static production assets (outputs to the `dist/` directory, suitable for Vercel, GitHub Pages, or CDN bucket hosting):
```bash
pnpm build
```

## Contact Us

If you have any questions, suggestions, or business cooperation proposals during use, feel free to contact us via WeChat or DingTalk.

## Donation Support

If this project helps you, feel free to support the author. Your support is the driving force for continuous maintenance and optimization.

| WeChat Pay | Alipay |
| :---: | :---: |
| <img src="asset/tenpay.png" width="180" alt="WeChat Pay" /> | <img src="asset/alipay.png" width="180" alt="Alipay" /> |

## License

This project is licensed under the MIT License.

Copyright (c) 2026. All rights reserved.
