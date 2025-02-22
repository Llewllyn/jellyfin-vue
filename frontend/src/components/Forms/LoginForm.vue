<template>
  <div>
    <VForm
      v-model="valid"
      :disabled="loading"
      @submit.prevent="userLogin">
      <VTextField
        v-if="isEmpty(user)"
        v-model="login.username"
        variant="outlined"
        hide-details
        autofocus
        :label="$t('username')"
        :rules="rules" />
      <VTextField
        v-model="login.password"
        variant="outlined"
        hide-details
        class="mt-4"
        :label="$t('password')"
        :append-inner-icon="showPassword ? IconEyeOff : IconEye"
        :type="showPassword ? 'text' : 'password'"
        @click:append="() => (showPassword = !showPassword)" />
      <VCheckbox
        v-model="login.rememberMe"
        hide-details
        class="mt-6 mb-6"
        color="primary"
        :label="$t('rememberMe')" />
      <VRow
        align="center"
        no-gutters>
        <VCol class="mr-2">
          <VBtn
            v-if="isEmpty(user)"
            to="/server/select"
            block
            size="large"
            variant="elevated">
            {{ $t('changeServer') }}
          </VBtn>
          <VBtn
            v-else
            block
            size="large"
            variant="elevated"
            @click="$emit('change')">
            {{ $t('changeUser') }}
          </VBtn>
        </VCol>
        <VCol class="mr-2">
          <VBtn
            :disabled="!valid"
            :loading="loading"
            block
            size="large"
            color="primary"
            variant="elevated"
            type="submit">
            {{ $t('signIn') }}
          </VBtn>
        </VCol>
      </VRow>
    </VForm>
  </div>
</template>

<script setup lang="ts">
import { remote } from '@/plugins/remote';
import { userLibraries } from '@/store/userLibraries';
import type { UserDto } from '@jellyfin/sdk/lib/generated-client';
import { isEmpty } from 'lodash-es';
import IconEye from 'virtual:icons/mdi/eye';
import IconEyeOff from 'virtual:icons/mdi/eye-off';
import { ref } from 'vue';
import { useI18n } from 'vue-i18n';
import { useRouter } from 'vue-router/auto';

const props = defineProps<{ user: UserDto }>();

defineEmits<{
  change: [];
}>();

const { t } = useI18n();

const router = useRouter();

const valid = ref(false);
const login = ref({ username: '', password: '', rememberMe: true });
const showPassword = ref(false);
const loading = ref(false);
const rules = [
  (v: string): boolean | string => !!v.trim() || t('required')
];

/**
 * Login the user into the client
 */
async function userLogin(): Promise<void> {
  if (!isEmpty(props.user)) {
    /**
     * If we have a user from the public user selector, set it as login
     */
    login.value.username = props.user.Name || '';
  }

  loading.value = true;

  try {
    await remote.auth.loginUser(
      login.value.username,
      login.value.password,
      login.value.rememberMe
    );

    await userLibraries.refresh();

    await router.replace('/');
  } finally {
    loading.value = false;
  }
}
</script>
