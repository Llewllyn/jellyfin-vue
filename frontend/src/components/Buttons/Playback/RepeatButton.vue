<template>
  <VBtn
    icon
    :size="size"
    :color="playbackManager.isRepeating ? 'primary' : undefined"
    @click="playbackManager.toggleRepeatMode">
    <VIcon
      :size="size"
      :icon="repeatIcon" />
  </VBtn>
</template>

<script setup lang="ts">
import { RepeatMode, playbackManager } from '@/store/playbackManager';
import IMdiRepeat from 'virtual:icons/mdi/repeat';
import IMdiRepeatOnce from 'virtual:icons/mdi/repeat-once';
import { computed } from 'vue';

defineProps<{ size?: string }>();

const repeatIcon = computed(() =>
  playbackManager.repeatMode === RepeatMode.RepeatOne
    ? IMdiRepeatOnce
    : IMdiRepeat
);
</script>
