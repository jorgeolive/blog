<template>
  <div>
    <h1>{{ aboutMe.title }}</h1>
    <p>{{ aboutMe.description }}</p>
    <nuxt-content :document="aboutMe"/>
  </div>
</template>

<script>
export default {
  async asyncData({ $content, params, error }) {
    const aboutMe = await $content("about-me")
      .fetch()
      .catch(err => {
        error({ statusCode: 404, message: "about me not found" });
      });

    return {
      aboutMe
    };
  }
};
</script>

