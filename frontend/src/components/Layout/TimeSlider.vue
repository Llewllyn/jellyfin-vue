<template>
  <VSlider
    v-model="sliderValue"
    hide-details
    :max="runtime"
    thumb-label
    validate-on="input"
    @start="clicked = true"
    @end="onRelease">
    <template #prepend>
      {{ formatTime(playbackManager.currentTime) }}
    </template>
    <template #thumb-label>
      {{ formatTime(sliderValue) }}
    </template>
    <template #append>
      {{ formatTime(runtime) }}
    </template>
  </VSlider>
</template>

<script setup lang="ts">
import { playbackManager } from '@/store/playbackManager';
import { formatTime } from '@/utils/time';
import { computed, ref } from 'vue';

const currentInput = ref(0);
const clicked = ref(false);
const runtime = computed(() => playbackManager.currentItemRuntime / 1000);
const sliderValue = computed({
  get() {
    return clicked.value ? currentInput.value : playbackManager.currentTime;
  },
  set(newValue) {
    currentInput.value = newValue;
  }
});

/**
 * Once the user releases the slider, change the time of the playbackManager with whatever
 * input value was provided by the user
 */
function onRelease(): void {
  playbackManager.currentTime = currentInput.value;
  clicked.value = false;
}
</script>
