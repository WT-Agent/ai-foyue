<template>
  <div class="app-container">
    <!-- 顶部生成成功浮动 Toast -->
    <transition name="fade">
      <div v-if="showSuccessToast" class="top-success-toast">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="20 6 9 17 4 12"></polyline>
        </svg>
        <span>佛曰开示谶语生成成功！</span>
      </div>
    </transition>

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

    <!-- 顶部 App Header (已移除未登录额度提示栏) -->
    <header>
      <h1>{{ appTitle }}</h1>
      <p>智能 AI 体验引擎 · 每日 AI 佛曰解压</p>
    </header>

    <!-- 活跃动态与使用人数轮播 -->
    <UserTicker />

    <!-- 核心卡片 / 结果与历史记录展示 -->
    <main class="glass-card input-group">
      <div class="card-tabs">
        <button class="tab-btn" :class="{ active: !showHistory }" @click="showHistory = false">
          数字禅修
        </button>
        <button class="tab-btn" :class="{ active: showHistory }" @click="showHistory = true">
          历史记录 ({{ historyList.length }})
        </button>
      </div>

      <!-- 本地历史记录视图 -->
      <div v-if="showHistory" class="history-view">
        <div class="history-header">
          <span>本地禅修谶语历史</span>
          <button v-if="historyList.length > 0" class="clear-all-btn" @click="clearAllHistory">清空全部</button>
        </div>

        <div v-if="historyList.length === 0" class="empty-state">
          <p>暂无历史解惑记录</p>
        </div>

        <div v-else class="history-grid">
          <div v-for="item in historyList" :key="item.id" class="history-card">
            <div class="h-card-header">
              <span class="h-card-style">{{ item.styleLabel }}</span>
              <span class="h-card-time">{{ item.timestamp }}</span>
            </div>
            
            <div class="h-card-body">
              <div class="h-card-nomad-title">
                <span class="h-city-tag">烦恼: {{ item.category }}</span>
                <span class="h-score-badge">综合禅修值: {{ getAverageScore(item) }}</span>
              </div>

              <p class="h-card-excerpt"><strong>佛曰开示：</strong>{{ cleanExcerpt(item.output) }}</p>
            </div>

            <div class="h-card-actions">
              <button class="h-action-btn load-btn" @click="selectHistoryItem(item)">
                加载详情
              </button>
              <button class="h-action-btn delete-btn" @click="deleteHistoryRecord(item.id)">
                删除
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-else>
        <!-- 主测评输入区域 -->
        <div v-if="!result" class="divination-setup">
          <div class="selector-group">
            <label class="selector-label">选择当前所遇的烦恼类型</label>
            <select v-model="inquiryCategory" class="style-select">
              <option value="职场内卷">职场内卷 (工作压力与疲惫)</option>
              <option value="财富焦虑">财富焦虑 (金钱、房贷与物价)</option>
              <option value="情感纠葛">情感纠葛 (人际交往与恋爱纠葛)</option>
              <option value="虚度光阴">虚度光阴 (拖延症与虚无感)</option>
              <option value="精神内耗">精神内耗 (深夜焦虑与胡思乱想)</option>
            </select>
          </div>

          <div class="selector-group">
            <label class="selector-label">输入您的烦恼或执念困惑 (选填)</label>
            <textarea 
              v-model="userInput" 
              placeholder="请输入您遇到的具体困惑，例如：工作十年遇到了瓶颈，想辞职当数字游民但又害怕失去稳定的月薪安全感..."
              rows="4"
            ></textarea>
          </div>

          <div class="selector-group">
            <label class="selector-label">选择解惑开示流派</label>
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
            :disabled="loading" 
            @click="handleGenerate"
          >
            {{ loading ? '禅思推演中...' : '虔诚问佛，求解红尘' }}
          </button>

          <!-- 异常提示 -->
          <div v-if="errorMsg" class="error-banner">
            {{ errorMsg }}
          </div>
        </div>

        <!-- 测评结果展现 -->
        <div v-else class="divination-result">
          <!-- 电子木鱼解压交互板块 -->
          <div class="wooden-fish-section">
            <div class="fish-canvas">
              <svg 
                class="wooden-fish-svg" 
                :class="{ knocking: isKnocking }" 
                @click="knockWoodenFish" 
                viewBox="0 0 200 160"
              >
                <!-- 绘制高保真拟物木鱼形状 -->
                <path d="M100,20 C40,20 20,60 20,100 C20,130 50,150 100,150 C150,150 180,130 180,100 C180,60 160,20 100,20 Z" fill="rgba(255,255,255,0.06)" stroke="rgba(255,255,255,0.15)" stroke-width="3" />
                <path d="M40,100 C40,70 60,50 100,50 C140,50 160,70 160,100" fill="none" stroke="rgba(168, 85, 247, 0.2)" stroke-width="2" />
                <circle cx="60" cy="80" r="10" fill="rgba(255,255,255,0.08)" />
                <path d="M140,110 C130,120 115,125 100,125 C85,125 70,120 60,110" fill="none" stroke="rgba(255,255,255,0.15)" stroke-width="3" stroke-linecap="round" />
              </svg>
              <!-- 浮空功德动效 (严格无 Emoji) -->
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
              累计功德：<strong style="color: var(--primary-color);">{{ totalMerit }}</strong>
              <p class="wood-fish-tip">点击上方木鱼敲击，功德 +1</p>
            </div>
          </div>

          <!-- 单轨禅修数据分析看板 (AI共识判定) -->
          <div v-if="aiScores" class="comparison-dashboard">
            <h3 class="dashboard-title">禅心数据分析 (AI共识测定值)</h3>
            <div class="comparison-grid">
              <div v-for="metric in metricsList" :key="metric.key" class="comparison-row">
                <div class="metric-info">
                  <span class="metric-label">{{ metric.label }}</span>
                  <span class="metric-scores-text">
                    判定值: <strong style="color: var(--accent-color)">{{ aiScores[metric.key] }} / 5</strong>
                  </span>
                </div>
                <div class="comparison-bars">
                  <div class="bar-container">
                    <div class="bar-bg">
                      <div class="bar-fill ai-fill" :style="{ width: aiScores[metric.key] * 20 + '%' }"></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="result-header">
            <div class="result-title-group">
              <span class="result-title">AI 佛开示：{{ inquiryCategory }}</span>
              <span class="success-badge">生成成功</span>
            </div>
            <div class="button-actions">
              <button class="icon-btn" @click="copyText">
                {{ copied ? '已复制谶语' : '复制谶语' }}
              </button>
              <button class="icon-btn highlight" @click="showShareCard = true">
                生成分享卡片
              </button>
              <button class="icon-btn restart-btn" @click="resetReview">
                重新求问
              </button>
            </div>
          </div>

          <!-- 加载中骨架屏 -->
          <div v-if="loading" class="skeleton">
            <div class="skeleton-line" style="width: 80%"></div>
            <div class="skeleton-line" style="width: 95%"></div>
            <div class="skeleton-line" style="width: 60%"></div>
            <div class="skeleton-line" style="width: 75%"></div>
          </div>

          <!-- 渲染回复 -->
          <div class="ai-response-wrapper">
            <div class="output-content scroll-box" style="text-align: left;">{{ displayResultText }}</div>
          </div>
        </div>
      </div>
    </main>

    <!-- 演示案例区组件 (模块三：30 条佛曰解惑精选案例展示) -->
    <DemoShowcase @use-sample="handleUseSample" />

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
          <p>我们非常重视您的隐私。您在本应用中选择的烦恼和倾诉仅用于实时大模型佛解语开示，我们不会在服务器端进行永久存储或记录。</p>
          <p>为了记录您的解惑历史、免费额度和功德总数，本应用会在您的浏览器本地（localStorage）记录相关状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 每日佛曰解惑与数字禅修微应用。本应用仅提供心理调试与禅意娱乐开示，不代表任何宗教信仰和医疗诊断。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 (自适应高度 + weixin.png & dingtalk.png 展示) -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
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

    <!-- 裂变拦截弹窗 (模块四：裂变机制) -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />

    <!-- 分享卡片弹窗 (模块二扩展) -->
    <ShareCardModal
      :visible="showShareCard"
      :content="displayResultText"
      :wechat-id="wechatId"
      @close="showShareCard = false"
    />

    <!-- 微信 H5 分享引导浮层 -->
    <div v-if="showShareGuide" class="share-guide-overlay" @click="handleShareClose">
      <div class="share-guide-arrow">↗</div>
      <div class="share-guide-content">
        <p>点击右上角菜单 <strong>•••</strong></p>
        <p>选择 <strong>「分享到朋友圈」</strong></p>
        <p class="share-guide-sub">分享这款高效率的 AI 智能微应用</p>
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

const appTitle = ref(appConfig.title || '网腾无限AI 每日佛曰');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const inquiryCategory = ref('职场内卷');
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);
const showSuccessToast = ref(false);
const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showShareCard = ref(false);

const isKnocking = ref(false);
const totalMerit = ref(parseInt(localStorage.getItem('muyu_total_merit') || '0', 10));
const floatingItems = ref<{ id: number; text: string; x: number; y: number }[]>([]);
let floatingId = 0;

// 禅定指标字段
const metricsList = [
  { key: 'merit', label: '功德值 (Merit)' },
  { key: 'mindfulness', label: '禅定度 (Mindfulness)' },
  { key: 'attachment', label: '执念度 (Attachment)' },
  { key: 'karma', label: '因果造化 (Karma)' },
  { key: 'impatience', label: '浮躁系数 (Impatience)' }
];

const aiScores = ref<Record<string, number> | null>(null);

const styleOptions = [
  { label: '传统禅宗开示 (经典佛教公案指点迷局)', value: '传统禅宗开示' },
  { label: '九巨擘禅意论 (马斯克/乔布斯等9大佬跨时空激辩)', value: '九巨擘禅意论' },
  { label: '赛博木鱼解压 (系统级清理妄念与本地业力缓存)', value: '赛博木鱼解压' },
  { label: '当头棒喝警醒 (极其毒舌辛辣拆穿自我感动)', value: '当头棒喝警醒' },
  { label: '温情心灵抚慰 (细腻温暖的心理调试与生活美学)', value: '温情心灵抚慰' }
];

const activeStyle = ref(styleOptions[0].value);

interface HistoryItem {
  id: string;
  timestamp: string;
  category: string;
  input: string;
  styleLabel: string;
  aiScores: Record<string, number> | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

const loadHistory = () => {
  try {
    const raw = localStorage.getItem('foyue_history_records');
    historyList.value = raw ? JSON.parse(raw) : [];
  } catch (e) {
    historyList.value = [];
  }
};

const saveHistory = () => {
  localStorage.setItem('foyue_history_records', JSON.stringify(historyList.value));
};

const addHistoryRecord = () => {
  const matched = styleOptions.find(o => o.value === activeStyle.value);
  const styleLabel = matched ? matched.label : '开示流派';

  const newItem: HistoryItem = {
    id: Date.now().toString(),
    timestamp: new Date().toLocaleString(),
    category: inquiryCategory.value,
    input: userInput.value,
    styleLabel,
    aiScores: aiScores.value ? { ...aiScores.value } : null,
    output: result.value
  };
  historyList.value.unshift(newItem);
  saveHistory();
};

const deleteHistoryRecord = (id: string) => {
  historyList.value = historyList.value.filter(item => item.id !== id);
  saveHistory();
};

const clearAllHistory = () => {
  if (confirm('确认清空所有历史禅修谶语记录吗？此操作不可恢复。')) {
    historyList.value = [];
    saveHistory();
  }
};

const selectHistoryItem = (item: HistoryItem) => {
  inquiryCategory.value = item.category;
  userInput.value = item.input;
  aiScores.value = item.aiScores ? { ...item.aiScores } : null;
  result.value = item.output;
  showHistory.value = false;
};

const getAverageScore = (item: HistoryItem) => {
  if (!item.aiScores) return '4.0';
  const s = item.aiScores;
  const avg = (s.merit + s.mindfulness + s.attachment + s.karma + s.impatience) / 5;
  return avg.toFixed(1);
};

// 纯前端利用 Web Audio API 动态合成敲击木鱼的声音
const knockWoodenFish = () => {
  isKnocking.value = true;
  setTimeout(() => {
    isKnocking.value = false;
  }, 100);

  // 累加功德
  totalMerit.value++;
  localStorage.setItem('muyu_total_merit', totalMerit.value.toString());

  // 新增漂浮提示 (严格无 Emoji)
  const id = floatingId++;
  const x = (Math.random() - 0.5) * 60;
  const y = -40 - Math.random() * 20;
  floatingItems.value.push({ id, text: '功德 +1', x, y });
  setTimeout(() => {
    floatingItems.value = floatingItems.value.filter(item => item.id !== id);
  }, 800);

  try {
    const audioCtx = new (window.AudioContext || (window as any).webkitAudioContext)();
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();

    osc.type = 'sine';
    osc.frequency.setValueAtTime(320, audioCtx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(100, audioCtx.currentTime + 0.15);

    gain.gain.setValueAtTime(0.5, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.15);

    osc.connect(gain);
    gain.connect(audioCtx.destination);

    osc.start();
    osc.stop(audioCtx.currentTime + 0.16);
  } catch (err) {
    // 捕获可能出现的音频上下文受限异常
  }
};

const parseAIScores = (text: string) => {
  const match = text.match(/\[ZEN_SCORES\](.*?)\[\/ZEN_SCORES\]/);
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
      merit: scores.merit || 3,
      mindfulness: scores.mindfulness || 3,
      attachment: scores.attachment || 3,
      karma: scores.karma || 3,
      impatience: scores.impatience || 3
    };
  }
  return null;
};

const cleanResponseText = (text: string) => {
  return text.replace(/\[ZEN_SCORES\][\s\S]*?\[\/ZEN_SCORES\]/gi, '').trim();
};

const displayResultText = computed(() => {
  return cleanResponseText(result.value);
});

const cleanExcerpt = (text: string) => {
  const cleaned = cleanResponseText(text);
  return cleaned.length > 80 ? cleaned.slice(0, 80) + '...' : cleaned;
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
  result.value = '';
  aiScores.value = null;

  try {
    const fullPrompt = `【烦恼类型】：${inquiryCategory.value}。${userInput.value.trim() ? '烦恼困惑倾诉：' + userInput.value : ''}\n【选定流派】：${activeStyle.value}\n${promptTopic.value}`;

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
      result.value = data.result;
      aiScores.value = parseAIScores(data.result);
      addHistoryRecord();
      triggerSuccessToast();
      
      const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
      localStorage.setItem('free_uses', (currentUses + 1).toString());
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const resetReview = () => {
  result.value = '';
  aiScores.value = null;
};

const handleUseSample = (sampleTrouble: string) => {
  userInput.value = sampleTrouble;
  showHistory.value = false;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(displayResultText.value);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>
