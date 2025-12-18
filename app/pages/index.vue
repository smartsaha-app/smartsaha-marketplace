<script setup>
import { onMounted } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();

onMounted(() => {
  router.push("/market"); // Redirection immédiate
});
</script>

<template>
  <!-- Optionnel : rien à afficher ici -->
  <div></div>
</template>
