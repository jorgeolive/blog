<template>
  <article class="post-wrapper">
      <!--<pre> {{ post }} </pre> -->
      <div class="post-content content-margins">
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
  text-align: center;
}

.post-content {
  margin-left: 12em;
  margin-right: 12em;
  background-color: white;
  box-shadow: 0 4px 8px 0 rgb(0 0 0 / 20%), 0 6px 20px 0 rgb(0 0 0 / 19%);
}

@media only screen and (min-width: 1200px) {
  .content-margins {
    margin-left: 12em;
    margin-right: 12em;
  }

  .post-title {
    font-family: MinionPro, Arial, sans-serif;
    font-size: 3.5rem;
    margin: 0;
    padding: 1em;
  }
}

@media only screen and (max-width: 1200px) {
  .content-margins {
    margin-left: 0;
    margin-right: 0;
  }

  .post-title {
    font-family: MinionPro, Arial, sans-serif;
    font-size: 2.5rem;
    margin: 0;
    padding: 1em;
  }
}

.nuxt-content {
  font-family: MinionPro, Arial, sans-serif;
  font-size: 1.3rem;
  text-align: left;
  padding: 20px;
  margin-left: 1.8rem;
  margin-right: 1.8rem;
}

.nuxt-content code {
  flex-shrink: 2;
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