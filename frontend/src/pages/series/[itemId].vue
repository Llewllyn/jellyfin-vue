<template>
  <ItemCols>
    <template #left>
      <VRow
        justify="center"
        justify-md="start">
        <VCol
          cols="7"
          md="3">
          <Card :item="item" />
        </VCol>
        <VCol
          cols="12"
          md="9">
          <h1
            class="text-h4 font-weight-light"
            :class="{ 'text-center': !$vuetify.display.mdAndUp }">
            {{ item.Name }}
          </h1>
          <h2
            v-if="item.OriginalTitle && item.OriginalTitle !== item.Name"
            class="text-subtitle-1"
            :class="{ 'text-center': !$vuetify.display.mdAndUp }">
            {{ item.OriginalTitle }}
          </h2>
          <div
            class="text-caption text-h4 font-weight-medium mt-2"
            :class="{ 'text-center': !$vuetify.display.mdAndUp }">
            <MediaInfo
              :item="item"
              year
              runtime
              rating
              ends-at />
          </div>
          <VRow
            class="my-4 align-center"
            :class="{
              'justify-center': !$vuetify.display.mdAndUp,
              'ml-0': $vuetify.display.mdAndUp
            }">
            <PlayButton
              class="mr-2"
              :item="item" />
            <LikeButton
              :item="item"
              class="mr-2" />
            <MarkPlayedButton
              :item="item"
              class="mr-2" />
            <ItemMenu :item="item" />
          </VRow>
          <VCol
            cols="12"
            md="10">
            <VRow
              v-if="item && item.GenreItems && item.GenreItems.length > 0"
              align="center">
              <VCol
                :cols="12"
                :sm="2"
                class="px-0 text-truncate">
                <label class="text--secondary">{{ $t('genres') }}</label>
              </VCol>
              <VCol
                class="px-0"
                :cols="12"
                :sm="10">
                <VSlideGroup>
                  <VSlideGroupItem
                    v-for="(genre, index) in item.GenreItems"
                    :key="`genre-${genre.Id}`">
                    <VChip
                      size="small"
                      link
                      :class="{ 'ml-2': index > 0 }"
                      :to="`/genre/${genre.Id}?type=${item.Type}`">
                      {{ genre.Name }}
                    </VChip>
                  </VSlideGroupItem>
                </VSlideGroup>
              </VCol>
            </VRow>
            <VRow
              v-if="item && directors.length > 0 && !$vuetify.display.smAndUp"
              align="center">
              <VCol
                :cols="12"
                :sm="2"
                class="mt-sm-3 py-sm-0 px-0 text-truncate">
                <label class="text--secondary">{{ $t('directing') }}</label>
              </VCol>
              <VCol
                :cols="12"
                :sm="10">
                <VRow dense>
                  <VCol
                    v-for="director in directors"
                    :key="director.Id"
                    cols="auto">
                    <VChip
                      size="small"
                      link
                      :to="getItemDetailsLink(director, 'Person')">
                      {{ director.Name }}
                    </VChip>
                  </VCol>
                </VRow>
              </VCol>
            </VRow>
            <VRow
              v-if="item && writers.length > 0 && !$vuetify.display.smAndUp"
              align="center">
              <VCol
                :cols="12"
                :sm="2"
                class="mt-sm-3 py-sm-0 px-0 text-truncate">
                <label class="text--secondary">{{ $t('writing') }}</label>
              </VCol>
              <VCol
                :cols="12"
                :sm="10">
                <VRow dense>
                  <VCol
                    v-for="writer in writers"
                    :key="writer.Id"
                    cols="auto">
                    <VChip
                      size="small"
                      link
                      :to="getItemDetailsLink(writer, 'Person')">
                      {{ writer.Name }}
                    </VChip>
                  </VCol>
                </VRow>
              </VCol>
            </VRow>
          </VCol>
          <div>
            <p
              v-if="item.Taglines && item.Taglines.length > 0"
              class="text-subtitle-1 text-truncate">
              {{ item.Taglines[0] }}
            </p>
            <!-- eslint-disable vue/no-v-html -
              Output is properly sanitized using sanitizeHtml -->
            <p
              v-if="item.Overview"
              class="item-overview"
              v-html="sanitizeHtml(item.Overview, true)" />
            <!-- eslint-enable vue/no-v-html -->
          </div>
        </VCol>
      </VRow>
      <VRow>
        <VCol cols="12">
          <SeasonTabs
            v-if="item.Type === 'Series'"
            :item="item" />
        </VCol>
      </VRow>
    </template>
    <template #right>
      <div v-if="crew.length > 0">
        <h2 class="text-h6 text-sm-h5">
          {{ $t('crew') }}
        </h2>
        <PeopleList :items="crew" />
      </div>
      <div v-if="actors.length > 0">
        <h2 class="text-h6 text-sm-h5">
          {{ $t('cast') }}
        </h2>
        <PeopleList :items="actors" />
      </div>
      <RelatedItems
        :item="item"
        vertical />
    </template>
  </ItemCols>
</template>

<script setup lang="ts">
import { remote } from '@/plugins/remote';
import { sanitizeHtml } from '@/utils/html';
import { getBlurhash } from '@/utils/images';
import { getItemDetailsLink } from '@/utils/items';
import {
  type BaseItemDto,
  type BaseItemPerson,
  ImageType
} from '@jellyfin/sdk/lib/generated-client';
import { getUserLibraryApi } from '@jellyfin/sdk/lib/utils/api/user-library-api';
import { computed, onMounted, ref } from 'vue';
import { useRoute } from 'vue-router/auto';

const route = useRoute<'/series/[itemId]'>();

const item = ref<BaseItemDto>({});

const crew = computed<BaseItemPerson[]>(() =>
  (item.value.People ?? []).filter((person) =>
    ['Director', 'Writer'].includes(person?.Type ?? '')
  )
);

const actors = computed<BaseItemPerson[]>(() =>
  (item.value.People ?? [])
    .filter((person) => person.Type === 'Actor')
    .slice(0, 10)
);

const directors = computed(() =>
  crew.value.filter((person) => person.Type === 'Director')
);

const writers = computed(() =>
  crew.value.filter((person) => person.Type === 'Writer')
);

onMounted(async () => {
  const { itemId } = route.params;

  item.value = (
    await remote.sdk.newUserApi(getUserLibraryApi).getItem({
      userId: remote.auth.currentUserId ?? '',
      itemId
    })
  ).data;

  route.meta.title = item.value.Name;
  route.meta.backdrop.blurhash = getBlurhash(item.value, ImageType.Backdrop);
});
</script>
