<template>
  <div class="app-container">
    <!-- 右上角常驻分享按钮 -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="share-icon">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享</span>
    </button>

    <header>
      <h1>{{ appTitle }}</h1>
      <p>Nomad-style 数据化效率榜 · AI 会议纪要与 Action Items 生成</p>
    </header>

    <!-- 活跃动态 -->
    <UserTicker />

    <!-- 顶部 Nomad 工具控制栏：多维过滤、搜索与新建按钮 -->
    <div class="nomad-toolbar glass-card">
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

      <div class="search-and-create">
        <input 
          type="text" 
          v-model="searchQuery" 
          placeholder="🔍 搜索会议主题、决议关键词..." 
          class="nomad-search-input"
        />
        <button class="create-trigger-btn" @click="showCreateModal = true">
          + 新建会议纪要
        </button>
      </div>
    </div>

    <!-- Nomad 无限 Card 网格墙 (Infinite Card Grid) -->
    <main class="nomad-grid-section">
      <div v-if="filteredCards.length === 0" class="empty-grid-state glass-card">
        <p>未找到符合条件的会议纪要卡片，点击上方 “+ 新建会议纪要” 开始生成。</p>
      </div>

      <div v-else class="nomad-card-grid">
        <div 
          v-for="card in filteredCards" 
          :key="card.id" 
          class="nomad-card glass-card"
          @click="openCardDetail(card)"
        >
          <!-- 卡片头部：主题与 Nomad Score 徽章 -->
          <div class="n-card-header">
            <div class="n-card-title-group">
              <span class="n-card-category">📌 {{ card.topic }}</span>
              <span class="n-card-date">{{ card.timestamp }}</span>
            </div>
            <div class="n-card-score-badge" :class="getScoreClass(card.averageScore)">
              🚀 Score: {{ card.averageScore.toFixed(1) }}
            </div>
          </div>

          <!-- Nomad 5 维数据胶囊 Pill 矩阵 -->
          <div class="n-card-metrics-grid">
            <div class="metric-pill">
              <span class="pill-icon">⚡ 效率</span>
              <span class="pill-val">{{ card.scores.efficiency }}</span>
            </div>
            <div class="metric-pill">
              <span class="pill-icon">🎯 决议</span>
              <span class="pill-val">{{ card.scores.clarity }}</span>
            </div>
            <div class="metric-pill">
              <span class="pill-icon">🤝 协同</span>
              <span class="pill-val">{{ card.scores.alignment }}</span>
            </div>
            <div class="metric-pill">
              <span class="pill-icon">⏰ 时间</span>
              <span class="pill-val">{{ card.scores.time }}</span>
            </div>
            <div class="metric-pill">
              <span class="pill-icon">🛡️ 风险</span>
              <span class="pill-val">{{ card.scores.risk }}</span>
            </div>
          </div>

          <!-- 卡片核心摘要 -->
          <div class="n-card-body">
            <div class="n-card-style-tag">{{ card.styleLabel }}</div>
            <p class="n-card-excerpt">{{ cleanExcerpt(card.output) }}</p>
          </div>

          <!-- 卡片底部操作与来源 -->
          <div class="n-card-footer">
            <span class="n-card-tag-badge">{{ card.isPreset ? '🔥 推荐基准' : '👤 用户生成' }}</span>
            <span class="n-card-expand-link">查看完整下钻报告 →</span>
          </div>
        </div>
      </div>
    </main>

    <!-- 下钻详情模态框 (Detail Modal) -->
    <div v-if="selectedCard" class="modal-overlay" @click.self="selectedCard = null">
      <div class="modal-content detail-modal-content">
        <div class="detail-modal-header">
          <h2>📌 {{ selectedCard.topic }} 纪要报告</h2>
          <button class="close-modal-icon" @click="selectedCard = null">✕</button>
        </div>

        <!-- 盖章解压与归档区域 -->
        <div class="stamp-section">
          <div class="stamp-canvas">
            <svg 
              class="stamp-svg" 
              :class="{ stamping: isStamping }" 
              @click="stampApproved" 
              viewBox="0 0 160 160"
            >
              <!-- 沉稳规整印章 -->
              <circle cx="80" cy="80" r="70" fill="rgba(59, 130, 246, 0.08)" stroke="#3b82f6" stroke-width="4" stroke-dasharray="6,3" />
              <circle cx="80" cy="80" r="62" fill="none" stroke="#3b82f6" stroke-width="2" />
              <text x="80" y="72" font-size="18" font-weight="900" fill="#3b82f6" text-anchor="middle">APPROVED</text>
              <text x="80" y="100" font-size="14" font-weight="bold" fill="#3b82f6" text-anchor="middle">会议归档</text>
            </svg>
            <!-- 浮空 Approved 动效 -->
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
            累计归档会议：<strong style="color: #3b82f6;">{{ totalMeetings }}</strong>
            <p class="wood-fish-tip">点击上方印章盖戳，Approved +1</p>
          </div>
        </div>

        <!-- 5维得分对比看板 -->
        <div class="comparison-dashboard">
          <h3 class="dashboard-title">📊 效率指标分析 (预期 vs AI评估)</h3>
          <div class="comparison-grid">
            <div v-for="metric in metricsList" :key="metric.key" class="comparison-row">
              <div class="metric-info">
                <span class="metric-label">{{ metric.icon }} {{ metric.label }}</span>
                <span class="metric-scores-text">
                  得分: <strong style="color: var(--primary-color)">{{ selectedCard.scores[metric.key] }}</strong> / 5
                </span>
              </div>
              <div class="bar-bg">
                <div class="bar-fill user-fill" :style="{ width: selectedCard.scores[metric.key] * 20 + '%' }"></div>
              </div>
            </div>
          </div>
        </div>

        <!-- 完整报告内容 -->
        <div class="ai-response-wrapper">
          <div class="output-content scroll-box" style="text-align: left;">{{ cleanResponseText(selectedCard.output) }}</div>
        </div>

        <div class="modal-actions-row">
          <button class="modal-btn-action" @click="copyText(selectedCard.output)">
            {{ copied ? '已复制报告' : '复制纪要全文' }}
          </button>
          <button v-if="!selectedCard.isPreset" class="modal-btn-delete" @click="deleteCard(selectedCard.id)">
            删除此卡片
          </button>
          <button class="modal-btn-close" @click="selectedCard = null">关闭</button>
        </div>
      </div>
    </div>

    <!-- 创建/生成模态框 (Create Modal) -->
    <div v-if="showCreateModal" class="modal-overlay" @click.self="showCreateModal = false">
      <div class="modal-content create-modal-content">
        <div class="detail-modal-header">
          <h2>+ 新建会议纪要</h2>
          <button class="close-modal-icon" @click="showCreateModal = false">✕</button>
        </div>

        <div class="divination-setup">
          <div class="selector-group">
            <label class="selector-label">输入会议主题</label>
            <input 
              type="text" 
              v-model="inquiryTopic" 
              class="city-text-input" 
              placeholder="例如：Q3季度产品迭代与架构升级会、运营增长研讨会..."
            />
          </div>

          <div class="selector-group">
            <label class="selector-label">请滑动评估预期会议效率指标</label>
            <div class="score-sliders">
              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">⚡ 会议效率指数 (Efficiency)</span>
                  <span class="slider-value">{{ userScores.efficiency }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.efficiency" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">🎯 决议明确度 (Clarity)</span>
                  <span class="slider-value">{{ userScores.clarity }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.clarity" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">🤝 参会协同度 (Alignment)</span>
                  <span class="slider-value">{{ userScores.alignment }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.alignment" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">⏰ 时间控制力 (Time)</span>
                  <span class="slider-value">{{ userScores.time }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.time" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">🛡️ 风险与阻碍 (Risk)</span>
                  <span class="slider-value">{{ userScores.risk }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.risk" class="range-slider" />
              </div>
            </div>
          </div>

          <div class="selector-group">
            <label class="selector-label">输入会议讨论速记/要点 (选填)</label>
            <textarea 
              v-model="userInput" 
              placeholder="请输入会议笔记或音频转文字片段，例如：参会人：张总、王工程师。讨论了预算超支20%与上线时间延期，张总要求增加2个前端人手..."
            ></textarea>
          </div>

          <div class="selector-group">
            <label class="selector-label">选择纪要生成流派</label>
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

          <button 
            class="action-btn" 
            :disabled="!inquiryTopic.trim() || loading" 
            @click="handleGenerate"
          >
            {{ loading ? '纪要生成中...' : '生成纪要并加入卡片墙' }}
          </button>

          <!-- 异常提示 -->
          <div v-if="errorMsg" style="color: var(--accent-color); font-size: 0.85rem; text-align: center; margin-top: 0.5rem;">
            {{ errorMsg }}
          </div>
        </div>
      </div>
    </div>

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的隐私。您在本应用中输入的会议主题、讨论速记以及指标设定等数据，均仅用于实时大模型纪要生成，我们不在服务器端持久记录您的内容。</p>
          <p>为了在您的浏览器本地保留“Approved 会议归档计数”和您的 Nomad 会议卡片历史，应用会使用浏览器的本地存储（localStorage）保存相应状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 会议纪要生成服务。使用本应用代表您同意遵守当地有关人工智能生成内容的各项管理条例。</p>
          <p>生成的内容包含执行摘要、Action Items 表格与风险提示，仅供团队协作与效率管理参考，请结合公司实际情况确认发布。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content" style="max-width: 420px;">
        <h3>Contact Us</h3>
        <div class="modal-text-content">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过微信或钉钉联系我们：</p>
          <div style="display: flex; gap: 1rem; justify-content: center; margin-top: 0.5rem; margin-bottom: 0.5rem; flex-wrap: wrap;">
            <div style="text-align: center;">
              <img :src="weixinImg" alt="微信二维码" style="width: 130px; height: 130px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.1);" />
              <div style="font-size: 0.75rem; margin-top: 0.25rem; color: var(--text-secondary);">微信</div>
            </div>
            <div style="text-align: center;">
              <img :src="dingtalkImg" alt="钉钉二维码" style="width: 130px; height: 130px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.1);" />
              <div style="font-size: 0.75rem; margin-top: 0.25rem; color: var(--text-secondary);">钉钉</div>
            </div>
          </div>
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

    <!-- 分享引导浮层 -->
    <div v-if="showShareGuide" class="share-guide-overlay" @click="handleShareClose">
      <div class="share-guide-arrow">↗</div>
      <div class="share-guide-content">
        <p>点击右上角菜单 <strong>•••</strong></p>
        <p>选择 <strong>「分享到朋友圈」</strong></p>
        <p class="share-guide-sub">分享这款 Nomad 风格的高效会议纪要神器</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

const appTitle = ref(appConfig.title || '网腾无限AI 会议纪要生成');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const activeFilter = ref('all');
const searchQuery = ref('');
const showCreateModal = ref(false);
const selectedCard = ref<NomadCard | null>(null);

const inquiryTopic = ref('');
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const copied = ref(false);
const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);

const userScores = ref({
  efficiency: 4,
  clarity: 4,
  alignment: 4,
  time: 4,
  risk: 2
});

const filterOptions = [
  { key: 'all', label: '全部 (All Grid)' },
  { key: 'top', label: '最高效率 🚀' },
  { key: 'action', label: '核心决议 🎯' },
  { key: 'recent', label: '最新生成 🔥' }
];

const styleOptions = [
  { label: '大厂高效 OKR 纪要', value: '大厂OKR流派：聚焦Action Items，责任人与完成时间清晰呈现。' },
  { label: '九巨擘企业开会圆桌', value: '九巨擘流派：模拟马斯克（第一性原理）、乔布斯（产品美学）、秦始皇（执行力）等9大高管激辩。' },
  { label: '敏捷站会与极简纪要', value: '敏捷站会流派：围绕已完成、进行中、风险阻碍三要素提炼。' },
  { label: '商务正规公文纪要', value: '正规公文流派：格式严谨规整，适合董事会或正规归档。' },
  { label: '头脑风暴与创意整理', value: '头脑风暴流派：发散讨论归类，整理创新点子与后续实验。' }
];

const activeStyle = ref(styleOptions[0].value);

const metricsList = [
  { key: 'efficiency', label: '会议效率 (Efficiency)', icon: '⚡' },
  { key: 'clarity', label: '决议明确 (Clarity)', icon: '🎯' },
  { key: 'alignment', label: '参会协同 (Alignment)', icon: '🤝' },
  { key: 'time', label: '时间控制 (Time)', icon: '⏰' },
  { key: 'risk', label: '风险与阻碍 (Risk)', icon: '🛡️' }
] as const;

interface NomadCard {
  id: string;
  timestamp: string;
  topic: string;
  input: string;
  styleLabel: string;
  scores: { efficiency: number; clarity: number; alignment: number; time: number; risk: number; };
  averageScore: number;
  output: string;
  isPreset?: boolean;
}

// 预置经典标杆卡片库（让 Nomad 卡片网格墙一开页就极其丰富！）
const presetCards: NomadCard[] = [
  {
    id: 'p1',
    timestamp: '2026-07-18 14:00',
    topic: 'Q3核心产品迭代与架构重构研讨会',
    input: '讨论重点：微服务架构升级、前端自动化压测、上线日期延期3天。',
    styleLabel: '大厂高效 OKR 纪要',
    scores: { efficiency: 5, clarity: 5, alignment: 4, time: 4, risk: 2 },
    averageScore: 4.8,
    isPreset: true,
    output: `📌 **一句话执行摘要与核心成果**\n会议一致同意启动微服务架构重构，同意将 Q3 上线时间微调至 7月25日，并明确自动化压测的负责人。\n\n📋 **Action Items 责任矩阵**\n| 待办事项 | 责任人 | 预计完成时间 | 交付标准 |\n|---|---|---|---|\n| 架构重构方案评审 | 张工 | 7月21日 | 完成文档与代码审查 |\n| 自动化压测脚本 | 李工 | 7月23日 | 通过 10万 QPS 压力测试 |\n\n⚠️ **潜在风险与后续跟踪**\n数据库迁移期间可能存在短暂停机，预计安排在夜间 2:00 进行。\n\n[HUIYI_SCORES]efficiency:5,clarity:5,alignment:4,time:4,risk:2[/HUIYI_SCORES]`
  },
  {
    id: 'p2',
    timestamp: '2026-07-17 10:30',
    topic: '全球游民市场拓展与品牌营销会',
    input: '讨论重点：巴厘岛与清迈线下工作坊预算、短视频裂变增长。',
    styleLabel: '头脑风暴与创意整理',
    scores: { efficiency: 4, clarity: 5, alignment: 5, time: 3, risk: 3 },
    averageScore: 4.6,
    isPreset: true,
    output: `📌 **一句话执行摘要与核心成果**\n确定启动清迈数字游民节赞助计划，拟定 5 万美金预算用于海外 TikTok 达人合作。\n\n📋 **Action Items 责任矩阵**\n| 待办事项 | 责任人 | 预计完成时间 | 交付标准 |\n|---|---|---|---|\n| TikTok 达人商务对接 | 迈克 | 7月20日 | 签约 10 位头部游民 KOC |\n| 场地预定与物料设计 | 莎拉 | 7月22日 | 完成主视觉设计 |\n\n[HUIYI_SCORES]efficiency:4,clarity:5,alignment:5,time:3,risk:3[/HUIYI_SCORES]`
  },
  {
    id: 'p3',
    timestamp: '2026-07-16 16:00',
    topic: '全员敏捷双周迭代复盘会',
    input: '讨论重点：修复了 14 个已知 Bug，前端组件库全量升级。',
    styleLabel: '敏捷站会与极简纪要',
    scores: { efficiency: 5, clarity: 4, alignment: 4, time: 5, risk: 1 },
    averageScore: 4.5,
    isPreset: true,
    output: `📌 **一句话执行摘要与核心成果**\n双周迭代任务完成率 96%，成功发布 V2.4 稳定版。\n\n[HUIYI_SCORES]efficiency:5,clarity:4,alignment:4,time:5,risk:1[/HUIYI_SCORES]`
  }
];

const cardsList = ref<NomadCard[]>([]);

interface FloatingItem {
  id: number;
  x: number;
  y: number;
  text: string;
}

const floatingItems = ref<FloatingItem[]>([]);
const isStamping = ref(false);
const totalMeetings = ref(0);
let floatId = 0;

// Web Audio API 合成金属盖章声
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

    const osc2 = ctx.createOscillator();
    const gain2 = ctx.createGain();
    osc2.type = 'square';
    osc2.frequency.setValueAtTime(750, now + 0.03);
    osc2.frequency.exponentialRampToValueAtTime(180, now + 0.08);
    gain2.gain.setValueAtTime(0.35, now + 0.03);
    gain2.gain.exponentialRampToValueAtTime(0.001, now + 0.09);
    osc2.connect(gain2);
    gain2.connect(ctx.destination);
    osc2.start(now + 0.03);
    osc2.stop(now + 0.11);
  } catch (e) {
    console.error('AudioContext error:', e);
  }
};

const stampApproved = () => {
  if (isStamping.value) return;
  isStamping.value = true;
  playStampSound();
  
  totalMeetings.value += 1;
  localStorage.setItem('huiyi_total_meetings', totalMeetings.value.toString());
  
  const id = floatId++;
  const x = Math.floor(Math.random() * 40) - 20;
  const y = -45;
  
  floatingItems.value.push({ id, x, y, text: 'Approved +1' });
  
  setTimeout(() => {
    floatingItems.value = floatingItems.value.filter(item => item.id !== id);
  }, 1000);

  setTimeout(() => {
    isStamping.value = false;
  }, 120);
};

const loadHistory = () => {
  try {
    const raw = localStorage.getItem('huiyi_history_records');
    const userCards: NomadCard[] = raw ? JSON.parse(raw) : [];
    cardsList.value = [...userCards, ...presetCards];
    
    const rawMeetings = localStorage.getItem('huiyi_total_meetings');
    totalMeetings.value = rawMeetings ? parseInt(rawMeetings, 10) : 12;
  } catch (e) {
    cardsList.value = [...presetCards];
  }
};

const saveHistory = (userCards: NomadCard[]) => {
  localStorage.setItem('huiyi_history_records', JSON.stringify(userCards));
};

const filteredCards = computed(() => {
  let list = [...cardsList.value];

  if (searchQuery.value.trim()) {
    const q = searchQuery.value.toLowerCase();
    list = list.filter(c => c.topic.toLowerCase().includes(q) || c.output.toLowerCase().includes(q));
  }

  if (activeFilter.value === 'top') {
    list.sort((a, b) => b.averageScore - a.averageScore);
  } else if (activeFilter.value === 'action') {
    list = list.filter(c => c.scores.clarity >= 4);
  } else if (activeFilter.value === 'recent') {
    list.sort((a, b) => parseInt(b.id) - parseInt(a.id));
  }

  return list;
});

const openCardDetail = (card: NomadCard) => {
  selectedCard.value = card;
};

const deleteCard = (id: string) => {
  if (confirm('确认删除此纪要卡片吗？')) {
    cardsList.value = cardsList.value.filter(c => c.id !== id);
    const userOnly = cardsList.value.filter(c => !c.isPreset);
    saveHistory(userOnly);
    selectedCard.value = null;
  }
};

const getScoreClass = (score: number) => {
  if (score >= 4.5) return 'score-high';
  if (score >= 3.5) return 'score-mid';
  return 'score-low';
};

const cleanExcerpt = (text: string) => {
  const cleaned = cleanResponseText(text);
  return cleaned.length > 90 ? cleaned.slice(0, 90) + '...' : cleaned;
};

const parseAIScores = (text: string) => {
  const match = text.match(/\[HUIYI_SCORES\](.*?)\[\/HUIYI_SCORES\]/);
  if (match) {
    const scoreStr = match[1].trim();
    const scores: Record<string, number> = {};
    scoreStr.split(',').forEach(item => {
      const [key, val] = item.split(':');
      if (key && val) {
        scores[key.trim()] = Math.min(5, Math.max(1, parseInt(val.trim(), 10) || 4));
      }
    });
    return {
      efficiency: scores.efficiency || 4,
      clarity: scores.clarity || 4,
      alignment: scores.alignment || 4,
      time: scores.time || 4,
      risk: scores.risk || 2
    };
  }
  return null;
};

const cleanResponseText = (text: string) => {
  return text.replace(/\[HUIYI_SCORES\].*?\[\/HUIYI_SCORES\]/g, '').trim();
};

onMounted(() => {
  loadHistory();
});

const handleShareClose = () => {
  showShareGuide.value = false;
  localStorage.setItem('shared_fission', 'true');
};

const isLimitReached = computed(() => {
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';

  try {
    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        taskType: 'text',
        prompt: `起卦主题：${promptTopic.value}。会议主题：${inquiryTopic.value}。用户期望效率指标：效率 ${userScores.value.efficiency}分，决议明确 ${userScores.value.clarity}分，协同 ${userScores.value.alignment}分，时间控制 ${userScores.value.time}分，风险阻碍 ${userScores.value.risk}分。会议讨论笔记：${userInput.value}。流派倾向：${activeStyle.value}`,
        style: activeStyle.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      const parsedScores = parseAIScores(data.result) || { ...userScores.value };
      const avg = (parsedScores.efficiency + parsedScores.clarity + parsedScores.alignment + parsedScores.time + (6 - parsedScores.risk)) / 5;

      const matched = styleOptions.find(o => o.value === activeStyle.value);
      const styleLabel = matched ? matched.label : '纪要流派';

      const newCard: NomadCard = {
        id: Date.now().toString(),
        timestamp: new Date().toLocaleString(),
        topic: inquiryTopic.value,
        input: userInput.value,
        styleLabel,
        scores: parsedScores,
        averageScore: avg,
        output: data.result
      };

      cardsList.value.unshift(newCard);
      const userOnly = cardsList.value.filter(c => !c.isPreset);
      saveHistory(userOnly);

      const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
      localStorage.setItem('free_uses', (currentUses + 1).toString());

      showCreateModal.value = false;
      selectedCard.value = newCard;

      // 清空输入
      inquiryTopic.value = '';
      userInput.value = '';
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const copyText = async (txt: string) => {
  try {
    const cleanedText = cleanResponseText(txt);
    await navigator.clipboard.writeText(cleanedText);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>

<style scoped>
/* Nomad 工具栏 */
.nomad-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.85rem 1.25rem;
  margin-bottom: 1.5rem;
  flex-wrap: wrap;
  gap: 1rem;
}

.filter-tabs {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.filter-pill {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  color: var(--text-secondary);
  padding: 0.4rem 0.85rem;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.filter-pill:hover {
  color: var(--text-primary);
  background: rgba(255, 255, 255, 0.08);
}

.filter-pill.active {
  background: var(--primary-gradient);
  color: white;
  border-color: transparent;
  box-shadow: 0 0 12px rgba(168, 85, 247, 0.4);
}

.search-and-create {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex: 1;
  justify-content: flex-end;
  min-width: 280px;
}

.nomad-search-input {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  padding: 0.45rem 1rem;
  color: var(--text-primary);
  font-size: 0.85rem;
  outline: none;
  width: 200px;
  transition: all 0.3s ease;
}

.nomad-search-input:focus {
  width: 240px;
  border-color: var(--primary-color);
}

.create-trigger-btn {
  background: var(--primary-gradient);
  color: white;
  border: none;
  padding: 0.5rem 1.1rem;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: bold;
  cursor: pointer;
  white-space: nowrap;
  box-shadow: 0 4px 15px rgba(168, 85, 247, 0.3);
  transition: transform 0.2s ease;
}

.create-trigger-btn:hover {
  transform: scale(1.05);
}

/* Nomad 卡片网格墙 */
.nomad-card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(290px, 1fr));
  gap: 1.25rem;
}

.nomad-card {
  padding: 1.25rem;
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  gap: 0.85rem;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  text-align: left;
  position: relative;
}

.nomad-card:hover {
  transform: translateY(-4px);
  border-color: rgba(168, 85, 247, 0.3);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.25);
}

.n-card-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 0.5rem;
}

.n-card-title-group {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.n-card-category {
  font-size: 1.05rem;
  font-weight: bold;
  color: var(--text-primary);
  line-height: 1.3;
}

.n-card-date {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.n-card-score-badge {
  padding: 0.25rem 0.6rem;
  border-radius: 8px;
  font-size: 0.78rem;
  font-weight: bold;
  white-space: nowrap;
}

.score-high {
  background: rgba(16, 185, 129, 0.18);
  color: #10b981;
  border: 1px solid rgba(16, 185, 129, 0.3);
}

.score-mid {
  background: rgba(245, 158, 11, 0.18);
  color: #f59e0b;
  border: 1px solid rgba(245, 158, 11, 0.3);
}

.score-low {
  background: rgba(239, 68, 68, 0.18);
  color: #ef4444;
  border: 1px solid rgba(239, 68, 68, 0.3);
}

/* Nomad 5维数据胶囊 Pill 矩阵 */
.n-card-metrics-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
}

.metric-pill {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.06);
  padding: 0.15rem 0.45rem;
  border-radius: 6px;
  font-size: 0.72rem;
  display: flex;
  gap: 0.3rem;
  align-items: center;
}

.pill-icon {
  color: var(--text-secondary);
}

.pill-val {
  color: var(--primary-color);
  font-weight: bold;
}

.n-card-body {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
  flex: 1;
}

.n-card-style-tag {
  font-size: 0.75rem;
  color: var(--primary-color);
  background: rgba(168, 85, 247, 0.1);
  padding: 0.15rem 0.4rem;
  border-radius: 4px;
  width: fit-content;
  font-weight: bold;
}

.n-card-excerpt {
  color: var(--text-secondary);
  font-size: 0.82rem;
  line-height: 1.4;
  margin: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}

.n-card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.05);
  padding-top: 0.6rem;
  margin-top: 0.2rem;
}

.n-card-tag-badge {
  color: var(--text-secondary);
}

.n-card-expand-link {
  color: var(--primary-color);
  font-weight: bold;
}

/* 模态框样式 */
.detail-modal-content, .create-modal-content {
  max-width: 650px;
  width: 92%;
  max-height: 85vh;
  overflow-y: auto;
  text-align: left;
}

.detail-modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  padding-bottom: 0.75rem;
  margin-bottom: 1rem;
}

.detail-modal-header h2 {
  font-size: 1.15rem;
  margin: 0;
  color: white;
}

.close-modal-icon {
  background: none;
  border: none;
  color: var(--text-secondary);
  font-size: 1.2rem;
  cursor: pointer;
}

.modal-actions-row {
  display: flex;
  gap: 0.75rem;
  margin-top: 1.25rem;
  justify-content: flex-end;
}

.modal-btn-action {
  background: var(--primary-gradient);
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  font-size: 0.85rem;
}

.modal-btn-delete {
  background: rgba(239, 68, 68, 0.15);
  color: #ef4444;
  border: 1px solid rgba(239, 68, 68, 0.3);
  padding: 0.5rem 1rem;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  font-size: 0.85rem;
}

.modal-btn-close {
  background: rgba(255, 255, 255, 0.08);
  color: var(--text-primary);
  border: 1px solid rgba(255, 255, 255, 0.1);
  padding: 0.5rem 1rem;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.85rem;
}

/* 印章与音效 */
.stamp-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  padding: 1.25rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 12px;
  margin-bottom: 1.25rem;
}

.stamp-canvas {
  position: relative;
  width: 120px;
  height: 120px;
}

.stamp-svg {
  width: 100%;
  height: 100%;
  cursor: pointer;
  transition: transform 0.1s ease-in-out;
}

.stamp-svg:hover {
  filter: drop-shadow(0 0 10px rgba(59, 130, 246, 0.4));
}

.stamp-svg.stamping {
  transform: scale(0.9) translateY(4px);
}

.floating-merit {
  position: absolute;
  left: 50%;
  top: 40%;
  font-size: 0.95rem;
  font-weight: bold;
  color: #3b82f6;
  text-shadow: 0 0 10px rgba(59, 130, 246, 0.6);
  pointer-events: none;
  white-space: nowrap;
}

.merit-counter-display {
  font-size: 0.85rem;
  color: var(--text-primary);
  text-align: center;
}

.merit-counter-display strong {
  font-size: 1.2rem;
}

.wood-fish-tip {
  font-size: 0.75rem;
  color: var(--text-secondary);
  margin: 0.2rem 0 0 0;
}

.score-sliders {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 1rem;
  margin-bottom: 0.5rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.04);
  padding: 1rem;
  border-radius: 10px;
}

.slider-group-item {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.slider-header {
  display: flex;
  justify-content: space-between;
  font-size: 0.78rem;
  color: var(--text-secondary);
}

.slider-value {
  color: var(--primary-color);
  font-weight: bold;
}

.range-slider {
  -webkit-appearance: none;
  width: 100%;
  height: 5px;
  border-radius: 3px;
  background: rgba(255, 255, 255, 0.1);
  outline: none;
  cursor: pointer;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: var(--primary-color);
  cursor: pointer;
}

.city-text-input {
  width: 100%;
  padding: 0.65rem 0.85rem;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  color: var(--text-primary);
  font-size: 0.88rem;
  outline: none;
}
</style>
