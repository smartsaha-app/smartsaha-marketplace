<template>
  <div
    class="h-screen flex flex-col items-center justify-center bg-[#112830] relative overflow-hidden"
  >
    <div
      class="absolute top-10 left-10 w-32 h-32 bg-emerald-200 dark:bg-emerald-900 rounded-full blur-3xl opacity-30"
    ></div>
    <div
      class="absolute bottom-10 right-10 w-40 h-40 bg-emerald-300 dark:bg-emerald-800 rounded-full blur-3xl opacity-30"
    ></div>

    <div class="w-full max-w-xl p-8 sm:p-10 relative">
      <div class="text-center mb-10">
        <img
          src="/marketplace_png.png"
          alt="SmartSaha logo"
          class="h-14 sm:h-16 mx-auto mb-4"
        />
        <h1
          class="text-3xl sm:text-4xl font-bold text-gray-100 dark:text-white mb-2"
        >
          Create Account
        </h1>
      </div>

      <form @submit.prevent="submitSignup" class="space-y-6">
        <div class="flex gap-4">
          <div class="relative flex-1">
            <input
              id="username"
              v-model="form.username"
              type="text"
              required
              placeholder="Enter your username "
              class="w-full px-4 py-3 rounded border border-white/20 bg-[#1D333B] text-gray-100 focus:ring-1 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition"
            />
          </div>
          <div class="relative flex-1">
            <input
              id="email"
              v-model="form.email"
              type="email"
              required
              placeholder="Enter your email "
              class="w-full px-4 py-3 rounded border border-white/20 bg-[#1D333B] text-gray-100 focus:ring-1 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition"
            />
          </div>
        </div>

        <!-- Username and Justification -->
        <div class="flex gap-4">
          <div class="relative flex-1">
            <input
              id="password"
              v-model="form.password"
              type="password"
              required
              minlength="6"
              placeholder="Enter your password"
              class="w-full px-4 py-3 rounded border border-white/20 bg-[#1D333B] text-gray-100 focus:ring-1 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition"
            />
          </div>

          <div class="relative flex-1">
            <input
              id="justification"
              v-model="form.justification"
              type="text"
              required
              placeholder="Link your justification"
              class="w-full px-4 py-3 rounded border border-white/20 bg-[#1D333B] text-gray-100 focus:ring-1 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition"
            />
          </div>
        </div>
        <!-- Choix catégorie -->
        <div class="space-y-2">
          <p class="text-gray-300 font-medium">Choose your category</p>

          <div class="grid grid-cols-3 gap-3">
            <div
              v-for="cat in categories"
              :key="cat.id"
              @click="selectedCategory = cat.id"
              :class="[
                'cursor-pointer px-4 py-3 rounded border transition flex items-center justify-center text-sm font-medium',
                selectedCategory === cat.id
                  ? 'bg-[#10b481]/20 border-[#10b481] text-white'
                  : 'bg-[#1D333B] border-white/20 text-gray-300 hover:border-emerald-500',
              ]"
            >
              {{ cat.displayName }}
            </div>
          </div>
        </div>

        <!-- Password -->

        <p class="text-sm text-gray-500 dark:text-gray-400">
          By signing up, you agree to the
          <a
            href="#"
            class="text-[#10b481] dark:text-emerald-400 hover:underline"
            >Terms and Conditions</a
          >.
        </p>

        <button
          type="submit"
          class="w-full bg-[#10b481] hover:bg-emerald-700 active:scale-[0.98] text-white font-semibold py-3 rounded shadow-md transition-all duration-200 flex items-center justify-center"
        >
          Sign Up
        </button>
      </form>

      <p class="text-center text-sm text-gray-500 dark:text-gray-400 mt-8">
        Already have an account?
        <NuxtLink
          to="/signin"
          class="text-[#10b481] dark:text-emerald-400 font-medium hover:underline"
        >
          Sign in
        </NuxtLink>
      </p>
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
            'text-sm font-semibold',
            notification.type === 'success' ? 'text-gray-700' : 'text-gray-700',
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

<script setup lang="ts">
import { ref } from "vue";
import { useRouter } from "vue-router";
import { API_URL } from "~/utils/config";

const router = useRouter();
const form = ref({
  email: "",
  justification: "",
  username: "",
  password: "",
});

const categories = ref([]);
const selectedCategory = ref(null);
const isLoading = ref(false);
const notification = ref({ visible: false, message: "", type: "success" });

const showNotification = (
  message: string,
  type: "success" | "error" = "success",
  duration = 3000
) => {
  notification.value = { visible: true, message, type };
  setTimeout(() => (notification.value.visible = false), duration);
};

onMounted(async () => {
  const res = await fetch(`${API_URL}/api/categorieuser/`);
  let data = await res.json();

  data = data.filter((cat: { categorie: string; }) => cat.categorie.toLowerCase() !== "admin");

  data = data.map((cat: { categorie: string; }) => {
    if (cat.categorie.toLowerCase() === "user") {
      return {
        ...cat,
        displayName: "Both",
      };
    }
    return {
      ...cat,
      displayName: cat.categorie,
    };
  });

  categories.value = data;
});

const submitSignup = async () => {
  if (!selectedCategory.value) {
    showNotification("Veuillez sélectionner une catégorie.", "error");
    return;
  }

  isLoading.value = true;
  try {
    const payload = {
      username: form.value.username,
      email: form.value.email,
      justificatif_url: form.value.justification,
      password: form.value.password,
      id_categorie_user_id: selectedCategory.value,
    };
    const response = await fetch(`${API_URL}/api/register/`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload),
    });
    if (!response.ok) {
      const error = await response.json();
      showNotification(error.detail, "error");
      setTimeout(() => {}, 3000);
      isLoading.value = false;
      return;
    }
    showNotification(
      "Inscription réussie Votre compte a été créé avec succès. " +
        "Cependant, vous devez attendre la vérification et l’activation de votre compte par un administrateur " +
        "afin d’accéder à votre dashboard et aux fonctionnalités de la plateforme. " +
        "Veuillez contacter l’administrateur si nécessaire.",
      "success",
      6000
    );

    setTimeout(() => {
      router.push("/signin");
    }, 6000);
  } catch (err) {
    console.error(err);
    showNotification(
      "Erreur réseau ou serveur. Veuillez réessayer plus tard.",
      "error"
    );
  }
};
</script>
