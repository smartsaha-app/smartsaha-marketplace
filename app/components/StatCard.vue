<template>
  <div
    class="relative flex items-center gap-4 p-3 bg-white rounded transition border border-gray-100"
  >
    <div
      class="flex items-center justify-center w-20 h-20 rounded"
      :style="{ backgroundColor: hexToRgba(color, 0.1) }"
    >
      <i :class="icon" class="text-4xl" :style="{ color: color }"></i>
    </div>
    <div class="flex flex-col text-left">
      <p class="text-3xl font-bold text-gray-800">
        {{ value }}
      </p>
      <p class="text-sm font-medium text-gray-500">
        {{ label }}
      </p>
    </div>
    <div
      class="absolute right-0 top-3 bottom-3 border-r-4"
      :style="{ borderColor: color }"
    ></div>
  </div>
</template>

<script setup>
defineProps({
  label: String,
  value: [String, Number],
  icon: String,
  color: String,
  bordered: Boolean,
  progress: Number,
});

const hexToRgba = (hex, alpha = 1) => {
  let r = 0,
    g = 0,
    b = 0;

  // #RRGGBB
  if (hex.length === 7) {
    r = parseInt(hex.slice(1, 3), 16);
    g = parseInt(hex.slice(3, 5), 16);
    b = parseInt(hex.slice(5, 7), 16);
  }

  return `rgba(${r}, ${g}, ${b}, ${alpha})`;
};
</script>

<style scoped>
/* Optional: add a subtle bounce effect when value changes */
</style>
