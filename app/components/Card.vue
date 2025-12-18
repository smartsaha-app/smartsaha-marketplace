<template>
  <div
    class="bg-white rounded shadow-sm p-6 min-w-[600px] border border-gray-100"
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
          <span class="font-semibold text-gray-800">{{
            post.user?.username
          }}</span>
          <div class="flex items-center gap-1 text-gray-400 text-xs">
            <span>{{ formattedDate }}</span>
          </div>
        </div>
      </div>

      <span
        class="px-2 py-2 text-xs font-semibold text-white rounded-full flex items-center justify-center hidden"
        :class="
          post.type_post?.type === 'Selling' ? 'bg-[#10b481]' : 'bg-[#f4a261]'
        "
      >
        <i
          :class="
            post.type_post?.type === 'Selling'
              ? 'bxr  bx-cart'
              : 'bxr  bx-badge-check'
          "
        ></i>
      </span>

      <div class="relative">
        <button
          @click="toggleReportMenu"
          class="text-gray-400 hover:text-gray-700"
        >
          <i class="bx bx-dots-vertical-rounded text-xl"></i>
        </button>

        <div
          v-if="showReportMenu"
          class="absolute right-0 mt-2 w-36 bg-white border rounded shadow-md z-10"
        >
          <button
            @click="openReportModal"
            class="w-full text-left px-4 py-2 text-sm text-red-600 hover:bg-gray-100"
          >
            Signaler
          </button>
        </div>
      </div>
    </div>

    <h2 class="text-xl font-semibold text-gray-800 mb-6">
      {{ post.title || "No title" }}
    </h2>

    <div class="flex gap-6 items-stretch">
      <div
        class="rounded overflow-hidden cursor-pointer flex-1"
        @click="openModal"
      >
        <img
          v-if="post.image_url"
          :src="post.image_url"
          class="w-full h-40 object-cover rounded"
        />
        <div
          v-else
          class="w-full h-full bg-gray-200 flex items-center justify-center text-gray-500 rounded"
        >
          No image
        </div>
      </div>

      <div
        class="flex-1 flex flex-col gap-2 text-gray-700 text-sm justify-between"
      >
        <div class="flex items-center gap-2">
          <i class="bxr bx-location-alt-2 text-gray-500"></i>
          <span>{{ post.location || "Unknown" }}</span>
        </div>

        <div class="flex items-center gap-2">
          <i class="bx bx-package text-gray-500"></i>
          <span>{{ post.product?.product }}</span>
        </div>

        <div class="flex flex-wrap gap-3 mt-3">
          <div
            class="flex items-center gap-2 bg-gray-50 border border-gray-100 backdrop-blur-sm text-gray-800 px-3 py-2 rounded shadow-sm text-sm"
          >
            <i class="bx bx-wallet text-base"></i>
            <span>{{ post?.price }} {{ post?.currency?.symbol }}</span>
          </div>

          <div
            class="flex items-center gap-2 bg-blue-50 text-[#112830] px-4 py-2 rounded shadow-sm text-sm font-semibold"
          >
            <i class="bx bx-package text-base"></i>
            <span
              >{{ post?.quantity }}
              {{ post?.product?.unit?.abbreviation }}</span
            >
          </div>
        </div>
      </div>
    </div>

    <div class="mt-6">
      <p class="text-gray-700 inline">
        {{
          showFullDescription
            ? post.description
            : post.description?.slice(0, 100) +
              (post.description?.length > 100 ? "..." : "")
        }}
      </p>
      <button
        v-if="post.description?.length > 100"
        @click="toggleDescription"
        class="text-blue-600 text-sm ml-2 inline"
      >
        {{ showFullDescription ? t("viewLess") : t("viewMore") }}
      </button>
    </div>

    <div class="flex justify-end mt-4">
      <button
        v-if="
          (post?.current_status === 'published' ||
            post?.current_status === 'négociation') &&
          post?.can_receive_bids
        "
        :class="
          post.type_post?.type === 'Selling' ? 'bg-[#10b481]' : 'bg-[#f4a261]'
        "
        class="px-4 py-2 text-white text-sm font-semibold rounded hover:brightness-110 transition"
        @click="openModal"
      >
        {{
          post.type_post?.type === "Selling"
            ? t("btnPlaceBid")
            : t("btnMakeOffer")
        }}
      </button>
    </div>

    <transition name="fade">
      <div
        v-if="modalOpen"
        class="fixed inset-0 bg-black/70 flex justify-center items-center z-50 p-4"
      >
        <div
          class="bg-white w-full max-w-4xl rounded shadow-2xl overflow-y-auto max-h-[90vh] relative p-6"
        >
          <button
            @click="closeModal"
            class="absolute top-4 right-4 text-gray-400 text-3xl hover:text-gray-700 transition"
          >
            <i class="bx bx-x"></i>
          </button>

          <div class="flex flex-col md:flex-row gap-6">
            <div class="md:w-1/2 relative">
              <img
                v-if="post.image_url"
                :src="post.image_url"
                alt="product"
                class="w-full h-80 md:h-[32rem] object-cover rounded shadow-md"
              />
              <div
                v-else
                class="w-full h-80 md:h-[32rem] bg-gray-100 rounded flex items-center justify-center text-gray-400 font-medium"
              >
                No image available
              </div>

              <div
                class="absolute top-4 left-4 right-4 flex justify-between items-start"
              >
                <div class="flex flex-col gap-2">
                  <div class="flex items-center gap-2">
                    <div
                      class="w-10 h-10 rounded-full flex items-center justify-center font-bold text-lg"
                      :class="getAvatarColor(post.user?.username)"
                    >
                      {{ post.user?.username?.charAt(0).toUpperCase() }}
                    </div>
                    <div class="flex flex-col">
                      <span class="font-semibold text-white">{{
                        post.user?.username
                      }}</span>
                      <div
                        class="flex items-center gap-1 text-gray-300 text-xs"
                      >
                        <span>{{ formattedDate }}</span>
                      </div>
                    </div>
                  </div>
                </div>

                <div>
                  <span
                    class="inline-block px-3 py-1 rounded-full text-xs text-white"
                    :class="{
                      'bg-[#10b481]': post.type_post?.type === 'Selling',
                      'bg-orange-500': post.type_post?.type === 'Buying',
                      'bg-gray-500': !post.type_post?.type,
                    }"
                  >
                    {{ post.type_post?.type || "Unknown" }}
                  </span>
                </div>
              </div>

              <div
                class="absolute bottom-4 left-4 bg-black/50 backdrop-blur-sm text-white px-3 py-2 rounded flex items-center gap-2 text-sm shadow-md"
              >
                <i class="bxr bx-location-alt-2"></i>
                <span class="">{{ post.location || "Not specified" }}</span>
              </div>
            </div>

            <div class="md:w-1/2 flex flex-col justify-between gap-4">
              <div>
                <h2 class="text-2xl md:text-3xl font-bold text-gray-900 mb-3">
                  {{ post.product?.product || "Product Name" }}
                </h2>

                <p class="text-gray-700 mb-3">
                  {{ post.description || "No description available." }}
                </p>

                <div class="flex flex-col gap-2 text-gray-700 font-medium">
                  <div class="flex flex-wrap gap-3 mt-3">
                    <div
                      class="flex items-center gap-2 bg-gray-50 border border-gray-100 backdrop-blur-sm text-gray-800 px-3 py-2 rounded shadow-sm text-sm"
                    >
                      <i class="bx bx-wallet text-base"></i>
                      <span
                        >{{ post?.price }} {{ post?.currency?.symbol }}</span
                      >
                    </div>

                    <div
                      class="flex items-center gap-2 bg-blue-50 text-[#112830] px-4 py-2 rounded shadow-sm text-sm font-semibold"
                    >
                      <i class="bx bx-package text-base"></i>
                      <span
                        >{{ post?.quantity }}
                        {{ post?.product?.unit?.abbreviation }}</span
                      >
                    </div>
                  </div>
                </div>
              </div>

              <div class="mt-4 flex flex-col gap-3">
                <input
                  v-model="price"
                  type="number"
                  placeholder="Enter your bid/offer price"
                  class="w-full p-3 border border-gray-200 rounded focus:ring-2 focus:ring-[#10b481] focus:border-[#10b481] shadow-sm"
                />
                <textarea
                  v-model="message"
                  placeholder="Write your message... (optional)"
                  class="w-full p-3 border border-gray-200 rounded resize-none focus:ring-2 focus:ring-[#10b481] focus:border-[#10b481] shadow-sm"
                  rows="4"
                ></textarea>
                <button
                  :disabled="loading"
                  :class="[
                    'w-full text-white font-semibold py-3 rounded transition shadow-md',
                    post.type_post?.type === 'Selling'
                      ? 'bg-[#10b481] hover:bg-[#0e9a72]'
                      : 'bg-[#f4a261] hover:bg-[#e07b3c]',
                    loading ? 'opacity-60 cursor-not-allowed' : '',
                  ]"
                  @click="sendBid"
                >
                  {{
                    loading
                      ? "Processing..."
                      : post.type_post?.type === "Selling"
                      ? t("btnPlaceBid")
                      : t("btnMakeOffer")
                  }}
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
  <transition name="fade">
    <div
      v-if="reportModalOpen"
      class="fixed inset-0 bg-black/60 flex items-center justify-center z-50 p-4"
    >
      <div class="bg-white rounded w-full max-w-md p-6">
        <h3 class="text-lg font-semibold mb-3 text-gray-800">
          {{ t("report") }}
        </h3>

        <textarea
          v-model="reportReason"
          class="w-full border rounded p-3 text-sm resize-none outline-none"
          rows="4"
          :placeholder="t('reportPlaceholder')"
        ></textarea>

        <div class="flex justify-end gap-2 mt-4">
          <button
            @click="reportModalOpen = false"
            class="px-4 py-2 text-sm rounded bg-gray-200 hover:bg-gray-300"
          >
            {{ t("cancel") }}
          </button>
          <button
            @click="sendReport"
            :disabled="reportLoading"
            class="px-4 py-2 text-sm rounded bg-red-600 text-white hover:bg-red-700 flex items-center justify-center gap-2 disabled:opacity-70"
          >
            <div
              v-if="reportLoading"
              class="w-4 h-4 border-2 border-white border-t-transparent rounded-full animate-spin"
            ></div>

            <span v-else>
              {{ t("send") }}
            </span>
          </button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script setup lang="ts">
import { ref, computed, defineProps } from "vue";
import {
  formatDistanceToNow,
  parseISO,
  format,
  differenceInDays,
} from "date-fns";
import { fr } from "date-fns/locale";
import { API_URL } from "~/utils/config";
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";
import { useRouter } from "vue-router";
const router = useRouter();

const languageStore = useLanguageStore();
const t = (key: string) => {
  const lang = languageStore.lang;
  return translate[lang][key] || key;
};

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

const showFullDescription = ref(false);

function toggleDescription() {
  showFullDescription.value = !showFullDescription.value;
}

const props = defineProps({
  post: {
    type: Object,
    required: true,
  },
});

const modalOpen = ref(false);
const message = ref("");
const price = ref(null);
const loading = ref(false);
const showReportMenu = ref(false);
const reportModalOpen = ref(false);
const reportReason = ref("");
const reportLoading = ref(false);
function toggleReportMenu() {
  showReportMenu.value = !showReportMenu.value;
}

function openReportModal() {
  showReportMenu.value = false;
  reportModalOpen.value = true;
}

async function sendReport() {
  if (!reportReason.value.trim()) {
    alert("Veuillez indiquer une raison.");
    return;
  }

  const token = localStorage.getItem("access_token");
  if (!token) {
    window.location.href = "/signin";
    return;
  }

  reportLoading.value = true;

  try {
    const res = await fetch(`${API_URL}/api/reports/`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`,
      },
      body: JSON.stringify({
        id_post: props.post.id,
        reason: reportReason.value,
      }),
    });

    if (!res.ok) {
      const err = await res.json();
      alert("Erreur lors du signalement : " + JSON.stringify(err));
      reportLoading.value = false;
      return;
    }

    alert("Signalement envoyé avec succès.");
    reportReason.value = "";
    reportModalOpen.value = false;
  } catch (err) {
    console.error(err);
    alert("Erreur interne.");
  }

  reportLoading.value = false;
}

function openModal() {
  const token = localStorage.getItem("access_token");
  if (!token) {
    window.location.href = "/signin";
    return;
  }
  modalOpen.value = true;
}
function closeModal() {
  modalOpen.value = false;
}

// Format date
const formattedDate = computed(() => {
  if (!props.post.created_at) return "";
  const date = new Date(props.post.created_at);
  const daysDiff = differenceInDays(new Date(), date);

  if (daysDiff <= 3) {
    return formatDistanceToNow(date, { addSuffix: true, locale: fr });
  } else {
    return format(date, "dd MMM yyyy", { locale: fr });
  }
});

async function sendBid() {
  if (!price.value || Number(price.value) <= 0) {
    alert("Veuillez entrer un prix valide.");
    return;
  }

  const token = localStorage.getItem("access_token");
  if (!token) {
    window.location.href = "/signin";
    return;
  }

  loading.value = true;

  try {
    if (!token) {
      window.location.href = "/signin";
      return;
    }

    const postId = props.post.id;

    // 1️⃣ CRÉATION DU CHAT
    const chatResponse = await fetch(`${API_URL}/api/chats/`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`,
      },
      body: JSON.stringify({
        id_post: postId,
        id_status_id: 1, // par défaut
      }),
    });

    if (!chatResponse.ok) {
      const err = await chatResponse.json();
      alert("Erreur création du chat : " + JSON.stringify(err));
      loading.value = false;
      return;
    }

    const chatData = await chatResponse.json();
    const chatId = chatData.id; // ← récupère l’ID du chat

    // 2️⃣ CRÉATION DU BID (optionnel avant/après)
    const bidResponse = await fetch(
      `${API_URL}/api/posts/${postId}/place_bid/`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify({
          price: Number(price.value),
          message: message.value || "",
        }),
      }
    );

    if (!bidResponse.ok) {
      const err = await bidResponse.json();
      alert("Erreur : " + JSON.stringify(err));
      loading.value = false;
      return;
    }

    // 3️⃣ CRÉATION DU MESSAGE AUTOMA-TIQUE
    const finalMessage =
      message.value && message.value.trim() !== ""
        ? message.value
        : `Bonjour, je propose ${price.value} ${props.post.currency?.symbol}`;

    const msgResponse = await fetch(`${API_URL}/api/messages/`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${token}`,
      },
      body: JSON.stringify({
        message: finalMessage,
        id_chat: chatId,
        id_type_message_id: 1,
      }),
    });

    if (!msgResponse.ok) {
      const err = await msgResponse.json();
      alert("Message non envoyé : " + JSON.stringify(err));
      loading.value = false;
      return;
    }

    alert("Votre offre & message ont été envoyés !");
    price.value = null;
    message.value = "";
    modalOpen.value = false;
  } catch (err) {
    console.error(err);
    alert("Erreur interne.");
  }

  loading.value = false;
}

function goToProfile(user: { id: any }) {
  router.push(`/user/${user.id}`);
}
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
