<template>
  <span ref="container">
    <template
      v-for="(item, index) of playbackManager.queue"
      :key="item.Id">
      <VHover v-slot="{ isHovering, props: hoverProps }">
        <VListItem
          v-bind="hoverProps"
          :title="item.Name ?? ''"
          :subtitle="getArtists(item)"
          class="grab-cursor"
          :class="{ 'text-primary font-weight-bold': isPlaying(index) }"
          @click="playbackManager.currentItemIndex = index">
          <template #prepend>
            <VListItemAction
              :key="index"
              start>
              <VIcon>
                <template v-if="!isHovering">
                  {{ index + 1 }}
                </template>
                <IMdiDragHorizontal v-else />
              </VIcon>
            </VListItemAction>
            <VAvatar>
              <BlurhashImage :item="item" />
            </VAvatar>
          </template>
          <template #append>
            <LikeButton
              v-hide="isPlaying(index)"
              :item="item" />
            <ItemMenu
              v-hide="isPlaying(index)"
              :item="item"
              queue />
          </template>
        </VListItem>
      </VHover>
    </template>
  </span>
</template>

<script setup lang="ts">
import { playbackManager } from '@/store/playbackManager';
import type { BaseItemDto } from '@jellyfin/sdk/lib/generated-client';
import Sortable from 'sortablejs';
import { onBeforeUnmount, shallowRef, watch } from 'vue';

let sortable: Sortable | undefined;
const container = shallowRef<HTMLSpanElement>();

/**
 * Destroys the sortable instance
 */
function destroy(): void {
  if (sortable) {
    sortable.destroy();
    sortable = undefined;
  }
}

/**
 * Checks if the item in the current position is playing
 */
function isPlaying(index: number): boolean {
  return index === playbackManager.currentItemIndex;
}

/**
 * Gets the artists of the item
 */
function getArtists(item: BaseItemDto): string | undefined {
  return item.Artists ? item.Artists.join(', ') : undefined;
}

watch(container, () => {
  destroy();

  if (container.value) {
    sortable = new Sortable(container.value, {
      animation: 500,
      delay: 0,
      dragoverBubble: true,
      onUpdate(e): void {
        const oldIndex = e.oldIndex;

        if (typeof oldIndex === 'number') {
          const item = playbackManager.queue[oldIndex];

          if (item?.Id && typeof e.newIndex === 'number') {
            playbackManager.changeItemPosition(item.Id, e.newIndex);
          }
        }
      }
    });
  }
});

onBeforeUnmount(destroy);
</script>

<style lang="scss" scoped>
.grab-cursor {
  cursor: grab;
}
</style>
