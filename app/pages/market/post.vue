<template>
  <div class="flex bg-gray-50 min-h-screen font-sans">
    <aside
      class="fixed top-20 left-0 w-80 h-[calc(100vh-4rem)] bg-[#112830] p-6 shadow-lg flex flex-col"
    >
      <div class="flex-1 overflow-y-auto pr-2 flex flex-col gap-6">
        <div class="relative">
          <i
            class="bx bx-search absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"
          ></i>
          <input
  v-model="filters.search"
  type="text"
  :placeholder="t('search')"
  class="w-full pl-10 pr-3 py-2 rounded border border-gray-200/10 bg-gray-50/10 text-gray-300 placeholder-gray-400 shadow-sm outline-none transition"
/>

        </div>

        <div>
          <button
            @click="showType = !showType"
            class="w-full text-left text-gray-300 font-semibold flex justify-between items-center py-2 px-3 rounded hover:bg-gray-50/10 transition"
          >
            {{ t('type') }}
            <i
              :class="showType ? 'bx bx-chevron-up' : 'bx bx-chevron-down'"
            ></i>
          </button>
          <transition name="fade">
            <div v-if="showType" class="flex flex-col gap-2 mt-2 pl-2">
              <label class="flex items-center gap-2 cursor-pointer">
                <input
                  type="radio"
                  value=""
                  v-model="filters.type"
                  class="accent-[#10b481]"
                />
                <span class="text-gray-300 hover:text-[#10b481] transition"
                  >{{ t('All') }}</span
                >
              </label>
              <label class="flex items-center gap-2 cursor-pointer">
                <input
                  type="radio"
                  value="Buying"
                  v-model="filters.type"
                  class="accent-[#10b481]"
                />
                <span class="text-gray-300 hover:text-[#10b481] transition"
                  >{{ t('Buying') }}</span
                >
              </label>
              <label class="flex items-center gap-2 cursor-pointer">
                <input
                  type="radio"
                  value="Selling"
                  v-model="filters.type"
                  class="accent-[#10b481]"
                />
                <span class="text-gray-300 hover:text-[#10b481] transition"
                  >{{ t('Selling') }}</span
                >
              </label>
            </div>
          </transition>
        </div>

        <div>
          <button
            @click="showLocation = !showLocation"
            class="w-full text-left text-gray-300 font-semibold flex justify-between items-center py-2 px-3 rounded hover:bg-gray-50/10 transition"
          >
            {{ t('location') }}
            <i
              :class="showLocation ? 'bx bx-chevron-up' : 'bx bx-chevron-down'"
            ></i>
          </button>
          <transition name="fade">
            <div
              v-if="showLocation"
              class="mt-2 flex flex-col gap-1 max-h-40 pl-2"
            >
              <input
                v-model="locationSearch"
                type="text"
                placeholder="Search location..."
                class="w-full p-2 rounded border border-gray-200/10 bg-gray-50/10 text-gray-300 outline-none mb-2"
              />
              <label
                v-for="loc in filteredLocations"
                :key="loc"
                class="flex items-center gap-2 px-2 py-1 rounded cursor-pointer hover:bg-gray-50/10 transition text-gray-300"
              >
                <input
                  type="checkbox"
                  :value="loc"
                  v-model="filters.locations"
                  class="accent-[#10b481]"
                />
                {{ loc }}
              </label>
            </div>
          </transition>
        </div>

        <div>
          <button
            @click="showPrice = !showPrice"
            class="w-full text-left text-gray-300 font-semibold flex justify-between items-center py-2 px-3 rounded hover:bg-gray-50/10 transition"
          >
            {{ t('priceRange') }}
            <i
              :class="showPrice ? 'bx bx-chevron-up' : 'bx bx-chevron-down'"
            ></i>
          </button>
          <transition name="fade">
            <div v-if="showPrice" class="flex gap-2 mt-2 pl-2">
              <input
                v-model.number="filters.minPrice"
                type="number"
                placeholder="Min"
                class="w-1/2 p-2 rounded border-gray-200/10 bg-gray-50/10 text-gray-300 outline-none"
              />
              <input
                v-model.number="filters.maxPrice"
                type="number"
                placeholder="Max"
                class="w-1/2 p-2 rounded border-gray-200/10 bg-gray-50/10 text-gray-300 outline-none"
              />
            </div>
          </transition>
        </div>
      </div>

      <button
        @click="applyFilters"
        class="mt-4 w-full bg-[#10b481] text-white py-2 rounded font-semibold hover:brightness-110 transition shadow-md flex-shrink-0"
      >
        {{ t('apply')}}
      </button>
    </aside>

    <main class="flex-1 p-6 ml-[410px] flex flex-col gap-6">
      <div v-if="loading" class="flex justify-center mt-10">
  <div class="w-8 h-8 border-4 border-white/20 border-t-[#10b481] rounded-full animate-spin"></div>
</div>

      <div
        v-else-if="filteredPosts.length === 0"
        class="text-gray-500 text-center mt-10"
      >
        No posts found.
      </div>
      <div v-else class="flex flex-col gap-6">
        <PostCard
          v-for="post in filteredPosts"
          :key="post.id"
          :post="post"
          class="w-full"
        />
      </div>
    </main>
  </div>
</template>

<script setup>
definePageMeta({
  layout: "market",
});
import { ref, computed, onMounted } from "vue";
import PostCard from "~/components/PostCard.vue";
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

const showType = ref(false);
const showLocation = ref(false);
const showPrice = ref(false);

const filters = ref({
  search: "",
  type: "",
  locations: [],
  minPrice: null,
  maxPrice: null,
});

const locationSearch = ref("");
const posts = ref([]);
const loading = ref(false);

const activeBtn =
  "bg-[#10b481] text-white px-3 py-1 rounded font-semibold shadow";
const inactiveBtn =
  "bg-gray-100 text-gray-700 px-3 py-1 rounded hover:bg-gray-200 transition shadow-sm";

const filteredLocations = computed(() => {
  const allLocations = posts.value.map((p) => p.location);
  const unique = [...new Set(allLocations)];
  if (!locationSearch.value) return unique;
  return unique.filter((loc) =>
    loc.toLowerCase().includes(locationSearch.value.toLowerCase())
  );
});

const filteredPosts = computed(() => {
  return posts.value.filter((post) => {
    const matchSearch = filters.value.search
      ? post.product.product
          .toLowerCase()
          .includes(filters.value.search.toLowerCase())
      : true;
    const matchType = filters.value.type
      ? post.type_post.type === filters.value.type
      : true;
    const matchLocation = filters.value.locations.length
      ? filters.value.locations.includes(post.location)
      : true;
    const matchMin =
      filters.value.minPrice != null
        ? post.price >= filters.value.minPrice
        : true;
    const matchMax =
      filters.value.maxPrice != null
        ? post.price <= filters.value.maxPrice
        : true;
    return matchSearch && matchType && matchLocation && matchMin && matchMax;
  });
});

// const fetchPosts = async () => {
//   loading.value = true;
//   try {
//     const res = await fetch(`${API_URL}/api/posts/`);
//     if (!res.ok) throw new Error("Failed to fetch posts");
//     posts.value = await res.json();
//   } catch (err) {
//     console.error(err);
//   } finally {
//     loading.value = false;
//   }
// };

const fetchPosts = async () => {
  loading.value = true;
  // errorMsg.value = "";
  try {
    const token = localStorage.getItem("access_token");
    const res = await fetch(`${API_URL}/api/posts/`);
    if (!res.ok) throw new Error("Failed to fetch posts");

    const data = await res.json();

    // Filtrer les posts
    posts.value = data.filter(
      (post) =>
        post.can_receive_bids &&
        (post.current_status === "published" ||
          post.current_status === "négociation")
    );

    console.log("Filtered posts:", JSON.stringify(posts.value, null, 2));
  } catch (err) {
    console.error(err);
    // errorMsg.value = "Error loading posts!";
  } finally {
    loading.value = false;
  }
};

const applyFilters = () => {};

onMounted(() => {
  fetchPosts();
});
</script>

<style scoped>
::-webkit-scrollbar {
  display: none;
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.2s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-5px);
}
</style>
