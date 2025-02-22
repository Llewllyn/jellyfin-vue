<template>
  <SettingsPage page-title="newUser">
    <template #actions>
      <VBtn
        color="primary"
        variant="elevated"
        class="ml-a"
        @click="$router.push('/settings/users/new')">
        {{ t('newUser') }}
      </VBtn>
      <VBtn
        variant="elevated"
        href="https://jellyfin.org/docs/general/server/users/adding-managing-users"
        rel="noreferrer noopener"
        target="_blank">
        {{ t('help') }}
      </VBtn>
    </template>
    <template #content>
      <VContainer>
        <VRow class="space">
          <VCard
            v-for="user in users"
            :key="user.Id"
            class="pa-2"
            @click="$router.push(`/settings/users/${user['Id']}`)">
            <VRow>
              <VCol>
                <UserImage
                  :user="user"
                  :size="48" />
              </VCol>
              <VCol>
                <VCardTitle class="pa-0 fixed-width">
                  {{ user.Name }}
                </VCardTitle>
                <VCardSubtitle
                  v-if="user.LastActivityDate"
                  class="pa-0 fixed-width">
                  {{
                    $t('lastActivityDate', {
                      value: useDateFns(formatDistanceToNow, new Date(user.LastActivityDate), { addSuffix: true })
                    })
                  }}
                </VCardSubtitle>
              </VCol>
            </VRow>
          </VCard>
        </VRow>
      </VContainer>
    </template>
  </SettingsPage>
</template>

<route lang="yaml">
meta:
  admin: true
</route>

<script setup lang="ts">
import { useDateFns } from '@/composables/use-datefns';
import { remote } from '@/plugins/remote';
import { getUserApi } from '@jellyfin/sdk/lib/utils/api/user-api';
import { formatDistanceToNow } from 'date-fns';
import { ref } from 'vue';
import { useI18n } from 'vue-i18n';

const { t } = useI18n();

const users = ref(
  (await remote.sdk.newUserApi(getUserApi).getUsers()).data ?? []
);
</script>

<style scoped>
.space {
  gap: 0.5rem;
}

.fixed-width {
  width: 200px;
  text-overflow: ellipsis;
}
</style>
