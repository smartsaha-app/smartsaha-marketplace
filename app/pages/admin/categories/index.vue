<template>
  <div class="space-y-8 p-12 bg-gray-50/60 min-h-screen">
    <Breadcrumb />

    <header
      class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mt-6"
    >
      <div>
        <h1 class="text-2xl font-bold text-[#112830] tracking-tight">
          {{ t("categoryProduct") }}
        </h1>
        <p class="text-sm text-gray-500 mt-1">
          {{ t("catText") }}
        </p>
      </div>

      <button
        @click="toggleForm"
        class="inline-flex items-center gap-2 px-5 py-2.5 rounded bg-[#10B481] text-white text-sm font-medium hover:bg-[#0c745f] transition shadow"
      >
        <i :class="showForm ? 'bx bx-x' : 'bx bx-plus'"></i>
        {{ showForm ? t("close") : t("addCategory") }}
      </button>
    </header>

    <transition name="fade">
      <div
        v-if="showForm"
        class="bg-white border border-gray-200 rounded p-6 shadow-sm"
      >
        <form
          @submit.prevent="submitCategory"
          class="flex flex-col sm:flex-row gap-4 items-end"
        >
          <div class="flex-1 w-full">
            <label class="text-sm font-medium text-gray-600 mb-1 block">
              {{ t("category") }}
            </label>
            <input
              v-model="categorie"
              maxlength="50"
              required
              :placeholder="t('catPlaceholder')"
              class="w-full border border-gray-300 rounded px-4 py-2.5 text-sm focus:ring-2 focus:ring-[#10B481]/30 focus:border-[#10B481] outline-none"
            />
          </div>

          <button
            type="submit"
            class="sm:w-44 w-full bg-[#10B481] text-white py-2.5 rounded text-sm font-semibold hover:bg-[#0c745f] transition disabled:opacity-60"
            :disabled="loading"
          >
            {{ loading ? t("loading") : editId ? t("edit") : t("add") }}
          </button>
        </form>

        <p
          v-if="message"
          :class="['mt-4 p-3 rounded text-sm font-medium', messageClass]"
        >
          {{ message }}
        </p>
      </div>
    </transition>

    <div class="grid gap-4">
      <div
        v-for="cat in categories"
        :key="cat.id"
        class="flex items-center justify-between bg-white border border-gray-200 rounded p-5 shadow-sm hover:shadow-md transition"
      >
        <div>
          <h3 class="font-semibold text-gray-800">
            {{ cat.categorie }}
          </h3>
          <p class="text-xs text-gray-500 mt-1 flex items-center gap-1">
            <i class="bx bx-calendar"></i>
            {{ new Date(cat.created_at).toLocaleDateString() }}
          </p>
        </div>

        <div class="flex items-center gap-2">
          <button
            @click="startEdit(cat)"
            class="w-9 h-9 flex items-center justify-center rounded-full bg-[#10B481]/10 text-[#10B481] hover:bg-[#10B481]/20 transition"
          >
            <i class="bx bx-edit text-lg"></i>
          </button>

          <button
            @click="deleteCategory(cat.id)"
            class="w-9 h-9 flex items-center justify-center rounded-full bg-red-500/10 text-red-600 hover:bg-red-500/20 transition"
          >
            <i class="bx bx-trash text-lg"></i>
          </button>
        </div>
      </div>

      <p
        v-if="categories.length === 0 && !loadingCategories"
        class="text-center text-gray-500 py-10 text-sm"
      >
        {{ t("noCategoryFound") }}
      </p>
    </div>

    <div v-if="loadingCategories" class="flex justify-center py-8">
      <div
        class="w-8 h-8 border-4 border-gray-300 border-t-[#10b481] rounded-full animate-spin"
      ></div>
    </div>
  </div>
</template>

<script setup lang="ts">
definePageMeta({
  layout: "dashboard",
});

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

const categories = ref<any[]>([]);
const loadingCategories = ref(false);

const showForm = ref(false);
const categorie = ref("");
const loading = ref(false);
const message = ref("");
const messageClass = ref("");

const editId = ref<number | null>(null);

function toggleForm() {
  showForm.value = !showForm.value;
  if (!showForm.value) resetForm();
}

function resetForm() {
  categorie.value = "";
  editId.value = null;
  message.value = "";
}

async function fetchCategories() {
  loadingCategories.value = true;
  try {
    const res = await fetch(`${API_URL}/api/categoriepost/`);
    const data = await res.json();
    if (res.ok) categories.value = data;
  } catch (err) {
    console.error(err);
  }
  loadingCategories.value = false;
}

async function submitCategory() {
  if (!categorie.value.trim()) return;

  loading.value = true;
  message.value = "";

  const token = localStorage.getItem("access_token");
  const method = editId.value ? "PUT" : "POST";
  const url = editId.value
    ? `${API_URL}/api/categoriepost/${editId.value}/`
    : `${API_URL}/api/categoriepost/`;

  try {
    const res = await fetch(url, {
      method,
      headers: {
        "Content-Type": "application/json",
        ...(token ? { Authorization: `Bearer ${token}` } : {}),
      },
      body: JSON.stringify({ categorie: categorie.value.trim() }),
    });

    const data = await res.json();

    if (res.ok) {
      message.value = editId.value
        ? `Catégorie modifiée avec succès !`
        : `Catégorie ajoutée avec succès !`;

      messageClass.value = "bg-green-100 text-green-800";
      resetForm();
      showForm.value = false;
      fetchCategories();
    } else {
      message.value = "Erreur : " + JSON.stringify(data);
      messageClass.value = "bg-red-100 text-red-800";
    }
  } catch {
    message.value = "Erreur serveur.";
    messageClass.value = "bg-red-100 text-red-800";
  }

  loading.value = false;
}

function startEdit(cat: any) {
  showForm.value = true;
  editId.value = cat.id;
  categorie.value = cat.categorie;
}

async function deleteCategory(id: number) {
  if (!confirm("Supprimer cette catégorie ?")) return;

  const token = localStorage.getItem("access_token");

  try {
    const res = await fetch(`${API_URL}/api/categoriepost/${id}/`, {
      method: "DELETE",
      headers: {
        ...(token ? { Authorization: `Bearer ${token}` } : {}),
      },
    });

    if (res.ok || res.status === 204) {
      categories.value = categories.value.filter((c) => c.id !== id);
    }
  } catch (err) {
    console.error(err);
  }
}

onMounted(fetchCategories);
</script>
