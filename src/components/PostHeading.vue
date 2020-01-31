<template>
  <span
    v-text="heading"
    class="font-semibold mb-2 text-lg"
    role="heading"
    aria-level="2"
  />
</template>

<script>
export default {
  props: {
    heading: {
      type: String,
      default: null
    }
  }
};
</script>
