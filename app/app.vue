<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>

<script setup lang="ts">
useHead({
  title: "Marketplace SmartSaha - Enchères Agricoles",
  titleTemplate: "%s | SmartSaha",
  meta: [
    { name: "description", content: "SmartSaha Marketplace facilite l’accès au marché pour les petits agriculteurs et coopératives grâce à une plateforme d’enchères agricole sécurisée et intuitive." },
    { name: "keywords", content: "SmartSaha, marketplace agricole, enchères agricoles, agriculteurs, coopératives, vente directe, agriculture" },
    { name: "robots", content: "index, follow" },
    { name: "viewport", content: "width=device-width, initial-scale=1" },
    { property: "og:title", content: "Marketplace SmartSaha - Enchères Agricoles" },
    { property: "og:description", content: "Plateforme sécurisée et intuitive pour connecter producteurs et acheteurs." },
    { property: "og:type", content: "website" },
    { property: "og:url", content: "https://www.smart-saha.com" },
    { property: "og:image", content: "/og-image.png" },
    { name: "twitter:card", content: "summary_large_image" },
    { name: "twitter:title", content: "Marketplace SmartSaha - Enchères Agricoles" },
    { name: "twitter:description", content: "Plateforme sécurisée et intuitive pour connecter producteurs et acheteurs." },
    { name: "twitter:image", content: "/og-image.png" }
  ],
  link: [
    { rel: "stylesheet", href: "https://unpkg.com/boxicons@2.1.4/css/boxicons.min.css" },
    { rel: "stylesheet", href: "https://cdn.boxicons.com/fonts/basic/boxicons.min.css" },
    { rel: "stylesheet", href: "https://fonts.googleapis.com/css2?family=Readex+Pro:wght@400;500;700;800&display=swap" }
  ],
  script: [
    { type: "module", src: "https://cdn.jsdelivr.net/npm/emoji-picker-element@1/index.js" }
  ]
})
</script>
