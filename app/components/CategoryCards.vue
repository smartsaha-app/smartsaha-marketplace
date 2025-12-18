<template>
  <div class="py-16 px-4 sm:px-6 lg:px-14 bg-[#f9f9f9]">
    <h2 class="text-2xl sm:text-3xl font-bold text-[#222831] mb-8 text-center">
      {{ t('titleCategorie') }}
    </h2>

    <div class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-3 gap-6">
      <div
        v-for="cat in categories"
        :key="cat.id"
        class="relative h-40 sm:h-48 lg:h-48 rounded overflow-hidden cursor-pointer group shadow-md hover:shadow-xl transition-shadow duration-300"
        @click="$emit('selectCategory', cat)"
      >
        <div
          class="absolute inset-0 bg-cover bg-center transition-transform duration-500 group-hover:scale-105"
          :style="{ backgroundImage: `url(${cat.image_url})` }"
        ></div>

        <div
          class="absolute inset-0 bg-black bg-opacity-40 transition-opacity duration-300 group-hover:bg-opacity-50"
        ></div>

        <div class="absolute inset-0 flex items-center justify-center">
          <h3 class="text-white text-xl sm:text-2xl lg:text-3xl font-bold text-center drop-shadow-md">
            {{ t(cat.categorie) }}
          </h3>
        </div>
      </div>
    </div>
  </div>
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
interface CategoriePost {
  id: number;
  categorie: string;
  image_url: string;
}

const categories: CategoriePost[] = [
  { id: 1, categorie: "catBeans", image_url: "/categories/beans.jpeg" },
  { id: 2, categorie: "catCereals", image_url: "/categories/cereals.jpeg" },
  {
    id: 3,
    categorie: "catVeg",
    image_url: "/categories/vegetables.jpeg",
  },
  { id: 4, categorie: "catNuts", image_url: "/categories/nuts.jpeg" },
  { id: 5, categorie: "catSpices", image_url: "/categories/spices.jpeg" },
  { id: 6, categorie: "catRoots", image_url: "/categories/roots.jpeg" },
];
</script>

<style scoped>
/* Zoom smooth sur le groupe entier */
.group:hover .group-hover\:scale-105 {
  transform: scale(1.05);
}
</style>
