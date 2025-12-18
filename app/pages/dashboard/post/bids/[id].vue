<template>
  <div class="flex h-full">
    <div class="flex-1 space-y-8 p-8">
      <Breadcrumb />
      <h2 class="text-2xl font-bold text-gray-800">
        {{ t("bidFor") }} "{{ postTitle }}"
      </h2>

      <div
        v-if="bids.length"
        class="grid gap-5 sm:grid-cols-1 md:grid-cols-2 lg:grid-cols-2"
      >
        <div
          v-for="bid in bids"
          :key="bid.id"
          class="rounded p-5 transition-all duration-300 border bg-white"
          :class="
            bid.is_highest
              ? 'border-[#10b481]/70 shadow-md bg-blue-50'
              : 'border-gray-200 shadow-sm'
          "
        >
          <div class="flex items-center justify-between">
            <div class="flex items-center gap-3">
              <div
                class="w-10 h-10 rounded-full flex items-center justify-center font-bold text-lg"
                :class="getAvatarColor(bid.user.username)"
              >
                {{ bid.user.username.charAt(0).toUpperCase() }}
              </div>

              <div class="flex flex-col">
                <span class="font-semibold text-gray-800">
                  {{ bid.user.username }}
                </span>
                <span class="text-xs text-gray-500">
                  {{ bid.user.email }}
                </span>
              </div>
            </div>

            <div class="flex items-center gap-3">
              <span
                v-if="bid.is_highest"
                class="text-[11px] font-medium text-[#10b481] bg-[#10b481]/10 px-2 py-1 rounded-md flex items-center gap-1"
              >
                <i class="bx bx-star text-sm"></i>
                {{ t("bestOffer") }}
              </span>

              <NuxtLink
                v-if="bid.current_status?.name === 'proposée'"
                :to="`/dashboard/chatbox?post=${bid.post.id}&bid=${bid.id}`"
                class=""
              >
                <i
                  class="bx bx-message-detail text-xl text-gray-400 hover:text-gray-600 cursor-pointer"
                ></i>
              </NuxtLink>
            </div>
          </div>

          <div
            class="flex items-center justify-between mt-5 text-xs text-gray-500"
          >
            <span
              class="px-3 py-1 rounded-full text-xs flex items-center gap-1"
              :class="{
                'bg-yellow-100 text-yellow-500':
                  bid.current_status?.name === 'proposée',
                'bg-[#10b481]/10 text-[#10b481]':
                  bid.current_status?.name === 'acceptée',
                'bg-red-100 text-red-600':
                  bid.current_status?.name === 'refusée',
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

            <span>{{ formatDate(bid.created_at) }}</span>
          </div>

          <p class="mt-4 text-gray-700 text-sm leading-relaxed">
            {{ truncated(bid.message, 150) }}
          </p>

          <hr class="my-4 border-gray-200" />

          <div class="flex items-center justify-between">
            <span
              class="font-semibold text-lg"
              :class="bid.is_highest ? 'text-[#10b481]' : 'text-gray-700'"
            >
              {{ bid.price }} {{ bid.currency.symbol }}
            </span>

            <button
              v-if="!postDetails?.accepted_bid"
              class="bg-[#10b481] text-white px-4 py-1.5 rounded shadow hover:bg-[#10b481]/70 transition"
              @click="openBidConfirmation(bid, 'accept')"
            >
              {{ t("btnaccept") }}
            </button>

            <!-- <button
                    @click="openBidConfirmation(item, 'accept')"
                    class="flex-1 bg-[#10b481] text-white px-3 py-2 rounded hover:bg-green-700 transition"
                  >
                    Accepter
                  </button>
                  <button
                    @click="openBidConfirmation(item, 'decline')"
                    class="flex-1 bg-gray-100 text-gray-800 px-3 py-2 rounded hover:bg-red-700 transition"
                  >
                    Refuser
                  </button> -->
          </div>
        </div>
      </div>

      <div v-else class="text-gray-500 italic">
        {{ t("noBidFound") }}
      </div>
    </div>

    <div
      class="hidden lg:flex w-96 border-l border-[#f1f1f1] bg-[#fff] p-4 overflow-y-auto justify-center"
    >
      <div
        class="fixed rounded shadow-sm border border-white/5 overflow-hidden mt-6 w-[350px]"
      >
        <div v-if="postDetails?.image_url" class="relative">
          <div class="relative">
            <img
              :src="postDetails.image_url"
              alt="Post Image"
              class="w-full h-48 object-cover"
            />
            <div
              class="absolute top-3 left-3 flex items-center gap-3 bg-white/10 backdrop-blur-md px-2 py-1 shadow-sm text-xs text-white"
            >
              <p>{{ t("publishedOn") }} {{ getPublishedDate() }}</p>
              <span class="h-4 w-px bg-gray-300"></span>
              <p class="flex items-center gap-1">
                <i class="bxr bx-location-alt-2"></i>
                <span>{{ postDetails?.location }}</span>
              </p>
            </div>
          </div>
          <div
            class="absolute top-3 left-3 flex items-center gap-3 bg-white/10 backdrop-blur-md px-2 py-1 shadow-sm text-xs text-white"
          >
            <p>{{ t("publishedOn") }} {{ getPublishedDate() }}</p>
            <span class="h-4 w-px bg-gray-300"></span>
            <p class="flex items-center gap-1">
              <i class="bxr bx-location-alt-2"></i>
              <span>{{ postDetails?.location }}</span>
            </p>
          </div>
        </div>

        <div class="p-4 flex flex-col gap-2">
          <h3 class="text-2xl font-bold text-gray-700">
            {{ postDetails?.title }}
          </h3>
          <p class="text-gray-600 text-sm leading-relaxed">
            {{ postDetails?.description }}
          </p>

          <div class="flex flex-wrap gap-3 mt-3">
            <div
              class="flex items-center gap-2 bg-gray-50 border border-gray-100 backdrop-blur-sm text-[#112830] px-3 py-2 rounded shadow-sm text-sm"
            >
              <i class="bx bx-wallet text-base"></i>
              <span
                >{{ postDetails?.price }}
                {{ postDetails?.currency?.symbol }}</span
              >
            </div>

            <div
              class="flex items-center gap-2 bg-[#10b481]/20 text-[#10b481] px-4 py-2 rounded shadow-sm text-sm font-semibold"
            >
              <i class="bx bx-package text-base"></i>
              <span
                >{{ postDetails?.quantity }}
                {{ postDetails?.product?.unit?.abbreviation }}</span
              >
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div
    v-if="confirmBidOpen"
    class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-30"
  >
    <div class="bg-white rounded p-6 w-96">
      <h3 class="text-lg font-semibold mb-4">
        {{
          bidAction === "accept"
            ? "Confirmer l’acceptation"
            : "Confirmer le refus"
        }}
      </h3>
      <p class="mb-4">
        {{
          bidAction === "accept"
            ? "Voulez-vous vraiment accepter cette enchère ?"
            : "Voulez-vous vraiment refuser cette enchère ? Cette action est irréversible."
        }}
      </p>

      <div v-if="bidAction === 'decline'" class="mb-4">
        <label class="block text-sm mb-1">{{ t("MsgBuyer") }} :</label>
        <textarea
          v-model="declineMessage"
          class="w-full border rounded p-2"
          rows="3"
        ></textarea>
      </div>

      <div class="flex justify-end gap-3">
        <button
          @click="cancelBid"
          class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300"
        >
          {{ t("cancel") }}
        </button>
        <button
          @click="performBidAction"
          :class="
            bidAction === 'accept'
              ? 'bg-[#10b481] hover:bg-[#10b481]/90'
              : 'bg-red-600 hover:bg-red-700'
          "
          class="px-4 py-2 text-white rounded"
        >
          {{ bidAction === "accept" ? t("btnAccept") : t("btnDecline") }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
definePageMeta({ layout: "dashboard" });
import { ref, onMounted } from "vue";
import { parseISO, formatDistanceToNow, format } from "date-fns";
import { useRoute } from "vue-router";
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

const route = useRoute();
const postId = route.params.id;

const bids = ref([]);
const postTitle = ref("");
const postDetails = ref(null);

const truncated = (text, length = 60) =>
  text.length > length ? text.slice(0, length) + "..." : text;

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

const getPublishedDate = () => {
  const history = postDetails.value?.status_history;
  if (!history) return null;

  const published = history.find((h) => h.status === "published");

  return published ? formatDate(published.date_changed) : null;
};

const fetchBids = async () => {
  try {
    const res = await fetch(`${API_URL}/api/posts/${postId}/bids/`, {
      headers: {
        Authorization: `Bearer ${localStorage.getItem("access_token")}`,
      },
    });
    if (!res.ok) throw new Error("Erreur lors de la récupération des enchères");
    const data = await res.json();
    bids.value = data;
    console.log("Bids:", JSON.stringify(bids.value, null, 2));

    postTitle.value = data[0]?.post?.title || "Post";
  } catch (err) {
    console.error(err);
  }
};

const fetchPostDetails = async () => {
  try {
    const res = await fetch(`${API_URL}/api/posts/${postId}/`, {
      headers: {
        Authorization: `Bearer ${localStorage.getItem("access_token")}`,
      },
    });
    if (!res.ok) throw new Error("Erreur lors du chargement du post");
    postDetails.value = await res.json();
    console.log("Post details:", JSON.stringify(postDetails.value, null, 2));
  } catch (err) {
    console.error(err);
  }
};
onMounted(() => {
  fetchPostDetails();
  fetchBids();
});

async function api(url, method = "GET", body = null) {
  const token = localStorage.getItem("access_token");
  const options = {
    method,
    headers: {
      "Content-Type": "application/json",
      ...(token && { Authorization: `Bearer ${token}` }),
    },
  };
  if (body) options.body = JSON.stringify(body);
  const res = await fetch(url, options);
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return await res.json();
}

const confirmBidOpen = ref(false);
const bidToConfirm = ref(null);
const bidAction = ref(null);
const declineMessage = ref("");

function openBidConfirmation(bid, action) {
  bidToConfirm.value = bid;
  bidAction.value = action;
  declineMessage.value = "";
  confirmBidOpen.value = true;
}

function cancelBid() {
  confirmBidOpen.value = false;
  bidToConfirm.value = null;
  bidAction.value = null;
  declineMessage.value = "";
}

async function performBidAction() {
  if (!bidToConfirm.value || !bidAction.value) return;

  try {
    if (bidAction.value === "accept") {
      await acceptBid(bidToConfirm.value);
    } else if (bidAction.value === "decline") {
      if (!declineMessage.value.trim()) {
        alert("Veuillez saisir un message pour l'acheteur !");
        return;
      }
      await declineBidWithMessage(bidToConfirm.value, declineMessage.value);
    }
  } catch (err) {
    console.error(err);
  } finally {
    cancelBid();
  }
}

async function declineBidWithMessage(bid, message) {
  const payload = {
    continue_negotiation: false,
    message: message,
  };
  const res = await api(
    `${API_URL}/api/bids/${bid.id}/reject/`,
    "POST",
    payload
  );
  console.log("Bid rejected:", res);
  await fetchBids();
}

async function acceptBid(bid) {
  if ((bid.current_status.name || "").toLowerCase() !== "proposée") {
    console.warn(
      "Cette enchère ne peut pas être acceptée :",
      bid.current_status
    );
    return;
  }

  const payload = {
    price: bid.price.toString(),
    post_id: bid.post_id,
    message: bid.message || "",
    is_active: bid.is_active ?? true,
    currency_id: bid.currency_id || null,
  };

  try {
    const res = await api(
      `${API_URL}/api/bids/${bid.id}/accept/`,
      "POST",
      payload
    );
    console.log("Bid accepted:", res);
    await fetchBids();
  } catch (err) {
    console.error("Erreur acceptBid :", err);
  }
}

async function declineBid(bid) {
  if (!bid?.id) return console.warn("Enchère invalide !");

  const continueNegotiation = !!confirm(
    "Voulez-vous continuer la négociation ?"
  );
  let msg = "";

  if (!continueNegotiation) {
    msg = prompt("Veuillez saisir un message pour l'acheteur");
    if (!msg) return alert("Message requis pour arrêter la négociation !");
  }

  const payload = {
    continue_negotiation: continueNegotiation,
    message: msg,
  };

  try {
    const res = await api(
      `${API_URL}/api/bids/${bid.id}/reject/`,
      "POST",
      payload
    );
    console.log("Bid rejected:", res);

    if (res.status >= 400) {
      console.warn("Erreur côté API :", res.status, res.data || res);
      alert(
        "Erreur lors du refus de l'enchère : " + JSON.stringify(res.data || res)
      );
      return;
    }

    await fetchBids();

    if (!continueNegotiation) {
      console.warn("La négociation est terminée avec l'acheteur.");
      return;
    }
    console.warn("La négociation continue avec l'acheteur.");
  } catch (err) {
    console.error("Erreur declineBid :", err.response?.data || err);
    alert(
      "Erreur lors du refus de l'enchère : " +
        JSON.stringify(err.response?.data || err)
    );
  }
}
</script>
