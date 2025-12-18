<template>
  <div class="min-h-screen flex flex-col bg-white">
    <Hero />
    <About />
    <StepCard />
    <CategoryCards />
    <Feature />
    <Abt />
    <Review/>
    <PreFooter />
  </div>
</template>

<script setup>
definePageMeta({
  layout: "market",
});

import Hero from "~/components/Hero.vue";
import Feature from "~/components/Feature.vue";
import About from "~/components/About.vue";
import Abt from "~/components/Abt.vue";
import PreFooter from "~/components/PreFooter.vue";
import CategoryCards from "~/components/CategoryCards.vue";
import Review from "~/components/Review.vue";
import StepCard from "~/components/StepCard.vue";
</script>

<style scoped>
section {
  scroll-margin-top: 80px;
}
</style>
