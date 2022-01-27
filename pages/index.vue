<template>
  <div class="layout">
    <div class="posts" >
      <div class="post" v-for ="post of posts" :key=post.slug> 
        <post-preview :post="post"></post-preview>
      </div>
      <!--<nuxt-content :document="index"/>-->
    </div>
  </div>
</template>

<script>

export default {
  async asyncData({ $content, params, error }) {
    const posts = await $content("posts")
      .sortBy("createdAt", "desc")
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
.layout {
  display: flex;
  justify-content: center;
}

.posts {
  flex: 0 0 70%;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.post {
  flex: 0 1 40%;
}

.nuxt-content {
  font-family: MinionPro, Arial, sans-serif;
  font-size: 0.75rem;
}
</style>