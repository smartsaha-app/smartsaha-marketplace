<template>
  <div class="space-y-8 p-12 bg-gray-50/60 min-h-screen">
    <Breadcrumb />

    <div class="max-w-3xl mx-auto p-8 transition-all duration-500">
      <h1
        class="text-2xl sm:text-3xl font-semibold text-gray-800 mb-6 text-left"
      >
        {{ t("createTitle") }}
      </h1>

      <div class="flex justify-between mb-8 hidden">
        <div
          v-for="(stepLabel, index) in steps"
          :key="index"
          class="flex flex-col items-center"
        >
          <div
            :class="[
              'w-10 h-10 flex items-center justify-center rounded-full border-2 transition-all duration-300',
              step === index + 1
                ? 'bg-[#10b481] text-white border-[#10b481]'
                : 'border-gray-300 text-gray-500',
            ]"
          >
            {{ index + 1 }}
          </div>
          <p class="text-xs mt-2">{{ stepLabel }}</p>
        </div>
      </div>

      <transition name="fade" mode="out-in">
        <form v-if="step === 1" @submit.prevent="nextStep" class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700">{{
              t("title")
            }}</label>
            <input
              v-model="form.title"
              type="text"
              class="input-field"
              required
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700">{{
              t("desc")
            }}</label>
            <textarea
              v-model="form.description"
              class="input-field"
              required
            ></textarea>
          </div>

          <div class="flex gap-4">
            <div class="flex-1">
              <label class="block text-sm font-medium text-gray-700">{{
                t("listingsType")
              }}</label>
              <select v-model="form.type_post_id" class="input-field">
                <option
                  v-for="type in filteredTypePosts"
                  :key="type.id"
                  :value="type.id"
                >
                  {{ type.type }}
                </option>
              </select>
            </div>

            <div class="flex-1">
              <label class="block text-sm font-medium text-gray-700">{{
                t("location")
              }}</label>
              <input
                v-model="form.location"
                type="text"
                class="input-field"
                placeholder="Select or type country"
                @input="filterCountries"
                list="countries-list"
                required
              />
            </div>
          </div>

          <div>
            <div
              class="mt-8 flex flex-col items-center justify-center w-full p-6 border border-dashed border-gray-300 rounded-xl cursor-pointer hover:border-blue-500 hover:bg-gray-50 transition-colors"
            >
              <label
                class="flex flex-col items-center justify-center w-full h-full cursor-pointer"
              >
                <i class="bx bx-upload text-6xl mb-3 text-gray-400"></i>

                <span class="text-sm font-medium text-gray-700 mb-2">
                  Cliquez ou glissez-déposez votre fichier
                </span>

                <span v-if="fileName" class="text-sm text-blue-500 mb-2">{{
                  fileName
                }}</span>

                <input type="file" @change="handleFileUpload" class="hidden" />
              </label>
            </div>
            <input
              v-model="form.image_url"
              type="url"
              placeholder=""
              class="input-field mb-2 border-none"
            />
            <div v-if="form.image_url" class="mt-4">
              <img
                :src="form.image_url"
                alt="Preview"
                class="rounded w-full h-60 object-cover"
              />
            </div>
          </div>

          <div class="flex justify-end gap-3 mt-4">
            <button
              type="submit"
              class="btn-primary px-4 py-2 rounded bg-blue-600 text-white font-medium transition"
            >
              {{ t("next") }}
            </button>
          </div>
        </form>
      </transition>
      <datalist id="countries-list">
        <option v-for="c in filteredCountries" :key="c" :value="c"></option>
      </datalist>

      <transition name="fade" mode="out-in">
        <form v-if="step === 2" @submit.prevent="nextStep" class="space-y-4">
          <div class="grid grid-cols-2 gap-4">
            <div>
              <label class="block text-sm font-medium text-gray-700">{{
                t("product")
              }}</label>
              <select v-model="form.product_id" class="input-field">
                <option v-for="p in products" :key="p.id" :value="p.id">
                  {{ p.product }}
                </option>
              </select>
            </div>

            <div>
              <label class="block text-sm font-medium text-gray-700">{{
                t("category")
              }}</label>
              <select v-model="form.categorie_post_id" class="input-field">
                <option v-for="cat in categories" :key="cat.id" :value="cat.id">
                  {{ cat.categorie }}
                </option>
              </select>
            </div>

            <div class="md:col-span-2">
              <label class="block text-sm font-medium text-gray-700">{{
                t("quantity")
              }}</label>
              <input
                v-model="form.quantity"
                type="number"
                min="1"
                class="input-field w-full"
                required
              />
            </div>

            <div class="grid grid-cols-2 gap-4 md:col-span-2">
              <div>
                <label class="block text-sm font-medium text-gray-700">{{
                  t("price")
                }}</label>
                <input
                  v-model="form.price"
                  type="text"
                  class="input-field w-full"
                  required
                />
              </div>

              <div>
                <label class="block text-sm font-medium text-gray-700">{{
                  t("currency")
                }}</label>
                <select v-model="form.currency_id" class="input-field w-full">
                  <option v-for="c in currencies" :key="c.id" :value="c.id">
                    {{ c.currency }} ({{ c.symbol }})
                  </option>
                </select>
              </div>
            </div>
          </div>

          <div class="flex justify-end gap-3 mt-4">
            <button
              type="button"
              class="btn-secondary px-4 py-2 rounded border border-gray-300 text-gray-700 hover:bg-gray-100 transition"
              @click="prevStep"
            >
              {{ t("prev") }}
            </button>

            <button
              type="submit"
              class="btn-primary px-4 py-2 rounded bg-blue-600 text-white font-medium transition"
            >
              {{ t("next") }}
            </button>
          </div>
        </form>
      </transition>

      <transition name="fade" mode="out-in">
        <div v-if="step === 3" class="space-y-6">
          <div class="border rounded-xl p-4 shadow-sm bg-gray-50">
            <div v-if="form.image_url" class="mt-4">
              <img
                :src="form.image_url"
                alt="Preview"
                class="rounded w-full h-60 object-cover"
              />
            </div>
            <h2 class="text-xl font-semibold text-gray-800">
              {{ form.title }}
            </h2>
            <p class="text-gray-600 mt-2">{{ form.description }}</p>
            <p class="text-gray-800 mt-2">
              <b>{{ t("price") }}: </b> {{ form.price }}
            </p>
            <p class="text-gray-600">
              <b>{{ t("qunatity") }}: </b> {{ form.quantity }}
            </p>
            <p class="text-gray-600">
              <b>{{ t("location") }}: </b> {{ form.location }}
            </p>
          </div>

          <div class="flex justify-end gap-3 mt-4">
            <button
              type="button"
              class="btn-secondary px-4 py-2 rounded border border-gray-300 text-gray-700 hover:bg-gray-100 transition"
              @click="prevStep"
            >
              {{ t("prev") }}
            </button>

            <button
              type="button"
              class="btn-success px-4 py-2 rounded bg-blue-600 text-white font-medium transition"
              @click="submitPost"
            >
              {{ t("save") }}
            </button>
          </div>
        </div>
      </transition>
    </div>
  </div>
  <div
    v-if="isLoading"
    class="absolute inset-0 bg-black/50 flex items-center justify-center"
  >
    <div
      class="w-12 h-12 border-4 border-t-[#10b481] border-white rounded-full animate-spin"
    ></div>
  </div>

  <transition name="fade">
    <div
      v-if="notification.visible"
      class="fixed inset-0 flex items-center justify-center z-50 bg-black/20 backdrop-blur-sm"
    >
      <div
        :class="[
          'bg-white rounded shadow-2xl px-8 py-6 flex flex-col items-center gap-4 w-[340px] text-center transition-all duration-300',
          notification.type === 'success'
            ? 'border-t-4 border-[#10b481]'
            : 'border-t-4 border-red-500',
        ]"
      >
        <div
          v-if="notification.type === 'success'"
          class="w-16 h-16 rounded-full bg-[#10b481] flex items-center justify-center"
        >
          <i class="bx bx-check text-4xl font-extrabold text-white"></i>
        </div>
        <div
          v-else
          class="w-16 h-16 rounded-full bg-red-500 flex items-center justify-center"
        >
          <i class="bx bx-x text-4xl font-extrabold text-white"></i>
        </div>

        <p
          :class="[
            'text-lg font-semibold',
            notification.type === 'success' ? 'text-[#10b481]' : 'text-red-500',
          ]"
        >
          {{ notification.message }}
        </p>

        <p class="text-gray-500 text-sm">
          {{
            notification.type === "success"
              ? "Redirecting to your dashboard..."
              : "Please try again."
          }}
        </p>
      </div>
    </div>
  </transition>
</template>

<script setup>
definePageMeta({ layout: "dashboard" });
import { ref, onMounted } from "vue";
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

const isLoading = ref(false);
const notification = ref({ visible: false, message: "", type: "success" });

const showNotification = (message, type = "success", duration = 3000) => {
  notification.value = { visible: true, message, type };
  setTimeout(() => (notification.value.visible = false), duration);
};

const step = ref(1);
const steps = ["General Info", "Details", "Preview"];

const form = ref({
  title: "",
  description: "",
  quantity: 1,
  price: "",
  location: "",
  image_url: "",
  type_post_id: "",
  product_id: "",
  categorie_post_id: "",
  currency_id: "",
});

const typePosts = ref([]);
const products = ref([]);
const categories = ref([]);
const currencies = ref([]);
const user = ref(null);

const countries = ref([]);
const filteredCountries = ref([]);

const filterCountries = () => {
  const search = form.value.location.toLowerCase();
  filteredCountries.value = countries.value.filter((c) =>
    c.toLowerCase().includes(search)
  );
};

const nextStep = () => {
  if (step.value < 3) step.value++;
};
const prevStep = () => {
  if (step.value > 1) step.value--;
};

const handleFileUpload = async (e) => {
  const file = e.target.files[0];
  if (!file) return;

  const formData = new FormData();
  formData.append("file", file);
  formData.append("upload_preset", "marketplace_posts");
  formData.append("folder", "posts");

  try {
    const res = await fetch(
      "https://api.cloudinary.com/v1_1/dhzaabn6m/image/upload",
      {
        method: "POST",
        body: formData,
      }
    );

    const data = await res.json();

    if (data.secure_url) {
      form.value.image_url = data.secure_url;
    } else {
      console.error("Erreur Cloudinary:", data);
      alert("Erreur lors de l'upload de l'image.");
    }
  } catch (err) {
    console.error(err);
    alert("Erreur lors de l'upload de l'image.");
  }
};

onMounted(async () => {
  const res = await fetch(
    "https://restcountries.com/v3.1/all?fields=name,cca2"
  );
  const data = await res.json();
  countries.value = data.map((c) => c.name.common).sort();
  filteredCountries.value = countries.value;

  const token = localStorage.getItem("access_token");
  if (!token) return;

  try {
    const me = await fetch(`${API_URL}/api/me/`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    const meData = await me.json();
    const userRes = await fetch(`${API_URL}/api/users/${meData.id}/`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    user.value = await userRes.json();

    const [tp, pr, cat, cur] = await Promise.all([
      fetch(`${API_URL}/api/typepost/`, {
        headers: { Authorization: `Bearer ${token}` },
      }),
      fetch(`${API_URL}/api/products/`, {
        headers: { Authorization: `Bearer ${token}` },
      }),
      fetch(`${API_URL}/api/categoriepost/`, {
        headers: { Authorization: `Bearer ${token}` },
      }),
      fetch(`${API_URL}/api/currencies/`, {
        headers: { Authorization: `Bearer ${token}` },
      }),
    ]);

    typePosts.value = await tp.json();
    products.value = await pr.json();
    categories.value = await cat.json();
    currencies.value = await cur.json();
  } catch (err) {
    console.error("Error loading data:", err);
  }
});

const submitPost = async () => {
  const token = localStorage.getItem("access_token");
  const payload = { ...form.value, is_active: true, user: user.value };
  isLoading.value = true;

  const res = await fetch(`${API_URL}/api/posts/`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${token}`,
    },
    body: JSON.stringify(payload),
  });

  if (res.ok) {
    showNotification(
      "Votre annonce a été enregistrée et sera publiée après validation par l’administrateur.",
      "success"
    );
    isLoading.value = false;
  } else {
    const err = await res.json();
    console.error(err);
    showNotification("Error publishing post!", "error");
    setTimeout(() => {}, 3000);
    isLoading.value = flase;
  }
};

const filteredTypePosts = computed(() => {
  if (!user.value || !typePosts.value.length) return [];

  const role = user.value.id_categorie_user.categorie;
  console.log("Role", role);
  console.log("User", JSON.stringify(user.value, null, 2));

  if (role === "Acheteur") {
    return typePosts.value.filter((t) => t.type.toLowerCase() === "buying");
  }

  if (role === "Vendeur") {
    return typePosts.value.filter((t) => t.type.toLowerCase() === "selling");
  }

  return typePosts.value;
});
</script>

<style scoped>
.input-field {
  @apply w-full border border-gray-300 rounded px-3 py-2 mt-1 text-gray-700 focus:ring-2 focus:ring-blue-500 focus:outline-none;
}
.btn-primary {
  @apply bg-[#10b481] text-white px-5 py-2 rounded transition-all;
}
.btn-secondary {
  @apply text-gray-700 px-5 py-2 transition-all;
}
.btn-success {
  @apply bg-green-600 text-white px-5 py-2 rounded hover:bg-green-700 transition-all;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
