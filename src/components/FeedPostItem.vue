<template>
  <PostLink :link="post.link">
    <PostHeading :heading="post.title" />
    <PostContent :content="post.content || post['content:encoded']" />
    <PostDate :date="post.isoDate || post.date" />
  </PostLink>
</template>

<script>
export default {
  components: {
    PostLink: () => import("@/components/PostLink.vue"),
    PostHeading: () => import("@/components/PostHeading.vue"),
    PostContent: () => import("@/components/PostContent.vue"),
    PostDate: () => import("@/components/PostDate.vue")
  },
  props: {
    post: {
      type: Object,
      default: null
    }
  }
};
</script>
