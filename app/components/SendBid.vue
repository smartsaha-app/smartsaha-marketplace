<template>
    <div class="">
      <div v-if="loading" class="flex justify-center mt-10">
        <div
          class="w-8 h-8 border-4 border-gray-300 border-t-[#10b481] rounded-full animate-spin"
        ></div>
      </div>
  
      <div v-if="errorMsg" class="text-center text-red-500">{{ errorMsg }}</div>
  
      <div
        v-if="bids.length"
        class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6"
      >
        <div
          v-for="bid in bids"
          :key="bid.id"
          class="rounded p-5 transition-all duration-300 border bg-white flex flex-col"
        >
          <div class="flex items-center justify-between mb-4">
            <div class="flex items-center gap-3">
              <div
                class="w-10 h-10 rounded-full flex items-center justify-center font-bold text-lg"
                :class="getAvatarColor(bid.post_detail?.user?.username)"
              >
                {{
                  bid.post_detail?.user?.username?.charAt(0).toUpperCase() || "U"
                }}
              </div>
              <div class="text-md">
                <p class="font-semibold text-gray-800">
                  {{ bid.post_detail?.user?.username || "Utilisateur" }}
                </p>
                <p class="text-gray-500 truncate text-sm">
                  {{ bid.post_detail?.user?.email || "-" }}
                </p>
              </div>
            </div>
            <div class="text-gray-400 text-xs">
              {{ formatBidDate(bid) }}
            </div>
          </div>
  
          <div
            class="flex flex-col md:flex-row md:items-center md:justify-between gap-3 mb-3"
          >
            <span
              class="px-3 py-1 rounded-full text-sm font-medium flex items-center gap-1"
              :class="{
                'bg-yellow-100 text-yellow-500':
                  bid.current_status?.name === 'proposée',
                'bg-[#10b481]/10 text-[#10b481]':
                  bid.current_status?.name === 'acceptée',
                'bg-red-100 text-red-600': bid.current_status?.name === 'refusée',
                'bg-gray-300 text-gray-700':
                  bid.current_status?.name === 'arrêtée' ||
                  bid.current_status?.name === 'annulée',
              }"
            >
              <template
                v-if="
                  (bid.current_status?.name === 'acceptée' ||
                    bid.current_status?.name === 'proposée') &&
                  bid.is_highest
                "
              >
                <i class="bx bxs-star text-sm"></i>
              </template>
  
              {{
                bid.current_status?.name
                  ? bid.current_status.name.charAt(0).toUpperCase() +
                    bid.current_status.name.slice(1)
                  : "Enchère"
              }}
            </span>
  
            <div class="flex items-center gap-2 text-gray-500 text-sm">
              <i class="bxr bx-location-alt-2"></i>
              <span>{{ bid.post_detail?.location || "-" }}</span>
            </div>
          </div>
  
          <h2 class="text-md font-semibold text-gray-800 mb-3">
            {{ bid.post_detail?.title || "Post sans titre" }}
          </h2>
  
          <button
        @click="openPostModal(bid.post_detail)"
        class="text-[#10b481] mb-6 text-sm hover:underline"
      >
        {{ t("gotopost") }}
      </button>

  
          <div class="bg-gray-50 border border-gray-200 p-3 rounded mb-4">
            <p class="text-sm text-gray-800 font-medium mb-1">
              {{ t("monOffre") }} :
            </p>
            <p class="text-gray-700 mb-1">{{ bid.message }}</p>
          </div>

          <div class="flex flex-wrap gap-3 mb-4">
            <div
              class="flex items-center gap-2 bg-gray-50 border border-gray-200 px-3 py-2 rounded shadow-sm text-sm text-gray-800"
            >
              <i class="bx bx-wallet text-base"></i>
              <span>
                {{ bid.price || "-" }}
                {{ bid.post_detail?.currency?.symbol || "-" }}
              </span>
            </div>
            <div
              class="flex items-center gap-2 bg-gray-50 border border-gray-200 px-3 py-2 rounded shadow-sm text-sm text-gray-800"
            >
              <i class="bx bx-package text-base"></i>
              <span>
                {{ bid.post_detail?.quantity || "-" }}
                {{ bid.post_detail?.product?.unit?.abbreviation || "-" }}
              </span>
            </div>
          </div>
          <div v-if="bid.current_status?.name !== 'acceptée'" class="mt-auto">
            <button
              @click="openCancelModal(bid.id)"
              class="w-full bg-red-600 text-white py-2 rounded hover:bg-red-700 transition"
            >
              {{ t("btnCancel") }}
            </button>
          </div>
        </div>
      </div>
  
      <div v-else class="text-center text-gray-500 mt-12">
        {{ t("noFoundMyBid") }}
      </div>
    </div>
  
    <div
      v-if="showCancelModal"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50"
    >
      <div class="bg-white rounded-lg p-6 w-80">
        <h2 class="text-lg font-bold mb-4">{{ t("comfirmCancel") }}</h2>
        <p class="mb-6">{{ t("confirmText") }}</p>
        <div class="flex justify-end gap-3">
          <button
            @click="showCancelModal = false"
            class="px-4 py-2 rounded border border-gray-300 hover:bg-gray-100"
          >
            {{ t("cancel") }}
          </button>
          <button
            @click="confirmCancelBid"
            class="px-4 py-2 rounded bg-red-500 text-white hover:bg-red-600"
          >
            {{ t("confirm") }}
          </button>
        </div>
      </div>
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
            <!-- Image -->
            <div class="md:w-1/2 relative">
              <img
                v-if="selectedPost.image_url"
                :src="selectedPost.image_url"
                alt="product"
                class="w-full h-80 md:h-[24rem] object-cover rounded shadow-md"
              />
              <div
                v-else
                class="w-full h-80 md:h-[32rem] bg-gray-100 rounded flex items-center justify-center text-gray-400 font-medium"
              >
                No image available
              </div>

              <div class="absolute top-4 left-4 right-4 flex justify-between items-start">
                

                <div>
                  <span
                    class="inline-block px-3 py-1 rounded-full text-xs text-white"
                    :class="{
                      'bg-[#10b481]': selectedPost.type_post?.type === 'Selling',
                      'bg-orange-500': selectedPost.type_post?.type === 'Buying',
                      'bg-gray-500': !selectedPost.type_post?.type,
                    }"
                  >
                    {{ selectedPost.type_post?.type || "Unknown" }}
                  </span>
                </div>
              </div>
            </div>

            <!-- Contenu du post -->
            <div class="md:w-1/2 flex flex-col justify-between gap-4">

              <div class="flex justify-between items-start">
                <div class="flex flex-col gap-2">
                  <div class="flex items-center gap-2">
                    <div
                      class="w-10 h-10 rounded-full flex items-center justify-center font-bold text-lg"
                      :class="getAvatarColor(selectedPost.user?.username)"
                    >
                      {{ selectedPost.user?.username?.charAt(0).toUpperCase() }}
                    </div>
                    <div class="flex flex-col">
                      <span class="font-semibold text-gray-800">{{ selectedPost.user?.username }}</span>
                      <div class="flex items-center gap-1 text-gray-500 text-xs">
                        <span>{{ formattedDate(selectedPost) }}</span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>

              <div>
                <h2 class="text-xl md:text-2xl font-bold text-gray-800 mb-3">
                  {{ selectedPost.title || "Product Name" }}
                </h2>
                <p class="text-gray-700 mb-3">
                  {{ selectedPost.description || "No description available." }}
                </p>
              </div>

              <div
        class="flex-1 flex flex-col gap-2 text-gray-700 text-sm justify-between"
      >
        <div class="flex items-center gap-2">
          <i class="bxr bx-location-alt-2 text-gray-500"></i>
          <span>{{ selectedPost.location || "Unknown" }}</span>
        </div>

        <div class="flex items-center gap-2">
          <i class="bx bx-package text-gray-500"></i>
          <span>{{ selectedPost.product?.product }}</span>
        </div>

        <div class="flex flex-wrap gap-3 mt-3">
          <div
            class="flex items-center gap-2 bg-gray-50 border border-gray-100 backdrop-blur-sm text-gray-800 px-3 py-2 rounded shadow-sm text-sm"
          >
            <i class="bx bx-wallet text-base"></i>
            <span>{{ selectedPost?.price }} {{ selectedPost?.currency?.symbol }}</span>
          </div>

          <div
            class="flex items-center gap-2 bg-blue-50 text-[#112830] px-4 py-2 rounded shadow-sm text-sm font-semibold"
          >
            <i class="bx bx-package text-base"></i>
            <span
              >{{ selectedPost?.quantity }}
              {{ selectedPost?.product?.unit?.abbreviation }}</span
            >
          </div>
        </div>
      </div>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </template>
  
  <script setup>
  import { ref, reactive, onMounted } from "vue";
  import { API_URL } from "~/utils/config";
  import {
    formatDistanceToNow,
    parseISO,
    format,
    differenceInDays,
  } from "date-fns";
  import { fr } from "date-fns/locale";
  import { useLanguageStore } from "~/stores/language";
  import { translate } from "~/utils/translate";
  import { useRouter } from "vue-router";
  const router = useRouter();
  
  const languageStore = useLanguageStore();
  const t = (key) => {
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
  
  function getAvatarColor(username) {
    let hash = 0;
    for (let i = 0; i < username.length; i++) {
      hash = username.charCodeAt(i) + ((hash << 5) - hash);
    }
    return avatarColors[Math.abs(hash) % avatarColors.length];
  }
  
  const bids = ref([]);
  const loading = ref(false);
  const errorMsg = ref("");
  const showFullDesc = reactive({});
  
  const fetchMyBids = async () => {
    loading.value = true;
    errorMsg.value = "";
    try {
      const res = await fetch(`${API_URL}/api/bids/my_bids/`, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
      });
      if (!res.ok) throw new Error("Erreur lors de la récupération des enchères");
      const data = await res.json();
  
      // Récupérer chaque post séparément
      for (const bid of data) {
        try {
          const postRes = await fetch(`${API_URL}/api/posts/${bid.post.id}/`, {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("access_token")}`,
            },
          });
          const postData = await postRes.json();
          bid.post_detail = postData; // Attacher le post au bid
        } catch (err) {
          console.error(`Erreur post ${bid.post.id}:`, err);
          bid.post_detail = {};
        }
      }
  
      bids.value = data;
      console.log("Mes bids avec post:", JSON.stringify(bids.value, null, 2));
    } catch (err) {
      console.error(err);
      errorMsg.value = "Erreur lors du chargement des enchères";
    } finally {
      loading.value = false;
    }
  };
  
  const formatBidDate = (bid) => {
    if (!bid.created_at) return "";
    const date = new Date(bid.created_at);
    const daysDiff = differenceInDays(new Date(), date);
  
    if (daysDiff <= 3) {
      return formatDistanceToNow(date, { addSuffix: true, locale: fr });
    } else {
      return format(date, "dd MMM yyyy", { locale: fr });
    }
  };
  
  const formattedDate = (post) => {
    if (!post.created_at) return "";
    const date = new Date(post.created_at);
    const daysDiff = differenceInDays(new Date(), date);
  
    if (daysDiff <= 3) {
      return formatDistanceToNow(date, { addSuffix: true, locale: fr });
    } else {
      return format(date, "dd MMM yyyy", { locale: fr });
    }
  };
  
  const toggleDesc = (id) => {
    showFullDesc[id] = !showFullDesc[id];
  };
  
  const showCancelModal = ref(false);
  const bidToCancel = ref(null);
  
  const openCancelModal = (bidId) => {
    bidToCancel.value = bidId;
    showCancelModal.value = true;
  };
  
  const confirmCancelBid = async () => {
    if (!bidToCancel.value) return;
  
    try {
      const res = await fetch(`${API_URL}/api/bids/${bidToCancel.value}/`, {
        method: "DELETE",
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
      });
  
      if (!res.ok) throw new Error("Erreur lors de l'annulation");
  
      showCancelModal.value = false;
      bidToCancel.value = null;
  
      await fetchMyBids(); // rafraîchir la liste
    } catch (err) {
      console.error(err);
      alert("Impossible d'annuler l'enchère.");
    }
  };

  const modalOpen = ref(false);
const selectedPost = ref({});

const openPostModal = (post) => {
  selectedPost.value = post;
  console.log("Post selected :", JSON.parse(JSON.stringify(selectedPost.value, null, 2)));

  modalOpen.value = true;
};

const closeModal = () => {
  modalOpen.value = false;
};
  
  onMounted(fetchMyBids);
  </script>
  
  <style scoped>
  div:hover {
    transition: all 0.3s ease;
  }

  .fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
  </style>
  