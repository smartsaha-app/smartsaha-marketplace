<template>
  <div class="flex flex-col h-screen">
    <!-- HEADER -->
    <header
      class="flex items-center justify-between bg-[#fff] shadow px-12 py-3 sticky top-0 z-50"
    >
      <div class="flex items-center gap-2">
        <button class="sm:hidden p-2 text-gray-800" @click="toggleMobileMenu">
          <i class="bx bx-menu text-2xl"></i>
        </button>

        <div class="flex items-center gap-4 -ml-0 sm:-ml-5">
          <div
            class="h-6 w-6 sm:h-8 sm:w-8 rounded flex items-center justify-center"
          >
            <img src="/marketplace.png" alt="Logo" />
          </div>

          <div class="hidden sm:flex flex-col text-left">
            <h1 class="text-md font-extrabold text-gray-700 tracking-tight">
              AgriTrade
            </h1>
            <p class="text-sm text-gray-500">SmartSaha Marketplace</p>
          </div>
        </div>
      </div>

      <h1 class="text-xl font-bold">Messagerie</h1>
      <div class="flex items-center gap-4">
        <div class="relative">
          <i class="bx bx-bell text-2xl cursor-pointer"></i>
          <span
            v-if="unreadCount > 0"
            class="absolute -top-1 -right-1 bg-red-600 text-white text-xs w-5 h-5 flex items-center justify-center rounded-full"
          >
            {{ unreadCount }}
          </span>
        </div>

        <div class="flex items-center gap-2 cursor-pointer">
          <button
            @click="userMenuOpen = !userMenuOpen"
            class="flex items-center gap-1 p-1 backdrop-blur-sm rounded transition"
          >
            <div
              class="w-10 h-10 bg-[#f4a261] text-white flex items-center justify-center font-bold rounded-full relative transition"
            >
              {{ user?.username?.charAt(0).toUpperCase() }}
            </div>
            <div class="ml-2 text-left">
              <p class="font-semibold text-gray-700 text-sm">
                {{ user?.username }}
              </p>
              <p class="text-xs text-gray-500">{{ user?.email }}</p>
            </div>
            <i class="bx bx-chevron-down ml-auto text-sm"></i>
          </button>
        </div>
      </div>
    </header>

    <!-- CONTENT -->
    <div class="flex flex-1 overflow-hidden">
      <!-- LEFT SIDEBAR : chats -->
      <aside
        class="w-80 sm:w-72 md:w-80 p-4 border-r border-gray-100 bg-[#112830] overflow-y-auto no-scrollbar"
      >
        <h2 class="p-4 text-lg font-semibold text-gray-800">Chats</h2>

        <div v-if="loadingChats" class="p-4 text-gray-400">
          Loading chats...
        </div>

        <div
          v-for="chat in chats"
          :key="chat.id"
          class="flex items-center gap-3 p-3 rounded cursor-pointer bg-gray-50/5 hover:bg-gray-50/10 mb-1 transition"
          :class="selectedChat?.id === chat.id ? 'bg-gray-100/10' : ''"
          @click="selectChat(chat)"
        >
          <div
            class="w-10 h-10 rounded-full flex items-center justify-center font-bold text-lg"
            :class="getAvatarColor(getOtherUsername(chat))"
          >
            {{ getOtherUsername(chat).charAt(0).toUpperCase() }}
          </div>
          <div class="flex-1 flex flex-col">
            <p class="text-gray-100 font-semibold">
              {{ getOtherUsername(chat) }}
            </p>
            <div
              class="flex justify-between items-center text-sm text-gray-400"
            >
              <span>{{
                truncate(getLastMessage(chat), 20) || "No message yet"
              }}</span>
              <span>{{ formatTime(getLastMessageDate(chat)) }}</span>
            </div>
          </div>
        </div>
      </aside>

      <!-- CENTER : messages -->
      <section class="flex-1 flex flex-col bg-[#fff] overflow-hidden">
        <div
          v-if="!selectedChat"
          class="flex-1 flex items-center justify-center text-gray-400"
        >
          Select a conversation
        </div>

        <div v-else class="flex-1 overflow-y-auto p-4 flex flex-col gap-3 no-scrollbar">
          <div
            v-for="item in sortedChatItems"
            :key="item.id + '-' + item.type"
            :class="item.user.id === userId ? 'self-end' : 'self-start'"
            class="max-w-xs rounded-xl px-4 py-2 text-sm"
            :style="
              item.user.id === userId
                ? 'background:#10b481;color:white;border-bottom-right-radius:0;'
                : 'background:#fefefe;border:1px solid #e5e5e5;border-bottom-left-radius:0;'
            "
          >
            <template v-if="item.type === 'message'">
              <p>{{ item.message }}</p>
            </template>

            <template v-else-if="item.type === 'bid'">
              <div class="max-w-md w-full py-4">
                <div class="flex justify-between items-center mb-2">
                  <div class="flex items-center gap-3">
                    <div
                      class="w-10 h-10 rounded-full flex items-center justify-center text-white font-bold"
                      :class="getAvatarColor(item.user.username)"
                    >
                      {{ item.user.username[0].toUpperCase() }}
                    </div>
                    <div>
                      <p class="font-semibold text-gray-900">
                        {{ item.user.username }}
                      </p>
                      <span class="text-xs text-gray-400">{{
                        formatDate(item.created_at)
                      }}</span>
                    </div>
                  </div>
                  <span class="text-green-700 font-bold"
                    >{{ item.price }} {{ item.currency.symbol }}</span
                  >
                </div>
                <p class="text-gray-700 border-t pt-2 mt-2">
                  {{ item.message }}
                </p>

                <div
                  v-if="post?.user.id === userId"
                  class="flex gap-2 mt-3 flex-wrap"
                >
                  <button
                    @click="acceptBid(item)"
                    class="flex-1 bg-green-600 text-white px-3 py-2 rounded hover:bg-green-700 transition"
                  >
                    Accepter
                  </button>
                  <button
                    @click="declineBid(item)"
                    class="flex-1 bg-gray-100 text-gray-800 px-3 py-2 rounded hover:bg-red-700 transition"
                  >
                    Refuser
                  </button>
                </div>
              </div>
            </template>
          </div>
        </div>

        <!-- INPUT -->
        <div
          v-if="selectedChat"
          class="p-4 border-t border-[#f1f1f1] bg-[#fff] flex gap-2"
        >
          <input
            v-model="newMessage"
            @keyup.enter="sendMessage"
            placeholder="Write a message..."
            class="flex-1 rounded border border-gray-200 px-3 py-2 bg-[#f1f1f1] outline-none"
          />
          <button
            @click="sendMessage"
            class="bg-[#10b481] text-white px-3 py-2 rounded font-semibold flex items-center justify-center"
          >
            <i class="bx bx-send-alt text-xl"></i>
          </button>
        </div>
      </section>

      <!-- RIGHT SIDEBAR : POST -->
      <aside
        v-if="post"
        class="hidden lg:flex w-96 border-l border-[#f1f1f1] bg-[#fff] p-4 overflow-y-auto"
      >
        <div
          class="rounded shadow-sm border border-white/5 overflow-hidden"
        >
          <!-- <img :src="post.image_url" class="w-full h-48 object-cover" /> -->

          <div class="relative">
            <img
              :src="post.image_url"
              alt="Post Image"
              class="w-full h-48 object-cover"
            />
            <div
              class="absolute top-3 left-3 flex items-center gap-3 bg-white/10 backdrop-blur-md px-2 py-1 shadow-sm text-xs text-white"
            >
              <p>Publié le {{ getPublishedDate() }}</p>
              <span class="h-4 w-px bg-gray-300"></span>
              <p class="flex items-center gap-1">
                <i class="bxr bx-location-alt-2"></i>
                <span>{{ post?.location }}</span>
              </p>
            </div>
          </div>

          <div class="p-4 flex flex-col gap-2">
            <h3 class="text-2xl font-bold text-gray-700">
              {{ post?.title }}
            </h3>
            <p class="text-gray-600 text-sm leading-relaxed">
              {{ post?.description }}
            </p>

            <div class="flex flex-wrap gap-3 mt-3">
              <div
                class="flex items-center gap-2 bg-gray-50 border border-gray-100 backdrop-blur-sm text-[#112830] px-3 py-2 rounded shadow-sm text-sm"
              >
                <i class="bx bx-wallet text-base"></i>
                <span
                  >{{ post?.price }} {{ post?.currency?.symbol }}/{{
                    post?.product?.unit?.abbreviation
                  }}</span
                >
              </div>

              <div
                class="flex items-center gap-2 bg-[#10b481]/20 text-[#10b481] px-4 py-2 rounded shadow-sm text-sm font-semibold"
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
      </aside>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import { API_URL } from "~/utils/config";

const chats = ref([]);
const messages = ref([]);
const post = ref(null);
const selectedChat = ref(null);
const newMessage = ref("");
const loadingChats = ref(true);
const username = ref("John Doe");
const userInitials = computed(() =>
  username.value
    .split(" ")
    .map((n) => n[0])
    .join("")
);
const unreadCount = ref(3);

let userId = null;
const user = ref(null);

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

function getAvatarColor(name) {
  const username = name || "unknown"; // fallback si null ou undefined

  let hash = 0;
  for (let i = 0; i < username.length; i++) {
    hash = username.charCodeAt(i) + ((hash << 5) - hash);
  }

  return avatarColors[Math.abs(hash) % avatarColors.length];
}

// Fetch helper
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

// Load chats
async function loadChats() {
  try {
    chats.value = await api(`${API_URL}/api/chats/`);
  } catch (err) {
    console.error(err);
  } finally {
    loadingChats.value = false;
  }
}

// Select chat
async function selectChat(chat) {
  selectedChat.value = chat;
  await loadMessages(chat.id);
  await loadPost(chat.id_post);
}

// Load messages
async function loadMessages(chatId) {
  const data = await api(`${API_URL}/api/messages/?chat=${chatId}`);
  messages.value = await Promise.all(
    data.map(async (msg) => {
      const user = await api(`${API_URL}/api/users/${msg.id_user}/`);
      return { ...msg, user };
    })
  );
}

// Send message
async function sendMessage() {
  if (!newMessage.value.trim()) return;
  const payload = {
    message: newMessage.value,
    id_chat: selectedChat.value.id,
    id_type_message_id: 1,
  };
  const newMsg = await api(`${API_URL}/api/messages/`, "POST", payload);
  const user = await api(`${API_URL}/api/users/${newMsg.id_user}/`);
  messages.value.push({ ...newMsg, user });
  newMessage.value = "";
}

// Load post
async function loadPost(postId) {
  post.value = await api(`${API_URL}/api/posts/${postId}/`);
  console.log("Loaded post:", JSON.stringify(post.value, null, 2));
}

// Format date
function formatDate(dateStr) {
  const date = new Date(dateStr),
    now = new Date(),
    diff = now - date,
    oneDay = 24 * 60 * 60 * 1000;
  if (date.toDateString() === now.toDateString())
    return date.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
  else if (diff < 2 * oneDay)
    return (
      "Hier " +
      date.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })
    );
  else
    return (
      date.toLocaleDateString() +
      " " +
      date.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })
    );
}

// Get other user
function getOtherUserId(chat) {
  const msg = messages.value.find(
    (m) => m.id_chat === chat.id && m.id_user !== userId
  );
  return msg ? msg.id_user : null;
}

function getOtherUsername(chat) {
  const otherId = getOtherUserId(chat);
  const msg = messages.value.find((m) => m.id_user === otherId);
  return msg?.user?.username || "Unknown";
}

function getLastMessage(chat) {
  const chatMessages = messages.value.filter((m) => m.id_chat === chat.id);
  if (!chatMessages.length) return null;
  return chatMessages[chatMessages.length - 1].message;
}

// Tronquer un texte et ajouter "..." si trop long
function truncate(text, maxLength) {
  if (!text) return "";
  return text.length > maxLength ? text.slice(0, maxLength) + "..." : text;
}

// Récupérer la date du dernier message
function getLastMessageDate(chat) {
  const chatMessages = messages.value.filter((m) => m.id_chat === chat.id);
  if (!chatMessages.length) return null;
  return chatMessages[chatMessages.length - 1].created_at;
}

// Formater uniquement l'heure HH:MM
function formatTime(dateStr) {
  if (!dateStr) return "";
  const date = new Date(dateStr);
  return date.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
}

// Compute sorted chat items (messages + bids)
const sortedChatItems = computed(() => {
  if (!selectedChat.value) return [];

  const msgs = messages.value
    .filter((m) => m.id_chat === selectedChat.value.id)
    .map((m) => ({ ...m, type: "message" }));

  const otherId = getOtherUserId(selectedChat.value);

  const bids =
    post.value?.active_bids
      ?.filter((b) => b.user.id === otherId)
      .map((b) => ({ ...b, type: "bid" })) || [];

  return [...msgs, ...bids].sort(
    (a, b) => new Date(a.created_at) - new Date(b.created_at)
  );
});

// Check current user
async function checkUser() {
  const token = localStorage.getItem("access_token");
  if (!token) return;
  try {
    const res = await fetch(`${API_URL}/api/me/`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    if (!res.ok) throw new Error("Unauthorized");
    const data = await res.json();
    userId = data.id;
    user.value = data;
  } catch {}
}

const getPublishedDate = () => {
  const history = post.value?.status_history;
  if (!history) return null;

  const published = history.find((h) => h.status === "published");

  return published ? formatDate(published.date_changed) : null;
};
// INIT
onMounted(() => {
  const token = localStorage.getItem("access_token");
  loadChats();
  checkUser();
});

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
    await loadPost(post.value.id); // rafraîchir la page ou le post
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

    // Tu peux récupérer le statut et les détails ici
    if (res.status >= 400) {
      console.warn("Erreur côté API :", res.status, res.data || res);
      alert(
        "Erreur lors du refus de l'enchère : " + JSON.stringify(res.data || res)
      );
      return;
    }

    await loadPost(bid.post_id);

    if (!continueNegotiation) {
      console.warn("La négociation est terminée avec l'acheteur.");
      return;
    }
    console.warn("La négociation continue avec l'acheteur.");
  } catch (err) {
    // Affiche toute l'info retournée par Django
    console.error("Erreur declineBid :", err.response?.data || err);
    alert(
      "Erreur lors du refus de l'enchère : " +
        JSON.stringify(err.response?.data || err)
    );
  }
}
</script>

<style scoped>
/* Chrome, Safari, Edge */
.no-scrollbar::-webkit-scrollbar {
  display: none;
}

/* Firefox */
.no-scrollbar {
  scrollbar-width: none;
}

</style>