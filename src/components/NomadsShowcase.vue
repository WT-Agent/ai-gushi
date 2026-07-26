<template>
  <section class="nomads-showcase-section">
    <div class="showcase-header">
      <div class="header-left">
        <h2 class="showcase-title">儿童故事与睡前童话实战模板库</h2>
        <p class="showcase-subtitle">精选睡前安抚、习惯培养、冒险成长与经典新编案例，点击“一键套用”快速创作</p>
      </div>
      <span class="showcase-badge">已收录 {{ showcaseItems.length }} 个童话故事模板</span>
    </div>

    <div class="showcase-grid">
      <div 
        v-for="item in showcaseItems" 
        :key="item.id" 
        class="glass-card showcase-card"
      >
        <div class="card-header">
          <span class="scenario-tag">{{ item.tag }}</span>
          <span class="usage-count">{{ item.usageCount }} 次应用</span>
        </div>

        <div class="card-content">
          <h3 class="item-title">{{ item.title }}</h3>
          <p class="item-prompt">“{{ item.prompt }}”</p>
        </div>

        <div class="card-action">
          <button class="apply-btn" @click="applyTemplate(item)">
            <span>一键套用</span>
            <svg class="arrow-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="5" y1="12" x2="19" y2="12"></line>
              <polyline points="12 5 19 12 12 19"></polyline>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const emit = defineEmits<{
  (e: 'apply-template', payload: { prompt: string; storyType?: string; ageGroup?: string; storyTheme?: string }): void;
}>();

export interface ShowcaseItem {
  id: string;
  tag: string;
  title: string;
  prompt: string;
  storyType?: string;
  ageGroup?: string;
  storyTheme?: string;
  usageCount: string;
}

const showcaseItems = computed<ShowcaseItem[]>(() => [
  {
    id: 'gushi-1',
    tag: '睡前安抚',
    title: '小熊嘟嘟按时刷牙睡觉的奇妙晚安梦',
    prompt: '小熊嘟嘟最喜欢吃甜甜的水蜜桃，但到了晚上总是磨蹭不想刷牙也不想上床。请创作一个充满奇幻色彩与安眠氛围的睡前童话，讲述牙齿小精灵与星光泡泡床的奇遇。',
    storyType: '温暖治愈睡前童话',
    ageGroup: '2-4岁幼儿',
    storyTheme: '按时睡觉不挑食',
    usageCount: '62.4k'
  },
  {
    id: 'gushi-2',
    tag: '勇敢成长',
    title: '小松鼠皮皮的迷雾森林大冒险',
    prompt: '小松鼠皮皮在彩虹森林里不小心迷路了，面对黑暗的树洞和怪异的声音，它如何凭借智慧、勇气以及小鸟朋友的帮助克服恐惧，找到回家的路。',
    storyType: '动物王国科幻探险故事',
    ageGroup: '5-7岁学龄前',
    storyTheme: '勇敢自信',
    usageCount: '58.1k'
  },
  {
    id: 'gushi-3',
    tag: '好习惯培养',
    title: '不挑食的小企鹅波波与五彩蔬菜王国',
    prompt: '小企鹅波波只喜欢吃冰淇淋，拒绝吃任何胡萝卜和西兰花，直到它乘坐魔法滑梯来到了五彩蔬菜王国，帮助蔬菜小精灵解除危机，学会了营养均衡。',
    storyType: '好习惯培养与行为引导故事',
    ageGroup: '2-4岁幼儿',
    storyTheme: '按时睡觉不挑食',
    usageCount: '49.8k'
  },
  {
    id: 'gushi-4',
    tag: '心理建构',
    title: '大耳朵小飞象克服自卑的云端飞翔记',
    prompt: '小飞象波波因为长着一对特别大的耳朵而被小伙伴嘲笑，但在一次大森林的风暴灾难中，它发现大耳朵能像翅膀一样飞翔，最终救了大家并建立自信。',
    storyType: '好习惯培养与行为引导故事',
    ageGroup: '5-7岁学龄前',
    storyTheme: '勇敢自信',
    usageCount: '44.3k'
  },
  {
    id: 'gushi-5',
    tag: '经典新编',
    title: '守株待兔成语新编：小农夫的勤劳农场',
    prompt: '用生动幽默且符合儿童理解的方式重编经典寓言守株待兔，让孩子们在快乐听故事的过程中明白不能寄希望于侥幸，只有勤劳付出才有收获。',
    storyType: '经典寓言与成语新编',
    ageGroup: '8-12岁小学生',
    storyTheme: '诚实友善',
    usageCount: '39.7k'
  },
  {
    id: 'gushi-6',
    tag: '科幻探索',
    title: '星际小兔子波比的月球果冻矿山探索',
    prompt: '小兔子波比驾驶着胡萝卜飞船前往遥远的草莓星球，途中遇到陨石带和外星伙伴，大家携手互助，共同寻找神奇的闪光能量矿石。',
    storyType: '动物王国科幻探险故事',
    ageGroup: '8-12岁小学生',
    storyTheme: '探索自然',
    usageCount: '36.5k'
  }
]);

function applyTemplate(item: ShowcaseItem) {
  emit('apply-template', {
    prompt: item.prompt,
    storyType: item.storyType,
    ageGroup: item.ageGroup,
    storyTheme: item.storyTheme
  });
}
</script>

<style scoped>
.nomads-showcase-section {
  margin-top: 2rem;
  width: 100%;
}

.showcase-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 1.25rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--card-border);
}

.showcase-title {
  font-size: 1.2rem;
  font-weight: 700;
  color: var(--text-primary);
  background: var(--primary-gradient);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.showcase-subtitle {
  font-size: 0.825rem;
  color: var(--text-secondary);
  margin-top: 0.25rem;
}

.showcase-badge {
  font-size: 0.75rem;
  color: #a5b4fc;
  background: rgba(99, 102, 241, 0.12);
  border: 1px solid rgba(99, 102, 241, 0.25);
  padding: 4px 10px;
  border-radius: 20px;
}

.showcase-grid {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  gap: 1.25rem;
}

@media (min-width: 640px) {
  .showcase-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .showcase-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.showcase-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
  padding: 1.25rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid var(--card-border);
  border-radius: 14px;
  transition: all 0.25s ease;
}

.showcase-card:hover {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(99, 102, 241, 0.4);
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.scenario-tag {
  font-size: 0.75rem;
  font-weight: 600;
  padding: 3px 8px;
  border-radius: 6px;
  background: rgba(168, 85, 247, 0.15);
  color: #c084fc;
  border: 1px solid rgba(168, 85, 247, 0.3);
}

.usage-count {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.card-content {
  margin-bottom: 1rem;
  flex: 1;
}

.item-title {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 0.4rem;
}

.item-prompt {
  font-size: 0.825rem;
  color: var(--text-secondary);
  line-height: 1.45;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  font-style: italic;
}

.card-action {
  padding-top: 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.04);
}

.apply-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 0.5rem 1rem;
  background: rgba(99, 102, 241, 0.1);
  border: 1px solid rgba(99, 102, 241, 0.3);
  border-radius: 8px;
  color: #a5b4fc;
  font-size: 0.825rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.showcase-card:hover .apply-btn {
  background: var(--primary-gradient);
  border-color: transparent;
  color: white;
}

.arrow-icon {
  width: 14px;
  height: 14px;
  transition: transform 0.2s ease;
}

.apply-btn:hover .arrow-icon {
  transform: translateX(3px);
}
</style>
