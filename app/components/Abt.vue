<template>
    <section class="py-16 px-4 md:px-12 bg-[#f9f9f9]">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div
          class="relative h-[300px] rounded overflow-hidden shadow-md group cursor-pointer p-4 flex flex-col justify-end"
          style="background-image: url('/img/image1.jpg'); background-size: cover; background-position: center;"
        >
          <div class="absolute inset-0 bg-black/40 group-hover:bg-black/50 transition"></div>
          <div class="relative z-10 text-white">
            <h3 class="text-2xl sm:text-3xl font-bold mb-1">{{ t('abttitle1') }}</h3>
            <h4 class="text-sm font-medium mb-1">{{ t('abttext1') }}</h4>
          </div>
        </div>
  
        <div
          class="relative h-[300px] rounded overflow-hidden shadow-md group cursor-pointer p-4 flex flex-col justify-end"
          style="background-image: url('/img/image2.jpg'); background-size: cover; background-position: center;"
        >
          <div class="absolute inset-0 bg-black/40 group-hover:bg-black/50 transition"></div>
          <div class="relative z-10 text-white">
            <h3 class="text-2xl sm:text-3xl font-bold mb-1">{{ t('abttitle2') }}</h3>
            <h4 class="text-sm font-medium mb-1">{{ t('abttext2') }}</h4>
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
  .group:hover h3,
  .group:hover h4 {
    transform: translateY(-2px);
    transition: transform 0.3s ease;
  }
  </style>
  