<template>
  <div
    class="box-border flex flex-col border-color-border"
    :class="uiSettingsStore.episodesPosition === 'right' ? 'h-full border-r border-t' : ''"
  >
    <div class="flex items-baseline space-x-1 p-2">
      <div class="text-nowrap text-base font-bold">{{ t('selectEpisode') }}</div>
      <div class="truncate text-color-disable">{{ data.vod_remarks }}</div>
    </div>

    <!-- 用于计算文本宽度的参考元素 -->
    <div
      class="pointer-events-none absolute -z-50 h-1 w-max overflow-hidden opacity-0"
      ref="textWidthRef"
    >
      <div v-for="item in data.vod_play_url" :key="item.url">
        {{ item.name }}
      </div>
    </div>

    <div
      class="episodes-box grid flex-1 gap-2 overflow-y-auto overflow-x-hidden p-2 pt-0"
      :style="{
        'grid-template-columns': `repeat(auto-fit, minmax(${textWidth}px, 1fr))`,
      }"
    >
      <div
        v-for="(item, idx) in data.vod_play_url"
        :class="{ active: videoDetailStore.curEpisodeIdx === idx }"
        class="relative flex cursor-pointer items-center justify-center rounded border border-color-border p-1 hover:bg-color-hover [&.active]:bg-color-primary/10"
        :key="item.url"
        :title="item.name"
        @click="videoDetailStore.changeEpisode(idx)"
        @contextmenu.prevent="handleRightClick($event, item.url)"
      >
        <div class="truncate">{{ item.name }}</div>

        <div 
          v-if="copiedUrl === item.url" 
          class="absolute -top-8 left-1/2 -translate-x-1/2 whitespace-nowrap rounded bg-gray-800 px-2 py-1 text-xs text-white shadow-lg"
          :style="{ animation: 'fade-slide 2s ease-out forwards' }"
        >
          链接已复制
        </div>

        <div class="absolute" :class="{ playon: videoDetailStore.curEpisodeIdx === idx }">
          <i></i><i></i><i></i><i></i>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { VideoDetailResponse } from '@/api/detail'
import useUISettingsStore from '@/stores/settings/ui'
import useVideoDetailStore from '@/stores/videoDetail'
import { computed, useTemplateRef, ref } from 'vue'
import { useI18n } from 'vue-i18n'

const { data } = defineProps<{ data: VideoDetailResponse }>()
const { t } = useI18n()
const videoDetailStore = useVideoDetailStore()
const uiSettingsStore = useUISettingsStore()
const textWidthRef = useTemplateRef<HTMLDivElement>('textWidthRef')

// 用于控制显示哪个项的复制提示
const copiedUrl = ref('')

const textWidth = computed(() => Math.max(textWidthRef.value?.offsetWidth || 0, 50))

// 【核心逻辑】处理右键点击
const handleRightClick = async (e: MouseEvent, url: string) => {
  e.preventDefault() // 防止默认的右键菜单弹出
  
  try {
    await navigator.clipboard.writeText(url)
    
    // 设置当前复制的 URL，用于触发提示显示
    copiedUrl.value = url
    
    // 2秒后清除提示（与 CSS 动画时间一致）
    setTimeout(() => {
      if (copiedUrl.value === url) {
        copiedUrl.value = ''
      }
    }, 2000)
    
  } catch (err) {
    console.error('复制失败:', err)
    alert('复制失败，请重试')
  }
}
</script>

<style scoped>
.episodes-box {
  grid-auto-rows: 50px;
  justify-items: stretch;
}

@keyframes playon {
  0% { transform: scaleY(0.7); }
  50% { transform: scaleY(1); }
  100% { transform: scaleY(0.35); }
}

/* 播放指示条样式 */
.playon {
  height: 8px;
  left: calc(50% - 11px);
  bottom: 0;
}
.playon i {
  width: 4px;
  height: 6px;
  border-radius: 4px 4px 0 0;
  background-color: var(--color-primary);
  position: absolute;
  bottom: 0;
  left: 0;
  transform-origin: center bottom;
}
.playon i:nth-last-child(1) { animation: playon 0.8s 0.3s infinite; }
.playon i:nth-last-child(2) { animation: playon 0.8s 0.1s infinite; left: 6px; }
.playon i:nth-last-child(3) { animation: playon 0.6s 0.2s infinite; left: 12px; }
.playon i:nth-last-child(4) { animation: playon 1s 0.3s infinite; left: 18px; }

/* 【新增】提示气泡的动画 */
@keyframes fade-slide {
  0% {
    opacity: 0;
    transform: translateY(10px) translateX(-50%);
  }
  10% {
    opacity: 1;
    transform: translateY(-10px) translateX(-50%);
  }
  90% {
    opacity: 1;
    transform: translateY(-10px) translateX(-50%);
  }
  100% {
    opacity: 0;
    transform: translateY(-20px) translateX(-50%);
  }
}
</style>
