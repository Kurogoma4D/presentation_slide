<template>
  <div :style="{ height: size + 'px', width: '100%' }"></div>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    size: Number,
  },
  setup(props) {
    const size = props.size;
    return { size };
  },
});
</script>
