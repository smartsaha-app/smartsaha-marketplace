<template>
  <section class="py-12 bg-gray-50">
    <div class="max-w-7xl mx-auto px-6">
      <div class="flex items-center justify-between mb-10 gap-4">
        <h2 class="text-2xl sm:text-3xl font-bold text-[#222831]">
          {{ t("titleFeature") }}
        </h2>

        <div class="flex items-center gap-4">
          <div class="flex gap-4">
            <button
              v-for="cat in ['All', 'Buying', 'Selling']"
              :key="cat"
              @click="filterCategory(cat)"
              :class="[
                'relative font-medium text-gray-700 hover:text-[#10b481] focus:text-[#10b481] transition-colors',
                selectedCategory === cat
                  ? 'after:absolute after:-bottom-1 after:left-0 after:w-full after:h-0.5 after:bg-[#10b481]'
                  : '',
              ]"
            >
              {{ t(cat) }}
            </button>
          </div>

          <div class="flex gap-2">
            <button
              @click="prev"
              class="bg-white border border-[#e0e0e0] p-1 px-2 rounded hover:bg-gray-100 transition"
            >
              <i class="bx bx-chevron-left text-lg"></i>
            </button>
            <button
              @click="next"
              class="bg-white border border-[#e0e0e0] p-1 px-2 rounded hover:bg-gray-100 transition"
            >
              <i class="bx bx-chevron-right text-lg"></i>
            </button>
          </div>
        </div>
      </div>

      <div class="relative overflow-x-auto scroll-smooth snap-x" ref="carousel">
        <div class="flex gap-6 flex-nowrap">
          <Card
            v-for="post in filteredPosts"
            :key="post.id"
            :post="post"
            class="flex-shrink-0 min-w-[600px] snap-start"
          />
        </div>
      </div>

      <div v-if="loading" class="text-center py-6 text-gray-500">
        Loading posts...
      </div>
      <div v-if="errorMsg" class="text-center py-6 text-red-500">
        {{ errorMsg }}
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import Card from "./Card.vue";
import { API_URL } from "~/utils/config";
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";
import { useRouter } from "vue-router";
const router = useRouter();

const languageStore = useLanguageStore();
const t = (key) => {
  const lang = languageStore.lang;
  return translate[lang][key] || key;
};

const posts = ref([]);
const loading = ref(false);
const errorMsg = ref("");

const fetchPosts = async () => {
  loading.value = true;
  errorMsg.value = "";
  try {
    const token = localStorage.getItem("access_token");
    const res = await fetch(`${API_URL}/api/posts/`);
    if (!res.ok) throw new Error("Failed to fetch posts");

    const data = await res.json();

    posts.value = data.filter(
      (post) =>
        post.can_receive_bids &&
        (post.current_status === "published" ||
          post.current_status === "négociation")
    );

    console.log("Filtered posts:", JSON.stringify(posts.value, null, 2));
  } catch (err) {
    console.error(err);
    errorMsg.value = "Error loading posts!";
  } finally {
    loading.value = false;
  }
};

const carousel = ref(null);

const getScrollAmount = () => {
  if (!carousel.value) return 624;
  const card = carousel.value.querySelector(".flex-shrink-0");
  if (!card) return 624;
  const gap = 24;
  return card.offsetWidth + gap;
};

const next = () => {
  if (carousel.value) {
    carousel.value.scrollBy({ left: getScrollAmount(), behavior: "smooth" });
  }
};

const prev = () => {
  if (carousel.value) {
    carousel.value.scrollBy({ left: -getScrollAmount(), behavior: "smooth" });
  }
};

const selectedCategory = ref("All");
const filterCategory = (cat) => (selectedCategory.value = cat);

const filteredPosts = computed(() =>
  selectedCategory.value === "All"
    ? posts.value
    : posts.value.filter(
        (post) => post.type_post?.type === selectedCategory.value
      )
);

onMounted(() => {
  fetchPosts();
});
</script>

<style scoped>
::-webkit-scrollbar {
  display: none;
}
</style>
