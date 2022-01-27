<template>
  <article class="post-wrapper">
      <!--<pre> {{ post }} </pre> -->
      <div class="post-content">
        <h1>
          {{ post.title }}
        </h1>
        <nuxt-content :document="post" />
      </div> 
  </article>
</template>

<script>
export default {
  async asyncData({ $content, params, error }) {
    const post = await $content(`posts/${params.slug}`)
      .fetch()
      .catch(err => {
        error({ statusCode: 404, message: "Página no encontrada" });
      });
    return {
      post
    };
  }
};
</script>

<style>
.post-wrapper {
  display: flex;
  justify-content: center;
  text-align: center;
}

.post-wrapper > div > h1 {
  font-size: 60px;
}

.post-content {
  background-color: white;
  flex: 0 0 75%;
  box-shadow: 0 4px 8px 0 rgb(0 0 0 / 20%), 0 6px 20px 0 rgb(0 0 0 / 19%);
}

.nuxt-content {
  text-align: left;
  font-size: 18px;
  padding: 10px;
}

.nuxt-content code {
  justify-content: left;
  font-size: 14px;
}
</style>