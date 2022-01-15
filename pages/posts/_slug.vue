<template>
  <div>
      <pre> {{ post }} </pre>
      <h1>
        {{ post.title }}
      </h1>
      <h3>{{ post.description }}</h3>
      <nuxt-content :document="post" />
  </div>
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
