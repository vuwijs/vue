<script lang="ts">
import { defineComponent } from 'vue-demi'

export default defineComponent({
  props: {
    component: {
      type: String,
      default: 'sa-alert',
    },
  },
})
</script>

<template>
  <div :class="component">
    <slot />
  </div>
</template>
