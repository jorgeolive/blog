<template>
  <div class="post-content">
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

<style scoped>
.post-content {
  font-family: MinionPro, Arial, sans-serif;
  font-size: 1.3rem;
  padding: 3rem;
  margin-left: 12rem;
  margin-right: 12rem;
  background-color: white;
  box-shadow: 0 4px 8px 0 rgb(0 0 0 / 20%), 0 6px 20px 0 rgb(0 0 0 / 19%);
}
</style>

