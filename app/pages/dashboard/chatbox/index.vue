<template>
  <div class="flex flex-col h-screen">
    <header
      class="flex items-center justify-between bg-[#fff] shadow px-12 py-3 sticky top-0 z-50"
    >
      <div class="flex items-center gap-2">
        <button class="sm:hidden p-2 text-gray-800" @click="toggleMobileMenu">
          <i class="bx bx-menu text-2xl"></i>
        </button>

        <div class="flex items-center gap-2">
          <button class="sm:hidden p-2 text-gray-800" @click="toggleMobileMenu">
            <i class="bx bx-menu text-2xl"></i>
          </button>

          <NuxtLink to="/market" class="flex items-center gap-3">
            <img
              src="/marketplace_png.png"
              alt="Logo"
              class="h-10 w-auto rounded-md"
            />
            <div class="leading-tight">
              <h1 class="text-xl font-semibold text-gray-800">AgriTrade</h1>
              <p class="text-xs text-gray-500 tracking-wide">
                SmartSaha Marketplace
              </p>
            </div>
          </NuxtLink>
        </div>
      </div>

      <div class="flex items-center gap-4">
        <div class="flex items-center gap-2 cursor-pointer">
          <div class="relative">
            <div
              class="flex items-center gap-2 p-2 pl-1 rounded-lg cursor-pointer"
              @click="toggleDropdown"
            >
              <div
                class="w-10 h-10 bg-[#10b481] text-white flex items-center justify-center font-semibold rounded-full"
              >
                {{ user?.username?.charAt(0).toUpperCase() }}
              </div>

              <div>
                <p class="text-sm font-semibold text-gray-800">
                  {{ user?.username }}
                </p>
                <p class="text-xs text-gray-500">{{ user?.email }}</p>
              </div>

              <i class="bx bx-chevron-down text-sm"></i>
            </div>

            <ul
              v-if="dropdownOpen"
              class="absolute right-0 mt-2 w-48 bg-white border border-gray-100 rounded-lg shadow-md overflow-hidden"
            >
              <li class="px-4 py-2 text-sm hover:bg-gray-50">
                <NuxtLink to="/dashboard">{{ t("dashboard") }}</NuxtLink>
              </li>
              <li class="px-4 py-2 text-sm hover:bg-gray-50">
                <button @click="logout">{{ t("logout") }}</button>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </header>

    <div class="flex flex-1 overflow-hidden">
      <aside
        class="w-80 sm:w-72 md:w-80 p-4 border-r border-gray-100 bg-[#112830] overflow-y-auto no-scrollbar"
      >
        <button
          @click="$router.back()"
          class="flex items-center gap-1 text-gray-100/60 transition"
        >
          <i class="bxr bx-chevron-left text-xl"></i>
          {{ t("back") }}
        </button>

        <h2 class="p-4 text-lg font-semibold text-gray-100">
          {{ t("chats") }}
        </h2>

        <div
          class="px-4 mb-6 flex items-center gap-2 relative bg-[#1f3945] rounded"
        >
          <i class="bx bx-search text-gray-400 text-lg"></i>

          <input
            v-model="searchQuery"
            placeholder="Search user..."
            class="w-full py-2 bg-transparent text-gray-200 placeholder-gray-400 outline-none"
          />
        </div>

        <div v-if="loadingChats" class="p-4 text-gray-400">
          <div
            class="w-8 h-8 border-4 border-gray-300 border-t-[#10b481] rounded-full animate-spin"
          ></div>
        </div>

        <div
          v-for="chat in filteredChats"
          :key="chat.id"
          @click="selectChat(chat)"
          class="group relative flex items-center gap-3 px-4 py-3 mb-1 rounded cursor-pointer transition-all"
          :class="[
            selectedChat?.id === chat.id
              ? 'bg-white/10 ring-1 ring-[#10b481]/30'
              : getUnreadCount(chat) > 0
              ? 'bg-white/5 hover:bg-white/10'
              : 'hover:bg-white/5',
          ]"
        >
          <span
            v-if="selectedChat?.id === chat.id"
            class="absolute left-0 top-1/2 -translate-y-1/2 w-1 h-8 rounded-r bg-[#10b481]"
          ></span>

          <div
            class="relative w-10 h-10 rounded-full flex items-center justify-center font-bold text-lg shrink-0"
            :class="getAvatarColor(getOtherUsername(chat))"
          >
            {{ getOtherUsername(chat).charAt(0).toUpperCase() }}
          </div>

          <div class="flex-1 min-w-0">
            <div class="flex justify-between items-center">
              <p
                class="truncate"
                :class="
                  getUnreadCount(chat) > 0
                    ? 'font-semibold text-white'
                    : 'text-gray-200 font-medium'
                "
              >
                {{ getOtherUsername(chat) }}
              </p>

              <span class="text-xs text-gray-400 shrink-0">
                {{ formatTime(getLastMessageDate(chat)) }}
              </span>
            </div>

            <div class="flex justify-between items-center gap-2">
              <p
                class="text-xs truncate flex-1"
                :class="
                  getUnreadCount(chat) > 0
                    ? 'text-gray-100 font-medium'
                    : 'text-gray-400'
                "
              >
                {{ truncate(getLastMessage(chat), 28) || "No message yet" }}
              </p>

              <span
                v-if="getUnreadCount(chat) > 0"
                class="min-w-[20px] h-5 px-1.5 flex items-center justify-center text-xs font-bold rounded-full bg-[#10b481] text-white shrink-0"
              >
                {{ getUnreadCount(chat) }}
              </span>
            </div>
          </div>
        </div>
      </aside>

      <section class="flex-1 flex flex-col bg-[#fff] overflow-hidden">
        <div
          v-if="selectedChat"
          class="flex items-center justify-between gap-3 p-4 border-b border-gray-200 sticky top-0 bg-white z-10"
        >
          <div class="flex items-center gap-3">
            <div
              class="w-10 h-10 rounded-full flex items-center justify-center font-bold text-lg"
              :class="getAvatarColor(getOtherUsername(selectedChat))"
            >
              {{ getOtherUsername(selectedChat).charAt(0).toUpperCase() }}
            </div>
            <div class="flex flex-col">
              <p class="text-gray-800 font-semibold text-sm">
                {{ getOtherUsername(selectedChat) }}
              </p>
              <p class="text-gray-500 text-xs">
                {{ getOtherUserEmail(selectedChat) }}
              </p>
            </div>
          </div>

          <div class="relative">
            <button @click="toggleChatMenu">
              <i class="bx bx-dots-vertical-rounded text-xl"></i>
            </button>

            <div
              v-if="chatMenuOpen"
              class="absolute right-0 mt-2 w-40 bg-white border rounded z-20"
            >
              <button
                @click="
                  confirmDelete(selectedChat);
                  chatMenuOpen = false;
                "
                class="w-full text-left px-4 py-2 text-red-600"
              >
                {{ t("deleteChat") }}
              </button>
            </div>
          </div>

          <div
            v-if="confirmDeleteOpen"
            class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-30"
          >
            <div class="bg-white rounded p-6 w-96">
              <h3 class="text-lg font-semibold mb-4">
                {{ t("confirmDelete") }}
              </h3>
              <p class="mb-6">
                {{ t("confirmDelText") }}
              </p>
              <div class="flex justify-end gap-3">
                <button
                  @click="cancelDelete"
                  class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300"
                >
                  {{ t("cancel") }}
                </button>
                <button
                  @click="performDelete"
                  class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
                >
                  {{ t("delete") }}
                </button>
              </div>
            </div>
          </div>
        </div>
        <div
          v-if="acceptedBidNotification"
          class="flex items-center justify-between gap-3 p-2 px-6 border-b border-yellow-200 sticky top-0 bg-yellow-100 z-10"
        >
          <div class="flex-1">
            <p class="text-yellow-800 font-semibold text-sm mb-1">
              {{ t("acceptedBid") }}
            </p>
            <p class="text-yellow-700 text-xs">
              {{ t("bidOf") }}
              <span class="font-medium"
                >{{ acceptedBidNotification.price }}
                {{ acceptedBidNotification.currency_symbol }}</span
              >
              {{ t("acceptedAt") }}
              {{ formatDateTime(acceptedBidNotification.created_at) }}
            </p>
          </div>
          <button
            @click="acceptedBidNotification = null"
            class="text-gray-500 text-md font-bold ml-3"
            aria-label="Fermer notification"
          >
            <i class="bxr bx-x"></i>
          </button>
        </div>

        <div
          v-if="!selectedChat"
          class="flex-1 flex items-center justify-center text-gray-400"
        ></div>

        <div
          v-else
          class="flex-1 overflow-y-auto flex flex-col gap-3 no-scrollbar"
        >
          <div
            v-for="item in sortedChatItems"
            :key="item.id + '-' + item.type"
            :class="item.user.id === userId ? 'self-end' : 'self-start'"
            class="max-w-xs rounded-xl px-4 py-2 text-sm"
          >
            <template v-if="item.type === 'message'">
              <div
                :class="item.user.id === userId ? 'self-end' : 'self-start'"
                class="max-w-xs rounded-xl px-4 py-2 text-sm"
                :style="
                  item.user.id === userId
                    ? 'background:#10b481;color:white;border-bottom-right-radius:0;'
                    : 'background:#f9f9f9;border:1px solid #e5e5e5;border-bottom-left-radius:0;'
                "
              >
                <p class="break-words">{{ item.message }}</p>
                <span class="text-xs mt-1">{{
                  formatDate(item.created_at)
                }}</span>
              </div>
            </template>

            <template v-else-if="item.type === 'bid'">
              <div
                class="w-72 rounded shadow-lg border-l-4 border-[#10b481] bg-gray-50 p-4 mb-3 self-start"
              >
                <div class="flex items-center gap-3 mb-2">
                  <div
                    class="w-10 h-10 rounded-full flex items-center justify-center font-bold"
                    :class="getAvatarColor(item.user.username)"
                  >
                    {{ item.user.username[0].toUpperCase() }}
                  </div>
                  <div>
                    <p class="font-semibold">{{ item.user.username }}</p>
                    <span class="text-xs text-gray-500">{{
                      formatDate(item.created_at)
                    }}</span>
                  </div>
                </div>

                <div class="mt-2 border-t border-gray-200 pt-2">
                  <!-- <p class="text-gray-700 mb-1">{{ item.message }}</p> -->
                  <p class="font-bold text-lg">
                    {{ item.price }} {{ item.currency.symbol }}
                  </p>
                </div>

                <div
                  v-if="
                    post?.user.id === userId &&
                    (item.current_status?.name || '').toLowerCase() ===
                      'proposée'
                  "
                  class="flex gap-2 mt-3 flex-wrap"
                >
                  <button
                    @click="openBidConfirmation(item, 'accept')"
                    class="flex-1 bg-[#10b481] text-white px-3 py-2 rounded hover:bg-green-700 transition"
                  >
                    Accepter
                  </button>
                  <button
                    @click="openBidConfirmation(item, 'decline')"
                    class="flex-1 border border-gray-400 text-gray-800 px-3 py-2 rounded bg-white transition"
                  >
                    Refuser
                  </button>
                </div>
              </div>
            </template>
          </div>
        </div>

        <div
          v-if="selectedChat"
          class="p-4 border-t border-[#f1f1f1] bg-[#fff] flex gap-2"
        >
          <button
            v-if="post?.current_status !== 'vendu' && post?.user?.id !== userId"
            @click="openCreateBidModal"
            class="w-12 h-full flex items-center justify-center rounded border border-gray-200 px-3 py-2 bg-gray-100 outline-none"
          >
            <i class="bx bx-plus text-xl text-gray-600"></i>
          </button>

          <div
            class="flex flex-1 items-center gap-3 rounded border border-gray-200 px-3 py-2 bg-gray-100 relative"
          >
            <button
              @click="showEmojiPicker = !showEmojiPicker"
              class="items-center justify-center outline-none"
            >
              <i class="bxr bx-smile text-lg text-gray-500"></i>
            </button>

            <emoji-picker
              v-if="showEmojiPicker"
              @emoji-click="addEmoji"
              class="absolute bottom-20 left-84 z-50 rounded shadow-lg"
              style="width: 300px; height: 250px"
              theme="light"
            ></emoji-picker>

            <input
              v-model="newMessage"
              @keyup.enter="sendMessage"
              placeholder="Write a message..."
              class="flex-1 bg-transparent outline-none"
            />
          </div>

          <button
            @click="sendMessage"
            class="bg-[#10b481] text-white px-3 py-2 rounded font-semibold flex items-center justify-center"
          >
            <i class="bx bx-send-alt text-xl"></i>
          </button>
        </div>
      </section>

      <aside
        v-if="post"
        class="hidden lg:flex w-96 border-l border-[#f1f1f1] bg-[#fff] p-4 overflow-y-auto"
      >
        <div class="rounded shadow-sm border border-white/5 overflow-hidden">
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
                <span>{{ post?.price }} {{ post?.currency?.symbol }}</span>
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
          Annuler
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
          {{ bidAction === "accept" ? t("btnaccept") : t("btndecline") }}
        </button>
      </div>
    </div>
  </div>
  <div
    v-if="openBidModal"
    class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-30"
  >
    <div class="bg-white rounded p-6 w-96">
      <label class="block text-sm mb-1">{{ t("priceProposed") }} :</label>
      <input
        type="number"
        v-model="bidPrice"
        class="w-full border rounded p-2 mb-4"
        placeholder="Montant en Ariary"
      />

      <label class="block text-sm mb-1">{{ t("msg") }} :</label>
      <textarea
        v-model="bidMessage"
        class="w-full border rounded p-2 mb-4"
        rows="3"
        placeholder="Votre message..."
      ></textarea>

      <div class="flex justify-end gap-3">
        <button
          @click="openBidModal = false"
          class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300"
        >
          {{ t("cancel") }}
        </button>

        <button
          @click="submitBid"
          class="px-4 py-2 bg-[#10b481] text-white rounded hover:bg-green-700"
        >
          {{ t("send") }}
        </button>
      </div>
    </div>
  </div>

  <div
    v-if="reviewPopupOpen"
    class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50"
  >
    <div class="bg-white rounded p-6 w-96 max-w-full">
      <h3 class="text-lg font-semibold mb-4">Évaluez votre transaction</h3>

      <div class="flex gap-1 mb-4">
        <template v-for="i in 5" :key="i">
          <button
            @click="reviewRating = i"
            class="text-2xl"
            :class="i <= reviewRating ? 'text-yellow-400' : 'text-gray-300'"
          >
            <i class="bxr bxs-star"></i>
          </button>
        </template>
      </div>

      <label class="block text-sm mb-1">Commentaire :</label>
      <textarea
        v-model="reviewComment"
        class="w-full border rounded p-2 mb-4"
        rows="3"
        placeholder="Optionnel..."
      ></textarea>

      <div class="flex justify-end gap-3">
        <button
          @click="reviewPopupOpen = false"
          class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300"
        >
          {{ t("cancel") }}
        </button>
        <button
          @click="submitReview"
          class="px-4 py-2 bg-[#10b481] text-white rounded hover:bg-green-700"
        >
          {{ t("send") }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import { API_URL } from "~/utils/config";
import { useRoute } from "vue-router";
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";
import { useRouter } from "vue-router";
const router = useRouter();

const languageStore = useLanguageStore();
const t = (key) => {
  const lang = languageStore.lang;
  return translate[lang][key] || key;
};

const route = useRoute();

const chats = ref([]);
const messages = ref([]);
const post = ref(null);
const selectedChat = ref(null);
const newMessage = ref("");
const loadingChats = ref(true);
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
  const username = name || "unknown";

  let hash = 0;
  for (let i = 0; i < username.length; i++) {
    hash = username.charCodeAt(i) + ((hash << 5) - hash);
  }

  return avatarColors[Math.abs(hash) % avatarColors.length];
}

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

async function loadChats() {
  try {
    chats.value = await api(`${API_URL}/api/chats/`);

    const postId = route.query.post;
    const bidId = route.query.bid;

    if (postId && bidId) {
      const chatToSelect = chats.value.find(
        (chat) =>
          chat.id_post == postId && chat.active_bids?.some((b) => b.id == bidId)
      );
      if (chatToSelect) {
        await selectChat(chatToSelect);
        return;
      }
    }

    if (chats.value.length > 0 && !selectedChat.value) {
      await selectChat(chats.value[0]);
    }
  } catch (err) {
    console.error(err);
  } finally {
    loadingChats.value = false;
  }
}

async function selectChat(chat) {
  selectedChat.value = chat;
  await loadMessages(chat.id);
  await markChatAsRead(chat.id);
  await loadPost(chat.id_post);
}

async function loadMessages(chatId) {
  const data = await api(`${API_URL}/api/messages/?chat=${chatId}`);
  messages.value = await Promise.all(
    data.map(async (msg) => {
      const user = await api(`${API_URL}/api/users/${msg.id_user}/`);
      return { ...msg, user };
    })
  );
}

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

async function loadPost(postId) {
  post.value = await api(`${API_URL}/api/posts/${postId}/`);
  console.log("Loaded post:", JSON.stringify(post.value, null, 2));
  checkAcceptedBid(post.value, selectedChat.value);
  openReview(post.value, selectedChat.value);
}

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

function getOtherUserId(chat) {
  if (!chat) return null;

  const otherMsg = messages.value.find(
    (m) => m.id_chat === chat.id && m.id_user !== userId
  );
  if (otherMsg) return otherMsg.id_user;

  if (post.value?.user?.id && post.value.user.id !== userId) {
    return post.value.user.id;
  }

  if (chat.user_id && chat.user_id !== userId) {
    return chat.user_id;
  }

  return null;
}

function getOtherUsername(chat) {
  if (!chat) return "";

  const otherId = getOtherUserId(chat);
  if (!otherId) return "";

  const msg = messages.value.find((m) => m.id_user === otherId);
  return msg?.user?.username || "";
}

function getOtherUserEmail(chat) {
  const otherId = getOtherUserId(chat);
  const msg = messages.value.find((m) => m.id_user === otherId);
  return msg?.user?.email || "Unknown email";
}

function getLastMessage(chat) {
  const chatMessages = messages.value.filter((m) => m.id_chat === chat.id);
  if (!chatMessages.length) return null;
  return chatMessages[chatMessages.length - 1].message;
}

function truncate(text, maxLength) {
  if (!text) return "";
  return text.length > maxLength ? text.slice(0, maxLength) + "..." : text;
}

function getLastMessageDate(chat) {
  const chatMessages = messages.value.filter((m) => m.id_chat === chat.id);
  if (!chatMessages.length) return null;
  return chatMessages[chatMessages.length - 1].created_at;
}

function formatTime(dateStr) {
  if (!dateStr) return "";
  const date = new Date(dateStr);
  return date.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
}

const sortedChatItems = computed(() => {
  if (!selectedChat.value) return [];

  const msgs = messages.value
    .filter((m) => m.id_chat === selectedChat.value.id)
    .map((m) => ({ ...m, type: "message" }));

  const otherId = getOtherUserId(selectedChat.value);

  const bids =
    post.value?.active_bids
      ?.filter((b) => b.user.id === userId || b.user.id === otherId)
      .map((b) => ({ ...b, type: "bid" })) || [];

  return [...msgs, ...bids].sort(
    (a, b) => new Date(a.created_at) - new Date(b.created_at)
  );
});

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

onMounted(() => {
  const token = localStorage.getItem("access_token");
  loadChats();
  checkUser();
});

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
  await loadPost(post.value.id);
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
    await loadPost(post.value.id);
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

    await loadPost(bid.post_id);

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

const chatMenuOpen = ref(false);
const confirmDeleteOpen = ref(false);
const chatToDelete = ref(null);

function toggleChatMenu() {
  chatMenuOpen.value = !chatMenuOpen.value;
}

function confirmDelete(chat) {
  chatToDelete.value = chat;
  confirmDeleteOpen.value = true;
}

async function performDelete() {
  if (!chatToDelete.value) return;

  try {
    const token = localStorage.getItem("access_token");
    const res = await fetch(`${API_URL}/api/chats/${chatToDelete.value.id}/`, {
      method: "DELETE",
      headers: { Authorization: `Bearer ${token}` },
    });

    if (!res.ok) throw new Error(`Erreur suppression chat : ${res.status}`);

    chats.value = chats.value.filter((c) => c.id !== chatToDelete.value.id);
    if (selectedChat.value?.id === chatToDelete.value.id) {
      selectedChat.value = null;
      messages.value = [];
    }

    console.log("Chat supprimé avec succès !");
  } catch (err) {
    console.error("Erreur suppression chat :", err);
  } finally {
    confirmDeleteOpen.value = false;
    chatToDelete.value = null;
  }
}

function cancelDelete() {
  confirmDeleteOpen.value = false;
  chatToDelete.value = null;
}

const acceptedBidNotification = ref(null);

function checkAcceptedBid(post, selectedChat) {
  if (!post || !selectedChat) {
    acceptedBidNotification.value = null;
    return;
  }

  const acceptedBid = post.accepted_bid;

  if (
    !acceptedBid ||
    acceptedBid.current_status?.name?.toLowerCase() !== "acceptée"
  ) {
    acceptedBidNotification.value = null;
    return;
  }

  const otherId = getOtherUserId(selectedChat);
  const bidsInChat =
    post.active_bids?.filter((b) => b.user.id === otherId).map((b) => b.id) ||
    [];

  if (bidsInChat.includes(acceptedBid.id)) {
    acceptedBidNotification.value = {
      price: acceptedBid.price,
      currency_symbol: acceptedBid.currency.symbol,
      created_at: acceptedBid.created_at,
      username: acceptedBid.user.username,
    };
  } else {
    acceptedBidNotification.value = null;
  }
}

function formatDateTime(dateStr) {
  const d = new Date(dateStr);
  return d.toLocaleString();
}

const openBidModal = ref(false);
const bidPrice = ref(null);
const bidMessage = ref("");
const loadingSubmitBid = ref(false);

function openCreateBidModal() {
  if (!post.value) {
    return alert("Post non chargé, réessaye.");
  }
  openBidModal.value = true;
}

async function submitBid() {
  if (!bidPrice.value) {
    alert("Veuillez entrer un prix.");
    return;
  }
  if (!post.value || !post.value.id) {
    alert("Impossible d'envoyer : post non trouvé.");
    return;
  }

  const token = localStorage.getItem("access_token");
  if (!token) {
    window.location.href = "/signin";
    return;
  }

  loadingSubmitBid.value = true;

  try {
    const payload = {
      price: bidPrice.value,
      message: bidMessage.value,
    };

    const bidResponse = await fetch(
      `${API_URL}/api/posts/${post.value.id}/place_bid/`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify(payload),
      }
    );

    if (!bidResponse.ok) {
      const err = await bidResponse.json().catch(() => ({}));
      alert("Erreur : " + JSON.stringify(err));
      return;
    }

    bidPrice.value = null;
    bidMessage.value = "";
    openBidModal.value = false;

    await loadPost(post.value.id);
    if (selectedChat.value) await loadMessages(selectedChat.value.id);
  } catch (err) {
    console.error(err);
    alert("Erreur lors de l’envoi de l’enchère.");
  } finally {
    loadingSubmitBid.value = false;
  }
}

const logout = () => {
  localStorage.removeItem("access_token");
  window.location.href = "/signin";
};
const dropdownOpen = ref(false);

function toggleDropdown() {
  dropdownOpen.value = !dropdownOpen.value;
}

const reviewPopupOpen = ref(false);
const reviewRating = ref(0);
const reviewComment = ref("");

function openReview(post, selectedChat) {
  if (!post || !selectedChat) {
    reviewPopupOpen.value = false;
    return;
  }

  const acceptedBid = post.accepted_bid;

  if (
    !acceptedBid ||
    acceptedBid.current_status?.name?.toLowerCase() !== "acceptée"
  ) {
    reviewPopupOpen.value = false;
    return;
  }

  const otherId = getOtherUserId(selectedChat);

  if (acceptedBid.user.id !== otherId && acceptedBid.user.id !== userId) {
    reviewPopupOpen.value = false;
    return;
  }

  reviewRating.value = 0;
  reviewComment.value = "";
  reviewPopupOpen.value = true;
}

async function submitReview() {
  if (!reviewRating.value) return alert("Veuillez donner une note !");

  try {
    const payload = {
      id_user_to: getOtherUserId(selectedChat.value),
      rating: reviewRating.value,
      comment: reviewComment.value,
    };
    console.log("Payload", payload);

    await api(`${API_URL}/api/reviews/`, "POST", payload);
    reviewPopupOpen.value = false;

    alert("Merci pour votre évaluation !");
  } catch (err) {
    console.error("Erreur review :", err);
    alert("Impossible d'envoyer la note.");
  }
}

function getUnreadCount(chat) {
  return messages.value.filter(
    (m) => m.id_chat === chat.id && !m.is_read && m.id_user !== userId
  ).length;
}

async function markChatAsRead(chatId) {
  try {
    await api(`${API_URL}/api/messages/mark_as_read/`, "POST", {
      chat_id: chatId,
    });

    messages.value = messages.value.map((m) =>
      m.id_chat === chatId && m.id_user !== userId ? { ...m, is_read: true } : m
    );
  } catch (err) {
    console.error("Erreur mark as read", err);
  }
}

const showEmojiPicker = ref(false);

function addEmoji(event) {
  newMessage.value += event.detail.unicode;
  showEmojiPicker.value = false;
}

const searchQuery = ref("");

const filteredChats = computed(() => {
  if (!searchQuery.value) return chats.value;

  return chats.value.filter((chat) => {
    const username = getOtherUsername(chat).toLowerCase();
    return username.includes(searchQuery.value.toLowerCase());
  });
});
</script>

<style scoped>
.no-scrollbar::-webkit-scrollbar {
  display: none;
}

.no-scrollbar {
  scrollbar-width: none;
}

emoji-picker {
  --background-color: #1f2937;
  --button-background-color: #10b481;
  --button-hover-background-color: #0ea371;
  --button-color: #fff;
  --font-size: 1.2rem;
  --border-radius: 12px;
}
</style>
