# NetTeng Infinite AI Meeting Template & Agenda Framework Generator

## Project Introduction

**NetTeng Infinite AI Meeting Template & Agenda Framework Generator** is an interactive productivity micro-application designed to generate customized meeting agendas, discussion frameworks, and decision-making rules based on user meeting intent. Built on Vue 3 + Vite + Vanilla CSS, the intent input box, quick intent tags, and 5-dimensional framework sliders are placed directly on the home page hero section. Upon generating a template, users can print it directly (or export as PDF via window.print()), download it as a `.md` file, or explore the Nomad List style template gallery grid below.

### Key Features
- **Front-and-Center Intent Generator**: Intent input text area, quick intent tags (Project Kick-off, OKR Review, Budget Alignment, Performance Interview, VC Pitch Practice), and 5-dimensional framework sliders placed on the home page.
- **5-Dimensional Metric Sliders**: Interactive 1-to-5 star sliders for Structure, Goal Focus, Timeline Rigor, Role Clarity, and Risk Preparation.
- **One-Click Print & PDF Export**: Customized `@media print` CSS rules allowing users to print or export high-definition PDF meeting templates.
- **Markdown Template Download**: One-click download of generated meeting templates in `.md` format.
- **Approved Sign-Off Stamp**: Local sound synthesis using the Web Audio API (metallic corporate stamp click). Clicking the stamp increments the Approved count and triggers a floating "Template Approved +1" animation.
- **Nomad List Template Gallery & Leaderboard**: Front-page card gallery featuring metric pills, score badges, quick filtering, and one-click "Load Template" functionality.
- **Floating Share Button**: A sleek glassmorphism purple-pink gradient share button at the top-right corner to invoke WeChat moments sharing guidance.
- **Side-by-Side QR Codes**: Parallel WeChat and Alipay payment codes in the donation section, and WeChat and DingTalk codes in the Contact Us modal.

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
