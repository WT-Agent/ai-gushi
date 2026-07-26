<template>
  <div class="glass-card ticker-card">
    <div class="ticker-wrapper">
      <div v-for="(item, index) in tickerItems" :key="index" class="ticker-item">
        <span>{{ item.text }}</span>
        <span style="color: var(--success-color); font-weight: bold;">[童话创作完成]</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const tickerItems = ref([
  { text: '用户 138****9281 成功生成了小熊按时刷牙与晚安治愈童话' },
  { text: '用户 177****4019 成功创作了小松鼠勇敢森林探索与互助故事' },
  { text: '用户 189****3320 成功生成了不挑食小企鹅营养健康睡前故事' },
  { text: '用户 131****8829 成功创作了小飞象克服自卑建立自信成长的童话' },
  { text: '用户 153****1290 成功生成了经典寓言守株待兔新编与成语启蒙故事' },
]);
</script>
