<template>
    <section class="bg-[#f9f9f9] py-8 px-3 md:px-8">

      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div
          class="group bg-white min-h-[100px] h-auto flex items-center gap-5 px-6 py-3 rounded shadow-sm border border-[#e0e0e0] hover:shadow-md hover:-translate-y-1 transition-all duration-300"
        >
          <div
            class="bg-[#112830]/10 text-[#112830] p-4 rounded-full text-3xl flex items-center justify-center group-hover:bg-[#10b481]/20 transition-colors"
          >
            <i class="bx bx-globe"></i>
          </div>
          <div class="text-left">
            <h3 class="text-lg font-semibold text-[#222831]">{{ t('abouttitle1') }}</h3>
            <p class="text-sm text-[#7a7a7a]">
              {{ t('abouttext1') }}
            </p>
          </div>
        </div>
  
        <div
          class="group bg-white min-h-[100px] h-auto flex items-center gap-5 px-6 py-3 rounded shadow-sm border border-[#e0e0e0] hover:shadow-md hover:-translate-y-1 transition-all duration-300"
        >
          <div
            class="bg-[#c99383]/10 text-[#c99383] p-4 rounded-full text-3xl flex items-center justify-center group-hover:bg-[#c99383]/20 transition-colors"
          >
            <i class="bx bx-badge-check"></i>
          </div>
          <div class="text-left">
            <h3 class="text-lg font-semibold text-[#222831]">{{ t('abouttitle2') }}</h3>
            <p class="text-sm text-[#7a7a7a]">
              {{ t('abouttext2') }}
            </p>
          </div>
        </div>
  
        <div
          class="group bg-white min-h-[100px] h-auto flex items-center gap-5 px-6 py-3 rounded shadow-sm border border-[#e0e0e0] hover:shadow-md hover:-translate-y-1 transition-all duration-300"
        >
          <div
            class="bg-[#6d4c41]/10 text-[#6d4c41] p-4 rounded-full text-3xl flex items-center justify-center group-hover:bg-[#6d4c41]/20 transition-colors"
          >
            <i class="bx bx-transfer-alt"></i>
          </div>
          <div class="text-left">
            <h3 class="text-lg font-semibold text-[#222831]">{{ t('abouttitle3') }}</h3>
            <p class="text-sm text-[#7a7a7a]">
              {{ t('abouttext3') }}
            </p>
          </div>
        </div>
      </div>
    </section>
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
  .group:hover i {
    transform: scale(1.15);
    transition: transform 0.3s ease;
  }
  </style>
  