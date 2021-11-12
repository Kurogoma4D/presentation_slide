<script setup lang="ts">
const props = defineProps({
  image: {
    type: String,
  },
});
</script>

<template>
  <div class="slidev-layout pip">
    <slot />
    <img
      class="
        absolute
        right-0
        bottom-0
        object-contain
        w-1/3
        h-full
        object-bottom
      "
      :src="props.image"
    />
  </div>
</template>