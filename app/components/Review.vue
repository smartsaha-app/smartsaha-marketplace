<template>
  <section v-if="topUsers.length" class="relative py-16 md:py-24 bg-white overflow-hidden">
    <div class="relative max-w-7xl mx-auto px-4">
      <h1
        class="text-2xl sm:text-3xl font-extrabold text-center text-[#112830] mb-14"
      >
        {{ t("userReview") }}
      </h1>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-10 items-end">
        <div
          v-for="(user, index) in topUsers"
          :key="user.id"
          class="relative group"
          :class="[
            index === 0 ? 'md:order-2' : '',
            index === 1 ? 'md:order-3' : '',
            index === 2 ? 'md:order-1' : '',
          ]"
        >
          <div
            class="backdrop-blur-xl bg-gray-100/20 border border-gray-100 rounded p-8 text-center shadow-sm transition-all duration-300 group-hover:scale-105"
          >
            <div class="flex justify-center mb-5">
              <div
                class="w-16 h-16 rounded-full flex items-center justify-center text-2xl font-bold shadow-lg"
                :class="getAvatarColor(user.username)"
              >
                {{ user.username.charAt(0).toUpperCase() }}
              </div>
            </div>

            <h2 class="text-xl font-semibold text-[#112830]">
              {{ user.username }}
            </h2>
            <p class="text-sm text-gray-600 mb-4 truncate">
              {{ user.email }}
            </p>

            <div class="flex justify-center items-center gap-1 mb-3">
              <template v-for="star in 5" :key="star">
                <i
                  class="bx text-xl"
                  :class="
                    star <= Math.round(user.average_rating)
                      ? 'bxs-star text-yellow-400'
                      : 'bxs-star text-gray-300'
                  "
                ></i>
              </template>
            </div>

            <p class="text-gray-300 text-sm">
              {{ user.average_rating.toFixed(1) || 0 }} / 5
            </p>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
i.bxs-star {
  transition: transform 0.2s;
}
i.bxs-star:hover {
  transform: scale(1.2);
}
</style>

<script setup>
import { ref, onMounted } from "vue";
import { API_URL } from "~/utils/config";
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";
import { useRouter } from "vue-router";
const router = useRouter();

const languageStore = useLanguageStore();
const t = (key) => {
  const lang = languageStore.lang;
  return translate[lang][key] || key;
};

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

function getAvatarColor(username) {
  let hash = 0;
  for (let i = 0; i < username.length; i++) {
    hash = username.charCodeAt(i) + ((hash << 5) - hash);
  }
  return avatarColors[Math.abs(hash) % avatarColors.length];
}

const rankColor = (index) => {
  return (
    [
      "bg-yellow-400 text-black",
      "bg-gray-300 text-black",
      "bg-orange-400 text-black",
    ][index] || "bg-gray-400"
  );
};

const topUsers = ref([]);

onMounted(async () => {
  const token = localStorage.getItem("access_token");
  if (!token) return;

  try {
    const resUsers = await fetch(`${API_URL}/api/users/`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    const usersList = await resUsers.json();

    const usersWithRatings = await Promise.all(
      usersList.map(async (user) => {
        const resReviews = await fetch(
          `${API_URL}/api/reviews/user/${user.id}/`,
          {
            headers: { Authorization: `Bearer ${token}` },
          }
        );
        const reviewsData = await resReviews.json();

        const reviews = Array.isArray(reviewsData.reviews)
          ? reviewsData.reviews
          : [];
        const average_rating = reviewsData.average_rating || 0;

        return { ...user, reviews, average_rating };
      })
    );

    // Trier par average_rating décroissant et prendre les 3 meilleurs
    topUsers.value = usersWithRatings
      .sort((a, b) => b.average_rating - a.average_rating)
      .slice(0, 3);
  } catch (err) {
    console.error(err);
  }
});
</script>
