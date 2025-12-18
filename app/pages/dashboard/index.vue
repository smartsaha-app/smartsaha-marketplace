<template>
  <div class="space-y-8 p-12 bg-gray-50/60 min-h-screen">
    <Breadcrumb />

    <div class="flex items-center justify-between">
      <div>
        <h1 class="text-2xl font-semibold text-[#112830]">
          {{ t('dashboard') }}
        </h1>
        <p class="text-sm text-gray-500">{{ t('dashboardText') }}</p>
      </div>

      <div class="flex items-center gap-4">
        <select
          v-model="period"
          class="bg-transparent outline-none border-none rounded px-3 py-2 text-sm font-medium"
        >
          <option value="global">{{ t('global') }}</option>
          <option value="last7">{{ t('last7') }}</option>
          <option value="lastMonth">{{ t('lastMonth') }}</option>
          <option value="year">{{ t('year') }}</option>
        </select>

        <div
          class="flex items-center gap-3 bg-white px-4 py-2 rounded border border-gray-100"
        >
          <div class="flex">
            <i
              v-for="n in 5"
              :key="n"
              :class="
                n <= Math.round(averageRating)
                  ? 'bx bxs-star text-yellow-400'
                  : 'bx bx-star text-gray-300'
              "
              class="text-lg"
            />
          </div>
          <span class="text-sm text-gray-600">
            <b class="text-[#112830]">{{ averageRating.toFixed(1) }}</b>
            /5 · {{ reviews.length }}
          </span>
        </div>
      </div>
    </div>

    <div
      class="grid grid-cols-1 sm:grid-cols-4 gap-4 overflow-hidden"
    >
      <StatCard
        label="Posts"
        :value="stats.posts"
        icon="bx bx-book"
        color="#10b481"
      />
      <StatCard
        label="Received bids"
        :value="stats.receivedBids"
        icon="bx bx-down-arrow-circle"
        color="#219ebc"
        bordered
      />
      <StatCard
        label="Sent bids"
        :value="stats.sentBids"
        icon="bx bx-up-arrow-circle"
        color="#f4a261"
        bordered
      />
      <StatCard
        label="Accepted revenue"
        :value="formattedRevenue"
        icon="bx bx-money"
        color="#5fd4a2"
        bordered
      />
    </div>

    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <div class="lg:col-span-2 bg-white rounded p-6 border border-gray-100">
        <h2 class="text-gray-500 font-semibold text-sm uppercase tracking-wide mb-4">{{ t('annoncesEvolution') }}</h2>
        <div class="h-72 w-full">
          <canvas ref="soldPostsChart" class="w-full h-full"></canvas>
        </div>
      </div>

      <div class="bg-white rounded p-6 border border-gray-100">
        <h2 class="text-gray-500 font-semibold text-sm uppercase tracking-wide mb-4">{{ t('postsStatus') }}</h2>
        <div class="h-72">
          <canvas ref="postsChart" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
definePageMeta({ layout: "dashboard" });

import { ref, onMounted, computed, watch } from "vue";
import { Chart, registerables } from "chart.js";
import Breadcrumb from "~/components/Breadcrumb.vue";
import StatCard from "~/components/StatCard.vue";
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

Chart.register(...registerables);

const posts = ref([]);
const receivedBids = ref([]);
const sentBids = ref([]);
const reviews = ref([]);

const period = ref("global");

const stats = ref({
  posts: 0,
  receivedBids: 0,
  sentBids: 0,
});

let soldChart = null;
let statusChart = null;

const postsChart = ref(null);
const soldPostsChart = ref(null);

async function fetchData(url, target) {
  const token = localStorage.getItem("access_token");
  const res = await fetch(url, {
    headers: { Authorization: `Bearer ${token}` },
  });
  if (res.ok) target.value = await res.json();
}

function isInPeriod(date) {
  const d = new Date(date);
  const now = new Date();

  if (period.value === "last7") {
    const start = new Date();
    start.setDate(now.getDate() - 7);
    return d >= start;
  }

  if (period.value === "lastMonth") {
    const lastMonth = new Date(now.getFullYear(), now.getMonth() - 1, 1);
    const endLastMonth = new Date(now.getFullYear(), now.getMonth(), 0);
    return d >= lastMonth && d <= endLastMonth;
  }

  if (period.value === "year") {
    return d.getFullYear() === now.getFullYear();
  }

  return true;
}

const averageRating = computed(() => {
  return reviews.value?.average_rating ?? 0;
});

const filteredPosts = computed(() =>
  posts.value.filter((p) => isInPeriod(p.created_at))
);
const filteredReceived = computed(() =>
  receivedBids.value.filter((b) => isInPeriod(b.created_at))
);
const filteredSent = computed(() =>
  sentBids.value.filter((b) => isInPeriod(b.created_at))
);

const totalRevenue = computed(() => {
  // On va créer un objet par currency
  const revenueByCurrency = {};

  filteredReceived.value
    .filter((b) => b.current_status.name === "acceptée")
    .forEach((b) => {
      const post = posts.value.find((p) => p.id === b.post.id);
      if (post && post.current_status === "vendu") {
        const currency = post.currency.symbol || "Ar"; // fallback
        revenueByCurrency[currency] = (revenueByCurrency[currency] || 0) + Number(b.price || 0);
      }
    });

  return revenueByCurrency;
});

// Formatage pour affichage
const formattedRevenue = computed(() => {
  const parts = [];
  for (const [currency, amount] of Object.entries(totalRevenue.value)) {
    parts.push(`${amount.toLocaleString()} ${currency}`);
  }
  return parts.join(" · "); // si plusieurs devises
});


watch([filteredPosts, filteredReceived, filteredSent], () => {
  stats.value.posts = filteredPosts.value.length;
  stats.value.receivedBids = filteredReceived.value.length;
  stats.value.sentBids = filteredSent.value.length;
});

function renderCharts() {
  soldChart?.destroy();
  statusChart?.destroy();

  const sold = filteredPosts.value.filter((p) => p.current_status === "vendu");

  const soldCounts = {};
  sold.forEach((p) => {
    const dateKey = new Date(p.created_at).toLocaleDateString();
    soldCounts[dateKey] = (soldCounts[dateKey] || 0) + 1;
  });

  soldChart = new Chart(soldPostsChart.value, {
    type: "line",
    data: {
      labels: Object.keys(soldCounts),
      datasets: [
        {
          data: Object.values(soldCounts),
          borderColor: "#10b481",
          backgroundColor: "rgba(16,180,129,0.3)",
          fill: true,
          tension: 0.4,
          pointRadius: 4,
        },
      ],
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: { legend: { display: false } },
      scales: {
        x: { grid: { display: false } },
        y: {
          beginAtZero: true,
          ticks: { stepSize: 1, precision: 0 },
        },
      },
    },
  });

  const statusColors = {
    brouillon: "#6B7280", // gris
    published: "#219ebc", // bleu
    négociation: "#f4a261", // jaune
    vendu: "#10B481", // vert
    supprimé: "#EF4444", // rouge
  };

  const statusCount = filteredPosts.value.reduce((a, p) => {
    a[p.current_status] = (a[p.current_status] || 0) + 1;
    return a;
  }, {});

  statusChart = new Chart(postsChart.value, {
    type: "doughnut",
    data: {
      labels: Object.keys(statusCount),
      datasets: [
        {
          data: Object.values(statusCount),
          backgroundColor: Object.keys(statusCount).map(
            (s) => statusColors[s] || "#D1D5DB"
          ), 
          borderWidth: 0,
        },
      ],
    },
    options: {
      cutout: "75%",
      plugins: {
        legend: { position: "bottom" },
      },
    },
  });
}

const userId = ref(0);

async function checkUser() {
  const token = localStorage.getItem("access_token");
  if (!token) return;

  try {
    const res = await fetch(`${API_URL}/api/me/`, {
      headers: { Authorization: `Bearer ${token}` },
    });

    if (!res.ok) throw new Error("Unauthorized");

    const data = await res.json();
    userId.value = data.id;
  } catch (err) {
    console.error("Failed to fetch user:", err);
  }
}

onMounted(async () => {
  await checkUser();

  await fetchData(`${API_URL}/api/posts/my_posts/`, posts);
  await fetchData(`${API_URL}/api/bids/received_bids/`, receivedBids);
  await fetchData(`${API_URL}/api/bids/my_bids/`, sentBids);
  await fetchData(`${API_URL}/api/reviews/user/${userId.value}/`, reviews);

  // console.log("received_bids :", JSON.parse(JSON.stringify(receivedBids.value, null, 2)));
  // console.log("posts :", JSON.parse(JSON.stringify(posts.value, null, 2)));
  // console.log("reviews :", JSON.parse(JSON.stringify(reviews.value, null, 2)));

  renderCharts();
});

watch(period, renderCharts);
</script>

