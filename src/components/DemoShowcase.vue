<template>
  <section class="glass-card showcase-container">
    <div class="showcase-header">
      <div class="showcase-title-group">
        <h2 class="showcase-title">会议议程与讨论框架案例库 (30 精选样例)</h2>
        <p class="showcase-subtitle">体验不同业务场景下的时间分配与引导提问，点击“一键同款生成”即可即刻生成</p>
      </div>
      <div class="showcase-badge">会议效能 · 免费体验</div>
    </div>

    <!-- 搜索与分类筛选 -->
    <div class="showcase-filter-bar">
      <div class="category-tabs">
        <button 
          v-for="cat in categories" 
          :key="cat"
          class="category-tab"
          :class="{ active: currentCategory === cat }"
          @click="currentCategory = cat"
        >
          {{ cat }}
        </button>
      </div>
      <div class="search-input-wrapper">
        <input 
          v-model="searchQuery"
          type="text"
          placeholder="搜索会议主题、框架流派或引导人关键词..."
          class="search-input"
        />
      </div>
    </div>

    <!-- 样例列表格 Grid -->
    <div class="sample-grid">
      <div 
        v-for="sample in paginatedSamples" 
        :key="sample.id" 
        class="sample-card"
      >
        <div class="sample-card-header">
          <span class="topic-category-tag">{{ sample.category }}</span>
          <span class="style-name-tag">{{ sample.style }}</span>
        </div>
        <div class="sample-original">
          <span class="sample-label">会议主题：</span>“{{ sample.topic }}”
        </div>
        <div class="sample-rewritten">
          <span class="sample-label">核心议程：</span>{{ sample.agenda }}
        </div>
        <div class="sample-card-footer">
          <button class="use-sample-btn" @click="$emit('use-sample', sample.topic)">
            一键同款生成
          </button>
        </div>
      </div>
    </div>

    <!-- 空状态提示 -->
    <div v-if="filteredSamples.length === 0" class="empty-showcase">
      未找到匹配的会议案例，请尝试切换分类或重置搜索关键词。
    </div>

    <!-- 分页组件 -->
    <div v-if="filteredSamples.length > pageSize" class="pagination-bar">
      <button 
        class="page-btn" 
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        上一页
      </button>
      <span class="page-info">第 {{ currentPage }} / {{ totalPages }} 页 (共 {{ filteredSamples.length }} 条)</span>
      <button 
        class="page-btn" 
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        下一页
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

defineEmits<{
  (e: 'use-sample', text: string): void;
}>();

const categories = ['全部', '大厂效能', '战略规划', '敏捷冲刺', '团队沟通', '创新发散'];
const currentCategory = ref('全部');
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = 6;

interface MeetingSample {
  id: number;
  category: string;
  topic: string;
  style: string;
  agenda: string;
}

// 精选 30 条自媒体会议框架案例
const raw30Samples: MeetingSample[] = [
  {
    id: 1,
    category: '大厂效能',
    topic: 'Q3季度产品延期与研发人力不足复盘会。',
    style: '大厂高效 OKR 框架',
    agenda: '延期模块归因通报(10min) -> 架构瓶颈突破方案讨论(20min) -> 里程碑调整与分工表决(15min)。'
  },
  {
    id: 2,
    category: '战略规划',
    topic: '新业务冷启动资源锁定与项目立项会议。',
    style: '九巨擘企业开会圆桌',
    agenda: '马斯克第一性原理提问(15min) -> 乔布斯体验直觉把控(15min) -> 秦始皇军令状签署(15min)。'
  },
  {
    id: 3,
    category: '敏捷冲刺',
    topic: '双周研发迭代计划与阻塞排除每日站会。',
    style: '敏捷站会与冲刺框架',
    agenda: '昨日成果核对(5min) -> 今日潜在阻塞汇报(10min) -> 跨团队联调阻碍即时攻坚(10min)。'
  },
  {
    id: 4,
    category: '团队沟通',
    topic: '跨部门年度绩效考核与员工晋升答辩评审。',
    style: '商务正规议程框架',
    agenda: '答辩人述职阐述(10min) -> 评审团提问答辩(15min) -> 无记名评分与最终合议(10min)。'
  },
  {
    id: 5,
    category: '创新发散',
    topic: '下一代AIGC辅助办公应用功能头脑风暴。',
    style: '头脑风暴与创新发散',
    agenda: '痛点场景破冰(10min) -> 无限制点子爆发(20min) -> 亲和图归类与可行性打分(15min)。'
  },
  {
    id: 6,
    category: '大厂效能',
    topic: '季度OKR目标对齐与跨部门依赖关系澄清。',
    style: '大厂高效 OKR 框架',
    agenda: '关键结果陈述(15min) -> 跨部门资源缺口讨论(20min) -> 协同依赖甘特图锁定(10min)。'
  },
  {
    id: 7,
    category: '战略规划',
    topic: '初创公司融资BP路演预演与问答漏洞防漏。',
    style: '九巨擘企业开会圆桌',
    agenda: 'VC视角路演讲解(15min) -> 九巨擘模拟评审刁钻追问(20min) -> 答辩修正与录制(10min)。'
  },
  {
    id: 8,
    category: '敏捷冲刺',
    topic: '系统重构上线回归测试缺陷清零攻坚会。',
    style: '敏捷站会与冲刺框架',
    agenda: '遗留高危Bug梳理(5min) -> 核心模块负责人分工(15min) -> 联调测试环境消灭Bug(25min)。'
  },
  {
    id: 9,
    category: '团队沟通',
    topic: '跨部门Q4营销预算分配冲突协调董事会议。',
    style: '商务正规议程框架',
    agenda: '各方诉求及数据通报(10min) -> 超支分配技术性抗辩(20min) -> 最终方案表决(15min)。'
  },
  {
    id: 10,
    category: '创新发散',
    topic: '自媒体短视频爆款内容策划选题会。',
    style: '头脑风暴与创新发散',
    agenda: '行业爆款分析(10min) -> 黄金三秒脑洞大开(20min) -> 选题性价比漏斗过滤(15min)。'
  },
  {
    id: 11,
    category: '大厂效能',
    topic: '用户流失率异常升高专项应急排查会议。',
    style: '大厂高效 OKR 框架',
    agenda: '数据波动与监控通报(10min) -> 产品与技术侧流失归因(20min) -> 紧急防卫举措锁定(15min)。'
  },
  {
    id: 12,
    category: '战略规划',
    topic: '企业出海本土化市场拓展三年规划决策会。',
    style: '九巨擘企业开会圆桌',
    agenda: '目标国家宏观数据报告(15min) -> 模拟竞争攻防推演(20min) -> 核心资源预算立案(15min)。'
  },
  {
    id: 13,
    category: '敏捷冲刺',
    topic: '应用架构微服务化拆分技术方案评审会。',
    style: '敏捷站会与冲刺框架',
    agenda: '总体拓扑介绍(10min) -> 数据一致性与调用链攻坚(25min) -> 迁移路线表决(10min)。'
  },
  {
    id: 14,
    category: '团队沟通',
    topic: '公司核心技术泄露合规事故问责整改会。',
    style: '商务正规议程框架',
    agenda: '安全防护漏洞报告(10min) -> 涉事责任认定与处置(15min) -> 零信任整改话术部署(15min)。'
  },
  {
    id: 15,
    category: '创新发散',
    topic: '下一年度雇主品牌建设与招聘话术设计。',
    style: '头脑风暴与创新发散',
    agenda: '离职访谈痛点归纳(10min) -> 亮点优势重新组合(20min) -> 渠道投放策略敲定(15min)。'
  },
  {
    id: 16,
    category: '大厂效能',
    topic: '核心业务主流程降本增效与外包服务控本会。',
    style: '大厂高效 OKR 框架',
    agenda: '外包产出性价比统计(10min) -> 核心流程平替降本讨论(25min) -> 减员节流表决(10min)。'
  },
  {
    id: 17,
    category: '战略规划',
    topic: '与头部互联网平台战略合作谈判对齐会。',
    style: '九巨擘企业开会圆桌',
    agenda: '谈判底线与核心诉求梳理(15min) -> 场景对抗模拟演练(20min) -> 授权边界最终裁决(10min)。'
  },
  {
    id: 18,
    category: '敏捷冲刺',
    topic: '新版本发布前核心链路压力测试指标评审会。',
    style: '敏捷站会与冲刺框架',
    agenda: '极限压测数据同步(10min) -> 性能滑坡模块攻坚(25min) -> 发布窗口确认(10min)。'
  },
  {
    id: 19,
    category: '团队沟通',
    topic: '企业文化升级宣导与核心价值观对齐会。',
    style: '商务正规议程框架',
    agenda: '新价值观体系推行方案(15min) -> 各部门行为准则修订(20min) -> 答疑交流(10min)。'
  },
  {
    id: 20,
    category: '创新发散',
    topic: '智能硬件外观工业设计初审头脑风暴。',
    style: '头脑风暴与创新发散',
    agenda: '目标受众喜好洞察(10min) -> 多方案概念大爆发(25min) -> 三维模型渲染方向打分(10min)。'
  },
  {
    id: 21,
    category: '大厂效能',
    topic: '年中核心资产盘点与闲置设备租赁处置会。',
    style: '大厂高效 OKR 框架',
    agenda: '闲置资产台账通报(10min) -> 外部租赁与回收询价(20min) -> 财务核销表决(15min)。'
  },
  {
    id: 22,
    category: '战略规划',
    topic: '传统企业数字化改造与线上商城冷启动会。',
    style: '九巨擘企业开会圆桌',
    agenda: '马斯克考察第一性数据(15min) -> 贝索斯飞轮效应验证(20min) -> 第一阶段预算批准(10min)。'
  },
  {
    id: 23,
    category: '敏捷冲刺',
    topic: '容器云服务故障自愈容灾方案技术对齐会。',
    style: '敏捷站会与冲刺框架',
    agenda: '宕机模拟与告警测试(10min) -> 自动备份与流量切换攻坚(25min) -> 实盘操作锁定(10min)。'
  },
  {
    id: 24,
    category: '团队沟通',
    topic: '跨区域分支机构销售返利合同纠纷应对会。',
    style: '商务正规议程框架',
    agenda: '涉案合同漏洞分析(15min) -> 调解边界与抗辩策略讨论(20min) -> 代表授权表决(10min)。'
  },
  {
    id: 25,
    category: '创新发散',
    topic: '线下门店精细化新零售陈列体验风暴会。',
    style: '头脑风暴与创新发散',
    agenda: '门店动线瓶颈梳理(10min) -> 极致人货场重组脑洞(25min) -> A角B角灰度落地定案(10min)。'
  },
  {
    id: 26,
    category: '大厂效能',
    topic: '公司核心高管股权激励第三期解锁考核会。',
    style: '大厂高效 OKR 框架',
    agenda: '业绩对赌指标核算报告(15min) -> 解锁资格与合规审议(20min) -> 董事会表决(10min)。'
  },
  {
    id: 27,
    category: '战略规划',
    topic: '探索接入区块链技术进行防伪存证可行性。',
    style: '九巨擘企业开会圆桌',
    agenda: '技术防伪边界分析(15min) -> 市场接受度激辩(20min) -> 概念论证小样分配(10min)。'
  },
  {
    id: 28,
    category: '敏捷冲刺',
    topic: '多云环境统一API网关高并发网关开发对齐。',
    style: '敏捷站会与冲刺框架',
    agenda: '并发流量指标与阻塞项(10min) -> 安全防御模块分工(25min) -> 技术路线图锁定(10min)。'
  },
  {
    id: 29,
    category: '团队沟通',
    topic: '办公研发大楼物业搬迁后综合成本复盘会。',
    style: '商务正规议程框架',
    agenda: '新旧大楼综合开支对比(15min) -> 降本痛点识别(20min) -> 绿色节能细则表决(10min)。'
  },
  {
    id: 30,
    category: '创新发散',
    topic: '跨平台智能车载车载助手交互UI头脑风暴。',
    style: '头脑风暴与创新发散',
    agenda: '车载安全性边界讨论(10min) -> 全声控极致脑洞爆发(25min) -> 模拟场景交互定音(10min)。'
  }
];

const samples = ref<MeetingSample[]>(raw30Samples);

const filteredSamples = computed(() => {
  return samples.value.filter(s => {
    const matchCat = currentCategory.value === '全部' || s.category === currentCategory.value;
    const matchQuery = !searchQuery.value.trim() || 
      s.topic.includes(searchQuery.value) || 
      s.style.includes(searchQuery.value) || 
      s.agenda.includes(searchQuery.value);
    return matchCat && matchQuery;
  });
});

const totalPages = computed(() => Math.ceil(filteredSamples.value.length / pageSize) || 1);

const paginatedSamples = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return filteredSamples.value.slice(start, start + pageSize);
});

watch([currentCategory, searchQuery], () => {
  currentPage.value = 1;
});
</script>
