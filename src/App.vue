<template>
  <div class="app-container">
    <!-- 成功提示 -->
    <div v-if="copied" class="top-success-toast">
      复制成功
    </div>
    <!-- 常驻悬浮分享按钮 (H5 / 移动端与桌面端通用) -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg class="share-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享童话故事工具</span>
    </button>

    <header>
      <h1>{{ appTitle }}</h1>
      <p>世界观主角设定 · 跌宕冒险情节 · 温暖睡前安抚 · 寓教于乐启发</p>
    </header>

    <!-- 动态广播轮播 -->
    <UserTicker />

    <!-- 核心操作区卡片 -->
    <main ref="inputCardRef" class="glass-card input-group">
      <!-- 故事类型选择 -->
      <div class="selector-group">
        <label class="selector-label">选择故事类型</label>
        <div class="style-selector">
          <button 
            v-for="stype in storyTypeOptions" 
            :key="stype.value"
            class="style-option"
            :class="{ active: activeStoryType === stype.value }"
            @click="activeStoryType = stype.value"
          >
            {{ stype.label }}
          </button>
        </div>
      </div>

      <!-- 适合年龄与故事主题 -->
      <div class="options-row" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem;">
        <div class="selector-group">
          <label class="selector-label">适合年龄</label>
          <div class="style-selector">
            <button 
              v-for="age in ageGroupOptions" 
              :key="age"
              class="style-option"
              :class="{ active: selectedAgeGroup === age }"
              @click="selectedAgeGroup = age"
            >
              {{ age }}
            </button>
          </div>
        </div>

        <div class="selector-group">
          <label class="selector-label">故事主题</label>
          <div class="style-selector">
            <button 
              v-for="theme in storyThemeOptions" 
              :key="theme"
              class="style-option"
              :class="{ active: selectedStoryTheme === theme }"
              @click="selectedStoryTheme = theme"
            >
              {{ theme }}
            </button>
          </div>
        </div>
      </div>

      <!-- 故事灵感与主角描述输入框 -->
      <div class="selector-group">
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <label class="selector-label">输入故事主角、背景场景或灵感梗概</label>
          <div style="display: flex; gap: 0.5rem;">
            <button v-if="userInput" class="text-link-btn" @click="userInput = ''">清空输入</button>
            <button class="text-link-btn" @click="showStoryGuideModal = true">儿童心理发展与故事安抚启发指南</button>
          </div>
        </div>
        <textarea 
          v-model="userInput" 
          placeholder="请描述想写的故事灵感或主角细节...（例如：故事主角是一只叫皮皮的小松鼠，平时总是磨蹭不肯按时睡觉。希望能创作一个带有梦幻树洞与星光泡泡的睡前童话，引导孩子养成规律作息习惯。）"
          style="min-height: 130px;"
        ></textarea>
        <div style="display: flex; justify-content: space-between; font-size: 0.75rem; color: var(--text-secondary);">
          <span>字符数: {{ userInput.length }} 字</span>
          <span>建议包含主角性格特点、故事核心矛盾与希望传达的正面价值观</span>
        </div>
      </div>

      <!-- 操作按钮区 -->
      <div style="display: flex; gap: 0.75rem;">
        <button 
          class="action-btn" 
          :disabled="loading || !userInput.trim()"
          @click="handleGenerate"
        >
          {{ loading ? '正在构建奇幻童话世界与睡前治愈故事中...' : '开始生成儿童故事与睡前童话' }}
        </button>
        <button class="icon-btn" style="padding: 0 1rem; border-radius: 10px;" @click="toggleHistoryDrawer">
          历史记录 ({{ historyList.length }})
        </button>
      </div>

      <!-- 异常提示 -->
      <div v-if="errorMsg" style="color: var(--accent-color); font-size: 0.85rem; text-align: center; margin-top: 0.5rem;">
        {{ errorMsg }}
      </div>
    </main>

    <!-- 生成结果卡片 -->
    <section v-if="result || loading" class="glass-card">
      <div class="result-header">
        <span class="result-title">儿童故事与睡前童话创作报告</span>
        <div class="button-actions">
          <button v-if="result" class="icon-btn" @click="copyText">
            {{ copied ? '已复制童话全文' : '复制故事内容' }}
          </button>
          <button v-if="result" class="icon-btn" @click="resetResult">
            重置
          </button>
        </div>
      </div>

      <!-- 加载中骨架屏 -->
      <div v-if="loading" class="skeleton">
        <div class="skeleton-line" style="width: 85%"></div>
        <div class="skeleton-line" style="width: 95%"></div>
        <div class="skeleton-line" style="width: 70%"></div>
        <div class="skeleton-line" style="width: 90%"></div>
        <div class="skeleton-line" style="width: 60%"></div>
      </div>

      <!-- 渲染结果 -->
      <div v-else-if="result">
        <!-- AI 共识打分可视化看板 -->
        <div v-if="aiScores" class="scores-container" style="margin-bottom: 1.5rem; padding: 1.25rem; background: rgba(0,0,0,0.25); border-radius: 12px; border: 1px solid rgba(255,255,255,0.06);">
          <div style="font-weight: 700; font-size: 0.95rem; margin-bottom: 1rem; color: #a5b4fc; display: flex; justify-content: space-between; align-items: center;">
            <span>AI 童话故事质量与睡前安抚评估看板</span>
            <span style="font-size: 0.8rem; font-weight: normal; color: var(--text-secondary);">综合质量分: {{ getAverageScoreFromMap(aiScores) }} / 5.0</span>
          </div>
          <div class="metrics-grid" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 1rem;">
            <div v-for="metric in metricsList" :key="metric.key" class="metric-item">
              <div style="display: flex; justify-content: space-between; font-size: 0.8rem; margin-bottom: 0.3rem;">
                <span style="color: var(--text-secondary);">{{ metric.label }}</span>
                <span style="font-weight: bold; color: var(--accent-color);">{{ aiScores[metric.key] || 4 }} / 5</span>
              </div>
              <div class="bar-bg" style="height: 6px; background: rgba(255,255,255,0.08); border-radius: 3px; overflow: hidden;">
                <div class="bar-fill" :style="{ width: ((aiScores[metric.key] || 4) * 20) + '%', background: 'var(--primary-gradient)', height: '100%', borderRadius: '3px', transition: 'width 0.5s ease' }"></div>
              </div>
            </div>
          </div>
        </div>

        <div class="output-content">{{ displayResultText }}</div>
      </div>
    </section>

    <!-- 历史记录面板 -->
    <section v-if="showHistory" class="glass-card" style="margin-top: 1rem;">
      <div class="result-header">
        <span class="result-title">本地童话故事历史记录</span>
        <button class="icon-btn" @click="showHistory = false">关闭记录</button>
      </div>

      <div v-if="historyList.length === 0" style="text-align: center; color: var(--text-secondary); padding: 1.5rem; font-size: 0.85rem;">
        暂无历史故事记录，快为孩子创作第一个温馨的睡前童话吧！
      </div>

      <div v-else class="history-grid" style="display: flex; flex-direction: column; gap: 0.75rem; max-height: 320px; overflow-y: auto;">
        <div v-for="item in historyList" :key="item.id" class="history-item" style="padding: 1rem; background: rgba(0,0,0,0.2); border-radius: 10px; border: 1px solid var(--card-border);">
          <div style="display: flex; justify-content: space-between; font-size: 0.8rem; color: var(--text-secondary); margin-bottom: 0.4rem;">
            <span>{{ item.timestamp }} · [{{ item.storyType }} / {{ item.ageGroup }}]</span>
            <span style="color: var(--primary-color);">评分: {{ getAverageScore(item) }}</span>
          </div>
          <div style="font-size: 0.85rem; font-weight: bold; margin-bottom: 0.4rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; color: var(--text-primary);">
            灵感: {{ item.input }}
          </div>
          <div style="display: flex; gap: 0.5rem;">
            <button class="icon-btn" style="font-size: 0.75rem;" @click="applyHistory(item)">套用灵感</button>
            <button class="icon-btn" style="font-size: 0.75rem;" @click="viewHistoryOutput(item)">查看童话全文</button>
          </div>
        </div>
      </div>
    </section>

    <!-- 童话故事模板 Showcase -->
    <NomadsShowcase
      @apply-template="handleApplyTemplate"
    />

    <!-- 儿童心理发展与故事安抚启发指南 Modal -->
    <div v-if="showStoryGuideModal" class="modal-overlay" @click.self="showStoryGuideModal = false">
      <div class="modal-content" style="max-width: 480px;">
        <h3>儿童心理发展与故事安抚启发指南</h3>
        <p style="text-align: left; font-size: 0.825rem; margin-bottom: 1rem; color: var(--text-secondary);">
          提升童话故事吸引力与睡前治愈效果的核心心理指导规则：
        </p>
        <div class="modal-scroll-area" style="text-align: left; font-size: 0.825rem;">
          <div v-for="(rule, idx) in storyRules" :key="idx" style="margin-bottom: 0.75rem; padding: 0.5rem 0.75rem; background: rgba(255,255,255,0.03); border-radius: 8px; border: 1px solid rgba(255,255,255,0.05);">
            <div style="color: var(--accent-color); font-weight: bold; margin-bottom: 0.2rem;">{{ rule.title }}</div>
            <div style="color: var(--text-primary); margin-bottom: 0.2rem;">实操建议: {{ rule.advice }}</div>
            <div style="color: var(--text-secondary); font-size: 0.775rem;">避坑红线: {{ rule.avoid }}</div>
          </div>
        </div>
        <button class="modal-btn" style="margin-top: 1rem;" @click="showStoryGuideModal = false">关闭</button>
      </div>
    </div>

    <!-- 微信 H5 悬浮分享引导 Modal -->
    <div v-if="showShareGuide" class="modal-overlay" @click.self="showShareGuide = false">
      <div class="modal-content">
        <h3>分享童话故事工具</h3>
        <p>扫码关注或将链接转发给有宝妈宝爸的朋友，轻松生成温暖治愈的睡前童话故事。</p>
        
        <div class="qr-code-placeholder">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" width="100%" height="100%">
            <rect width="100" height="100" fill="white"/>
            <rect x="5" y="5" width="25" height="25" fill="#110e24"/>
            <rect x="9" y="9" width="17" height="17" fill="white"/>
            <rect x="13" y="13" width="9" height="9" fill="#110e24"/>
            <rect x="70" y="5" width="25" height="25" fill="#110e24"/>
            <rect x="74" y="9" width="17" height="17" fill="white"/>
            <rect x="78" y="13" width="9" height="9" fill="#110e24"/>
            <rect x="5" y="70" width="25" height="25" fill="#110e24"/>
            <rect x="9" y="74" width="17" height="17" fill="white"/>
            <rect x="13" y="78" width="9" height="9" fill="#110e24"/>
            <rect x="35" y="10" width="8" height="8" fill="#110e24"/>
            <rect x="48" y="5" width="6" height="12" fill="#110e24"/>
            <rect x="60" y="15" width="5" height="5" fill="#110e24"/>
            <rect x="35" y="35" width="10" height="10" fill="#110e24"/>
            <rect x="50" y="45" width="15" height="8" fill="#110e24"/>
            <rect x="40" y="70" width="8" height="16" fill="#110e24"/>
            <rect x="55" y="65" width="10" height="10" fill="#110e24"/>
            <rect x="75" y="40" width="12" height="12" fill="#110e24"/>
            <rect x="75" y="75" width="15" height="15" fill="#110e24"/>
            <rect x="45" y="80" width="8" height="8" fill="#110e24"/>
          </svg>
        </div>

        <div style="font-size: 0.8rem; color: var(--text-secondary); margin-bottom: 1.5rem;">
          微信号: <span style="color: var(--primary-color); font-weight: bold;">{{ wechatId }}</span>
        </div>

        <button class="modal-btn" @click="showShareGuide = false">关闭</button>
      </div>
    </div>

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
      <a href="https://api.wuxian.xyz/sign-up?aff=OyRY" target="_blank" rel="noopener noreferrer" class="footer-link-btn">API 平台</a>
      <a href="https://www.kutuyun.com/aff/IPJKCKWF" target="_blank" rel="noopener noreferrer" class="footer-link-btn">酷兔云</a>
      <a href="https://bandwagonhost.com/aff.php?aff=48115" target="_blank" rel="noopener noreferrer" class="footer-link-btn">搬瓦工</a>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的隐私与儿童成长数据安全。您在本工具中输入的所有灵感梗概与自定义角色均仅用于 AI 实时童话创作，系统不会在云端公开或留存您的隐私数据。</p>
          <p>为了保障免费使用额度，本应用会在您的浏览器本地（localStorage）记录试用次数与解锁状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用网腾无限 AI 儿童故事与睡前童话创作专家。本工具生成的童话故事、品格启发与睡前安抚文本仅供家长及幼教工作者阅读与互动引导使用。</p>
          <p>生成内容均经过儿童亲和力与积极价值观审核，建议家长陪伴孩子共同进行亲子阅读。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信交流" class="contact-qr-img" />
              <span class="contact-qr-label">微信交流</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉联系" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉联系</span>
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
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import NomadsShowcase from './components/NomadsShowcase.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

// 配置参数
const appTitle = ref(appConfig.title || '网腾无限AI - 儿童故事与睡前童话创作专家');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const inputCardRef = ref<HTMLElement | null>(null);
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);

const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showStoryGuideModal = ref(false);

// 解析 Cookie
const getCookie = (name: string): string | null => {
  const nameEQ = name + "=";
  const ca = document.cookie.split(';');
  for (let i = 0; i < ca.length; i++) {
    let c = ca[i];
    while (c.charAt(0) === ' ') c = c.substring(1, c.length);
    if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
  }
  return null;
};

// 用户登录状态
const userToken = ref(getCookie('wuxian_session'));
const isLoggedIn = computed(() => !!userToken.value);
const authUsesCount = ref(parseInt(localStorage.getItem('auth_uses') || '0', 10));

// 故事类型预设
const storyTypeOptions = [
  { label: '温暖治愈睡前童话', value: '温暖治愈睡前童话' },
  { label: '好习惯培养与行为引导故事', value: '好习惯培养与行为引导故事' },
  { label: '动物王国科幻探险故事', value: '动物王国科幻探险故事' },
  { label: '经典寓言与成语新编', value: '经典寓言与成语新编' }
];
const activeStoryType = ref(storyTypeOptions[0].value);

// 适合年龄与故事主题
const ageGroupOptions = ['2-4岁幼儿', '5-7岁学龄前', '8-12岁小学生'];
const selectedAgeGroup = ref('2-4岁幼儿');

const storyThemeOptions = ['勇敢自信', '诚实友善', '按时睡觉不挑食', '探索自然'];
const selectedStoryTheme = ref('按时睡觉不挑食');

// 评估指标列表
const metricsList = [
  { key: 'storyImaginativeness', label: '故事想象力' },
  { key: 'moralEducationalValue', label: '品格教育蕴涵' },
  { key: 'languageChildFriendliness', label: '儿童语体亲和力' },
  { key: 'plotEngagement', label: '情节吸引力' },
  { key: 'soothingBedtimeEffect', label: '睡前安抚治愈力' }
];

const aiScores = ref<Record<string, number> | null>(null);

// 历史记录定义
interface HistoryItem {
  id: string;
  timestamp: string;
  storyType: string;
  ageGroup: string;
  storyTheme: string;
  input: string;
  aiScores: Record<string, number> | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

// 儿童心理与睡前引导规则法则
const storyRules = [
  { title: '睡前情绪安抚降频法则', advice: '选用舒缓平稳的语调，避免过于惊险悬疑的冲突情节，帮助孩子大脑逐步进入放松安眠状态', avoid: '切忌在结尾设置突兀惊悚的怪兽或未解决的恐怖悬念' },
  { title: '具象行为替代说教法则', advice: '将按时刷牙、不挑食等好习惯融入小动物主角的趣味互动，让孩子产生角色共鸣', avoid: '切忌使用严厉的成人说教语言或惩罚性后果威吓孩子' },
  { title: '正面心理暗示强化法则', advice: '在故事中多强调勇敢、友善与自信的积极回响，通过温暖包容的结局赋予孩子安全感', avoid: '切忌对主角的失误进行否定性评价或过度标签化' }
];

// 计算纯结果文本 (剔除打分标签 [GUSHI_SCORES])
const displayResultText = computed(() => {
  if (!result.value) return '';
  return result.value.replace(/\[GUSHI_SCORES\][\s\S]*?\[\/GUSHI_SCORES\]/g, '').trim();
});

// 解析打分标签
const parseAiScores = (rawText: string) => {
  const match = rawText.match(/\[GUSHI_SCORES\](.*?)\[\/GUSHI_SCORES\]/);
  if (!match) return null;
  const content = match[1];
  const scoresObj: Record<string, number> = {};
  content.split(',').forEach(item => {
    const [key, val] = item.split(':');
    if (key && val) {
      scoresObj[key.trim()] = parseInt(val.trim(), 10) || 4;
    }
  });
  return Object.keys(scoresObj).length > 0 ? scoresObj : null;
};

// 计算平均分
const getAverageScoreFromMap = (scores: Record<string, number>) => {
  const keys = Object.keys(scores);
  if (keys.length === 0) return '4.5';
  const sum = keys.reduce((acc, k) => acc + (scores[k] || 4), 0);
  return (sum / keys.length).toFixed(1);
};

const getAverageScore = (item: HistoryItem) => {
  if (!item.aiScores) return '4.5';
  return getAverageScoreFromMap(item.aiScores);
};

// 本地历史记录读取与保存
const loadHistory = () => {
  try {
    const raw = localStorage.getItem('gushi_history_records');
    historyList.value = raw ? JSON.parse(raw) : [];
  } catch (e) {
    historyList.value = [];
  }
};

const saveHistory = () => {
  localStorage.setItem('gushi_history_records', JSON.stringify(historyList.value));
};

const addHistoryRecord = () => {
  const newItem: HistoryItem = {
    id: Date.now().toString(),
    timestamp: new Date().toLocaleString(),
    storyType: activeStoryType.value,
    ageGroup: selectedAgeGroup.value,
    storyTheme: selectedStoryTheme.value,
    input: userInput.value,
    aiScores: aiScores.value,
    output: result.value
  };
  historyList.value.unshift(newItem);
  if (historyList.value.length > 20) {
    historyList.value = historyList.value.slice(0, 20);
  }
  saveHistory();
};

const toggleHistoryDrawer = () => {
  loadHistory();
  showHistory.value = !showHistory.value;
};

const applyHistory = (item: HistoryItem) => {
  userInput.value = item.input;
  activeStoryType.value = item.storyType;
  selectedAgeGroup.value = item.ageGroup;
  selectedStoryTheme.value = item.storyTheme;
  showHistory.value = false;
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const viewHistoryOutput = (item: HistoryItem) => {
  userInput.value = item.input;
  result.value = item.output;
  aiScores.value = item.aiScores;
  showHistory.value = false;
};

// 限制与额度检测
const isLimitReached = computed(() => {
  if (isLoggedIn.value) {
    return authUsesCount.value >= 15;
  }
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
  result.value = '';
  aiScores.value = null;

  try {
    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: `任务指导: ${promptTopic.value}\n【故事类型】: ${activeStoryType.value}\n【适合年龄】: ${selectedAgeGroup.value}\n【故事主题】: ${selectedStoryTheme.value}\n【故事灵感/背景描述】: ${userInput.value}`,
        style: activeStoryType.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      aiScores.value = parseAiScores(data.result);
      
      addHistoryRecord();

      if (isLoggedIn.value) {
        const nextAuthUses = authUsesCount.value + 1;
        localStorage.setItem('auth_uses', nextAuthUses.toString());
        authUsesCount.value = nextAuthUses;
      } else {
        const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
        localStorage.setItem('free_uses', (currentUses + 1).toString());
      }
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const handleApplyTemplate = (payload: { prompt: string; storyType?: string; ageGroup?: string; storyTheme?: string }) => {
  userInput.value = payload.prompt;
  if (payload.storyType) activeStoryType.value = payload.storyType;
  if (payload.ageGroup) selectedAgeGroup.value = payload.ageGroup;
  if (payload.storyTheme) selectedStoryTheme.value = payload.storyTheme;
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const resetResult = () => {
  result.value = '';
  aiScores.value = null;
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

onMounted(() => {
  loadHistory();
});
</script>

<style scoped>
.text-link-btn {
  background: none;
  border: none;
  color: #a5b4fc;
  font-size: 0.775rem;
  cursor: pointer;
  transition: color 0.2s ease;
}
.text-link-btn:hover {
  color: var(--text-primary);
  text-decoration: underline;
}
</style>
