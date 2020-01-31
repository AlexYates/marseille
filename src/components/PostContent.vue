<template>
  <div v-html="content" class="mb-4 text-gray-800 text-sm" />
</template>

<script>
export default {
  props: {
    content: {
      type: String,
      default: null
    }
  }
};
</script>
