<template>
  <section
    class="h-[410px] relative flex items-center bg-center bg-cover"
    style="background-image: url('/img/hero8.png')"
  >
    <div
      class="container mx-auto px-6 lg:px-20 relative z-10 flex flex-col justify-center h-full"
    >
      <div class="max-w-xl text-left text-white">
        <h1
          class="text-3xl md:text-4xl lg:text-5xl font-extrabold mb-6 drop-shadow-lg"
        >
          {{ t('accroche') }}
        </h1>
        <p class="text-base md:text-lg lg:text-xl mb-8 drop-shadow-md">
          {{ t('textacrroche') }}
        </p>
        <div class="flex flex-col sm:flex-row gap-4">
          <button
            class="border border-white text-white hover:bg-white/20 px-6 py-3 rounded font-normal transition"
          >
            {{ t('btnStart') }}
          </button>
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
@media (max-width: 768px) {
  section {
    height: 500px;
  }
  h1 {
    font-size: 2.5rem;
  }
  p {
    font-size: 1rem;
  }
}
@media (max-width: 480px) {
  section {
    height: 450px;
  }
  h1 {
    font-size: 2rem;
  }
  p {
    font-size: 0.9rem;
  }
}
</style>
