<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>


<script setup lang="ts">
useHead({
  script: [
    {
      type: 'module',
      src: 'https://cdn.jsdelivr.net/npm/emoji-picker-element@1/index.js'
    }
  ],
  link: [
    {
      rel: 'stylesheet',
      href: 'https://unpkg.com/boxicons@2.1.4/css/boxicons.min.css'
    },
    {
      rel: "stylesheet",
      href: "https://cdn.boxicons.com/fonts/basic/boxicons.min.css"
    },
    {
      rel: "stylesheet",
      href: "https://fonts.googleapis.com/css2?family=Readex+Pro:wght@400;500;700;800&display=swap"
    }
  ]
})

</script>