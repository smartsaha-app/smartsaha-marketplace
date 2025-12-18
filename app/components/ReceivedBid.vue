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
                class="w-12 h-12 rounded-full flex items-center justify-center font-bold text-lg"
                :class="getAvatarColor(bid.user?.username)"
              >
                {{ bid.user?.username?.charAt(0).toUpperCase() || "U" }}
              </div>
              <div class="text-md">
                <p class="font-semibold text-gray-800">
                  {{ bid.user?.username || "Utilisateur" }}
                </p>
                <p class="text-gray-500 truncate text-sm">
                  {{ bid.user?.email || "-" }}
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
            {{ bid.post?.title || "Post sans titre" }}
          </h2>
          <NuxtLink
            :to="`/dashboard/post/bids/${bid.post.id}`"
            class="text-[#10b481] mb-6 text-sm hover:underline"
          >
            {{ t("gotopost") }}
          </NuxtLink>
  
          
  
          <div class="bg-gray-50 border border-gray-200 p-3 rounded mb-4">
            <p class="text-sm text-gray-800 font-medium mb-1">
              {{ t("msgBid") }} :
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
          <NuxtLink
            v-if="bid.current_status?.name === 'proposée'"
            :to="`/dashboard/chatbox?post=${bid.post.id}&bid=${bid.id}`"
            class="w-full bg-[#10b481] text-white py-2 rounded hover:bg-red-700 transition text-center"
          >
            {{ t("negocier") }}
          </NuxtLink>
        </div>
      </div>
  
      <div v-else class="text-center text-gray-500 mt-12">
        {{ t("noReceivedBidFound") }}
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from "vue";
  import { API_URL } from "~/utils/config";
  import { formatDistanceToNow, differenceInDays, format } from "date-fns";
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
    if (!username) return avatarColors[0];
    let hash = 0;
    for (let i = 0; i < username.length; i++) {
      hash = username.charCodeAt(i) + ((hash << 5) - hash);
    }
    return avatarColors[Math.abs(hash) % avatarColors.length];
  }
  
  const bids = ref([]);
  const loading = ref(false);
  const errorMsg = ref("");
  
  const fetchReceivedBids = async () => {
    loading.value = true;
    errorMsg.value = "";
    try {
      const res = await fetch(`${API_URL}/api/bids/received_bids/`, {
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
      errorMsg.value = "Erreur lors du chargement des enchères reçues";
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
  
  onMounted(fetchReceivedBids);
  </script>
  