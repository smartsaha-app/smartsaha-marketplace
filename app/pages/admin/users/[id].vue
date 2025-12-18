<template>
  <div class="space-y-8 p-12 bg-gray-50/60 min-h-screen">
    <Breadcrumb />
    <div v-if="!user.id" class="text-gray-500">
      <div
        class="w-8 h-8 border-4 border-gray-300 border-t-[#10b481] rounded-full animate-spin"
      ></div>
    </div>
    <div class="flex items-center gap-4 mb-10">
      <div
        class="w-24 h-24 rounded-full flex items-center justify-center text-3xl font-bold shadow-md"
        :class="getAvatarColor(user.username)"
      >
        {{ user.username.charAt(0).toUpperCase() }}
      </div>

      <div>
        <h1 class="text-3xl font-bold text-gray-900">{{ user.username }}</h1>
        <p class="text-gray-600">{{ user.email }}</p>

        <div class="mt-1 flex items-center gap-8">
          <p
            class="text-sm flex items-center gap-1"
            :class="user.is_verified ? 'text-[#10b481]' : 'text-red-600'"
          >
            <i
              :class="user.is_verified ? 'bx bx-badge-check' : 'bx bx-error'"
            ></i>
            {{
              user.is_verified ? t("verifiedAccount") : t("notverifiedAccount")
            }}
          </p>

          <button
            v-if="!user.is_verified"
            class="px-4 py-1.5 rounded bg-[#10b481] text-white text-sm font-medium hover:bg-[#0e9a72] transition"
            @click="verifyUser(user.id)"
          >
            {{ t("verify") }}
          </button>
        </div>
      </div>
    </div>

    <div class="mb-12">
      <div
        class="flex flex-col md:flex-row md:items-center md:justify-between gap-4 mb-8"
      >
        <div>
          <h3 class="text-lg font-semibold text-gray-900">
            {{ t("userReview") }}
          </h3>
          <div class="flex items-center gap-1 mt-2">
            <template v-for="star in 5" :key="star">
              <i
                class="bxr text-xl"
                :class="
                  star <= Math.round(user.rating_avg || 0)
                    ? 'bxs-star text-yellow-400'
                    : 'bxs-star bx-star-empty text-gray-300'
                "
              ></i>
            </template>
            <span class="text-gray-500 text-sm ml-2">
              ({{ (user.rating_avg || 0).toFixed(1) }})
            </span>
          </div>
        </div>

        <button
          @click="toggleSlideReport"
          class="inline-flex items-center gap-2 px-4 py-2 rounded bg-red-50 text-red-600 text-sm font-medium hover:bg-red-100 transition"
        >
          {{ t("viewReports") }}
          <i class="bxr bx-arrow-right-stroke-circle text-xl"></i>
        </button>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-4 gap-6">
        <div
          class="relative overflow-hidden rounded bg-gradient-to-br from-emerald-500 to-emerald-600 p-5 text-white"
        >
          <i
            class="bx bx-category absolute -right-4 -top-4 text-7xl opacity-10"
          ></i>
          <p
            class="opacity-80 uppercase tracking-wide text-xs font-semibold mb-1"
          >
            {{ t("category") }}
          </p>
          <p class="text-lg font-semibold">
            {{ user.id_categorie_user?.categorie }}
          </p>
        </div>

        <div
          class="relative overflow-hidden rounded p-5 text-white"
          :class="
            user.is_active
              ? 'bg-gradient-to-br from-green-500 to-green-600'
              : 'bg-gradient-to-br from-red-500 to-red-600'
          "
        >
          <i
            class="bx bx-shield absolute -right-4 -top-4 text-7xl opacity-10"
          ></i>
          <p
            class="opacity-80 uppercase tracking-wide text-xs font-semibold mb-1"
          >
            {{ t("status") }}
          </p>
          <p class="text-lg font-semibold">
            {{ user.is_active ? t("actif") : t("inactif") }}
          </p>
        </div>

        <div
          class="relative overflow-hidden rounded bg-gray-900 p-5 text-white"
        >
          <i
            class="bx bx-news absolute -right-4 -top-4 text-7xl opacity-10"
          ></i>
          <p
            class="opacity-80 uppercase tracking-wide text-xs font-semibold mb-1"
          >
            {{ t("annonces") }}
          </p>
          <p class="text-3xl font-bold">
            {{ posts.length }}
          </p>
        </div>

        <div
          class="relative overflow-hidden rounded bg-gradient-to-br from-red-600 to-red-700 p-5 text-white"
        >
          <i
            class="bx bx-error-circle absolute -right-4 -top-4 text-7xl opacity-10"
          ></i>
          <p
            class="opacity-80 uppercase tracking-wide text-xs font-semibold mb-1"
          >
            {{ t("reports") }}
          </p>
          <p class="text-3xl font-bold">
            {{ totalReports }}
          </p>
        </div>
      </div>

      <div
        class="mt-8 border-t pt-4 flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3 text-sm text-gray-500"
      >
        <span>
          {{ t("justificatif") }} :
          <strong class="text-gray-800">
            {{ user.justificatif_url ? t("available") : t("notAvailable") }}
          </strong>
        </span>

        <a
          v-if="user.justificatif_url"
          :href="user.justificatif_url"
          target="_blank"
          class="text-[#10b481] font-medium hover:underline"
        >
          {{ t("viewDoc") }}
        </a>
      </div>
    </div>

    <h2 class="text-2xl font-semibold text-gray-900 mb-4">
      {{ t("annonces") }}
    </h2>

    <div v-if="posts.length === 0" class="text-gray-500">
      {{ t("noAnnoncesFound") }}
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <Card
        v-for="post in posts"
        :key="post.id"
        :post="post"
        class="flex-shrink-0 w-80"
      />
    </div>
  </div>

  <transition name="fade">
    <div
      v-if="slideReportOpen"
      class="fixed inset-0 z-30 bg-black/40 backdrop-blur-sm"
      @click="toggleSlideReport"
    ></div>
  </transition>

  <transition name="slide-right">
    <div
      v-if="slideReportOpen"
      class="fixed top-20 right-0 w-[380px] h-screen bg-white z-40 flex flex-col shadow-xl"
    >
      <div class="flex items-center justify-between px-5 py-4 border-b">
        <div class="flex items-center gap-2">
          <h2 class="text-lg font-semibold text-gray-800">
            {{ t("reportstitle") }}
          </h2>
        </div>

        <button
          @click="toggleSlideReport"
          class="w-8 h-8 flex items-center justify-center rounded-full hover:bg-gray-200 text-gray-500 transition"
        >
          ✕
        </button>
      </div>

      <div class="flex-1 overflow-y-auto px-5 py-4 space-y-4">
        <div
          v-for="group in groupedReports"
          :key="group.postId"
          class="bg-white rounded border border-gray-100 p-4 hover:shadow-sm transition"
        >
          <h3 class="text-sm font-semibold text-gray-900 mb-1 line-clamp-2">
            {{ getPostById(group.postId)?.title || `Post #${group.postId}` }}
          </h3>

          <p class="text-xs text-gray-500 mb-3">
            {{ group.reports.length }} report(s)
          </p>

          <button
            v-if="group.reports.some((r) => r.status === 'pending')"
            @click="openConfirmModal(group.postId)"
            class="w-full mb-4 px-3 py-2 rounded bg-red-500/90 text-white text-sm font-medium hover:bg-red-600 transition"
          >
            {{ t("approveReports") }}
          </button>

          <ul class="space-y-2">
            <li
              v-for="r in group.reports"
              :key="r.id"
              class="flex items-center justify-between text-sm"
            >
              <div class="flex items-center gap-2">
                <div
                  class="w-7 h-7 rounded-full flex items-center justify-center text-xs font-semibold shadow"
                  :class="getAvatarColor(getUserById(r.id_user)?.username)"
                >
                  {{ getUserById(r.id_user)?.username.charAt(0).toUpperCase() }}
                </div>

                <span class="text-gray-700 truncate max-w-[170px]">
                  {{ r.reason }}
                </span>
              </div>

              <span
                class="text-xs px-2 py-1 rounded-full justify-center items-center"
                :class="
                  r.status === 'pending'
                    ? 'bg-yellow-100 text-yellow-700'
                    : 'bg-green-100 text-green-700'
                "
              >
                {{ r.status }}
              </span>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </transition>

  <transition name="fade">
    <div
      v-if="confirmModalOpen"
      class="fixed inset-0 z-50 flex items-center justify-center bg-black/40"
    >
      <div class="bg-white rounded shadow-xl w-full max-w-md p-6">
        <h3 class="text-xl font-semibold text-gray-900 mb-3">
          {{ t("confirmDelete") }}
        </h3>

        <p class="text-gray-600 mb-6" v-html="t('confirmDeleteReport')"></p>

        <div class="flex justify-end gap-3">
          <button
            @click="closeConfirmModal"
            class="px-4 py-2 rounded border text-gray-600 hover:bg-gray-100 transition"
          >
            {{ t("cancel") }}
          </button>

          <button
            @click="confirmApprove"
            class="px-4 py-2 rounded bg-red-600 text-white hover:bg-red-700 transition"
          >
            {{ t("confirm") }}
          </button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script setup lang="ts">
definePageMeta({ layout: "dashboard" });

import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";
import { API_URL } from "~/utils/config";
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";
import { useRouter } from "vue-router";
import Breadcrumb from "~/components/Breadcrumb.vue";
const router = useRouter();

const languageStore = useLanguageStore();
const t = (key: string) => {
  const lang = languageStore.lang;
  return translate[lang][key] || key;
};

const route = useRoute();

const user = ref<any>({
  username: "",
  email: "",
  id_categorie_user: { categorie: "" },
  justificatif_url: "",
  is_verified: false,
  is_active: false,
  id: null,
});

const avatarColors = [
  "bg-red-200 text-red-700",
  "bg-orange-200 text-orange-700",
  "bg-amber-200 text-amber-700",
  "bg-lime-200 text-lime-700",
  "bg-green-200 text-green-700",
  "bg-teal-200 text-teal-700",
  "bg-cyan-200 text-cyan-700",
  "bg-blue-200 text-blue-700",
  "bg-indigo-200 text-indigo-700",
  "bg-purple-200 text-purple-700",
  "bg-pink-200 text-pink-700",
];

function getAvatarColor(username: string) {
  let hash = 0;
  for (let i = 0; i < username.length; i++) {
    hash = username.charCodeAt(i) + ((hash << 5) - hash);
  }
  return avatarColors[Math.abs(hash) % avatarColors.length];
}

const posts = ref<any[]>([]);

function getColor(name?: string) {
  if (!name) return "hsl(0, 0%, 70%)";
  let hash = 0;
  for (let i = 0; i < name.length; i++) {
    hash = name.charCodeAt(i) + ((hash << 5) - hash);
  }
  const hue = Math.abs(hash) % 360;
  return `hsl(${hue}, 60%, 45%)`;
}

async function fetchUser() {
  const id = route.params.id;
  const token = localStorage.getItem("access_token");

  const res = await fetch(`${API_URL}/api/users/${id}/`, {
    headers: { Authorization: `Bearer ${token}` },
  });

  if (res.ok) user.value = await res.json();
  await fetchUserRating(id);
}

async function fetchPosts() {
  const token = localStorage.getItem("access_token");
  const res = await fetch(`${API_URL}/api/posts/`, {
    headers: { Authorization: `Bearer ${token}` },
  });

  if (res.ok) {
    const all = await res.json();
    if (user.value.id) {
      posts.value = all.filter((p: any) => p.user?.id === user.value.id);
      console.log(
        "Posts de l'utilisateur :",
        JSON.stringify(posts.value, null, 2)
      );
    } else {
      posts.value = [];
    }
  }
}

async function verifyUser(userId: number) {
  const token = localStorage.getItem("access_token");
  const res = await fetch(`${API_URL}/api/users/${userId}/verify/`, {
    method: "POST",
    headers: {
      Authorization: `Bearer ${token}`,
      "Content-Type": "application/json",
    },
  });

  if (res.ok) {
    alert("Utilisateur vérifié !");
    fetchUser();
  } else {
    const data = await res.json();
    alert("Erreur : " + JSON.stringify(data));
  }
}

const reports = ref<any[]>([]);
async function fetchReport() {
  const token = localStorage.getItem("access_token");
  const res = await fetch(`${API_URL}/api/reports/`, {
    headers: { Authorization: `Bearer ${token}` },
  });

  if (res.ok) {
    const all = await res.json();
    // reports.value = all;
    reports.value = all.filter(
      (r: any) =>
        r.id_post !== null && posts.value.some((p: any) => p.id === r.id_post)
    );
    console.log(
      "Signalement de l'utilisateur :",
      JSON.stringify(reports.value, null, 2)
    );
  } else {
    reports.value = [];
  }
}

const totalReports = computed(() => reports.value.length);

const slideReportOpen = ref(false);
function toggleSlideReport() {
  slideReportOpen.value = !slideReportOpen.value;
}

const groupedReports = computed(() => {
  const map: Record<number, any> = {};

  for (const report of reports.value) {
    const postId = report.id_post;
    if (!map[postId]) {
      map[postId] = {
        postId,
        reports: [],
      };
    }
    map[postId].reports.push(report);
  }

  return Object.values(map);
});

const confirmModalOpen = ref(false);
const selectedPostId = ref<number | null>(null);

function openConfirmModal(postId: number) {
  selectedPostId.value = postId;
  confirmModalOpen.value = true;
}

function closeConfirmModal() {
  confirmModalOpen.value = false;
  selectedPostId.value = null;
}

async function approveReport() {
  if (!selectedPostId.value) return;

  const token = localStorage.getItem("access_token");

  const res = await fetch(`${API_URL}/api/reports/approve/`, {
    method: "POST",
    headers: {
      Authorization: `Bearer ${token}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ post_id: selectedPostId.value }),
  });

  const data = await res.json();

  if (!res.ok) {
    alert(
      "Erreur : " +
        (data.detail || data.error || "Impossible d'approuver les reports")
    );
    return;
  }

  alert("Tous les reports du post approuvés et post supprimé");

  // Refresh
  await fetchPosts();
  await fetchReport();

  closeConfirmModal();
}

async function confirmApprove() {
  if (!selectedPostId.value) return;

  await approveReport(selectedPostId.value);
  closeConfirmModal();
}

function getPostById(postId: number) {
  return posts.value.find((p) => p.id === postId);
}

const users = ref<any[]>([]);
async function fetchUsers() {
  const token = localStorage.getItem("access_token");
  const res = await fetch(`${API_URL}/api/users/`, {
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${token}`,
    },
  });

  if (res.ok) {
    users.value = await res.json();
    console.log("Users:", JSON.stringify(users.value, null, 2));
  } else if (res.status === 401) {
    router.push("/login");
  } else {
    users.value = [];
  }
}

function getUserById(userId: number) {
  return users.value.find((u) => u.id === userId);
}

async function fetchUserRating(userId: number) {
  const token = localStorage.getItem("access_token");
  const res = await fetch(`${API_URL}/api/reviews/user/${userId}/`, {
    headers: { Authorization: `Bearer ${token}` },
  });

  if (res.ok) {
    const data = await res.json();
    user.value.rating_avg = data.average_rating || 0;
  } else {
    user.value.rating_avg = 0;
  }
}

onMounted(async () => {
  await fetchUser();
  await fetchPosts();
  await fetchReport();
  await fetchUsers();
});
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-right-enter-active {
  transition: transform 0.3s ease;
}
.slide-right-leave-active {
  transition: transform 0.3s ease;
}
.slide-right-enter-from {
  transform: translateX(100%);
}
.slide-right-enter-to {
  transform: translateX(0%);
}
.slide-right-leave-from {
  transform: translateX(0%);
}
.slide-right-leave-to {
  transform: translateX(100%);
}
</style>
