<template>
  <footer class="bg-gradient-to-r from-[#112830] to-[#112830] text-white py-16 px-6 md:px-12">
    <div class="max-w-7xl mx-auto grid grid-cols-1 md:grid-cols-3 gap-12">
      <!-- Logo + Mission -->
      <div class="flex flex-col gap-6">
        <NuxtLink to="/" class="flex items-center gap-3">
          <img src="/marketplace.png" alt="Logo" class="h-12 w-auto rounded-lg shadow-md" />
          <h1 class="text-2xl font-bold text-white">AgriTrade</h1>
        </NuxtLink>
        <p class="text-gray-400 text-sm">
          {{ t('FtText') }}
        </p>
        <div class="flex gap-4 mt-2">
          <a href="#" class="hover:text-[#10b481] transition-all duration-200 text-2xl"><i class="bx bxl-facebook"></i></a>
          <a href="#" class="hover:text-[#10b481] transition-all duration-200 text-2xl"><i class="bx bxl-twitter"></i></a>
          <a href="#" class="hover:text-[#10b481] transition-all duration-200 text-2xl"><i class="bx bxl-instagram"></i></a>
          <a href="#" class="hover:text-[#10b481] transition-all duration-200 text-2xl"><i class="bx bxl-linkedin"></i></a>
        </div>
      </div>

      <div class="transition-all duration-300">
        <h3 class="font-bold text-lg mb-4 border-b border-gray-700 pb-2">{{ t('quickLinks') }}</h3>
        <ul class="space-y-3">
          <li><a href="#" class="hover:text-[#10b481] transition-all duration-200">{{ t('home') }}</a></li>
          <li><NuxtLink to="/market/about" class="hover:text-[#10b481] transition-all duration-200">{{ t('about') }}</NuxtLink></li>
          <li><a href="#" class="hover:text-[#10b481] transition-all duration-200">{{ t('contact') }}</a></li>
          <li><a href="#" class="hover:text-[#10b481] transition-all duration-200">{{ t('faq') }}</a></li>
        </ul>
      </div>

      <div class="transition-all duration-300">

        <h3 class="font-bold text-lg mb-4 border-b border-gray-700 pb-2">{{ t('resources') }}</h3>
        <ul class="space-y-3">
          <li><a href="https://app.smart-saha.com" class="hover:text-[#10b481] transition-all duration-200">{{ t('pa') }}</a></li>
          <li><a href="#" class="hover:text-[#10b481] transition-all duration-200">{{ t('shipping') }}</a></li>
          <li><NuxtLink to="/market/legal?tab=privacy" :class="activeTab === 'privacy' ? 'text-[#10b481] border-b-2 border-[#10b481] pb-1' : 'hover:text-[#10b481] transition-all duration-200'">{{ t('privacyPolicy') }}</NuxtLink></li>
          <li><NuxtLink to="/market/legal?tab=terms" :class="activeTab === 'terms' ? 'text-[#10b481] border-b-2 border-[#10b481] pb-1' : 'hover:text-[#10b481] transition-all duration-200'">{{ t('termsOfUse') }}</NuxtLink></li>
        </ul>
      </div>
    </div>

    <div class="mt-12 border-t border-gray-700 pt-6 text-center text-gray-500 text-sm">
      &copy; {{ t('copy') }}
    </div>
  </footer>
</template>


<script setup lang="ts">
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";
import { useRouter } from "vue-router";
const router = useRouter();

const languageStore = useLanguageStore();
const t = (key: string) => {
  const lang = languageStore.lang;
  return translate[lang][key] || key;
};
</script>

<style scoped>
a i {
  transition: transform 0.2s ease;
}
a i:hover {
  transform: scale(1.2);
}
</style>
