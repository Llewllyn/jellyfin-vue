<template>
  <VMain v-if="playbackManager.queue">
    <VAppBar color="transparent">
      <AppBarButtonLayout @click="$router.back()">
        <template #icon>
          <VIcon>
            <IMdiArrowLeft />
          </VIcon>
        </template>
      </AppBarButtonLayout>
      <VSpacer />
      <AppBarButtonLayout @click="isVisualizing = !isVisualizing">
        <template #icon>
          <VIcon>
            <IDashiconsAlbum v-if="isVisualizing" />
            <IMdiChartBar v-else />
          </VIcon>
        </template>
      </AppBarButtonLayout>
    </VAppBar>
    <VCol class="px-0">
      <VFadeTransition mode="out-in">
        <Swiper
          v-if="!isVisualizing"
          class="d-flex justify-center align-center user-select-none"
          :modules="modules"
          :slides-per-view="4"
          centered-slides
          :autoplay="false"
          effect="coverflow"
          :coverflow-effect="coverflowEffect"
          keyboard
          a11y
          virtual
          @swiper="(swiper) => swiperInstance = swiper"
          @slide-change="onSlideChange">
          <SwiperSlide
            v-for="(item, index) in playbackManager.queue"
            :key="`${item.Id}-${index}`"
            :virtual-index="`${item.Id}-${index}`"
            class="d-flex justify-center">
            <div class="album-cover presentation-height">
              <BlurhashImage :item="item" />
            </div>
          </SwiperSlide>
        </Swiper>
        <MusicVisualizer
          v-else
          class="d-flex justify-center align-center user-select-none presentation-height" />
      </VFadeTransition>
      <VRow class="justify-center align-center mt-3">
        <VCol cols="6">
          <VRow class="justify-center align-center">
            <VCol>
              <VRow>
                <h1 class="text-h4">
                  {{ playbackManager.currentItem?.Name }}
                </h1>
              </VRow>
              <VRow>
                <span class="text-subtitle">
                  {{ artistString }}
                </span>
              </VRow>
            </VCol>
            <!-- TODO: Fix alignment with the end time of TimeSlider -->
            <VCol class="d-flex justify-end">
              <LikeButton
                v-if="playbackManager.currentItem"
                :item="playbackManager?.currentItem"
                size="x-large" />
            </VCol>
          </VRow>
          <VRow class="justify-center align-center mt-3">
            <TimeSlider />
          </VRow>
          <VRow class="justify-center align-center">
            <ShuffleButton size="x-large" />
            <PreviousTrackButton size="x-large" />
            <PlayPauseButton size="x-large" />
            <NextTrackButton size="x-large" />
            <RepeatButton size="x-large" />
          </VRow>
        </VCol>
      </VRow>
    </VCol>
  </VMain>
</template>

<route lang="yaml">
meta:
  layout: fullpage
  backdrop:
    opacity: 0.75
  transition:
    enter: 'scroll-y-reverse-transition'
    leave: 'scroll-y-transition'
</route>

<script setup lang="ts">
import { playbackManager } from '@/store/playbackManager';
import { getBlurhash } from '@/utils/images';
import { ImageType } from '@jellyfin/sdk/lib/generated-client';
import { isNil } from 'lodash-es';
import type SwiperType from 'swiper';
import 'swiper/css';
import 'swiper/css/a11y';
import 'swiper/css/effect-coverflow';
import 'swiper/css/keyboard';
import 'swiper/css/virtual';
import { A11y, EffectCoverflow, Keyboard, Virtual } from 'swiper/modules';
import { Swiper, SwiperSlide } from 'swiper/vue';
import { computed, ref, shallowRef, watchEffect } from 'vue';
import { useRoute } from 'vue-router/auto';

const modules = [A11y, Keyboard, Virtual, EffectCoverflow];
const route = useRoute();

const coverflowEffect = {
  depth: 500,
  slideShadows: false,
  rotate: 0,
  stretch: -400
};

const isVisualizing = ref(false);

const backdropHash = computed(() => {
  return playbackManager.currentItem
    ? getBlurhash(playbackManager.currentItem, ImageType.Primary)
    : '';
});
const artistString = computed(() =>
  playbackManager.currentItem?.Artists?.join(', ')
);

const swiperInstance = shallowRef<SwiperType>();

watchEffect(() => {
  if (swiperInstance.value && !isNil(playbackManager.currentItemIndex)) {
    swiperInstance.value.slideTo(playbackManager.currentItemIndex);
    route.meta.title = playbackManager.currentItem?.Name ?? '';
  }

  if (backdropHash.value) {
    route.meta.backdrop.blurhash = backdropHash.value;
  }
}
);

/**
 * Handle slide changes
 */
function onSlideChange(): void {
  const index = swiperInstance.value?.activeIndex ?? 0;

  playbackManager.currentItemIndex = index;
}
</script>

<style lang="scss" scoped>
.album-cover {
  position: relative;
  min-width: 65vh;
  width: 65vh;
}

.presentation-height {
  height: 65vh;
}
</style>
