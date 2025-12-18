<template>
  <main
    class="min-h-screen bg-gradient-to-b from-[#f9f9f9] to-[#eef4f2] text-[#222831] py-20 px-6 sm:px-12"
  >
    <div
      class="max-w-5xl mx-auto bg-white shadow-lg rounded-3xl p-10 sm:p-16 transition-all duration-500"
    >
      <header class="text-center mb-12">
        <h1 class="text-5xl font-extrabold mb-3 tracking-tight text-[#10b481]">
          Legal Information
        </h1>
        <p class="text-[#666] max-w-2xl mx-auto">
          Please read our Privacy Policy and Terms of Service carefully before
          using our platform.
        </p>
      </header>

      <nav
        class="flex justify-center gap-6 sm:gap-10 border-b border-[#e0e0e0] pb-4 mb-10"
      >
        <button
          @click="selectTab('privacy')"
          :class="[
            'transition-all duration-300 pb-2 text-lg font-semibold relative',
            currentTab === 'privacy'
              ? 'text-[#10b481]'
              : 'text-[#777] hover:text-[#10b481]',
          ]"
        >
          Privacy Policy
          <span
            v-if="currentTab === 'privacy'"
            class="absolute -bottom-[2px] left-0 w-full h-[2px] bg-[#10b481] rounded-full transition-all duration-300"
          ></span>
        </button>
        <button
          @click="selectTab('terms')"
          :class="[
            'transition-all duration-300 pb-2 text-lg font-semibold relative',
            currentTab === 'terms'
              ? 'text-[#10b481]'
              : 'text-[#777] hover:text-[#10b481]',
          ]"
        >
          Terms of Service
          <span
            v-if="currentTab === 'terms'"
            class="absolute -bottom-[2px] left-0 w-full h-[2px] bg-[#10b481] rounded-full transition-all duration-300"
          ></span>
        </button>
      </nav>

      <transition name="fade-slide" mode="out-in">
        <section
          v-if="currentTab === 'privacy'"
          key="privacy"
          class="space-y-6"
        >
          <h2
            class="text-2xl font-bold text-[#10b481] border-b-2 border-[#10b481] inline-block pb-1"
          >
            Privacy Policy
          </h2>
          <p class="text-[#444]">
            We value your privacy. This policy explains how we collect, use, and
            protect your personal information when you use our marketplace.
          </p>
          <ul class="list-disc list-inside space-y-3 text-[#555]">
            <li>
              <strong>Information Collection:</strong> We collect details such
              as your name, email, and transaction data to provide our services
              efficiently.
            </li>
            <li>
              <strong>Data Usage:</strong> Your data is used solely to manage
              your account, process orders, and improve your experience.
            </li>
            <li>
              <strong>Security:</strong> We use encryption and advanced
              safeguards to protect your information.
            </li>
            <li>
              <strong>Cookies:</strong> Cookies enhance your browsing experience
              and site performance.
            </li>
            <li>
              <strong>Third Parties:</strong> Your data will never be sold or
              shared without your consent.
            </li>
          </ul>
          <p>
            For any privacy-related questions, contact us at
            <a
              href="mailto:support@smartsaha.com"
              class="text-[#10b481] font-semibold underline hover:no-underline"
            >
              support@smartsaha.com </a
            >.
          </p>
        </section>

        <section v-else key="terms" class="space-y-6">
          <h2
            class="text-2xl font-bold text-[#10b481] border-b-2 border-[#10b481] inline-block pb-1"
          >
            Terms of Service
          </h2>
          <p class="text-[#444]">
            By using our platform, you agree to comply with these Terms of
            Service. Please read them carefully.
          </p>
          <ul class="list-disc list-inside space-y-3 text-[#555]">
            <li>
              <strong>Account Responsibility:</strong> You are responsible for
              maintaining the security of your account.
            </li>
            <li>
              <strong>Marketplace Use:</strong> The platform connects farmers,
              cooperatives, and buyers worldwide. Misuse or fraud is prohibited.
            </li>
            <li>
              <strong>Transactions:</strong> Sellers and buyers manage their own
              agreements. The platform acts as an intermediary only.
            </li>
            <li>
              <strong>Intellectual Property:</strong> All content and visuals
              are protected by copyright.
            </li>
            <li>
              <strong>Termination:</strong> We reserve the right to suspend or
              terminate accounts that violate these terms.
            </li>
          </ul>
          <p>
            Continued use of the platform means you accept these terms. Updates
            will be posted on this page.
          </p>
        </section>
      </transition>

      <footer
        class="text-center text-[#888] mt-16 border-t border-[#e0e0e0] pt-8 text-sm"
      >
        <p>
          &copy; {{ new Date().getFullYear() }}
          <span class="text-[#10b481] font-semibold">SmartSaha Marketplace</span
          >. All rights reserved.
        </p>
      </footer>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const router = useRouter();
const currentTab = ref(route.query.tab === "terms" ? "terms" : "privacy");

const selectTab = (tab) => {
  currentTab.value = tab;
  router.replace({ query: { tab } });
};

watch(
  () => route.query.tab,
  (val) => {
    if (val === "terms" || val === "privacy") currentTab.value = val;
  }
);

onMounted(() => {
  if (!route.query.tab) {
    router.replace({ query: { tab: "privacy" } });
  }
});
</script>

<style scoped>
main {
  font-family: "Montserrat", sans-serif;
  line-height: 1.8;
}

.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.5s ease;
}
.fade-slide-enter-from {
  opacity: 0;
  transform: translateY(10px);
}
.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>
