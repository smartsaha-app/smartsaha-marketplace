<template>
  <div class="p-6 mt-12 max-w-6xl mx-auto">
    <div v-if="!user.id" class="text-gray-500">Chargement…</div>
    <div class="flex items-center gap-4 mb-10">
      <div
        class="w-20 h-20 rounded-full flex items-center justify-center text-3xl font-bold shadow-md"
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
            :class="user.is_verified ? 'text-green-600' : 'text-red-600'"
          >
            <i
              :class="user.is_verified ? 'bx bx-badge-check' : 'bx bx-error'"
            ></i>
            {{ user.is_verified ? "Compte vérifié" : "Non vérifié" }}
          </p>
        </div>
      </div>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-4 gap-6 mb-12">

      <div
        class="bg-white p-5 rounded border border-gray-100 shadow-sm hover:shadow-md transition-all duration-300 text-gray-800 flex flex-col items-start gap-2"
      >
        <div class="flex items-center gap-2">
          <i class="bx bx-news text-xl text-[#3b82f6]"></i>
          <h2 class="text-lg font-semibold">Publications</h2>
        </div>
        <p class="text-sm text-gray-500 font-medium">{{ posts.length }}</p>
      </div>
    </div>

    <h2 class="text-2xl font-semibold text-gray-900 mb-4">Ses publications</h2>

    <div v-if="posts.length === 0" class="text-gray-500">
      Cet utilisateur n’a encore rien publié.
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
</template>

<script setup lang="ts">
definePageMeta({ layout: "dashboard" });

import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";
import { API_URL } from "~/utils/config";

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

onMounted(async () => {
  await fetchUser();
  await fetchPosts();
});

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
</script>
