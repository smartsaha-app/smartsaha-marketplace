<template>
  <div class="space-y-8 p-12 bg-gray-50/60 min-h-screen">
    <Breadcrumb />
    <header class="mb-10 mt-6">
      <h1 class="text-2xl font-bold text-gray-900 tracking-tight">
        {{ t('productTitle') }}
      </h1>
      <p class="text-gray-500 mt-1 text-sm">
        {{ t('productText') }}
      </p>
    </header>

    <div class="flex border-b border-gray-200 mb-8">
      <button
        @click="showSection('products')"
        class="px-4 py-2 -mb-px text-sm font-medium transition"
        :class="
          activeSection === 'products'
            ? 'border-b-2 border-[#10B481] text-[#10B481]'
            : 'text-gray-500 hover:text-gray-700'
        "
      >
        {{ t('products') }}
      </button>

      <button
        @click="showSection('units')"
        class="ml-6 px-4 py-2 -mb-px text-sm font-medium transition"
        :class="
          activeSection === 'units'
            ? 'border-b-2 border-[#10B481] text-[#10B481]'
            : 'text-gray-500 hover:text-gray-700'
        "
      >
        {{ t('units') }}
      </button>
    </div>

    <section v-if="activeSection === 'products'">
      <div class="flex justify-end mb-6">
        <button
          @click="toggleProductForm"
          class="px-4 py-2 rounded text-white bg-[#10B481] hover:bg-[#0c745f] text-sm transition"
        >
        <i :class="showProductForm ? 'bx bx-x' : 'bx bx-plus'"></i>
          {{ showProductForm ? t('close') : t('add') }}
        </button>
      </div>

      <div
        v-if="showProductForm"
        class="border border-gray-200 rounded p-6 mb-10 bg-white"
      >
        <h2 class="font-semibold text-gray-800 mb-4 text-lg">
          {{ editProductId ? "Modifier un produit" : "Créer un produit" }}
        </h2>

        <form
          @submit.prevent="submitProduct"
          class="grid grid-cols-1 md:grid-cols-3 gap-6"
        >
          <div class="flex flex-col gap-1">
            <label class="text-sm text-gray-600">{{ t('productName') }}</label>
            <input
              v-model="productName"
              required
              maxlength="50"
              class="border border-gray-300 rounded px-3 py-2 text-sm focus:border-[#10B481] outline-none"
              placeholder="Ex : Haricots"
            />
          </div>

          <div class="flex flex-col gap-1">
            <label class="text-sm text-gray-600">{{ t('productDesc') }}</label>
            <input
              v-model="productDescription"
              class="border border-gray-300 rounded px-3 py-2 text-sm focus:border-[#10B481] outline-none"
              placeholder="Optionnel"
            />
          </div>

          <div class="flex flex-col gap-1">
            <label class="text-sm text-gray-600">{{ t('unit') }}</label>
            <select
              v-model="productUnit"
              required
              class="border border-gray-300 rounded px-3 py-2 text-sm focus:border-[#10B481] outline-none"
            >
              <option v-for="u in units" :key="u.id" :value="u.id">
                {{ u.unit }} ({{ u.abbreviation }})
              </option>
            </select>
          </div>

          <div class="md:col-span-3 flex justify-end">
            <button
              type="submit"
              class="px-6 py-2 rounded text-white bg-[#10B481] text-sm hover:bg-[#0c745f] transition"
            >
              {{ editProductId ? t('edit') : t('add') }}
            </button>
          </div>
        </form>
      </div>

      <div class="space-y-4">
        <div
          v-for="p in products"
          :key="p.id"
          class="flex justify-between items-center border border-gray-200 rounded p-4 bg-white"
        >
          <div>
            <h3 class="font-medium text-gray-800">{{ p.product }}</h3>
            <p class="text-sm text-gray-500">{{ p.description || "—" }}</p>
            <span
              class="inline-block mt-2 text-xs text-gray-700 bg-gray-100 px-3 py-1 rounded"
            >
              {{ p.unit?.abbreviation }}
            </span>
          </div>

          <div class="flex gap-4 text-xl">
            <button
              @click="startEditProduct(p)"
              class="text-[#10B481] hover:opacity-70"
            >
              <i class="bx bx-edit"></i>
            </button>
            <button
              @click="deleteProduct(p.id)"
              class="text-red-600 hover:text-red-700"
            >
              <i class="bx bx-trash"></i>
            </button>
          </div>
        </div>
      </div>
    </section>

    <section v-if="activeSection === 'units'" class="mt-10">
      <div class="flex justify-end mb-6">
        <button
          @click="toggleUnitForm"
          class="px-4 py-2 rounded text-white bg-[#10B481] hover:bg-[#0c745f] text-sm transition"
        >
          {{ showUnitForm ? "Fermer" : t('add') }}
        </button>
      </div>

      <div
        v-if="showUnitForm"
        class="border border-gray-200 rounded p-6 bg-white mb-8"
      >
        <form
          @submit.prevent="submitUnit"
          class="grid grid-cols-1 md:grid-cols-3 gap-6"
        >
          <div class="flex flex-col gap-1">
            <label class="text-sm text-gray-600">{{ t('unit') }}</label>
            <input
              v-model="unitName"
              required
              class="border border-gray-300 rounded px-3 py-2 text-sm focus:border-[#10B481] outline-none"
              placeholder="Kilogramme"
            />
          </div>

          <div class="flex flex-col gap-1">
            <label class="text-sm text-gray-600">{{ t('abrev') }}</label>
            <input
              v-model="unitAbbreviation"
              required
              class="border border-gray-300 rounded px-3 py-2 text-sm focus:border-[#10B481] outline-none"
              placeholder="kg"
            />
          </div>

          <div class="md:col-span-3 flex justify-end">
            <button
              type="submit"
              class="px-6 py-2 rounded text-white bg-[#10B481] text-sm hover:bg-[#0c745f] transition"
            >
              {{ editUnitId ? t('edit') : t('add') }}
            </button>
          </div>
        </form>
      </div>

      <div class="space-y-4">
        <div
          v-for="u in units"
          :key="u.id"
          class="flex justify-between items-center border border-gray-200 rounded p-4 bg-white"
        >
          <div>
            <h3 class="font-medium text-gray-800">{{ u.unit }}</h3>
            <p class="text-sm text-gray-500">{{ u.abbreviation }}</p>
          </div>

          <div class="flex gap-4 text-xl">
            <button
              @click="startEditUnit(u)"
              class="text-[#10B481] hover:opacity-70"
            >
              <i class="bx bx-edit"></i>
            </button>
            <button
              @click="deleteUnit(u.id)"
              class="text-red-600 hover:text-red-700"
            >
              <i class="bx bx-trash"></i>
            </button>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
definePageMeta({ layout: "dashboard" });
import { ref, onMounted } from "vue";
import { API_URL } from "~/utils/config";
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";
import { useRouter } from "vue-router";
import Breadcrumb from "~/components/Breadcrumb.vue";
const router = useRouter();

const languageStore = useLanguageStore();
const t = (key: string) => {
  const lang = languageStore.lang;
  return translate[lang][key] || key;
};

const activeSection = ref<"products" | "units">("products");

const showProductForm = ref(false);
const showUnitForm = ref(false);

const productName = ref("");
const productDescription = ref("");
const productUnit = ref("");
const editProductId = ref<number | null>(null);

const unitName = ref("");
const unitAbbreviation = ref("");
const editUnitId = ref<number | null>(null);

const products = ref<any[]>([]);
const units = ref<any[]>([]);
const message = ref("");
const messageClass = ref("");

function toggleProductForm() {
  showProductForm.value = !showProductForm.value;
  if (!showProductForm.value) resetProductForm();
}
function toggleUnitForm() {
  showUnitForm.value = !showUnitForm.value;
  if (!showUnitForm.value) resetUnitForm();
}

function showSection(section: "products" | "units") {
  activeSection.value = section;
  showProductForm.value = false;
  showUnitForm.value = false;
}

function resetProductForm() {
  productName.value = "";
  productDescription.value = "";
  productUnit.value = "";
  editProductId.value = null;
}
function resetUnitForm() {
  unitName.value = "";
  unitAbbreviation.value = "";
  editUnitId.value = null;
}

async function fetchUnits() {
  const res = await fetch(`${API_URL}/api/units/`);
  units.value = await res.json();
}
async function fetchProducts() {
  const res = await fetch(`${API_URL}/api/products/`);
  products.value = await res.json();
}

async function submitProduct() {
  const token = localStorage.getItem("access_token");
  const payload = {
    product: productName.value,
    description: productDescription.value,
    unit_id: productUnit.value,
  };
  const url = editProductId.value
    ? `${API_URL}/api/products/${editProductId.value}/`
    : `${API_URL}/api/products/`;
  const method = editProductId.value ? "PUT" : "POST";
  const res = await fetch(url, {
    method,
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${token}`,
    },
    body: JSON.stringify(payload),
  });
  const data = await res.json();
  if (res.ok) {
    message.value = editProductId.value
      ? "Produit modifié !"
      : "Produit ajouté !";
    messageClass.value = "bg-green-100 text-green-800";
    await fetchProducts();
    resetProductForm();
    showProductForm.value = false;
  } else {
    message.value = "Erreur: " + JSON.stringify(data);
    messageClass.value = "bg-red-100 text-red-800";
  }
}

async function submitUnit() {
  const token = localStorage.getItem("access_token");
  const payload = {
    unit: unitName.value,
    abbreviation: unitAbbreviation.value,
  };
  const url = editUnitId.value
    ? `${API_URL}/api/units/${editUnitId.value}/`
    : `${API_URL}/api/units/`;
  const method = editUnitId.value ? "PUT" : "POST";
  const res = await fetch(url, {
    method,
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${token}`,
    },
    body: JSON.stringify(payload),
  });
  const data = await res.json();
  if (res.ok) {
    message.value = editUnitId.value ? "Unit modifiée !" : "Unit ajoutée !";
    messageClass.value = "bg-green-100 text-green-800";
    await fetchUnits();
    resetUnitForm();
    showUnitForm.value = false;
  } else {
    message.value = "Erreur: " + JSON.stringify(data);
    messageClass.value = "bg-red-100 text-red-800";
  }
}

function startEditProduct(p: any) {
  showProductForm.value = true;
  editProductId.value = p.id;
  productName.value = p.product;
  productDescription.value = p.description;
  productUnit.value = p.unit_id;
}
function startEditUnit(u: any) {
  showUnitForm.value = true;
  editUnitId.value = u.id;
  unitName.value = u.unit;
  unitAbbreviation.value = u.abbreviation;
}
async function deleteProduct(id: number) {
  if (!confirm("Supprimer ce produit ?")) return;
  const token = localStorage.getItem("access_token");
  const res = await fetch(`${API_URL}/api/products/${id}/`, {
    method: "DELETE",
    headers: { Authorization: `Bearer ${token}` },
  });
  if (res.ok) fetchProducts();
}
async function deleteUnit(id: number) {
  if (!confirm("Supprimer cette unité ?")) return;
  const token = localStorage.getItem("access_token");
  const res = await fetch(`${API_URL}/api/units/${id}/`, {
    method: "DELETE",
    headers: { Authorization: `Bearer ${token}` },
  });
  if (res.ok) fetchUnits();
}

onMounted(() => {
  fetchUnits();
  fetchProducts();
});
</script>
