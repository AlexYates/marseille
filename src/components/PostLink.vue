<template>
  <a
    :href="link"
    class="news bg-gray-200 flex flex-col mb-2 p-4 rounded focus:text-red-800 hover:text-red-800"
    rel="noreferrer"
    target="_blank"
  >
    <slot />
  </a>
</template>

<script>
export default {
  props: {
    link: {
      type: String,
      default: null
    }
  }
};
</script>
