<template>
  <header
    :class="[
      'fixed top-0 left-0 w-full z-50 backdrop-blur-md transition-all duration-300',
      isScrolled ? 'bg-white/90 shadow-sm' : 'bg-white/60',
    ]"
  >
    <div class="max-w-7xl mx-auto flex items-center justify-between py-4 px-4">
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

      <nav class="hidden md:flex items-center gap-8">
        <NuxtLink
          v-for="(item, i) in menuItems"
          :key="i"
          :to="item.link"
          class="relative text-gray-700 font-medium hover:text-[#10b481] transition group"
        >
          {{ t(item.label) }}
          <span
            class="absolute bottom-[-3px] left-0 w-0 h-[2px] bg-[#10b481] transition-all duration-300 group-hover:w-full"
          ></span>
        </NuxtLink>
      </nav>

      <div class="hidden md:flex items-center gap-5">
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

        <div
          class="relative p-2 rounded-lg hover:bg-gray-100 cursor-pointer transition hidden"
          @click="$router.push('/messages')"
        >
          <i class="bx bx-message-dots text-2xl text-gray-700"></i>
          <span
            v-if="hasNewMessages"
            class="absolute -top-1 -right-1 w-3.5 h-3.5 bg-red-500 rounded-full border-2 border-white"
          ></span>
        </div>

        <div class="relative">
          <template v-if="isLoggedIn">
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
          </template>

          <template v-else>
            <NuxtLink
              to="/signin"
              class="px-4 py-2 text-sm font-medium text-white bg-[#10b481] rounded-lg hover:bg-[#0e946a] transition"
            >
              Sign in
            </NuxtLink>
          </template>
        </div>
      </div>

      <button
        class="md:hidden flex flex-col justify-between w-6 h-6"
        @click="toggleMobileMenu"
      >
        <span class="w-full h-0.5 bg-gray-700"></span>
        <span class="w-full h-0.5 bg-gray-700"></span>
        <span class="w-3/4 h-0.5 bg-gray-700"></span>
      </button>
    </div>

    <transition name="slide-down h-screen">
      <div
        v-if="mobileMenuOpen"
        class="md:hidden bg-white shadow-lg border-t border-gray-100 w-full"
      >
        <div class="flex items-center gap-3 p-4 border-b border-gray-100">
          <template v-if="isLoggedIn">
            <div
              class="w-12 h-12 bg-[#10b481] text-white flex items-center justify-center font-bold rounded-full"
            >
              {{ user?.username?.charAt(0).toUpperCase() }}
            </div>
            <div class="flex flex-col">
              <p class="text-sm font-semibold text-gray-800">
                {{ user?.username }}
              </p>
              <p class="text-xs text-gray-500">{{ user?.email }}</p>
            </div>
          </template>
          <template v-else>
            <NuxtLink
              to="/signin"
              class="px-4 py-2 text-sm font-medium text-white bg-[#10b481] rounded-lg hover:bg-[#0e946a] transition"
            >
              Sign in
            </NuxtLink>
          </template>
        </div>

        <div class="flex items-center gap-2 py-3 px-4 border-b border-gray-100">
          <button
            @click="open = !open"
            class="flex items-center gap-2 py-1.5 px-3 w-full border rounded-lg hover:bg-gray-50 transition"
          >
            <img :src="currentLocale.flag" class="w-5 h-5 rounded-full" />
            <span class="text-sm font-medium">{{ currentLocale.name }}</span>
            <i class="bx bx-chevron-down text-sm ml-auto"></i>
          </button>

          <transition name="fade">
            <ul
              v-if="open"
              class="absolute mt-2 w-40 bg-white border border-gray-100 rounded-lg shadow-md overflow-hidden z-50"
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

        <nav class="flex flex-col py-4 px-6 gap-3">
          <NuxtLink
            v-for="(item, i) in menuItems"
            :key="i"
            :to="item.link"
            class="py-2 font-medium text-gray-700 hover:text-[#10b481] transition"
            @click="mobileMenuOpen = false"
          >
            {{ t(item.label) }}
          </NuxtLink>

          <button
            @click="
              () => {
                goToDashboard();
                mobileMenuOpen = false;
              }
            "
            class="py-2 font-medium text-gray-700 hover:text-[#10b481] transition text-left"
          >
            Dashboard
          </button>

          <NuxtLink
            class="py-2 font-medium text-gray-700 hover:text-[#10b481] transition"
            @click="logout()"
          >
            Log out
          </NuxtLink>
        </nav>
      </div>
    </transition>
  </header>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
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

const isLoggedIn = ref(false);
const dropdownOpen = ref(false);
const langDropdownOpen = ref(false);
const mobileMenuOpen = ref(false);
const currentLang = ref("EN");
const user = ref(null);
const hasNewMessages = ref(true);
const isScrolled = ref(false);

const menuItems = [
  { label: "home", link: "/market/" },
  { label: "about", link: "/market/about" },
  { label: "posts", link: "/market/post" },
  // { label: "Services", link: "/services" },
];

function toggleDropdown() {
  dropdownOpen.value = !dropdownOpen.value;
}

function toggleLangDropdown() {
  langDropdownOpen.value = !langDropdownOpen.value;
}

function toggleMobileMenu() {
  mobileMenuOpen.value = !mobileMenuOpen.value;
}

function setLanguage(lang) {
  currentLang.value = lang;
  langDropdownOpen.value = false;
}

function handleClickOutside(event) {
  const header = document.querySelector("header");
  if (header && !header.contains(event.target)) {
    dropdownOpen.value = false;
    langDropdownOpen.value = false;
    mobileMenuOpen.value = false;
  }
}

function handleScroll() {
  isScrolled.value = window.scrollY > 20;
}

onMounted(() => {
  document.addEventListener("click", handleClickOutside);
  window.addEventListener("scroll", handleScroll);
  checkUser();
});

onBeforeUnmount(() => {
  document.removeEventListener("click", handleClickOutside);
  window.removeEventListener("scroll", handleScroll);
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
    user.value = data;
    isLoggedIn.value = true;
  } catch {
    isLoggedIn.value = false;
  }
}

const logout = () => {
  localStorage.removeItem("access_token");
  // Optionnel : rediriger l'utilisateur vers la page de connexion
  window.location.href = "/signin";
};

function goToDashboard() {
  if (user.value?.id_categorie_user_id === 4) {
    router.push("/admin");
  } else {
    router.push("/dashboard");
  }
}
</script>

<style scoped>
/* Transitions */
.slide-down-enter-active {
  transition: all 0.3s ease-out;
}
.slide-down-leave-active {
  transition: all 0.2s ease-in;
}
.slide-down-enter-from {
  transform: translateY(-20px);
  opacity: 0;
}
.slide-down-leave-to {
  transform: translateY(-20px);
  opacity: 0;
}
</style>
