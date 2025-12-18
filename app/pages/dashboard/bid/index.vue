<template>
  <div class="space-y-8 p-12 bg-gray-50/60 min-h-screen">
    <Breadcrumb />

    <h1 class="text-2xl font-bold mb-6">Mes Enchères</h1>

    <div class="flex border-b border-gray-200 mb-8">
      <button
        @click="tab = 'sent'"
        class="px-4 py-2 -mb-px text-sm font-medium transition flex items-center gap-1"
        :class="getTabClass('sent')"
      >
        Enchères envoyées
      </button>

      <button
        @click="tab = 'received'"
        class="ml-6 px-4 py-2 -mb-px text-sm font-medium transition flex items-center gap-1"
        :class="getTabClass('received')"
      >
        Enchères reçues
      </button>
    </div>

    <div>
      <SendBid v-if="tab === 'sent'" />
      <ReceivedBid v-if="tab === 'received'" />
    </div>
  </div>
</template>

<script setup>
definePageMeta({ layout: "dashboard" });
import { ref } from 'vue';
import SendBid from '~/components/SendBid.vue';
import ReceivedBid from '~/components/ReceivedBid.vue';
import Breadcrumb from '~/components/Breadcrumb.vue';

const tab = ref('sent');

// Fonction pour gérer la classe des tabs
const getTabClass = (name) => {
  return tab.value === name
    ? 'border-b-2 border-[#10B481] text-[#10B481]'
    : 'text-gray-500 hover:text-gray-700';
};
</script>

<style scoped>
/* Ajoute d'autres styles si nécessaire */
</style>
