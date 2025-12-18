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
    title: "Instant access to your workspace",
    description:
      "Un tableau de bord centralisé pour gérer annonces, transactions et messages en toute simplicité.",
    icon: "bx bx-grid-alt",
    tag: "Dashboard",
    color: "bg-emerald-500",
  },
  {
    key: "listing",
    title: "Flexible buying & selling",
    description:
      "Publiez des offres d’achat ou de vente en quelques clics avec un contrôle total.",
    icon: "bx bx-store",
    tag: "Marketplace",
    color: "bg-blue-500",
  },
  {
    key: "negotiation",
    title: "Smart negotiation system",
    description:
      "Discutez, négociez et concluez des deals en toute transparence.",
    icon: "bx bx-conversation",
    tag: "Bidding",
    color: "bg-purple-500",
  },
  {
    key: "trust",
    title: "Built on trust & reviews",
    description:
      "Un système d’évaluation clair pour renforcer la crédibilité de chaque utilisateur.",
    icon: "bx bx-shield-quarter",
    tag: "Reputation",
    color: "bg-rose-500",
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
