<template>
  <VMain
    class="fullscreen-video-container fill-height"
    :class="{ 'cursor-none': !overlay }"
    @mousemove="handleMouseMove"
    @touchend="handleMouseMove">
    <VOverlay
      v-model="overlay"
      contained
      scrim="transparent"
      width="100%"
      height="100%">
      <div
        class="d-flex flex-column justify-space-between align-center player-overlay">
        <div class="osd-top pt-s pl-s pr-s">
          <div class="d-flex align-center py-2 px-4">
            <div class="d-flex">
              <VBtn
                :icon="IMdiClose"
                @click="playbackManager.stop" />
              <VBtn
                :icon="IMdiChevronDown"
                @click="playerElement.toggleFullscreenVideoPlayer" />
            </div>
            <div class="d-flex ml-auto">
              <CastButton />
            </div>
          </div>
        </div>
        <div class="osd-bottom pb-s pl-s pr-s">
          <div class="pa-4">
            <TimeSlider />
            <div
              class="controls-wrapper d-flex align-stretch justify-space-between">
              <div
                v-if="$vuetify.display.mdAndUp"
                class="d-flex flex-column align-start justify-center mr-auto video-title">
                <template
                  v-if="
                    playbackManager.currentlyPlayingType ===
                      BaseItemKind.Episode
                  ">
                  <span class="mt-1 text-subtitle-1 text-truncate">
                    {{ playbackManager.currentItem?.Name }}
                  </span>
                  <span class="text-subtitle-2 text--secondary text-truncate">
                    {{ playbackManager.currentItem?.SeriesName }}
                  </span>
                  <span class="text-subtitle-2 text--secondary text-truncate">
                    {{
                      $t('seasonEpisode', {
                        seasonNumber:
                          playbackManager.currentItem?.ParentIndexNumber,
                        episodeNumber: playbackManager.currentItem?.IndexNumber
                      })
                    }}
                  </span>
                </template>
                <template v-else>
                  <span>{{ playbackManager.currentItem?.Name }}</span>
                </template>
                <br />
                <span
                  v-if="playbackManager.currentItem?.RunTimeTicks"
                  class="text-subtitle-2 text--secondary text-truncate">
                  {{
                    getEndsAtTime(playbackManager.currentItem.RunTimeTicks)
                      .value
                  }}
                </span>
              </div>
              <div
                class="d-flex player-controls align-center justify-start justify-md-center">
                <PreviousTrackButton class="mx-1" />
                <PlayPauseButton class="mx-1" />
                <NextTrackButton class="mx-1" />
              </div>
              <div class="d-flex aligh-center ml-auto ml-md-0">
                <VolumeSlider
                  v-if="$vuetify.display.smAndUp"
                  class="mr-2" />
                <QueueButton :close-on-click="true" />
                <SubtitleSelectionButton
                  v-if="$vuetify.display.smAndUp"
                  v-model="subtitleSelectionButtonOpened" />
                <PlaybackSettingsButton
                  v-model="playbackSettingsButtonOpened" />
                <VBtn
                  v-if="mediaControls.supportsPictureInPicture"
                  class="align-self-center"
                  icon
                  @click="mediaControls.togglePictureInPicture">
                  <VIcon>
                    <IMdiPictureInPictureBottomRight />
                  </VIcon>
                </VBtn>
                <VBtn
                  v-if="fullscreen.isSupported"
                  class="align-self-center"
                  icon
                  @click="fullscreen.toggle">
                  <VIcon>
                    <IMdiFullscreen v-if="fullscreen.isFullscreen" />
                    <IMdiFullscreenExit v-else />
                  </VIcon>
                  <VTooltip
                    :text="$t('fullScreen')"
                    location="top" />
                </VBtn>
              </div>
            </div>
          </div>
        </div>
      </div>
    </VOverlay>
  </VMain>
</template>

<route lang="yaml">
meta:
  layout: fullpage
  transition:
    enter: 'scroll-y-reverse-transition'
    leave: 'scroll-y-transition'
</route>

<script setup lang="ts">
import {
  mediaControls,
  mediaElementRef
} from '@/store';
import { playbackManager } from '@/store/playbackManager';
import { playerElement } from '@/store/playerElement';
import { getEndsAtTime } from '@/utils/time';
import { BaseItemKind } from '@jellyfin/sdk/lib/generated-client';
import {
  useFullscreen,
  useMagicKeys,
  useTimeoutFn,
  whenever
} from '@vueuse/core';
import IMdiChevronDown from 'virtual:icons/mdi/chevron-down';
import IMdiClose from 'virtual:icons/mdi/close';
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue';

/**
 * - iOS's Safari fullscreen API is only available for the video element
 */
const fullscreen = useFullscreen().isSupported.value
  ? useFullscreen(undefined, { autoExit: true })
  : useFullscreen(mediaElementRef, { autoExit: true });
const keys = useMagicKeys();
const osd = ref(true);
const subtitleSelectionButtonOpened = ref(false);
const playbackSettingsButtonOpened = ref(false);
const staticOverlay = computed(
  () =>
    playbackManager.isPaused ||
    subtitleSelectionButtonOpened.value ||
    playbackSettingsButtonOpened.value
);

const overlay = computed({
  get: () => staticOverlay.value || osd.value,
  set: (newValue) => (osd.value = newValue)
});

const timeout = useTimeoutFn(() => {
  overlay.value = false;
}, 5000);

/**
 * Shows the overlay on mouse move and starts the timeout to hide it, unless the overlay must stay static
 */
function handleMouseMove(): void {
  overlay.value = true;
  timeout.start();
}

onBeforeUnmount(() => {
  if (playerElement.isFullscreenVideoPlayer) {
    playbackManager.stop();
  }

  /**
   * We need to destroy JASSUB so the canvas can be recreated in the other view
   */
  playerElement.freeSsaTrack();
  playerElement.isFullscreenMounted = false;
});

onMounted(() => {
  playerElement.isFullscreenMounted = true;
});

whenever(keys.space, playbackManager.playPause);
whenever(keys.k, playbackManager.playPause);
whenever(keys.right, playbackManager.skipForward);
whenever(keys.l, playbackManager.skipForward);
whenever(keys.left, playbackManager.skipBackward);
whenever(keys.j, playbackManager.skipBackward);
whenever(keys.f, fullscreen.toggle);
whenever(keys.m, playbackManager.toggleMute);

watch(staticOverlay, (val) => {
  if (val) {
    timeout.stop();
  } else {
    timeout.start();
  }
});
</script>

<style>
.fullscreen-video-container video {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  max-width: 100%;
  max-height: 100%;
}

.fullscreen-video-container video.stretched {
  width: 100%;
  height: 100%;
}
</style>

<style scoped>
.fullscreen-video-container {
  background: black;
}

.controls-wrapper {
  position: relative;
}

.player-overlay {
  height: 100%;
}

.osd-top,
.osd-bottom {
  width: 100%;
  padding: 8px;
}

.osd-bottom > div,
.osd-top > div {
  max-width: calc(100vh * 1.77 - 2vh);
  margin: auto;
}

.osd-top {
  padding-bottom: 5em;
  background: linear-gradient(
    to bottom,
    rgb(var(--v-theme-background), 0.75) 0%,
    rgb(var(--v-theme-background), 0.74) 8.1%,
    rgb(var(--v-theme-background), 0.714) 15.5%,
    rgb(var(--v-theme-background), 0.672) 22.5%,
    rgb(var(--v-theme-background), 0.618) 29%,
    rgb(var(--v-theme-background), 0.556) 35.3%,
    rgb(var(--v-theme-background), 0.486) 41.2%,
    rgb(var(--v-theme-background), 0.412) 47.1%,
    rgb(var(--v-theme-background), 0.338) 52.9%,
    rgb(var(--v-theme-background), 0.264) 58.8%,
    rgb(var(--v-theme-background), 0.194) 64.7%,
    rgb(var(--v-theme-background), 0.132) 71%,
    rgb(var(--v-theme-background), 0.078) 77.5%,
    rgb(var(--v-theme-background), 0.036) 84.5%,
    rgb(var(--v-theme-background), 0.01) 91.9%,
    rgb(var(--v-theme-background), 0) 100%
  );
}

.osd-bottom {
  padding-top: 6em;
  background: linear-gradient(
    to top,
    rgb(var(--v-theme-background), 0.75) 0%,
    rgb(var(--v-theme-background), 0.74) 8.1%,
    rgb(var(--v-theme-background), 0.714) 15.5%,
    rgb(var(--v-theme-background), 0.672) 22.5%,
    rgb(var(--v-theme-background), 0.618) 29%,
    rgb(var(--v-theme-background), 0.556) 35.3%,
    rgb(var(--v-theme-background), 0.486) 41.2%,
    rgb(var(--v-theme-background), 0.412) 47.1%,
    rgb(var(--v-theme-background), 0.338) 52.9%,
    rgb(var(--v-theme-background), 0.264) 58.8%,
    rgb(var(--v-theme-background), 0.194) 64.7%,
    rgb(var(--v-theme-background), 0.132) 71%,
    rgb(var(--v-theme-background), 0.078) 77.5%,
    rgb(var(--v-theme-background), 0.036) 84.5%,
    rgb(var(--v-theme-background), 0.01) 91.9%,
    rgb(var(--v-theme-background), 0) 100%
  );
}

.player-controls {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
}

.video-title {
  max-width: 40vw;
  height: 6em;
}
</style>
