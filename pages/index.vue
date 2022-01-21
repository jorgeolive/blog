<template>
  <div>
    <div style="margin: auto; width: 70%;" id="post-snapshot" v-for ="post of posts" :key=post.slug> 
      <post-preview :post="post"></post-preview>
    </div>
    <!--<nuxt-content :document="index"/>-->
  </div>
</template>

<script>

export default {
  async asyncData({ $content, params, error }) {
    const posts = await $content("posts")
      .fetch()
      .catch(err => {
        error({ statusCode: 404, message: "index not found" });
      });

    return {
      posts
    };
  },
  methods: {
    formatDate(date) {
      const options = { year: 'numeric', month: 'long', day: 'numeric' }
      return new Date(date).toLocaleDateString('en', options)
    }
  }
};
</script>

<style>
#post-snapshot {
  border-color: cadetblue;
}

.nuxt-content {
  font-family: OpenSans, Arial, sans-serif;
  font-size: 0.75rem;
}
</style>