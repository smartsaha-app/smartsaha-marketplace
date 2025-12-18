<template>
  <section class="bg-gray-50 py-12 md:py-28">
    <div class="max-w-7xl mx-auto px-4">
      <div class="max-w-3xl mx-auto text-center mb-16">
        <h2 class="text-2xl sm:text-3xl font-bold tracking-tight text-gray-900">
          {{ t('howItWork') }}
        </h2>
      </div>

      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
        <article
          v-for="feature in features"
          :key="feature.key"
          class="relative bg-white rounded border border-gray-200 p-6 transition hover:border-gray-300 hover:shadow-sm"
        >
          <div
            class="w-12 h-12 rounded flex items-center justify-center text-white mb-5"
            :class="feature.color"
          >
            <i :class="feature.icon" class="text-xl"></i>
          </div>

          <h3 class="text-base font-semibold text-gray-900 mb-2">
            {{ feature.title }}
          </h3>

          <p class="text-sm text-gray-600 leading-relaxed">
            {{ feature.description }}
          </p>
        </article>
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

const features = [
  {
    key: "access",
    title: t('featuresTitle1'),
    description: t('featuresText1'),
    icon: "bx bx-grid-alt",
    tag: "Dashboard",
    color: "bg-[#10b481]",
  },
  {
    key: "listing",
    title: t('featuresTitle2'),
    description: t('featuresText2'),
    icon: "bx bx-store",
    tag: "Marketplace",
    color: "bg-[#219ebc]",
  },
  {
    key: "negotiation",
    title: t('featuresTitle3'),
    description: t('featuresText3'),
    icon: "bx bx-conversation",
    tag: "Bidding",
    color: "bg-[#f4a261]",
  },
  {
    key: "trust",
    title: t('featuresTitle4'),
    description: t('featuresText4'),
    icon: "bx bx-shield-quarter",
    tag: "Reputation",
    color: "bg-[##5fd4a2]",
  },
];
</script>

<style scoped>
div.flex-shrink-0:hover {
  transform: scale(1.05);
  transition: all 0.3s ease-in-out;
}

@media (hover: hover) {
  .shadow-md:hover {
    transform: scale(1.03);
  }
}
</style>
