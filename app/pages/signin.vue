<template>
  <div
    class="min-h-screen flex flex-col items-center justify-center bg-[#112830] transition-colors duration-300"
  >
    <div class="w-full max-w-md p-8 sm:p-10">
      <div class="text-center mb-10">
        <img
          src="/marketplace_png.png"
          alt="SmartSaha logo"
          class="h-14 sm:h-16 mx-auto mb-4"
        />
        <h1
          class="text-3xl sm:text-4xl font-bold text-gray-100 dark:text-white mb-2"
        >
          Welcome back
        </h1>
      </div>

      <form @submit.prevent="submitSignin" class="space-y-6">
        <div class="space-y-2">
          <input
            id="username"
            v-model="form.username"
            type="text"
            placeholder="Enter your username"
            required
            class="w-full px-4 py-3 rounded border border-white/20 bg-[#1D333B] text-gray-100 focus:ring-1 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition"
          />
        </div>

        <div class="relative space-y-2 justify-center">
          <input
            id="password"
            v-model="form.password"
            :type="showPassword ? 'text' : 'password'"
            placeholder="Enter your password"
            required
            minlength="6"
            class="w-full px-4 py-3 pr-12 rounded border border-white/20 bg-[#1D333B] text-gray-100 focus:ring-1 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition"
          />

          <i
            :class="[
              'bx text-xl absolute top-4 right-4 -translate-y-1/2 cursor-pointer text-gray-400 hover:text-white transition',
              showPassword ? 'bx-hide' : 'bx-show',
            ]"
            @click="showPassword = !showPassword"
          ></i>
        </div>

        <div class="flex items-center justify-between text-sm hidden">
          <label class="flex items-center space-x-2">
            <input
              type="checkbox"
              class="rounded text-emerald-600 dark:bg-gray-800 focus:ring-emerald-500"
            />
            <span class="text-gray-600 dark:text-gray-400">Remember me</span>
          </label>
          <NuxtLink
            to="#"
            class="text-emerald-600 dark:text-emerald-400 hover:underline"
          >
            Forgot password?
          </NuxtLink>
        </div>

        <button
          type="submit"
          class="w-full bg-emerald-600 hover:bg-emerald-700 active:scale-[0.98] text-white font-semibold py-3 rounded shadow-md transition-all duration-200 flex items-center justify-center gap-3"
          :disabled="loading"
        >
          <template v-if="loading">
            <span
              class="w-5 h-5 border-4 border-white border-t-transparent rounded-full animate-spin"
            ></span>
            <span>Signing in...</span>
          </template>
          <template v-else>Sign In</template>
        </button>

        <div class="flex items-center my-4 hidden">
          <div class="flex-grow h-px bg-gray-200 dark:bg-gray-700"></div>
          <span class="px-3 text-gray-400 text-sm">or</span>
          <div class="flex-grow h-px bg-gray-200 dark:bg-gray-700"></div>
        </div>

        <button
          type="button"
          @click="signinWithGoogle"
          class="w-full border border-white/20 py-3 rounded flex items-center justify-center gap-3 font-medium text-gray-700 hover:bg-gray-50 dark:hover:bg-gray-800 transition-all hidden"
        >
          <i class="bx bxl-google text-lg"></i>
          <span>Sign in with Google</span>
        </button>
      </form>

      <p class="text-center text-sm text-gray-500 dark:text-gray-400 mt-8">
        Don’t have an account?
        <NuxtLink
          to="/signup"
          class="text-emerald-600 dark:text-emerald-400 font-medium hover:underline"
        >
          Create one
        </NuxtLink>
      </p>
    </div>

    <div
      class="absolute top-10 left-10 w-32 h-32 bg-emerald-200 dark:bg-emerald-900 rounded-full blur-3xl opacity-30"
    ></div>
    <div
      class="absolute bottom-10 right-10 w-40 h-40 bg-emerald-300 dark:bg-emerald-800 rounded-full blur-3xl opacity-30"
    ></div>
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

<script setup lang="ts">
import { ref } from "vue";
import { useRouter } from "vue-router";
import { API_URL } from "~/utils/config";

const router = useRouter();
const isLoading = ref(false);
const notification = ref({ visible: false, message: "", type: "success" });
const showPassword = ref(false);

const showNotification = (
  message: string,
  type: "success" | "error" = "success",
  duration = 3000
) => {
  notification.value = { visible: true, message, type };
  setTimeout(() => (notification.value.visible = false), duration);
};

const form = ref({
  username: "",
  password: "",
});
const loading = ref(false);

const submitSignin = async () => {
  isLoading.value = true;
  try {
    const response = await fetch(`${API_URL}/api/token/`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(form.value),
    });

    if (!response.ok) {
      const error = await response.json();
      showNotification(error.detail, "error");
      setTimeout(() => {}, 3000);
      loading.value = false;
      isLoading.value = false;
      return;
    }

    const data = await response.json();
    localStorage.setItem("access_token", data.access);
    router.push("/market");
  } catch (error) {
    console.error("Network error:", error);
    alert("Network or server error.");
  } finally {
    loading.value = false;
  }
};
</script>
