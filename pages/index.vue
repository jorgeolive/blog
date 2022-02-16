<template>
    <div class="posts" >
      <div class="post" v-for ="post of posts" :key=post.slug> 
        <post-preview :post="post"></post-preview>
      </div>
      <!--<nuxt-content :document="index"/>-->
    </div>
</template>

<script>

export default {
  async asyncData({ $content, params, error }) {
    const posts = await $content("posts")
      .sortBy("postNumber", "desc")
      .fetch()
      .catch(err => {
        error({ statusCode: 404, message: "index not found" });
      });

    return {
      posts
    };
  }
};
</script>

<style>
.posts {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.post {
  flex: 0 4 60%;
}
</style>