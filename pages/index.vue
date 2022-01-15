<template>
  <div>
    <h1>{{ index.title }}</h1>
    <p>{{ index.description }}</p>
    <nuxt-content :document="index"/>
  </div>
</template>

<script>
export default {
  async asyncData({ $content, params, error }) {
    const index = await $content("index")
      .fetch()
      .catch(err => {
        error({ statusCode: 404, message: "index not found" });
      });

    return {
      index
    };
  }
};
</script>