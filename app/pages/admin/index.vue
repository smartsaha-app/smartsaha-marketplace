<template>
  <div class="space-y-8 p-8 bg-gray-50 min-h-screen">
    <Breadcrumb />
    <h1 class="text-2xl font-bold mb-6">{{ t('dashboard') }}</h1>

    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-6">
      <div
        class="p-5 rounded shadow-lg flex flex-col justify-between transition-transform duration-300 hover:scale-105 bg-white"
      >
        <div class="flex items-center justify-between">
          <div
            class="text-gray-500 font-semibold text-sm uppercase tracking-wide"
          >
            {{ t('totalUsers') }}
          </div>
          <div
            class="flex items-center justify-center w-10 h-10 rounded bg-[#10b481]/10"
          >
            <i class="bx bx-user text-2xl text-[#10b481]"></i>
          </div>
        </div>
        <div class="text-2xl font-bold mt-3">{{ stats.users.total }}</div>
      </div>

      <div
        class="p-5 rounded shadow-lg flex flex-col justify-between transition-transform duration-300 hover:scale-105 bg-white"
      >
        <div class="flex items-center justify-between">
          <div
            class="text-gray-500 font-semibold text-sm uppercase tracking-wide"
          >
            {{ t('verifiedUsers') }}
          </div>
          <div
            class="flex items-center justify-center w-10 h-10 rounded bg-blue-500/10"
          >
            <i class="bx bx-check-circle text-2xl text-blue-500"></i>
          </div>
        </div>
        <div class="text-2xl font-bold mt-3">{{ stats.users.verified }}</div>
      </div>

      <div
        class="p-5 rounded shadow-lg flex flex-col justify-between transition-transform duration-300 hover:scale-105 bg-white"
      >
        <div class="flex items-center justify-between">
          <div
            class="text-gray-500 font-semibold text-sm uppercase tracking-wide"
          >
            {{ t('unverifiedUsers') }}
          </div>
          <div
            class="flex items-center justify-center w-10 h-10 rounded bg-red-500/10"
          >
            <i class="bx bx-x-circle text-2xl text-red-500"></i>
          </div>
        </div>
        <div class="text-2xl font-bold mt-3">{{ stats.users.unverified }}</div>
      </div>

      <div
        class="relative p-5 rounded shadow-lg flex flex-col transition-transform duration-300 hover:scale-105"
      >
        <div
          class="text-[#10b481] font-semibold text-sm uppercase tracking-wide"
        >
          {{ t('usersByCategory') }}
        </div>

        <ul class="mt-3 space-y-2 text-gray-700 max-w-[90%]">
          <li
            v-for="(count, cat) in stats.users.byCategory"
            :key="cat"
            class="flex justify-between py-1"
          >
            <span>{{ cat }}</span>
            <span class="font-semibold">{{ count }}</span>
          </li>
        </ul>
        <div
          class="absolute right-0 top-3 bottom-3 border-r-4 border-[#10b481]"
        ></div>
      </div>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <div
        class="bg-white p-6 rounded shadow-lg hover:shadow-2xl transition-shadow duration-300"
      >
        <h2 class="text-lg font-semibold mb-4">{{ t('postsStatus') }}</h2>
        <div class="h-64">
          <canvas ref="postsChart"></canvas>
        </div>
      </div>

      <div
        class="bg-white p-6 rounded shadow-lg hover:shadow-2xl transition-shadow duration-300"
      >
        <h2 class="text-lg font-semibold mb-4">{{ t('soldPosts') }}</h2>
        <div class="h-64">
          <canvas ref="soldPostsChart"></canvas>
        </div>
      </div>

      <div
        class="bg-white p-6 rounded shadow-lg hover:shadow-2xl transition-shadow duration-300 md:col-span-2"
      >
        <h2 class="text-lg font-semibold mb-4">{{ t('reports') }}</h2>
        <div class="h-64">
          <canvas ref="reportsChart"></canvas>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
definePageMeta({ layout: "dashboard" });

import { ref, onMounted } from "vue";
import Breadcrumb from "~/components/Breadcrumb.vue";
import { API_URL } from "~/utils/config";
import { Chart, registerables } from "chart.js";
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

const users = ref([]);
const posts = ref([]);
const reports = ref([]);
const stats = ref({
  users: { total: 0, verified: 0, unverified: 0, byCategory: {} },
});

const postsChart = ref(null);
const soldPostsChart = ref(null);
const reportsChart = ref(null);

async function fetchUsers() {
  const token = localStorage.getItem("access_token");
  const res = await fetch(`${API_URL}/api/users/`, {
    headers: { Authorization: `Bearer ${token}` },
  });
  if (res.ok) users.value = await res.json();
  stats.value.users.total = users.value.length;
  stats.value.users.verified = users.value.filter((u) => u.is_verified).length;
  stats.value.users.unverified = users.value.filter(
    (u) => !u.is_verified
  ).length;
  // stats.value.users.byCategory = users.value.reduce((acc, u) => {
  //   const cat = u.id_categorie_user?.categorie || "Other";
  //   acc[cat] = (acc[cat] || 0) + 1;
  //   return acc;
  // }, {});

  stats.value.users.byCategory = users.value.reduce((acc, u) => {
    if (
      !u.id_categorie_user?.categorie ||
      u.id_categorie_user?.categorie.toLowerCase() === "admin"
    )
      return acc;

    let cat = "Other";
    const role = u.id_categorie_user?.categorie.toLowerCase();

    if (role.includes("acheteur") || role === "buyer") cat = "Acheteur";
    else if (role.includes("vendeur") || role === "seller") cat = "Vendeur";
    else if (role.includes("user")) cat = "Both";

    acc[cat] = (acc[cat] || 0) + 1;
    return acc;
  }, {});
}

async function fetchPosts() {
  const token = localStorage.getItem("access_token");
  const res = await fetch(`${API_URL}/api/posts/`, {
    headers: { Authorization: `Bearer ${token}` },
  });
  if (res.ok) posts.value = await res.json();
}

async function fetchReports() {
  const token = localStorage.getItem("access_token");
  const res = await fetch(`${API_URL}/api/reports/`, {
    headers: { Authorization: `Bearer ${token}` },
  });
  if (res.ok) reports.value = await res.json();
  console.log("Fetched reports:", reports.value);
}

function createChart(refEl, type, data, options = {}) {
  if (!refEl) return;

  const defaultOptions = {
    responsive: true,
    maintainAspectRatio: false,
    plugins: {
      legend: {
        position: "top",
        labels: {
          boxWidth: 15,
          padding: 20,
          color: "#334155",
          font: { weight: "600", size: 13 },
        },
      },
      tooltip: {
        mode: "index",
        intersect: false,
        backgroundColor: "rgba(51,65,85,0.9)",
        titleColor: "#fff",
        bodyColor: "#fff",
        cornerRadius: 6,
        padding: 10,
      },
    },
    interaction: { mode: "nearest", axis: "x", intersect: false },
    scales: {
      x: { 
        grid: { display: false }, 
        ticks: { color: "#64748b", font: { size: 12 } } 
      },
      y: { 
        grid: { color: "#e2e8f0" }, 
        ticks: { color: "#64748b", beginAtZero: true, stepSize: 1, precision: 0 } 
      },
    },
    animations: {
      tension: { duration: 800, easing: "easeInOutQuad", from: 0.4, to: 0.4, loop: false },
    },
  };

  return new Chart(refEl, {
    type,
    data,
    options: { ...defaultOptions, ...options },
  });
}

onMounted(async () => {
  await fetchUsers();
  await fetchPosts();
  await fetchReports();

  // POSTS STATUS
  /* POSTS STATUS DOUGHNUT */
const statusColors = {
  brouillon: "#9ca3af",
  published: "#3b82f6",
  négociation: "#f59e0b",
  vendu: "#10b981",
  supprimé: "#ef4444"
};

const statusCount = posts.value.reduce((acc, post) => {
  acc[post.current_status] = (acc[post.current_status] || 0) + 1;
  return acc;
}, {});

createChart(postsChart.value, "doughnut", {
  labels: Object.keys(statusCount),
  datasets: [{
    label: t("posts"),
    data: Object.values(statusCount),
    backgroundColor: Object.keys(statusCount).map(key => statusColors[key] || "#94a3b8"),
    borderWidth: 2,
    borderColor: "#fff",
    hoverOffset: 12,
  }],
});

/* SOLD POSTS LINE */
const soldPosts = posts.value.filter(p => p.current_status === "vendu");
const weekCounts = {};
soldPosts.forEach(p => {
  const date = new Date(p.created_at);
  const week = `${date.getFullYear()}-W${Math.ceil((date.getTime() / 86400000 + 4 - (date.getDay() || 7)) / 7)}`;
  weekCounts[week] = (weekCounts[week] || 0) + 1;
});

createChart(soldPostsChart.value, "line", {
  labels: Object.keys(weekCounts),
  datasets: [{
    label: t("soldPostsPerWeek"),
    data: Object.values(weekCounts),
    borderColor: "#10b981",
    backgroundColor: ctx => {
      const gradient = ctx.chart.ctx.createLinearGradient(0, 0, 0, 300);
      gradient.addColorStop(0, "rgba(16,185,129,0.4)");
      gradient.addColorStop(1, "rgba(16,185,129,0.05)");
      return gradient;
    },
    fill: true,
    tension: 0.3,
    pointRadius: 6,
    pointHoverRadius: 10,
    pointBackgroundColor: "#10b981",
    pointHoverBackgroundColor: "#fff",
    pointBorderColor: "#10b981",
    pointHoverBorderColor: "#10b981",
    borderWidth: 3,
  }],
}, {
  plugins: { legend: { position: "top" } },
});

/* REPORTS BAR */
const reportCounts = {};
reports.value.forEach(r => {
  const date = new Date(r.created_at);
  const week = `${date.getFullYear()}-W${Math.ceil((date.getTime() / 86400000 + 4 - (date.getDay() || 7)) / 7)}`;
  reportCounts[week] = (reportCounts[week] || 0) + 1;
});

createChart(reportsChart.value, "bar", {
  labels: Object.keys(reportCounts),
  datasets: [{
    label: t("reportsPerWeek"),
    data: Object.values(reportCounts),
    backgroundColor: "#f87171",
    borderRadius: 6,
  }],
});
});
</script>
