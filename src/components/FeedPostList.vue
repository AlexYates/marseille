<template>
  <ol>
    <li v-for="(post, index) in posts" :key="index">
      <FeedPostItem :post="post" />
    </li>
  </ol>
</template>

<script>
export default {
  components: {
    FeedPostItem: () => import("@/components/FeedPostItem.vue")
  },
  props: {
    posts: {
      type: Array,
      default: null
    }
  }
};
</script>
