<template>
  <div class="min-h-screen flex flex-col overflow-x-hidden">
    <div class="flex flex-1 pt-16">
      <!-- Sidebar Desktop -->
      <aside
        class="hidden sm:flex peer group fixed top-20 left-0 h-[calc(100vh-4rem)] flex-col justify-between bg-[#112830] shadow-lg w-20 hover:w-56 transition-all duration-300 z-40"
      >
        <nav class="flex flex-col space-y-2 py-4">
          <button
            v-for="item in sidebarLinks"
            :key="item.to"
            @click="router.push(item.to)"
            class="group relative flex items-center gap-3 px-4 py-2 text-white transition-all duration-300 hover:bg-white/10"
            active-class="bg-white/10"
          >
            <span
              class="absolute left-0 top-1/2 -translate-y-1/2 h-[60%] border-l-[3px] border-white opacity-0 hover:opacity-100 group-focus:opacity-100 transition-all duration-300"
            ></span>
            <i :class="item.icon + ' text-xl ml-2 font-light'"></i>
            <span
              class="font-light opacity-0 group-hover:opacity-100 transition-opacity duration-300"
              >{{ item.label }}</span
            >
          </button>
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

// Utilisateur
const user = ref(null);
const isLoggedIn = ref(false);

// Langue
const currentLang = ref("EN");
const langDropdownOpen = ref(false);

// Définir les liens sidebar
const sidebarLinks = computed(() => {
  if (!user.value) return [];

  const links = [
    {
      label: "Dashboard",
      icon: "bxr bx-dashboard",
      to: "/dashboard",
    },
    {
      label: "Posts",
      icon: "bxr  bx-list-plus",
      to: "/dashboard/post",
    },
    {
      label: "ChatBox",
      icon: "bx-chat",
      to: "/dashboard/chatbox",
    },
  ];
  // Si admin
  if (user.value.id_categorie_user_id === 6) {
    links.push(
      {
        label: "Posts",
        icon: "bx-shield",
        to: "/admin/posts",
      },
      {
        label: "Categories",
        icon: "bx-category",
        to: "/admin/categories",
      },
      {
        label: "Products",
        icon: "bx-category",
        to: "/admin/products",
      },
      {
        label: "Users",
        icon: "bx-category",
        to: "/admin/users",
      }
    );
  }

  return links;
});

// Mobile menu
const isMobileMenuOpen = ref(false);
const toggleMobileMenu = () => {
  isMobileMenuOpen.value = !isMobileMenuOpen.value;
};

// Récupérer l'utilisateur
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
    console.log("User data:", data);
    isLoggedIn.value = true;
  } catch {
    isLoggedIn.value = false;
    user.value = null;
  }
}

onMounted(() => {
  checkUser();
});
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
