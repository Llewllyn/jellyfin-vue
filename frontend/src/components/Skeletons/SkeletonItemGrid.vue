<template>
  <VRow>
    <VCol
      cols="12"
      :class="useResponsiveClasses('card-grid-container')">
      <SkeletonCard
        v-for="n in 24"
        :key="n"
        :card-shape="skeletonCardShape"
        text />
    </VCol>
  </VRow>
</template>

<script setup lang="ts">
import { useResponsiveClasses } from '@/composables/use-responsive-classes';
import { CardShapes, getShapeFromItemType } from '@/utils/items';
import type { BaseItemKind } from '@jellyfin/sdk/lib/generated-client';
import { computed } from 'vue';

const props = withDefaults(defineProps<{ viewType?: BaseItemKind }>(), {
  viewType: 'Movie'
});

const skeletonCardShape = computed(() => {
  return getShapeFromItemType(props.viewType) || CardShapes.Portrait;
});
</script>

<style lang="scss" scoped>
.card-grid-container {
  display: grid;
}

.card-grid-container.sm-and-down {
  grid-template-columns: repeat(3, minmax(calc(100% / 3), 1fr));
}

.card-grid-container.sm-and-up {
  grid-template-columns: repeat(4, minmax(calc(100% / 4), 1fr));
}

.card-grid-container.lg-and-up {
  grid-template-columns: repeat(6, minmax(calc(100% / 6), 1fr));
}

.card-grid-container.xl {
  grid-template-columns: repeat(8, minmax(calc(100% / 8), 1fr));
}
</style>
