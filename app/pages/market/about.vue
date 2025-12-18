<template>
  <main class="bg-gradient-to-b from-[#f9f9f9] to-[#eef4f2] text-[#222831]">
    <section class="relative py-24 px-6 sm:px-12 text-center overflow-hidden">
      <div class="absolute inset-0">
        <img
          src="/img/image4.jpg"
          alt="SmartSaha Marketplace"
          class="w-full h-full object-cover opacity-20"
        />
      </div>
      <div class="relative max-w-4xl mx-auto">
        <h1 class="text-5xl sm:text-6xl font-extrabold mb-6 text-[#10b481]">
          {{ t('aboutTitle') }}
        </h1>
        <p class="text-lg text-[#112830] max-w-2xl mx-auto">
          {{ t('aboutText') }}
        </p>
      </div>
    </section>

    <section
      class="max-w-6xl mx-auto py-20 px-6 sm:px-12 grid md:grid-cols-2 gap-12 items-center"
    >
      <div>
        <h2 class="text-3xl font-bold mb-4 text-[#10b481]">{{ t('whoWeAre') }}</h2>
        <p class="text-[#555] leading-relaxed mb-4">
          {{ t('whores1') }}
        </p>
        <p class="text-[#555] leading-relaxed">
          {{ t('whores2') }}
        </p>
      </div>
      <div>
        <img
          src="/img/image1.jpg"
          alt="Farmers working together"
          class="rounded shadow-lg object-cover w-full h-[380px]"
        />
      </div>
    </section>

    <section class="bg-white py-20 px-6 sm:px-12">
      <div class="max-w-6xl mx-auto text-center mb-12">
        <h2 class="text-4xl font-bold text-[#10b481] mb-4">
          {{ t('mission') }}
        </h2>
        <p class="text-[#555] max-w-2xl mx-auto">
          {{ t('missionRes1') }}
        </p>
      </div>

      <div class="grid sm:grid-cols-2 lg:grid-cols-3 gap-8 max-w-6xl mx-auto">
        <div
          class="bg-gradient-to-b from-[#10b481]/10 to-white p-8 rounded shadow hover:shadow-lg transition-all duration-300"
        >
          <h3 class="text-xl font-semibold text-[#10b481] mb-3">Our Mission</h3>
          <p class="text-[#444]">
            To empower farmers and producers by providing them access to digital
            tools, logistics, and international markets that increase
            profitability and sustainability.
          </p>
        </div>

        <div
          class="bg-gradient-to-b from-[#10b481]/10 to-white p-8 rounded shadow hover:shadow-lg transition-all duration-300"
        >
          <h3 class="text-xl font-semibold text-[#10b481] mb-3">Our Vision</h3>
          <p class="text-[#444]">
            To become the leading agricultural marketplace in Africa — a trusted
            platform connecting local production to global demand.
          </p>
        </div>

        <div
          class="bg-gradient-to-b from-[#10b481]/10 to-white p-8 rounded shadow hover:shadow-lg transition-all duration-300"
        >
          <h3 class="text-xl font-semibold text-[#10b481] mb-3">Our Values</h3>
          <ul class="text-[#444] space-y-2 text-left list-disc list-inside">
            <li>Transparency & Trust</li>
            <li>Empowerment through Technology</li>
            <li>Sustainability and Fair Trade</li>
          </ul>
        </div>
      </div>
    </section>

    <section
      class="max-w-6xl mx-auto py-20 px-6 sm:px-12 grid md:grid-cols-2 gap-12 items-center"
    >
      <div class="order-2 md:order-1">
        <img
          src="/img/image1.jpg"
          alt="Marketplace connection"
          class="rounded shadow-lg object-cover w-full h-[380px]"
        />
      </div>
      <div class="order-1 md:order-2">
        <h2 class="text-3xl font-bold mb-4 text-[#10b481]">
          How SmartSaha Works
        </h2>
        <p class="text-[#555] leading-relaxed mb-4">
          SmartSaha allows farmers and producers to post their offers and
          products directly on the marketplace. Buyers and businesses can browse
          listings, negotiate, and make purchases securely through our platform.
        </p>
        <p class="text-[#555] leading-relaxed">
          Every transaction is transparent, and our verification process ensures
          authenticity, reliability, and fair pricing for all parties involved.
        </p>
      </div>
    </section>

    <section class="bg-[#10b481] text-white text-center py-20 px-6 sm:px-12">
      <h2 class="text-4xl font-extrabold mb-6">
        Join the Future of Agriculture
      </h2>
      <p class="max-w-2xl mx-auto mb-8 text-white/90">
        Be part of a global community that values innovation, connection, and
        sustainability in agriculture. Start buying or selling today.
      </p>
      <div class="flex justify-center gap-4 flex-wrap">
        <button
        @click="goToSignin"
          class="border border-white text-white px-6 py-3 rounded hover:bg-white/10 transition-all"
        >
          Get Started
        </button>
      </div>
    </section>
  </main>
</template>
<script setup lang="ts">
import { useLanguageStore } from "~/stores/language";
import { translate } from "~/utils/translate";
import { useRouter } from "vue-router";
const router = useRouter();

const languageStore = useLanguageStore();
const t = (key: string) => {
  const lang = languageStore.lang;
  return translate[lang][key] || key;
};
definePageMeta({
  layout: "market",
});

function goToSignin() {
  window.location.href = "/signin";
}
</script>

<style scoped>
main {
  font-family: "Montserrat", sans-serif;
  line-height: 1.7;
}

img {
  transition: transform 0.5s ease, box-shadow 0.5s ease;
}

img:hover {
  transform: scale(1.03);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
}
</style>
