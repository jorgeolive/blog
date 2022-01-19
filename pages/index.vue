<template>
  <div>
    <client-only placeholder="newPostCarousel">
    <!--<carousel :per-page="1" :mouse-drag="false">
    <slide>
      Slide 1 Content
    </slide>
    <slide>
      Slide 2 Content
    </slide>
  </carousel> -->
  </client-only>
  <div style="display: flex; flex-flow: column nowrap;" >
    <div id="post-snapshot" v-for ="post of posts" :key=post.slug> 
      <post-preview :post="post"></post-preview>
    </div>
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