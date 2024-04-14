<script setup lang="ts">
  import { navigateTo, useNuxtApp, onMounted, useCookie, ref } from '#imports';

  const { $trpc } = useNuxtApp();

  const data = ref<any>(null);

  onMounted(async () => {
    const {
      defaultOrgShortcode,
      twoFactorEnabledCorrectly,
      userHasOrgMembership,
      username
    } = await $trpc.account.defaults.redirectionData.query({});

    const usernameCookie = useCookie('un-join-username', {
      maxAge: 3600
    });

    data.value = twoFactorEnabledCorrectly;

    if (!userHasOrgMembership) {
      if (!twoFactorEnabledCorrectly) {
        if (
          process.client && // running in client
          !usernameCookie.value && // doesn't have a username cookie
          username && // has username set
          Boolean(defaultOrgShortcode) && // has a default org
          !twoFactorEnabledCorrectly // 2fa is not enabled correctly
        ) {
          usernameCookie.value = username;
        }
        return navigateTo(`/join/2fa`);
      }
      return navigateTo(`/join/org`);
    }

    if (!twoFactorEnabledCorrectly) {
      return navigateTo(
        `/${defaultOrgShortcode}/settings/user/security?2fa=error`
      );
    }

    // We need to redirect to the index page of [orgShortcode] due to this nuxt issue https://github.com/nuxt/nuxt/issues/25214
    // the index page will reload nuxt, then redirect to the convos view - we should not directly navigate to convos page!!!
    setTimeout(() => {
      navigateTo(`/${defaultOrgShortcode}`);
    }, 500);
  });
</script>

<template>
  <div class="flex h-full w-full items-center justify-center">
    <span class="font-display text-3xl">Redirecting...</span>
    {{ data }}
  </div>
</template>
