<template>
  <section class="space-y-8 p-12 bg-gray-50/60 min-h-screen">
    <Breadcrumb />
    <div class="max-w-7xl mx-auto px-6 space-y-6">

      <!-- Afficher le post filtré -->
      <div v-if="post" class="max-w-[300px] gap-6">
        <Card :post="post" :key="post.id" />
      </div>

      <!-- Loading -->
      <div v-if="loading" class="text-center py-6 text-gray-500">
        Loading post...
      </div>

      <!-- Message d'erreur -->
      <div v-if="errorMsg" class="text-center py-6 text-red-500">
        {{ errorMsg }}
      </div>

    </div>
  </section>
</template>

<script setup>
definePageMeta({ layout: "dashboard" });

import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";
import Card from "~/components/Card.vue";
import Breadcrumb from "~/components/Breadcrumb.vue";
import { API_URL } from "~/utils/config";
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";

const route = useRoute();
const languageStore = useLanguageStore();

const t = (key) => {
  const lang = languageStore.lang;
  return translate[lang][key] || key;
};

const postId = route.params.id;

const posts = ref([]); // tous les posts
const post = ref(null); // le post filtré
const loading = ref(false);
const errorMsg = ref("");

const fetchPosts = async () => {
  loading.value = true;
  errorMsg.value = "";
  try {
    const token = localStorage.getItem("access_token");
    const res = await fetch(`${API_URL}/api/posts/`, {
      headers: {
        Authorization: token ? `Bearer ${token}` : undefined,
      },
    });
    if (!res.ok) throw new Error("Failed to fetch posts");

    const data = await res.json();
    posts.value = Array.isArray(data) ? data : [data];

    // Filtrer le post correspondant à l'id du paramètre
    post.value = posts.value.find(p => p.id.toString() === postId.toString()) || null;
    if (!post.value) {
      errorMsg.value = "Post non trouvé";
    }
  } catch (err) {
    console.error(err);
    errorMsg.value = "Erreur lors du chargement du post !";
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchPosts();
});
</script>

<style scoped>
::-webkit-scrollbar {
  display: none;
}
</style>
