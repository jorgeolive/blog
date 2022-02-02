<template>
  <article class="post-wrapper">
      <!--<pre> {{ post }} </pre> -->
      <div class="post-content">
        <h1 class="post-title">
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

.post-title {
  font-family: MinionPro, Arial, sans-serif;
  font-size: 45px;
}

.post-content {
  background-color: white;
  flex: 0 0 75%;
  box-shadow: 0 4px 8px 0 rgb(0 0 0 / 20%), 0 6px 20px 0 rgb(0 0 0 / 19%);
}

.nuxt-content {
  font-family: MinionPro, Arial, sans-serif;
  font-size: 1.3rem;
  text-align: left;
  padding: 20px;
  margin-left: 100px;
  margin-right: 100px;
}

.nuxt-content code {
  color: brown;
  font-size: 1rem;
  text-align: left;
  font-weight: bold;
}

.nuxt content h1 {
  font-size: 2rem;
  font-family: MinionPro, Arial, sans-serif;
}

a[href^="https"] {
  font-size: 1.3rem;
  font-family: MinionPro, Arial, sans-serif;
  color: blue;
}
</style>