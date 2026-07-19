<template>
  <div class="app-container">
    <!-- 顶部生成成功浮动 Toast -->
    <transition name="fade">
      <div v-if="showSuccessToast" class="top-success-toast">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="20 6 9 17 4 12"></polyline>
        </svg>
        <span>会议议程与讨论框架模板生成成功！</span>
      </div>
    </transition>

    <!-- 右上角常驻分享按钮 -->
    <button class="floating-share-btn no-print" @click="showShareGuide = true">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="share-icon">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享</span>
    </button>

    <header class="no-print">
      <h1>{{ appTitle }}</h1>
      <p>智能 AI 会议引擎 · 核心讨论议程与框架生成</p>
    </header>

    <!-- 活跃动态 -->
    <UserTicker class="no-print" />

    <!-- 首页中心核心生成区 -->
    <main class="glass-card main-generator-card no-print">
      <div class="generator-header">
        <h2>定制您今天的会议模板与框架</h2>
        <p class="generator-sub">输入您的开会意图与目标，AI 为您定制分钟级议程、讨论提问与决议分工规则</p>
      </div>

      <div class="divination-setup">
        <!-- 开会意图输入框 -->
        <div class="selector-group">
          <label class="selector-label">输入您的开会意图与核心讨论目标</label>
          <textarea 
            v-model="inquiryIntent" 
            placeholder="例如：今天我们要开一个季度季度产品延期与人员招聘对齐会，需要明确延期原因、确定新的上线节点并重新分配前端开发任务..."
            class="intent-textarea"
          ></textarea>
        </div>

        <!-- 快捷意图标签 -->
        <div class="selector-group">
          <label class="selector-label">常用开会意图快捷填入：</label>
          <div class="quick-tags-wrapper">
            <button 
              v-for="tag in quickTags" 
              :key="tag.label" 
              class="quick-tag-btn"
              @click="applyQuickTag(tag.text)"
            >
              {{ tag.label }}
            </button>
          </div>
        </div>

        <!-- 框架流派选择 -->
        <div class="selector-group">
          <label class="selector-label">选择开会框架流派</label>
          <select v-model="activeStyle" class="style-select">
            <option 
              v-for="style in styleOptions" 
              :key="style.value" 
              :value="style.value"
            >
              {{ style.label }}
            </option>
          </select>
        </div>

        <!-- 生成主按钮 -->
        <button 
          class="action-btn main-generate-btn" 
          :disabled="!inquiryIntent.trim() || loading" 
          @click="handleGenerate"
        >
          {{ loading ? '会议模板生成中...' : '生成专业会议模板与框架' }}
        </button>

        <div v-if="errorMsg" class="error-banner">
          {{ errorMsg }}
        </div>
      </div>
    </main>

    <!-- 生成结果区与打印导出工具区 -->
    <section v-if="currentTemplate" class="glass-card result-card printable-area">
      <!-- 印章与归档区域 -->
      <div class="stamp-section no-print">
        <div class="stamp-canvas">
          <svg 
            class="stamp-svg" 
            :class="{ stamping: isStamping }" 
            @click="stampApproved" 
            viewBox="0 0 160 160"
          >
            <circle cx="80" cy="80" r="70" fill="rgba(59, 130, 246, 0.08)" stroke="#3b82f6" stroke-width="4" stroke-dasharray="6,3" />
            <circle cx="80" cy="80" r="62" fill="none" stroke="#3b82f6" stroke-width="2" />
            <text x="80" y="72" font-size="18" font-weight="900" fill="#3b82f6" text-anchor="middle">APPROVED</text>
            <text x="80" y="100" font-size="14" font-weight="bold" fill="#3b82f6" text-anchor="middle">模板归档</text>
          </svg>
          <transition-group name="float-up">
            <span 
              v-for="item in floatingItems" 
              :key="item.id" 
              class="floating-merit"
              :style="{ transform: `translate(${item.x}px, ${item.y}px)` }"
            >
              {{ item.text }}
            </span>
          </transition-group>
        </div>
        <div class="merit-counter-display">
          累计打印/使用模板：<strong style="color: #3b82f6;">{{ totalTemplates }}</strong>
          <p class="wood-fish-tip">点击印章完成归档，Template Approved +1</p>
        </div>
      </div>

      <!-- 操作与导出工具栏 -->
      <div class="result-toolbar no-print">
        <div class="result-title-group">
          <span class="result-title-tag">会议模板结果</span>
          <span class="success-badge">生成成功</span>
        </div>
        <div class="export-actions">
          <button class="export-btn print-btn" @click="printTemplate">
            打印 / 导出 PDF
          </button>
          <button class="export-btn download-btn" @click="downloadTemplate">
            下载模板 (.md)
          </button>
          <button class="export-btn copy-btn" @click="copyText">
            {{ copied ? '已复制模板' : '复制全文' }}
          </button>
          <button class="export-btn highlight-btn" @click="showShareCard = true">
            生成分享卡片
          </button>
        </div>
      </div>

      <!-- 五维权重与卡片即时评分区 -->
      <div class="comparison-dashboard no-print">
        <h3 class="dashboard-title">框架维度权重 (支持微调评分)</h3>
        <div class="card-rating-interactive-row">
          <div v-for="metric in metricsList" :key="metric.key" class="metric-rating-box">
            <span class="metric-rating-label">{{ metric.shortLabel }}</span>
            <div class="metric-rating-controls">
              <button class="rating-btn" @click="adjustCardMetric(currentTemplate, metric.key, -1)">-</button>
              <span class="rating-num">{{ currentTemplate.scores[metric.key] }}</span>
              <button class="rating-btn" @click="adjustCardMetric(currentTemplate, metric.key, 1)">+</button>
            </div>
          </div>
        </div>
      </div>

      <!-- 模板正文 -->
      <div class="template-document">
        <div class="print-only-header">
          <h1>{{ currentTemplate.topic }} - 会议议程与讨论框架</h1>
          <p>生成日期: {{ currentTemplate.timestamp }} | 框架流派: {{ currentTemplate.styleLabel }}</p>
          <hr />
        </div>
        <!-- 骨架屏加载 -->
        <div v-if="loading" class="skeleton">
          <div class="skeleton-line" style="width: 80%"></div>
          <div class="skeleton-line" style="width: 95%"></div>
          <div class="skeleton-line" style="width: 60%"></div>
          <div class="skeleton-line" style="width: 75%"></div>
        </div>
        <div v-else class="markdown-body scroll-box">{{ cleanResponseText(currentTemplate.output) }}</div>
      </div>
    </section>

    <!-- 演示案例区组件 (模块三：30 条会议框架精选案例展示) -->
    <DemoShowcase @use-sample="handleUseSample" />

    <!-- 首页下方：Nomad 热门会议模板库排行榜网格墙 (Nomad Template Gallery) -->
    <section class="nomad-gallery-section no-print">
      <div class="gallery-header glass-card">
        <div class="gallery-title-group">
          <h2>Nomad 热门会议模板排行榜</h2>
          <p>浏览精选与历史高分开会框架，可直接在卡片上微调权重与评分</p>
        </div>

        <div class="gallery-filter-row">
          <div class="filter-tabs">
            <button 
              v-for="filter in filterOptions" 
              :key="filter.key"
              class="filter-pill"
              :class="{ active: activeFilter === filter.key }"
              @click="activeFilter = filter.key"
            >
              {{ filter.label }}
            </button>
          </div>

          <input 
            type="text" 
            v-model="searchQuery" 
            placeholder="搜索模板意图..." 
            class="nomad-search-input"
          />
        </div>
      </div>

      <!-- 卡片网格墙 -->
      <div class="nomad-card-grid">
        <div 
          v-for="card in filteredCards" 
          :key="card.id" 
          class="nomad-card glass-card"
        >
          <!-- 头部与 Score 徽章 -->
          <div class="n-card-header">
            <div class="n-card-title-group">
              <span class="n-card-category">主题: {{ card.topic }}</span>
              <span class="n-card-date">{{ card.timestamp }}</span>
            </div>
            <div class="n-card-score-badge" :class="getScoreClass(card.averageScore)">
              Score: {{ card.averageScore.toFixed(1) }}
            </div>
          </div>

          <!-- 5 维胶囊 Pill 与 Card 内部评分组件 -->
          <div class="n-card-metrics-grid">
            <div 
              v-for="metric in metricsList" 
              :key="metric.key" 
              class="metric-pill-interactive"
            >
              <span class="pill-icon">{{ metric.shortLabel }}</span>
              <div class="pill-score-controls">
                <button class="score-step-btn" @click.stop="adjustCardMetric(card, metric.key, -1)">-</button>
                <span class="pill-val">{{ card.scores[metric.key] }}</span>
                <button class="score-step-btn" @click.stop="adjustCardMetric(card, metric.key, 1)">+</button>
              </div>
            </div>
          </div>

          <div class="n-card-body">
            <div class="n-card-style-tag">{{ card.styleLabel }}</div>
            <p class="n-card-excerpt">{{ cleanExcerpt(card.output) }}</p>
          </div>

          <!-- 操作栏 -->
          <div class="n-card-footer">
            <button class="card-use-btn" @click="loadCardToGenerator(card)">
              载入此框架
            </button>
            <button class="card-view-btn" @click="openCardDetail(card)">
              查看 / 打印
            </button>
          </div>
        </div>
      </div>
    </section>

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links no-print">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay no-print" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的隐私。您在本应用中输入的开会意图与讨论主题等数据，均仅用于实时大模型会议模板生成，我们不在服务器端持久记录您的内容。</p>
          <p>为了在您的浏览器本地保留“Approved 模板归档计数”和您的 Nomad 模板卡片历史，应用会使用浏览器的本地存储（localStorage）保存相应状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay no-print" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 会议模板与框架生成服务。使用本应用代表您同意遵守当地有关人工智能生成内容的各项管理条例。</p>
          <p>生成的内容包含分钟级议程、讨论框架与表决规则，仅供团队开会准备与效能提升参考，请结合公司实际情况灵活调整。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 (自适应高度 + weixin.png & dingtalk.png 展示) -->
    <div v-if="showContact" class="modal-overlay no-print" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信联系" class="contact-qr-img" />
              <span class="contact-qr-label">微信联系</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉交流" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉交流</span>
            </div>
          </div>
          <p class="contact-email">反馈邮箱: <span style="color: var(--primary-color);">us@wuxian.xyz</span></p>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />

    <!-- 分享卡片弹窗 (模块二扩展) -->
    <ShareCardModal
      :visible="showShareCard"
      :content="currentTemplate ? currentTemplate.output : ''"
      :wechat-id="wechatId"
      @close="showShareCard = false"
    />

    <!-- 分享引导浮层 -->
    <div v-if="showShareGuide" class="share-guide-overlay no-print" @click="handleShareClose">
      <div class="share-guide-arrow">↗</div>
      <div class="share-guide-content">
        <p>点击右上角菜单 <strong>•••</strong></p>
        <p>选择 <strong>「分享到朋友圈」</strong></p>
        <p class="share-guide-sub">分享这款 Nomad 风格的会议模板神器</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import DemoShowcase from './components/DemoShowcase.vue';
import ShareCardModal from './components/ShareCardModal.vue';
import appConfig from './config.json';
const weixinImg = 'https://ai.wuxian.xyz/assets/weixin.png';
const dingtalkImg = 'https://ai.wuxian.xyz/assets/dingtalk.png';

const appTitle = ref(appConfig.title || '网腾无限AI 会议模板与框架生成');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const inquiryIntent = ref('');
const loading = ref(false);
const errorMsg = ref('');
const copied = ref(false);
const showSuccessToast = ref(false);
const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showShareCard = ref(false);

const activeFilter = ref('all');
const searchQuery = ref('');

const quickTags = [
  { label: '新项目启动会', text: '针对新项目启动，需要明确项目目标、里程碑节点、各部门接口人分工与风险防控。' },
  { label: '季度 OKR 复盘会', text: '针对季度 OKR 完成度进行复盘，讨论未达标项归因、关键结果对齐与下季度资源申请。' },
  { label: '跨部门预算对齐会', text: '跨部门讨论营销与研发预算分配，解决超支争议，确定审批流与资金投放优先级。' },
  { label: '绩效考核与晋升面谈', text: '对团队成员进行半年度绩效评估与晋升答辩对齐，给予结构化反馈与下一步提升计划。' },
  { label: '创投路演预演会', text: '针对创业项目 VC 融资路演预演，严查商业模式漏洞、财务预测真实性与 Q&A 提问。' }
];

const styleOptions = [
  { label: '大厂高效 OKR 框架', value: '大厂OKR流派：时间线严格到分钟，责任人与表决规则清晰。' },
  { label: '九巨擘企业开会圆桌', value: '九巨擘流派：模拟马斯克（第一性原理）、乔布斯（产品心）、秦始皇（军令状）等9大高管破局提问。' },
  { label: '敏捷站会与冲刺框架', value: '敏捷冲刺流派：围绕已完成、今日阻塞、风险攻坚3环节极简设计。' },
  { label: '商务正规议程框架', value: '商务公文流派：格式严谨规整，适合董事会或正规归档。' },
  { label: '头脑风暴与创新发散', value: '头脑风暴流派：破冰、发散点子、聚类分组与可行性筛选。' }
];

const activeStyle = ref(styleOptions[0].value);

const filterOptions = [
  { key: 'all', label: '全部模板' },
  { key: 'top', label: '最高推荐' },
  { key: 'action', label: '高效议程' },
  { key: 'recent', label: '近期定制' }
];

const metricsList = [
  { key: 'struct', label: '结构化程度', shortLabel: '结构', icon: '' },
  { key: 'focus', label: '目标聚焦度', shortLabel: '聚焦', icon: '' },
  { key: 'time', label: '时间规划力', shortLabel: '时间', icon: '' },
  { key: 'role', label: '角色分工明确度', shortLabel: '分工', icon: '' },
  { key: 'risk', label: '风险预案严密度', shortLabel: '预案', icon: '' }
] as const;

type MetricKey = typeof metricsList[number]['key'];

interface NomadCard {
  id: string;
  timestamp: string;
  topic: string;
  input: string;
  styleLabel: string;
  scores: { struct: number; focus: number; time: number; role: number; risk: number; };
  averageScore: number;
  output: string;
  isPreset?: boolean;
}

const presetCards: NomadCard[] = [
  {
    id: 'p1',
    timestamp: '2026-07-18 14:00',
    topic: 'Q3核心产品延期与架构重构对齐会模板',
    input: '针对Q3产品延期与研发人力不足，制定对齐议程。',
    styleLabel: '大厂高效 OKR 框架',
    scores: { struct: 5, focus: 5, time: 4, role: 4, risk: 4 },
    averageScore: 4.4,
    isPreset: true,
    output: `📌 **【1. 会议目标与基调定义】**\n明确 Q3 延期真实归因，在 45 分钟内锁定新的上线节点与前端人力支援表决方案。\n\n⏱️ **【2. 分钟级议程与时间分配表】**\n| 时间段 | 议题环节 | 主讲/引导人 | 预期产出 |\n|---|---|---|---|\n| 00-05 min | 破冰与延期现状通报 | 产品负责人 | 确认延期 5 天的具体模块列表 |\n| 05-25 min | 架构瓶颈攻坚与人力调配 | 研发架构师 | 达成前端支援 2 人的表决共识 |\n| 25-40 min | 新里程碑排期与交付物确认 | 项目经理 | 输出精确到日的排期甘特图 |\n| 40-45 min | 责任人确认与总结收尾 | 会议主持人 | 锁定表决签名 |\n\n💬 **【3. 议题引导提问与讨论框架】**\n- 提问 1: 如果不增加人力，仅裁剪非核心功能，能否按原定日期上线？\n- 提问 2: 微服务重构带来的风险，是否有灰度发布的降级预案？\n\n🗳️ **【4. 决议表决机制与分工规则】**\n采用技术负责人否决制与多数表决制，所有 Action Items 当场指定唯一责任人。\n\n⚠️ **【5. 常见讨论坑点与防跑题指南】**\n防范相互推诿，规定发言必须围绕“解决方案与交付节点”，禁止无建设性的抱怨。\n\n[HUIYI_SCORES]struct:5,focus:5,time:4,role:4,risk:4[/HUIYI_SCORES]`
  },
  {
    id: 'p2',
    timestamp: '2026-07-17 10:30',
    topic: '新项目启动会与资源锁定议程模板',
    input: '新项目启动，明确团队分工与关键里程碑。',
    styleLabel: '商务正规议程框架',
    scores: { struct: 5, focus: 4, time: 5, role: 5, risk: 4 },
    averageScore: 4.6,
    isPreset: true,
    output: `📌 **【1. 会议目标与基调定义】**\n对齐全员项目愿景，锁定各部门 Q4 资源投入与接口人名册。\n\n⏱️ **【2. 分钟级议程与时间分配表】**\n| 时间段 | 议题环节 | 主讲人 | 预期产出 |\n|---|---|---|---|\n| 00-10 min | 项目背景与商业价值 | 项目发起人 | 全员理解项目战略意义 |\n| 10-30 min | 总体架构与阶段里程碑 | 交付负责人 | 确定四大阶段验收标准 |\n\n[HUIYI_SCORES]struct:5,focus:4,time:5,role:5,risk:4[/HUIYI_SCORES]`
  }
];

const cardsList = ref<NomadCard[]>([]);
const currentTemplate = ref<NomadCard | null>(presetCards[0]);

interface FloatingItem {
  id: number;
  x: number;
  y: number;
  text: string;
}

const floatingItems = ref<FloatingItem[]>([]);
const isStamping = ref(false);
const totalTemplates = ref(0);
let floatId = 0;

const playStampSound = () => {
  try {
    const AudioContext = window.AudioContext || (window as any).webkitAudioContext;
    if (!AudioContext) return;
    const ctx = new AudioContext();
    const now = ctx.currentTime;
    
    const osc1 = ctx.createOscillator();
    const gain1 = ctx.createGain();
    osc1.frequency.setValueAtTime(280, now);
    osc1.frequency.exponentialRampToValueAtTime(70, now + 0.05);
    gain1.gain.setValueAtTime(0.75, now);
    gain1.gain.exponentialRampToValueAtTime(0.001, now + 0.07);
    osc1.connect(gain1);
    gain1.connect(ctx.destination);
    osc1.start(now);
    osc1.stop(now + 0.09);
  } catch (e) {
    console.error('AudioContext error:', e);
  }
};

const stampApproved = () => {
  if (isStamping.value) return;
  isStamping.value = true;
  playStampSound();
  
  totalTemplates.value += 1;
  localStorage.setItem('huiyi_total_templates', totalTemplates.value.toString());
  
  const id = floatId++;
  const x = Math.floor(Math.random() * 40) - 20;
  const y = -45;
  
  floatingItems.value.push({ id, x, y, text: 'Template Approved +1' });
  
  setTimeout(() => {
    floatingItems.value = floatingItems.value.filter(item => item.id !== id);
  }, 1000);

  setTimeout(() => {
    isStamping.value = false;
  }, 120);
};

const applyQuickTag = (text: string) => {
  inquiryIntent.value = text;
};

const adjustCardMetric = (card: NomadCard | null, key: MetricKey, delta: number) => {
  if (!card) return;
  const newScore = Math.min(5, Math.max(1, card.scores[key] + delta));
  card.scores[key] = newScore;
  card.averageScore = (card.scores.struct + card.scores.focus + card.scores.time + card.scores.role + card.scores.risk) / 5;
  const userOnly = cardsList.value.filter(c => !c.isPreset);
  saveHistory(userOnly);
};

const loadHistory = () => {
  try {
    const raw = localStorage.getItem('huiyi_template_records');
    const userCards: NomadCard[] = raw ? JSON.parse(raw) : [];
    cardsList.value = [...userCards, ...presetCards];
    const rawCount = localStorage.getItem('huiyi_total_templates');
    totalTemplates.value = rawCount ? parseInt(rawCount, 10) : 18;
  } catch (e) {
    cardsList.value = [...presetCards];
  }
};

const saveHistory = (userCards: NomadCard[]) => {
  try {
    localStorage.setItem('huiyi_template_records', JSON.stringify(userCards));
    const raw = localStorage.getItem('huiyi_template_records');
    const parsed = raw ? JSON.parse(raw) : [];
    cardsList.value = [...parsed, ...presetCards];
  } catch (e) {
    console.error('Save history failed:', e);
  }
};

const filteredCards = computed(() => {
  let list = cardsList.value;

  if (activeFilter.value === 'top') {
    list = list.filter(c => c.averageScore >= 4.5);
  } else if (activeFilter.value === 'action') {
    list = list.filter(c => c.scores.time >= 4);
  } else if (activeFilter.value === 'recent') {
    list = list.filter(c => !c.isPreset);
  }

  if (searchQuery.value.trim()) {
    list = list.filter(c => 
      c.topic.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      c.input.toLowerCase().includes(searchQuery.value.toLowerCase())
    );
  }

  return list;
});

const getScoreClass = (score: number) => {
  if (score >= 4.5) return 'score-high';
  if (score >= 3.5) return 'score-mid';
  return 'score-low';
};

const cleanExcerpt = (text: string) => {
  const cleaned = cleanResponseText(text);
  return cleaned.length > 90 ? cleaned.slice(0, 90) + '...' : cleaned;
};

const loadCardToGenerator = (card: NomadCard) => {
  inquiryIntent.value = card.input;
  const matched = styleOptions.find(o => o.label === card.styleLabel);
  if (matched) {
    activeStyle.value = matched.value;
  }
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const openCardDetail = (card: NomadCard) => {
  currentTemplate.value = card;
  const resultEl = document.querySelector('.printable-area');
  if (resultEl) {
    resultEl.scrollIntoView({ behavior: 'smooth' });
  }
};

const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const isLimitReached = computed(() => {
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const parseAIScores = (text: string) => {
  const match = text.match(/\[HUIYI_SCORES\](.*?)\[\/HUIYI_SCORES\]/);
  if (match) {
    const scoreStr = match[1].trim();
    const scores: Record<string, number> = {};
    scoreStr.split(',').forEach(item => {
      const [key, val] = item.split(':');
      if (key && val) {
        scores[key.trim()] = Math.min(5, Math.max(1, parseInt(val.trim(), 10) || 3));
      }
    });
    return {
      struct: scores.struct || 3,
      focus: scores.focus || 3,
      time: scores.time || 3,
      role: scores.role || 3,
      risk: scores.risk || 3
    };
  }
  return null;
};

const cleanResponseText = (text: string) => {
  if (!text) return '';
  return text.replace(/\[HUIYI_SCORES\][\s\S]*?\[\/HUIYI_SCORES\]/gi, '').trim();
};

const triggerSuccessToast = () => {
  showSuccessToast.value = true;
  setTimeout(() => {
    showSuccessToast.value = false;
  }, 3000);
};

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';

  const matched = styleOptions.find(o => o.value === activeStyle.value);
  const styleLabel = matched ? matched.label : '会议框架';

  try {
    const fullPrompt = `【会议意图与目标】：${inquiryIntent.value}。\n【框架流派】：${activeStyle.value}\n${promptTopic.value}`;

    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: fullPrompt,
        style: activeStyle.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      const generatedScores = parseAIScores(data.result) || { struct: 4, focus: 4, time: 4, role: 4, risk: 4 };
      const avg = (generatedScores.struct + generatedScores.focus + generatedScores.time + generatedScores.role + generatedScores.risk) / 5;

      const newRecord: NomadCard = {
        id: Date.now().toString(),
        timestamp: new Date().toISOString().slice(0, 16).replace('T', ' '),
        topic: inquiryIntent.value.length > 25 ? inquiryIntent.value.slice(0, 25) + '...' : inquiryIntent.value,
        input: inquiryIntent.value,
        styleLabel,
        scores: generatedScores,
        averageScore: avg,
        output: data.result
      };

      const userOnly = cardsList.value.filter(c => !c.isPreset);
      userOnly.unshift(newRecord);
      saveHistory(userOnly);

      currentTemplate.value = newRecord;
      triggerSuccessToast();

      const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
      localStorage.setItem('free_uses', (currentUses + 1).toString());
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地开发服务器代理。';
  } finally {
    loading.value = false;
  }
};

const handleShareClose = () => {
  showShareGuide.value = false;
  localStorage.setItem('shared_fission', 'true');
};

const printTemplate = () => {
  window.print();
};

const downloadTemplate = () => {
  if (!currentTemplate.value) return;
  try {
    const text = cleanResponseText(currentTemplate.value.output);
    const blob = new Blob([text], { type: 'text/markdown;charset=utf-8;' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.setAttribute('download', `${currentTemplate.value.topic.replace(/\s+/g, '_')}_会议框架模板.md`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  } catch (e) {
    console.error('Download failed:', e);
  }
};

const copyText = async () => {
  if (!currentTemplate.value) return;
  try {
    await navigator.clipboard.writeText(cleanResponseText(currentTemplate.value.output));
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};

const handleUseSample = (sampleTopic: string) => {
  inquiryIntent.value = sampleTopic;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

onMounted(() => {
  loadHistory();
});
</script>
