<template>
  <div class="space-y-8 p-12 bg-gray-50/60 min-h-screen">
    <Breadcrumb />
    <div class="flex justify-between items-center">
      <h2 class="text-2xl font-bold text-gray-800">{{ t('myListings') }}</h2>
      <button
        @click="refreshPosts"
        class="flex items-center gap-2 text-gray-600 hover:text-[#10b481] transition-colors hidden"
      >
        <i class="bx bx-refresh text-xl animate-spin-once"></i>
        <span class="hidden sm:inline text-sm font-medium"></span>
      </button>

      <button
        @click="$router.push('/dashboard/post/create')"
        class="flex items-center gap-2 text-white bg-[#10b481] hover:bg-[#0e9c70] transition-colors duration-300 py-2 px-4 rounded"
      >
        <i class="bx bx-plus text-xl"></i>
        <span class="hidden sm:inline text-sm font-medium">{{ t('newListings') }}</span>
      </button>
    </div>

    <div class="flex flex-wrap gap-6 border-b">
      <button
        @click="filterCategory('All')"
        :class="[
          'relative pb-1 font-medium text-gray-700 hover:text-[#10b481] transition-colors',
          selectedType === 'All'
            ? 'after:absolute after:-bottom-0.5 after:left-0 after:w-full after:h-0.5 after:bg-[#10b481]'
            : '',
        ]"
      >
        {{ t('All') }}
      </button>
      <button
        @click="filterCategory('Buying')"
        :class="[
          'relative pb-1 font-medium text-gray-700 hover:text-[#10b481] transition-colors',
          selectedType === 'Buying'
            ? 'after:absolute after:-bottom-0.5 after:left-0 after:w-full after:h-0.5 after:bg-[#10b481]'
            : '',
        ]"
      >
        {{ t('Buying') }}
      </button>
      <button
        @click="filterCategory('Selling')"
        :class="[
          'relative pb-1 font-medium text-gray-700 hover:text-[#10b481] transition-colors',
          selectedType === 'Selling'
            ? 'after:absolute after:-bottom-0.5 after:left-0 after:w-full after:h-0.5 after:bg-[#10b481]'
            : '',
        ]"
      >
        {{ t('Selling') }}
      </button>
    </div>

    <div
      v-if="filteredPosts.length"
      class="overflow-x-auto bg-white shadow-sm rounded border border-gray-200"
    >
      <table class="min-w-full w-full">
        <thead class="bg-gray-50 border-b">
          <tr>
            <th class="px-3 py-2 text-left text-sm font-semibold text-gray-700">
              {{ t('product') }}
            </th>
            <!-- <th class="px-3 py-2 text-left text-sm font-semibold text-gray-700">
              Description
            </th> -->
            <th class="px-3 py-2 text-left text-sm font-semibold text-gray-700">
              {{ t('quantity') }}
            </th>
            <th class="px-3 py-2 text-left text-sm font-semibold text-gray-700">
              {{ t('price') }}
            </th>
            <th class="px-3 py-2 text-left text-sm font-semibold text-gray-700">
              {{ t('category') }}
            </th>
            <th class="px-3 py-2 text-left text-sm font-semibold text-gray-700">
              {{ t('date') }}
            </th>
            <th
              class="px-3 py-2 text-center text-sm font-semibold text-gray-700"
            >
              {{ t('actions') }}
            </th>
          </tr>
        </thead>

        <tbody>
          <tr
            v-for="post in paginatedPosts"
            :key="post.id"
            class="border-t transition"
          >
            <td class="px-3 py-2 flex items-center gap-3">
              <img
                v-if="post.image_url"
                :src="post.image_url"
                alt=""
                class="w-12 h-12 object-cover rounded-md border"
              />
              <span class="font-medium text-gray-800">{{
                post.product.product
              }}</span>
            </td>

            <!-- <td class="px-3 py-2 text-sm text-gray-600">
              {{ truncated(post.description) }}
            </td> -->
            <td class="px-3 py-2 text-sm text-gray-600">
              {{ post.quantity }} {{ post.product.unit.abbreviation }}
            </td>
            <td class="px-3 py-2 text-sm text-gray-600">
              {{ post.price }} {{ post.currency.symbol }}
            </td>

            <td class="px-3 py-2">
              <span
                :class="[
                  'px-3 py-1 text-xs font-semibold rounded-full border',
                  post.type_post?.type === 'Selling'
                    ? 'border-[#10b481] text-[#10b481] bg-[#10b481]/10'
                    : 'border-[#e76f51] text-[#e76f51] bg-[#f4a261]/10',
                ]"
              >
                {{ post.type_post?.type }}
              </span>
            </td>

            <td class="px-3 py-2 text-sm text-gray-500">
              {{ formatDate(post.created_at) }}
            </td>

            <td class="px-3 py-2 flex justify-center gap-4">
              <NuxtLink
                :to="`/dashboard/post/bids/${post.id}`"
                class="relative text-gray-600 hover:text-[#10b481] transition-colors"
                title="Voir les offres"
              >
                <i class="bx bx-bell text-xl"></i>

                <span
                  v-if="post.total_bids > 0"
                  class="absolute -top-2 -right-2 bg-red-500 text-white text-xs font-bold rounded-full h-5 w-5 flex items-center justify-center shadow-md"
                >
                  {{ post.total_bids }}
                </span>
              </NuxtLink>

              <button
                class="text-blue-500 hover:text-blue-700 transition-colors"
                title="Modifier"
              >
                <i class="bx bx-edit text-lg"></i>
              </button>

              <button
                class="text-red-500 hover:text-red-700 transition-colors"
                title="Supprimer"
              >
                <i class="bx bx-trash text-lg"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>

      <div class="flex justify-between items-center p-4 border-t bg-gray-50">
        <p class="text-sm text-gray-600">
          {{ t('page') }} {{ currentPage }} {{ t('sur') }} {{ totalPages }}
        </p>
        <div class="flex gap-2">
          <button
            @click="prevPage"
            :disabled="currentPage === 1"
            class="px-3 py-1 border rounded text-sm hover:bg-gray-100 disabled:opacity-40"
          >
            {{ t('prev') }}
          </button>
          <button
            @click="nextPage"
            :disabled="currentPage === totalPages"
            class="px-3 py-1 border rounded text-sm hover:bg-gray-100 disabled:opacity-40"
          >
            {{ t('next') }}
          </button>
        </div>
      </div>
    </div>

    <div v-else class="text-gray-500 italic">{{ t('noListingsFound') }}</div>
  </div>
</template>

<script setup>
definePageMeta({ layout: "dashboard" });

import { ref, computed, onMounted } from "vue";
import { formatDistanceToNow, parseISO, format } from "date-fns";
import { API_URL } from "~/utils/config";
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";
import { useRouter } from "vue-router";
import Breadcrumb from "~/components/Breadcrumb.vue";
const router = useRouter();

const languageStore = useLanguageStore();
const t = (key) => {
  const lang = languageStore.lang;
  return translate[lang][key] || key;
};

const isLoggedIn = ref(true);
const userPosts = ref([]);

const selectedType = ref("All");
const filterCategory = (category) => {
  selectedType.value = category;
};

const filteredPosts = computed(() =>
  selectedType.value === "All"
    ? userPosts.value
    : userPosts.value.filter((p) => p.type_post?.type === selectedType.value)
);

const currentPage = ref(1);
const itemsPerPage = 4;
const totalPages = computed(() =>
  Math.ceil(filteredPosts.value.length / itemsPerPage)
);

const paginatedPosts = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  return filteredPosts.value.slice(start, start + itemsPerPage);
});

const nextPage = () =>
  currentPage.value < totalPages.value && currentPage.value++;
const prevPage = () => currentPage.value > 1 && currentPage.value--;

// Fonctions utilitaires
const truncated = (text, length = 60) =>
  text.length > length ? text.slice(0, length) + "..." : text;

// ⚡ Récupérer les posts de l'utilisateur depuis l'API
const fetchUserPosts = async () => {
  try {
    const res = await fetch(`${API_URL}/api/posts/my_posts/`, {
      headers: {
        Authorization: `Bearer ${localStorage.getItem("access_token")}`,
      },
    });
    if (!res.ok) throw new Error("Erreur lors de la récupération des posts");
    const data = await res.json();
    userPosts.value = data;
    console.log("Post:", JSON.stringify(userPosts.value, null, 2));
  } catch (err) {
    console.error(err);
  }
};

// Charger les posts au montage
onMounted(() => {
  fetchUserPosts();
});

// Rafraîchir
const refreshPosts = () => fetchUserPosts();

const formatDate = (dateStr) => {
  if (!dateStr) return "Date inconnue";

  try {
    const date = parseISO(dateStr);
    const diffDays = Math.floor((new Date() - date) / (1000 * 60 * 60 * 24));

    return diffDays > 7
      ? format(date, "dd/MM/yyyy")
      : formatDistanceToNow(date, { addSuffix: true });
  } catch (e) {
    console.error("Invalid date:", dateStr);
    return "Date invalide";
  }
};
</script>

<style scoped>
tr:hover {
  background-color: rgba(16, 180, 129, 0.08);
}
.animate-spin-once {
  animation: spin 0.7s linear;
}
@keyframes spin {
  100% {
    transform: rotate(360deg);
  }
}
</style>
