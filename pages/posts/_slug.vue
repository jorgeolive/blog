<template>
  <article class="post-wrapper">
      <!--<pre> {{ post }} </pre> -->
      <div class="post-content content-margins">
        <h1 class="post-title">
          {{ post.title }}
        </h1>
        <p style="font-family: MinionPro, Arial, sans-serif; font-size: 1.3rem; font-style: italic;">On {{formatDate(post.createdAt)}}</p> 
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
  },
   head() {
     return {
      title: "jorge-olive.net - " + this.post.title,
      meta: [
        { charset: 'utf-8' },
        { name: 'viewport', content: 'width=device-width, initial-scale=1' },
        {
          hid: 'description',
          name: this.post.title,
          content: this.post.htmlMetadata
        },
        { hid: 'og:title', property: 'og:title', content: this.post.title },
        { hid: 'og:description', property: 'og:description', content: this.post.htmlMetadata },
        { hid: 'twitter:title', name: 'twitter:title', content: this.post.title },
        { hid: 'twitter:description', name: 'twitter:description', content: this.post.htmlMetadata }
      ],
      link: [{ rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }]
    }
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