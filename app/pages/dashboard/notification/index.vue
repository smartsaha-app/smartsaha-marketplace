<template>
    <section class="space-y-8 p-12 bg-gray-50/60">
      <Breadcrumb />

      <div class="flex items-center justify-between mb-8">
        <h1 class="text-3xl font-bold text-gray-900">
          Notifications
        </h1>
  
        <span class="text-sm text-gray-500">
          {{ unreadCount }} unread
        </span>
      </div>
  
      <div v-if="loading" class="flex justify-center py-20">
        <div class="w-8 h-8 border-4 border-gray-300 border-t-[#10b481] rounded-full animate-spin"></div>
      </div>
  
      <div v-if="error" class="text-red-500 text-center py-10">
        {{ error }}
      </div>
  
      <div v-if="notifications.length" class="space-y-4">
        <div
          v-for="notif in notifications"
          :key="notif.id"
          @click="markAsRead(notif)"
          class="group flex gap-4 p-5 rounded border cursor-pointer transition-all"
          :class="notif.is_read
            ? 'bg-white border-gray-100'
            : 'bg-[#f7fdfb] border-[#10b481]/30 shadow-sm hover:shadow-md'"
        >
          <div
            class="w-11 h-11 flex items-center justify-center rounded-full shrink-0"
            :class="notif.is_read ? 'bg-gray-100 text-gray-400' : 'bg-[#10b481]/10 text-[#10b481]'"
          >
            <i class="bx bx-bell text-xl"></i>
          </div>
  
          <div class="flex-1">
            <div class="flex items-center gap-3">
              <h3 class="font-semibold text-gray-900">
                {{ notif.notification_type || 'Notification' }}
              </h3>
  
              <span
                v-if="!notif.is_read"
                class="text-xs px-2 py-0.5 rounded-full bg-[#10b481]/10 text-[#10b481] font-medium"
              >
                New
              </span>
            </div>
  
            <p class="text-gray-600 text-sm mt-1 leading-relaxed">
              {{ notif.message }}
            </p>
  
            <p class="text-xs text-gray-400 mt-2">
              {{ formatDate(notif.created_at) }}
            </p>
          </div>
  
          <div class="self-center text-gray-300 group-hover:text-gray-500 transition">
            <i class="bx bx-chevron-right text-xl"></i>
          </div>
        </div>
      </div>
  
      <div v-else-if="!loading" class="text-center py-24 text-gray-500">
        <i class="bx bx-bell-off text-5xl mb-4"></i>
        <p>No notifications yet</p>
      </div>
    </section>
  </template>
  
  
  <script setup>
    definePageMeta({
    layout: "dashboard",
  });
  import { ref, onMounted } from "vue";
  import { API_URL } from "~/utils/config";
  import Breadcrumb from "~/components/Breadcrumb.vue";
  
  const notifications = ref([]);
  const loading = ref(false);
  const error = ref("");
  
  function formatDate(dateStr) {
    const date = new Date(dateStr);
    return date.toLocaleString();
  }
  const unreadCount = computed(
  () => notifications.value.filter(n => !n.is_read).length
);
  
  function markAsRead(notif) {
    if (!notif.is_read) {
      notif.is_read = true;
      fetch(`${API_URL}/api/notifications/${notif.id}/`, {
        method: "PATCH",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${localStorage.getItem("access_token")}`,
        },
        body: JSON.stringify({ is_read: true }),
      }).catch(console.error);
    }
  }
  
  async function fetchNotifications() {
    loading.value = true;
    error.value = "";
    try {
      const token = localStorage.getItem("access_token");
      if (!token) throw new Error("Not authenticated");
  
      const res = await fetch(`${API_URL}/api/notifications/`, {
        headers: { Authorization: `Bearer ${token}` },
      });
      if (!res.ok) throw new Error("Failed to fetch notifications");
  
      notifications.value = await res.json();
    } catch (err) {
      console.error(err);
      error.value = err.message || "Error fetching notifications";
    } finally {
      loading.value = false;
    }
  }
  
  onMounted(() => {
    fetchNotifications();
  });
  </script>
  
  <style scoped>
  ul {
    max-height: 70vh;
    overflow-y: auto;
  }
  </style>
  