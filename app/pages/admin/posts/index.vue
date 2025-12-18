<template>
  <div class="space-y-8 p-12 bg-gray-50/60 min-h-screen">
    <Breadcrumb />
    <h1 class="text-2xl font-bold mb-4">{{ t("validationPosts") }}</h1>

    <div class="grid grid-cols-1 sm:grid-cols-2 gap-6">
      <div
        v-for="post in draftPosts"
        :key="post.id"
        class="bg-white rounded shadow-sm p-6 w-full max-w-[600px] border border-gray-100"
      >
        <div class="flex justify-between items-center mb-3">
          <div class="flex items-center gap-2 mb-4">
            <div
              class="w-10 h-10 rounded-full flex items-center justify-center font-bold text-lg cursor-pointer"
              :class="getAvatarColor(post.user?.username)"
              @click="goToProfile(post.user)"
            >
              {{ post.user?.username?.charAt(0).toUpperCase() }}
            </div>
            <div class="flex flex-col">
              <span class="font-semibold text-gray-800">{{ post.user?.username }}</span>
              <div class="flex items-center gap-1 text-gray-400 text-xs">
                <span>{{ formattedDate(post.created_at) }}</span>
              </div>
            </div>
          </div>

          <span
            class="px-2 py-2 text-xs font-semibold text-white rounded-full flex items-center justify-center"
            :class="post.type_post?.type === 'Selling' ? 'bg-[#10b481]' : 'bg-[#f4a261]'"
          >
            <i :class="post.type_post?.type === 'Selling' ? 'bxr bx-cart' : 'bxr bx-badge-check'"></i>
          </span>
        </div>

        <h2 class="text-xl font-semibold text-gray-800 mb-6">{{ post.title || "No title" }}</h2>

        <div class="flex gap-6 items-stretch">
          <div class="rounded overflow-hidden cursor-pointer flex-1" @click="openModal(post)">
            <img v-if="post.image_url" :src="post.image_url" class="w-full h-40 object-cover rounded" />
            <div v-else class="w-full h-full bg-gray-200 flex items-center justify-center text-gray-500 rounded">
              No image
            </div>
          </div>

          <div class="flex-1 flex flex-col gap-2 text-gray-700 text-sm justify-between">
            <div class="flex items-center gap-2">
              <i class="bxr bx-location-alt-2 text-gray-500"></i>
              <span>{{ post.location || "Unknown" }}</span>
            </div>

            <div class="flex items-center gap-2">
              <i class="bx bx-package text-gray-500"></i>
              <span>{{ post.product?.product }}</span>
            </div>

            <div class="flex items-center gap-2">
              <i class="bx bx-package text-gray-500"></i>
              <span>{{ post.current_status }}</span>
            </div>

            <div class="flex flex-wrap gap-3 mt-3">
              <div class="flex items-center gap-2 bg-gray-50 border border-gray-100 backdrop-blur-sm text-gray-800 px-3 py-2 rounded shadow-sm text-sm">
                <i class="bx bx-wallet text-base"></i>
                <span>{{ post?.price }} {{ post?.currency?.symbol }}</span>
              </div>

              <div class="flex items-center gap-2 bg-blue-50 text-[#112830] px-4 py-2 rounded shadow-sm text-sm font-semibold">
                <i class="bx bx-package text-base"></i>
                <span>{{ post?.quantity }} {{ post?.product?.unit?.abbreviation }}</span>
              </div>
            </div>
          </div>
        </div>

        <div class="mt-6">
          <p class="text-gray-700 inline">
            {{ showFullDescription ? post.description : post.description?.slice(0, 100) + (post.description?.length > 100 ? "..." : "") }}
          </p>
          <button
            v-if="post.description?.length > 100"
            @click="toggleDescription"
            class="text-blue-600 text-sm ml-2 inline"
          >
            {{ showFullDescription ? t("viewLess") : t("viewMore") }}
          </button>
        </div>

        <div class="flex gap-2 mt-3">
          <button
            class="flex-1 bg-[#10b481] text-white px-3 py-2 rounded transition-colors"
            @click="openValidationModal(post.id, true)"
          >
            {{ t("validate") }}
          </button>
          <button
            class="flex-1 bg-red-500 text-white px-3 py-2 rounded hover:bg-red-600 transition-colors"
            @click="openValidationModal(post.id, false)"
          >
            {{ t("reject") }}
          </button>
        </div>
      </div>
    </div>
  </div>

  <div v-if="loading" class="flex justify-center mt-10">
    <div class="w-8 h-8 border-4 border-gray-300 border-t-[#10b481] rounded-full animate-spin"></div>
  </div>

  <div v-if="showValidationModal" class="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
    <div class="bg-white rounded shadow-lg w-full max-w-md p-6">
      <h3 class="text-lg font-semibold mb-4">{{ isValidAction ? t("validatePost") : t("rejectPost") }}</h3>
      <textarea
        v-model="validationMessage"
        placeholder="Message optionnel..."
        class="w-full border border-gray-300 rounded p-2 resize-none h-24 mb-4"
      ></textarea>
      <div class="flex justify-end gap-3">
        <button class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300" @click="closeValidationModal">{{ t("cancel") }}</button>
        <button class="px-4 py-2 bg-[#10b481] text-white rounded hover:bg-green-600" @click="submitValidation">{{ t("confirm") }}</button>
      </div>
    </div>
  </div>

  <transition name="fade">
    <div
      v-if="notification.visible"
      class="fixed inset-0 flex items-center justify-center z-50 bg-black/20 backdrop-blur-sm"
    >
      <div
        :class="[
          'bg-white rounded shadow-2xl px-8 py-6 flex flex-col items-center gap-4 w-[340px] text-center transition-all duration-300',
          notification.type === 'success' ? 'border-t-4 border-[#10b481]' : 'border-t-4 border-red-500',
        ]"
      >
        <div
          :class="notification.type === 'success' ? 'w-16 h-16 rounded-full bg-[#10b481] flex items-center justify-center' : 'w-16 h-16 rounded-full bg-red-500 flex items-center justify-center'"
        >
          <i :class="notification.type === 'success' ? 'bx bx-check text-4xl text-white' : 'bx bx-x text-4xl text-white'"></i>
        </div>
        <p :class="notification.type === 'success' ? 'text-[#10b481] text-lg font-semibold' : 'text-red-500 text-lg font-semibold'">
          {{ notification.message }}
        </p>
      </div>
    </div>
  </transition>
</template>

<script setup>
definePageMeta({ layout: "dashboard" });
import { ref, onMounted, computed } from "vue";
import { API_URL } from "~/utils/config";
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";
import Breadcrumb from '~/components/Breadcrumb.vue';

const languageStore = useLanguageStore();
const t = (key) => translate[languageStore.lang][key] || key;

const posts = ref([]);
const loading = ref(false);

const showFullDescription = ref(false);
function toggleDescription() { showFullDescription.value = !showFullDescription.value; }

const avatarColors = [
  "bg-red-200 text-red-700","bg-orange-200 text-orange-700","bg-amber-200 text-amber-700",
  "bg-lime-200 text-lime-700","bg-green-200 text-green-700","bg-teal-200 text-teal-700",
  "bg-cyan-200 text-cyan-700","bg-blue-200 text-blue-700","bg-indigo-200 text-indigo-700",
  "bg-purple-200 text-purple-700","bg-pink-200 text-pink-700",
];
function getAvatarColor(username) {
  let hash = 0;
  for (let i = 0; i < username.length; i++) hash = username.charCodeAt(i) + ((hash << 5) - hash);
  return avatarColors[Math.abs(hash) % avatarColors.length];
}

// const draftPosts = computed(() => posts.value);
const draftPosts = computed(() => posts.value.filter(post => post.current_status === 'brouillon'));

function formattedDate(dateStr) {
  if (!dateStr) return "tsisy";
  const date = new Date(dateStr);
  return date.toLocaleDateString();
}

const showValidationModal = ref(false);
const validationMessage = ref("");
const currentPostId = ref(null);
const isValidAction = ref(true);

function openValidationModal(postId, isValid) {
  currentPostId.value = postId;
  isValidAction.value = isValid;
  validationMessage.value = "";
  showValidationModal.value = true;
}
function closeValidationModal() { showValidationModal.value = false; validationMessage.value = ""; }

const notification = ref({ visible: false, message: "", type: "success" });
function showNotification(message, type = "success", duration = 3000) {
  notification.value = { visible: true, message, type };
  setTimeout(() => (notification.value.visible = false), duration);
}

async function submitValidation() {
  try {
    const token = localStorage.getItem("access_token");
    const response = await fetch(`${API_URL}/api/posts/${currentPostId.value}/validation_post/`, {
      method: "POST",
      headers: { "Content-Type": "application/json", Authorization: `Bearer ${token}` },
      body: JSON.stringify({ comment: validationMessage.value }),
    });
    const data = await response.json();
    if (response.ok) {
      showNotification(data.message, "success");
      await fetchPosts();
      closeValidationModal();
    } else showNotification("Erreur: " + JSON.stringify(data), "error");
  } catch (err) {
    console.error(err);
    showNotification("Erreur serveur.", "error");
  }
}

async function fetchPosts() {
  loading.value = true;
  try {
    const token = localStorage.getItem("access_token");
    const response = await fetch(`${API_URL}/api/posts/`, { headers: { Authorization: `Bearer ${token}` } });
    const data = await response.json();
    if (response.ok) posts.value = data;
    else showNotification("Erreur: impossible de récupérer les posts.", "error");
  } catch (err) {
    console.error(err);
    showNotification("Erreur serveur.", "error");
  }
  loading.value = false;
}

onMounted(fetchPosts);
</script>

<style scoped>
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>
