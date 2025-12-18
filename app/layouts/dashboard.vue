<template>
  <div class="min-h-screen flex flex-col overflow-x-hidden">
    <header
      class="fixed top-0 left-0 right-0 z-50 flex items-center px-4 sm:px-8 h-20 bg-[#fefefe] backdrop-blur-md shadow-sm"
    >
      <div class="flex items-center gap-2">
        <button class="sm:hidden p-2 text-gray-800" @click="toggleMobileMenu">
          <i class="bx bx-menu text-2xl"></i>
        </button>

        <NuxtLink to="/" class="flex items-center gap-3">
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

      <div class="hidden sm:flex items-center gap-6 ml-auto">
        <div class="relative">
          <button
            @click="open = !open"
            class="flex items-center gap-2 py-1.5 px-3 transition"
          >
            <img :src="currentLocale.flag" class="w-5 h-5 rounded-full" />
            <span class="text-sm font-medium">{{ currentLocale.name }}</span>
            <i class="bx bx-chevron-down text-sm"></i>
          </button>

          <transition name="fade">
            <ul
              v-if="open"
              class="absolute mt-2 w-40 bg-white border border-gray-100 rounded-lg shadow-md overflow-hidden"
            >
              <li
                v-for="loc in locales"
                :key="loc.code"
                @click="selectLocale(loc.code)"
                class="flex items-center gap-2 px-3 py-2 hover:bg-[#10b481]/10 cursor-pointer"
              >
                <img :src="loc.flag" class="w-5 h-5 rounded-full" />
                <span class="text-sm">{{ loc.name }}</span>
              </li>
            </ul>
          </transition>
        </div>

        <div class="flex items-center gap-4">
          <div class="relative">
            <button
              @click="goToNotif"
              class="relative w-11 h-11 flex items-center justify-center rounded-full hover:bg-gray-100 transition focus:outline-none focus:ring-2 focus:ring-[#10b481]/30"
            >
              <i class="bx bx-bell text-xl text-gray-700"></i>

              <span
                v-if="unreadNotifCount > 0"
                class="absolute -top-0.5 -right-0.5 min-w-[18px] h-[18px] px-1 flex items-center justify-center text-[10px] font-semibold bg-red-600 text-white rounded-full shadow"
              >
                {{ unreadNotifCount > 9 ? "9+" : unreadNotifCount }}
              </span>
            </button>
          </div>

          <div class="relative">
            <button
              @click="goToChat"
              class="relative w-11 h-11 flex items-center justify-center rounded-full hover:bg-gray-100 transition focus:outline-none focus:ring-2 focus:ring-[#10b481]/30"
            >
              <i class="bx bx-message-dots text-xl text-gray-700"></i>

              <span
                v-if="unreadCount > 0"
                class="absolute -top-0.5 -right-0.5 min-w-[18px] h-[18px] px-1 flex items-center justify-center text-[10px] font-semibold bg-red-600 text-white rounded-full shadow"
              >
                {{ unreadCount > 9 ? "9+" : unreadCount }}
              </span>
            </button>
          </div>
        </div>

        <div class="relative">
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
                <button @click="goToDashboard" class="w-full text-left">
                  {{ t("dashboard") }}
                </button>
              </li>
              <li class="px-4 py-2 text-sm hover:bg-gray-50">
                <button @click="logout">{{ t("logout") }}</button>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </header>

    <transition name="slide">
      <aside
        v-if="isMobileMenuOpen"
        class="fixed inset-0 z-50 bg-[#112830]/95 backdrop-blur-sm flex flex-col sm:hidden"
      >
        <div
          class="flex justify-between items-center p-4 border-b border-gray-700"
        >
          <h2 class="text-white font-bold text-lg">
            <div class="flex items-center gap-3">
              <div
                class="h-10 w-10 rounded-full bg-gradient-to-br from-[#f4a261] to-[#f4a261] flex items-center justify-center"
              >
                <i class="bx bxs-user text-white text-lg"></i>
              </div>
              <div class="flex flex-col">
                <p class="text-white font-semibold text-sm">
                  {{ user?.username }}
                </p>
                <p class="text-gray-300 text-xs font-light">
                  {{ user?.email }}
                </p>
              </div>
            </div>
          </h2>
          <button @click="toggleMobileMenu" class="text-white text-2xl">
            <i class="bx bx-x"></i>
          </button>
        </div>

        <div class="flex flex-col gap-4 px-4 py-4 border-b border-gray-700">
          <div>
            <select
              v-model="languageStore.lang"
              class="w-full p-2 rounded bg-[#112830] text-white"
            >
              <option v-for="loc in locales" :key="loc.code" :value="loc.code">
                {{ loc.name }}
              </option>
            </select>
          </div>
        </div>

        <nav class="flex-1 overflow-y-auto py-4 px-2 flex flex-col gap-2">
          <button
            v-for="item in sidebarLinks"
            :key="item.to"
            @click="
              router.push(item.to);
              toggleMobileMenu();
            "
            class="flex items-center gap-3 px-4 py-3 text-white hover:bg-[#10b481]/20 rounded w-full text-left"
          >
            <i :class="item.icon + ' text-xl'"></i>
            <span>{{ item.label }}</span>
          </button>
          <button
            @click="router.push('/help')"
            class="flex items-center gap-3 px-4 py-3 text-white hover:bg-[#10b481]/20 rounded w-full text-left"
          >
            <i class="bxr bx-info-circle text-xl text-white"></i>
            <span>{{ t("help") }}</span>
          </button>
          <button
            @click="logout"
            class="flex items-center gap-3 px-3 py-2 hover:bg-red-500/20 rounded text-white"
          >
            <i class="bx-light bx bx-log-out text-xl"></i>
            <span>{{ t("logout") }}</span>
          </button>
        </nav>
      </aside>
    </transition>

    <div class="flex flex-1 pt-16">
      <aside
        class="hidden sm:flex peer group fixed top-20 left-0 h-[calc(100vh-4rem)] flex-col justify-between bg-[#112830] shadow-lg w-20 hover:w-56 transition-all duration-300 z-40"
      >
        <nav class="flex flex-col space-y-2 py-4">
          <template v-for="item in sidebarLinks" :key="item.label">
            <div v-if="item.dropdown" class="relative">
              <button
                @click="
                  dropdownStates[item.label] = !dropdownStates[item.label]
                "
                class="group relative flex items-center gap-3 px-4 py-2 text-white transition-all duration-300 hover:bg-white/10"
              >
                <div class="flex items-center gap-3">
                  <i :class="item.icon + ' text-xl ml-2 font-light'"></i>
                  <span
                    class="font-light opacity-0 group-hover:opacity-100 transition-opacity duration-300"
                  >
                    {{ item.label }}
                  </span>
                </div>
                <i
                  :class="
                    dropdownStates[item.label]
                      ? 'bx bx-chevron-up'
                      : 'bx bx-chevron-down'
                  "
                  class="opacity-0 group-hover:opacity-100 transition-opacity duration-300"
                ></i>
              </button>

              <div
                v-if="dropdownStates[item.label]"
                class="ml-2 flex flex-col space-y-1 mt-1 overflow-hidden transition-all duration-300"
              >
                <button
                  v-for="sub in item.dropdown"
                  :key="sub.label"
                  @click="router.push(sub.to)"
                  class="flex items-center gap-2 px-4 py-2 text-white hover:bg-white/20 rounded w-full text-left"
                >
                  <i :class="sub.icon + ' text-lg'"></i>
                  <span
                    class="font-light opacity-0 group-hover:opacity-100 transition-opacity duration-300"
                    >{{ sub.label }}</span
                  >
                </button>
              </div>
            </div>

            <button
              v-else
              @click="router.push(item.to)"
              class="group relative flex items-center gap-3 px-4 py-2 text-white transition-all duration-300 hover:bg-white/10"
            >
              <span
                class="absolute left-0 top-1/2 -translate-y-1/2 h-[60%] border-l-[3px] border-white bg-white/10 opacity-0 hover:opacity-100 group-focus:opacity-100 transition-all duration-300"
              ></span>
              <i :class="item.icon + ' text-xl ml-2 font-light'"></i>
              <span
                class="font-light opacity-0 group-hover:opacity-100 transition-opacity duration-300"
                >{{ item.label }}</span
              >
            </button>
          </template>
        </nav>
      </aside>

      <main
        class="flex-1 sm:ml-20 transition-all duration-300 peer-hover:ml-56"
      >
        <slot />
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
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

const locales = [
  { code: "en", name: "English", flag: "/flags/en.png" },
  { code: "fr", name: "Français", flag: "/flags/fr.png" },
  { code: "mg", name: "Malagasy", flag: "/flags/mg.png" },
];

const open = ref(false);
const currentLocale = computed(
  () => locales.find((l) => l.code === languageStore.lang) || locales[0]
);

const selectLocale = (code: string) => {
  languageStore.setLang(code);
  open.value = false;
};

const user = ref(null);
const isLoggedIn = ref(false);

const currentLang = ref("EN");
const langDropdownOpen = ref(false);

const dropdownStates = reactive<{ [key: string]: boolean }>({});

const sidebarLinks = computed(() => {
  if (!user.value) return [];

  const baseLinks = [
    { label: t("dashboard"), icon: "bxr bx-dashboard", to: "/dashboard" },
    { label: t("posts"), icon: "bxr bx-newspaper", to: "/dashboard/post" },
    { label: t("bid"), icon: "bxr bx-price-tag", to: "/dashboard/bid" },
    {
      label: t("chatbox"),
      icon: "bx-chat",
      to: "/dashboard/chatbox",
      newTab: true,
    },
  ];

  const links: any[] = [];

  if (user.value.id_categorie_user_id === 4) {
    links.push({
      label: t("gererCompte"),
      icon: "bx bx-user-circle",
      dropdown: baseLinks,
      open: false,
    });

    dropdownStates[t("gererCompte")] = false;

    links.push(
      { label: t("posts"), icon: "bxr bx-newspaper", to: "/admin/posts" },
      { label: t("category"), icon: "bx-category", to: "/admin/categories" },
      { label: t("product"), icon: "bx bx-package", to: "/admin/products" },
      { label: t("user"), icon: "bxr bx-user", to: "/admin/users" }
    );
  } else {
    links.push(...baseLinks);
  }

  return links;
});

const isMobileMenuOpen = ref(false);
const toggleMobileMenu = () => {
  isMobileMenuOpen.value = !isMobileMenuOpen.value;
};

let userId = 0;

async function checkUser() {
  const token = localStorage.getItem("access_token");
  if (!token) return;

  try {
    const res = await fetch(`${API_URL}/api/me/`, {
      headers: { Authorization: `Bearer ${token}` },
    });

    if (!res.ok) throw new Error("Unauthorized");

    const data = await res.json();

    user.value = data;
    userId = data.id;
    console.log("User data:", data);

    isLoggedIn.value = true;
  } catch {
    isLoggedIn.value = false;
    user.value = null;
  }
}

onMounted(() => {
  checkUser();

  if (!isLoggedIn.value) {
    router.push("/signin");
  }
});

async function countUnreadMessages(currentUserId: number) {
  const token = localStorage.getItem("access_token");
  if (!token) return 0;

  try {
    const [chatsRes, messagesRes] = await Promise.all([
      fetch(`${API_URL}/api/chats/`, {
        headers: { Authorization: `Bearer ${token}` },
      }),
      fetch(`${API_URL}/api/messages/`, {
        headers: { Authorization: `Bearer ${token}` },
      }),
    ]);

    const chats = await chatsRes.json();
    const messages = await messagesRes.json();

    const chatIds = new Set(chats.map((chat: any) => chat.id));

    const unreadUsers = new Set<number>();

    messages.forEach((msg: any) => {
      if (
        chatIds.has(msg.id_chat) &&
        !msg.is_read &&
        msg.id_user !== currentUserId
      ) {
        unreadUsers.add(msg.id_user);
      }
    });

    return unreadUsers.size;
  } catch (error) {
    console.error("Erreur unread messages:", error);
    return 0;
  }
}

const unreadCount = ref(0);

onMounted(async () => {
  unreadCount.value = await countUnreadMessages(userId);
});

function goToChat() {
  window.location.href = "/dashboard/chatbox";
}

function goToNotif() {
  window.location.href = "/dashboard/notification";
}

const logout = () => {
  localStorage.removeItem("access_token");
  window.location.href = "/signin";
};
const dropdownOpen = ref(false);

function toggleDropdown() {
  dropdownOpen.value = !dropdownOpen.value;
}

const notifications = ref([]);

const unreadNotifCount = computed(
  () => notifications.value.filter((n) => !n.is_read).length
);

async function fetchNotifications() {
  try {
    const token = localStorage.getItem("access_token");
    if (!token) throw new Error("Not authenticated");

    const res = await fetch(`${API_URL}/api/notifications/`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    if (!res.ok) throw new Error("Failed to fetch notifications");

    notifications.value = await res.json();
  } catch (err) {
    console.error(err);
  } finally {
  }
}

onMounted(() => {
  fetchNotifications();
});

function goToDashboard() {
  if (user.value?.id_categorie_user_id === 4) {
    router.push("/admin");
  } else {
    router.push("/dashboard");
  }
}
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
